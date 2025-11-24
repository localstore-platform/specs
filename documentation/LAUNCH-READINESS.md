LAUNCH READINESS CHECKLIST — QR-Session Pilot (10 stores)

Purpose: A concise pre-flight checklist for PM/ops to confirm readiness to start the pilot.

Status: fill each item with ✅ when done.

1. Business & selection

   - [ ] Pilot merchants confirmed (10 stores) with primary contact and manager availability.
   - [ ] Signed pilot participation agreement (email confirmation/consent).

2. Product & Docs
   - [x] API docs: `architecture/api-specification.md` (QR Sessions & Table Ordering).
   - [x] DB schema: `architecture/qr-session-database-schema.md`.
   - [x] Reconciliation spec: `architecture/reconciliation-job-spec.md`.
   - [x] Merchant quickstart: `documentation/qr-session-quickstart.md`.
   - [x] Merchant FAQ: `documentation/merchant-faq.md`.
   - [x] Reconciliation runbook: `documentation/reconciliation-runbook.md`.
   - [x] Reconciliation API & OpenAPI draft: `architecture/reconciliation-api-spec.md` and `architecture/reconciliation-openapi.yaml`.

3. Engineering readiness (docs-only checklist)
   - [ ] Backend hooks & endpoints documented (POST events, cursor GET, payments binding).
   - [ ] Nightly reconciliation job spec documented and scheduled (see `architecture/reconciliation-job-spec.md`).
   - [ ] Analytics ingestion & retention documented (`architecture/qr-session-database-schema.md`).
   - [ ] Observability: metrics and alerts configured for pilot tenants (session_created_count, payment_match_rate, duplicate_event_rate).

4. Merchant onboarding & ops
   - [ ] Quickstart PDF and 15-min training slot scheduled with each merchant.
   - [ ] Printed QR stickers/laminates prepared and delivered to merchants.
   - [ ] Support channel created (Slack/WhatsApp) and on-call owner assigned for first 2 weeks.

5. Pilot telemetry & KPI tracking
   - [ ] Telemetry flags enabled for pilot tenants (verbose event logging).
   - [ ] Dashboard/reporting for KPIs configured (order_capture_rate, payment_match_rate, session_conversion_rate).
   - [ ] Daily health-check schedule and owner assigned for first week.

6. Compliance & security
   - [ ] Payment gateway webhook test vectors verified in staging.
   - [ ] Data retention policy agreed (events 1 year, payments 2 years, raw_payload PII redaction policy).
   - [ ] Access control for reconciliation UI (OWNER, MANAGER, ACCOUNTANT) documented.

7. Rollback & escalation
   - [ ] Rollback criteria defined (e.g., payment_match_rate < 60% after 2 weeks).
   - [ ] Escalation path and support SLAs documented.

8. Launch approval
   - [ ] PM sign-off
   - [ ] Tech lead sign-off
   - [ ] Ops/support sign-off

Notes:

- This repository is documentation-first; engineering implementation will follow the spec files here. Use this checklist to confirm the docs and pilot materials are aligned before coding starts.

# End
