# Nightly Reconciliation Job — Spec

Purpose

This document specifies the nightly reconciliation job that compares QR-session totals, payment gateway records, and manual merchant entries to surface mismatches for operator review. The job reduces revenue leakage, improves data quality for analytics/AI, and increases merchant trust.

Overview

- Run frequency: nightly (local store timezone) — scheduled shortly after business close (configurable per location).
- Inputs:
  - `session_events` (append-only event store)
  - `session_payments` (payments recorded via gateway webhooks or manual operator entries)
  - Optional: external POS export (if merchant provides CSV) ingested to `external_pos_imports`
- Output:
  - `reconciliation_issues` table populated with mismatches and suggested matches
  - Email/push report to merchant if mismatches exceed thresholds
  - Dashboard entries for manual reconciliation UI

Matching algorithm (heuristic)

1. Group events by session_id and compute session totals (subtotal, tax, discount, total). Use `payload` values from session_events or compute line totals.
2. For each session with `status = paid` or recent payments, attempt to match session totals to payments by `payment_reference` (exact match) and amount (exact or within tolerance).
3. If no `payment_reference` available, use fuzzy match by location + amount + timestamp window (±10 minutes) to find a candidate payment.
4. For unmatched sessions, mark as `UNMATCHED_SESSION`; for payments without a matching session, mark `UNMATCHED_PAYMENT`.
5. For partial matches (sum of multiple payments equals session total), mark `PARTIAL_MATCH` and list contributing payments.
6. For amount mismatches beyond tolerance, mark `AMOUNT_MISMATCH` and include details.

Thresholds & tolerances (defaults, configurable per tenant)

- Time window for fuzzy matching: ±10 minutes
- Amount tolerance: 0 VND for e-wallet webhooks (strict); ±1000 VND for vendor rounding discrepancies (configurable)
- Auto-resolve: if fuzzy match confidence > 0.95 and merchant has enabled auto-match, automatically bind and mark resolved.

Reconciliation_issues table (SQL)

```sql
CREATE TABLE reconciliation_issues (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL,
  location_id uuid NOT NULL,
  issue_type varchar NOT NULL, -- UNMATCHED_SESSION | UNMATCHED_PAYMENT | AMOUNT_MISMATCH | PARTIAL_MATCH | DUPLICATE_PAYMENT
  session_id uuid NULL,
  payment_id uuid NULL,
  candidate_payment_ids uuid[] NULL,
  session_total bigint NULL,
  payment_total bigint NULL,
  confidence numeric NULL,
  details jsonb NULL,
  status varchar NOT NULL DEFAULT 'open', -- open | resolved | ignored
  created_at timestamptz NOT NULL DEFAULT now(),
  resolved_at timestamptz NULL,
  resolved_by uuid NULL,
  resolution_note text NULL
);
```

Nightly job steps (detailed)

1. Identify sessions to reconcile: sessions with `created_at` in last N days and `status IN ('paid','pending_reconciliation')`.
2. Aggregate session totals per session (server-side SQL using `session_events` payload).
3. Load payments from `session_payments` for same interval.
4. Run exact-match step: match by `payment_reference` and amount.
5. Run fuzzy-match step where needed: join on `location_id` + time window + amount tolerance. Compute confidence (based on time proximity, amount closeness, and if payment method matches expected method).
6. Generate `reconciliation_issues` rows for any unmatched, ambiguous, or mismatched records.
7. Send summary to merchant (email/push) if open issues exceed threshold (e.g., 2 issues or >₫500k mismatch).
8. Update monitoring metrics: unmatched_sessions_count, unmatched_payments_count, auto_resolve_count.

APIs for Reconciliation UI

- GET /api/v1/tenants/{tenantId}/reconciliation/issues?status=open — list issues
- POST /api/v1/tenants/{tenantId}/reconciliation/issues/{issueId}/resolve — body: {action: 'match'|'mark_cash'|'ignore', payment_id?, note}
- POST /api/v1/tenants/{tenantId}/reconciliation/manual-payment — create manual session_payment bound to session_id (used by staff)

Operational notes

- Audit trail: every resolve action must be logged with `resolved_by` and `resolution_note`.
- Escalation: if unresolved issues persist for >72 hours and affect daily revenue > threshold, create an ops ticket for support team.
- Backfill: support a backfill run for prior days if merchants import external POS CSVs.

Monitoring & Alerts

- Daily email to product owner with reconciliation metrics for pilot stores.
- Alerts when payment_match_rate < 90% for a location for 3 consecutive days.

Privacy & Security

- Do not send raw customer PII in emails; use payment_reference and masked identifiers.
- Limit access to reconciliation UI to roles: OWNER, MANAGER, ACCOUNTANT.

Migration

- When POS integration is added, POS orders should be ingested and included in the reconciliation matching (prefer exact pos_order_id match to session_id where provided).

# End
