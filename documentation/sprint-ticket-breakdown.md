Sprint & PR Ticket Breakdown — Weeks 1–3 (QR-Session MVP)

Purpose

Provide concrete, PR-sized engineering tickets for Weeks 1–3 from the implementation timeline. Each ticket includes a short title, description, owner suggestion, acceptance criteria, estimated effort, and a suggested branch/PR naming convention.

How to use

- Create issues in your tracker (GitHub) using the titles below.
- Attach the corresponding doc links (`architecture/api-specification.md`, `architecture/qr-session-database-schema.md`).
- Use the PR checklist at the end before merging.

Week 1 — Core Event Writes & Idempotency (Sprint goal)

W1-01: API: POST /api/v1/locations/{locationId}/qr-sessions (create session)

- Description: Implement session creation endpoint that returns a `session_id`, `qr_url`, and initial snapshot. Persist row to `qr_sessions` with TTL and `tenant_id`/`location_id`/`table_id`.
- Owner: Backend Eng
- Estimate: 3 SP (2–3 days)
- Acceptance:
  - POST returns 201 with session_id and qr_url
  - DB row in `qr_sessions` with correct tenant/location/table
  - Unit tests for TTL behavior
- Branch: feat/w1-session-create

W1-02: API: POST /api/v1/sessions/{sessionId}/events (append event)

- Description: Implement append-only event write. Accepts idempotency_key, event_type (add_item/remove_item/modify_qty), payload. Persist to `session_events` with server-assigned `event_seq`.
- Owner: Backend Eng
- Estimate: 5 SP (3–5 days)
- Acceptance:
  - Duplicate POST with same idempotency_key does not create duplicate event (idempotency enforced)
  - Response contains `event_id` and `event_seq`
  - Unit tests for concurrency ordering (simulate concurrent writes)
- Branch: feat/w1-session-events

W1-03: DB: idempotency & unique constraints + migration

- Description: Add DB migration to enforce unique constraint on (session_id, idempotency_key) and sequence generation for `event_seq` (monotonic per-session or global monotonic sequence as chosen).
- Owner: Backend Eng / DBA
- Estimate: 2 SP (1–2 days)
- Acceptance:
  - Migration applied in staging
  - Tests for unique constraint violation return 409 with clear error payload
- Branch: db/migration/w1-idempotency

W1-04: Tests: unit tests & integration tests for write/retry

- Description: Create unit tests and an integration test that verifies idempotency under retries.
- Owner: QA / Backend Eng
- Estimate: 3 SP
- Acceptance:
  - CI runs tests and they pass
  - Test covers retry and duplicate idempotency_key
- Branch: test/w1-idempotency

Week 2 — Cursor GET, Snapshot & Session Lifecycle

W2-01: API: GET /api/v1/sessions/{sessionId}/events?cursor={event_seq}

- Description: Implement cursor-based incremental retrieval. Returns events after the provided cursor and a next_cursor value.
- Owner: Backend Eng
- Estimate: 4 SP
- Acceptance:
  - GET returns ordered events and correct next_cursor
  - Handles empty result (no new events) with 200 and next_cursor unchanged
  - Unit tests for ordering and consistency during concurrent writes
- Branch: feat/w2-cursor-get

W2-02: API: GET /api/v1/sessions/{sessionId}/snapshot

- Description: Provide current session snapshot (current items, quantities, total, last_event_seq) for UI hydration.
- Owner: Backend Eng, Frontend Eng (contract)
- Estimate: 3 SP
- Acceptance:
  - Snapshot matches replaying events up to last_event_seq
  - Snapshot endpoint is lightweight and cacheable (TTL)
- Branch: feat/w2-snapshot

W2-03: Session lifecycle worker (TTL/expire)

- Owner: Backend Eng
- Estimate: 3 SP
- Acceptance:
  - Worker marks expired sessions and creates an event if configured
  - Tests for TTL edge cases
- Branch: feat/w2-session-lifecycle

W2-04: Frontend: Basic client hydration + incremental polling

- Description: Implement client behavior to fetch snapshot then poll via cursor GET. Provide sample Next.js page or component mock for demo.
- Description: Implement background worker that marks sessions as `expired` after TTL and optionally emits an expiration event to `session_events`.
- Owner: Frontend Eng
- Estimate: 5 SP
- Acceptance:
  - Demo page shows add/remove actions reflected in UI via polling
  - Handles reconnects and resumes from last_cursor
- Branch: feat/w2-client-polling

Week 3 — Payments Binding & Webhooks

W3-01: API: POST /api/v1/sessions/{sessionId}/payments (manual & gateway-normalized)

- Description: API for recording payments (manual cash or normalized gateway payment). Accept payment_reference, amount, method, status.
- Owner: Backend Eng
- Estimate: 3 SP
- Acceptance:
  - Payment row created and linked to session
  - Basic validation for amount vs session total (warning mismatch flag)
- Branch: feat/w3-payments-api

W3-02: Integration: Webhook handlers for MoMo/VNPay/ZaloPay (stubs)

- Description: Implement generic webhook handlers that accept provider payloads, normalize to payment_reference, and call internal payments API. Add verification for signature when possible.
- Owner: Integration Eng
- Estimate: 5 SP
- Acceptance:
  - Webhook test vectors produce payments linked to sessions
  - Webhook signature verification paths exist (configurable)
- Branch: feat/w3-webhooks

W3-03: Tests: E2E test: payment webhook → session marked paid

- Description: Add an end-to-end test that simulates a payment webhook and expects the session to transition to `paid`.
- Owner: QA / Integration Eng
- Estimate: 3 SP
- Acceptance:
  - E2E test passes in CI or on local staging harness
- Branch: test/w3-e2e-payment

W3-04: Admin API: manual-payment reconciliation endpoints

- Description: Admin endpoint to mark cash payments and to list payments for reconciliation UI.
- Owner: Backend Eng
- Estimate: 2 SP
- Acceptance:
  - Admin can create manual payments and see them in reconciliation list
- Branch: feat/w3-admin-payments

PR Checklist (required before merging)

- [ ] Issue linked in PR description
- [ ] Branch follows naming convention: feat|fix|test/<short-desc>
- [ ] Unit tests added for new logic; CI passes
- [ ] Integration or E2E tests for critical flows (idempotency, payments) included where applicable
- [ ] Swagger/OpenAPI updated for new endpoints or DTOs
- [ ] DB migrations included for schema changes with up/down
- [ ] Security checks: auth guards, input validation, webhook signature verification where applicable
- [ ] Observability: metrics (counts/errors) and structured logs added for the feature
- [ ] README or API docs updated with endpoint contracts
- [ ] PR description contains testing steps for reviewers

Branch / PR naming example

- feat/w1-session-create
- feat/w1-session-events
- db/migration/w1-idempotency
- test/w3-e2e-payment

Exporting to issues

- Copy each W1/W2/W3 item to GitHub Issues and assign owners.
- Prioritize W1-02 (session events) and W1-03 (idempotency DB) as blocking for other features.

End of sprint & PR ticket breakdown
