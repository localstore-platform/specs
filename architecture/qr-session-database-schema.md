# QR Session Database Schema

This document defines the recommended schema and migration notes for Phase 1 QR-per-table sessions and session events. The goal is a simple append-only event store that supports idempotent writes, cursor-based reads, and later publication to a pub/sub system for real-time upgrades.

## Principles

- Append-only event store for `session_events` with server-assigned monotonic `event_seq` for ordering.
- Idempotency via `idempotency_key` to dedupe client retries.
- Tenant and location scoping on all records (`tenant_id`, `location_id`).
- Keep raw vendor/client payload (`raw_payload`) for auditing and migration.
- TTL/expiration for ephemeral browsing sessions but retention of event history for analytics (default: events retained 1 year).

## Tables

### qr_sessions (one row per active session)

Columns:

- id uuid PRIMARY KEY DEFAULT gen_random_uuid()
- tenant_id uuid NOT NULL
- location_id uuid NOT NULL
- session_code varchar NOT NULL -- short code used in QR URL
- table_id varchar NULL -- printed table code, for merchant convenience
- status varchar NOT NULL DEFAULT 'active' -- active | paid | cancelled | expired
- created_at timestamptz NOT NULL DEFAULT now()
- last_activity_at timestamptz NOT NULL DEFAULT now()
- expires_at timestamptz NULL
- source varchar NULL -- qr, zalo_miniapp, direct
- device_info jsonb NULL
- metadata jsonb NULL
- raw_payload jsonb NULL -- store initial creation payload for audit

Indexes & constraints:

- UNIQUE (tenant_id, location_id, session_code)
- INDEX on (location_id, status)
- INDEX on (tenant_id, created_at)

Retention/TTL:

- When a session transitions to `paid` or `cancelled`, `expires_at` may be set to now() + 365 days to keep for audit.
- For `expired` sessions due to inactivity, retain rows for analytics for 365 days then archive.

### session_events (append-only event stream)

Columns:

- id uuid PRIMARY KEY DEFAULT gen_random_uuid()
- event_seq bigserial NOT NULL -- monotonic sequence
- session_id uuid NOT NULL REFERENCES qr_sessions(id) ON DELETE CASCADE
- tenant_id uuid NOT NULL
- location_id uuid NOT NULL
- idempotency_key varchar NULL -- client-provided for dedupe
- event_type varchar NOT NULL -- item_add|item_remove|quantity_update|submit_order|payment_attempt|payment_success|payment_failed|refund
- client_ts timestamptz NULL -- timestamp from client device
- server_ts timestamptz NOT NULL DEFAULT now()
- payload jsonb NOT NULL -- canonical event payload (items, amounts, modifiers)
- raw_payload jsonb NULL -- vendor/raw client payload
- processed boolean NOT NULL DEFAULT false -- for background processors

Indexes & constraints:

- UNIQUE(idempotency_key, session_id) WHERE idempotency_key IS NOT NULL
- INDEX on (session_id, event_seq)
- INDEX on (tenant_id, location_id, server_ts)

Notes:

- Use `event_seq` as the cursor offset for GET operations. The server should return `next_cursor` encoding the last event_seq.
- Keep `payload` minimal and normalized: item_id, sku, name, unit_price_vnd, quantity, modifiers, line_total.

### session_payments

Columns:

- id uuid PRIMARY KEY DEFAULT gen_random_uuid()
- session_id uuid NOT NULL REFERENCES qr_sessions(id) ON DELETE CASCADE
- tenant_id uuid NOT NULL
- location_id uuid NOT NULL
- payment_method varchar NOT NULL -- momo|vnpay|zalopay|cash|external_pos
- payment_reference varchar NULL -- gateway txn id or external pos id
- amount_bigint bigint NOT NULL -- amount in VND
- currency varchar NOT NULL DEFAULT 'VND'
- status varchar NOT NULL -- pending|success|failed
- operator_id uuid NULL -- staff who recorded manual payment
- metadata jsonb NULL
- created_at timestamptz NOT NULL DEFAULT now()
- raw_payload jsonb NULL -- original gateway webhook/body

Indexes:

- INDEX on (payment_reference)
- INDEX on (session_id, status)

Behavior:

- When a payment with status `success` is recorded and validated, update `qr_sessions.status = 'paid'` and append a `payment_success` event to `session_events` (transactional write recommended).
- Support binding manual cash payments via merchant UI: create session_payments row with status `success` and operator_id, then mark session paid.

## Audit & Reconciliation Helpers

- Keep full `raw_payload` JSON for 1 year for dispute resolution; archive thereafter.
- Provide materialized views for daily aggregates (daily_metrics) that consume `session_events` and `session_payments`.
- Implement nightly reconciliation job that compares payment totals vs session totals and flags mismatches to a `reconciliation_issues` table for merchant review.

## Migration & POS readiness

- Ensure every event and payment row keeps `session_id` and `raw_payload` so when POS integrations arrive, you can map POS orders to existing sessions.
- Build SKU mapping table: `sku_mappings(tenant_id, platform_item_id, vendor_sku, confidence, created_at, manual_override)` to resolve vendor SKUs to platform products.
- When onboarding POS integrations, support dual-write mode where POS order writes a `session_events` record with `event_type=external_pos_order` linking to pos_order_id and session_id when available.

## Retention policy (suggested)

- session_events: keep 365 days in primary DB, then move to cold storage (S3) with Parquet for ML training.
- session_payments, qr_sessions: keep 2 years in DB for accounting convenience, archive to cold storage after.
- raw_payload: keep 1 year; redact PII where not required.

## Observability

- Metrics: events_per_minute, duplicate_event_rate (idempotency hits), late_arrival_rate (>5 minutes after client_ts), payment_match_rate.
- Alerts: duplicate_event_rate > 0.5%, payment_match_rate < 90%, nightly reconciliation mismatches > threshold.

## Example SQL (Postgres)

```sql
-- QR sessions
CREATE TABLE qr_sessions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL,
  location_id uuid NOT NULL,
  session_code varchar NOT NULL,
  table_id varchar NULL,
  status varchar NOT NULL DEFAULT 'active',
  created_at timestamptz NOT NULL DEFAULT now(),
  last_activity_at timestamptz NOT NULL DEFAULT now(),
  expires_at timestamptz NULL,
  source varchar NULL,
  device_info jsonb NULL,
  metadata jsonb NULL,
  raw_payload jsonb NULL
);
CREATE UNIQUE INDEX idx_qr_sessions_tenant_loc_code ON qr_sessions(tenant_id, location_id, session_code);

-- Session events
CREATE TABLE session_events (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  event_seq bigserial NOT NULL,
  session_id uuid NOT NULL REFERENCES qr_sessions(id) ON DELETE CASCADE,
  tenant_id uuid NOT NULL,
  location_id uuid NOT NULL,
  idempotency_key varchar NULL,
  event_type varchar NOT NULL,
  client_ts timestamptz NULL,
  server_ts timestamptz NOT NULL DEFAULT now(),
  payload jsonb NOT NULL,
  raw_payload jsonb NULL,
  processed boolean NOT NULL DEFAULT false
);
CREATE UNIQUE INDEX idx_session_events_idempotency ON session_events(idempotency_key, session_id) WHERE idempotency_key IS NOT NULL;
CREATE INDEX idx_session_events_seq ON session_events(session_id, event_seq);

-- Payments
CREATE TABLE session_payments (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id uuid NOT NULL REFERENCES qr_sessions(id) ON DELETE CASCADE,
  tenant_id uuid NOT NULL,
  location_id uuid NOT NULL,
  payment_method varchar NOT NULL,
  payment_reference varchar NULL,
  amount_bigint bigint NOT NULL,
  currency varchar NOT NULL DEFAULT 'VND',
  status varchar NOT NULL,
  operator_id uuid NULL,
  metadata jsonb NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  raw_payload jsonb NULL
);
CREATE INDEX idx_session_payments_ref ON session_payments(payment_reference);
```
