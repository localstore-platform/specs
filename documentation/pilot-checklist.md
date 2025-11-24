Pilot Checklist — 10-store QR-session pilot

Purpose: run a focused pilot to validate QR session capture quality, reconciliation flows, AI signal usefulness, and merchant UX.

Pilot goals (success criteria):

- Order capture rate ≥ 80% (sessions paid or manually reconciled vs expected sales) after two weeks.
- Payment match rate ≥ 90% for e-wallet flows.
- Session conversion (paid_sessions / created_sessions) ≥ 10% for casual cafés; adjust per business type.
- Collect minimum 14 days of item-level sales per store to seed initial AI models.

Store selection (mix & criteria):

- 4 single-location coffee shops (cash + e-wallet mix)
- 3 small restaurants (higher dwell time, need longer TTL)
- 3 small chains (2–4 locations) to validate multi-location comparison
- Selection criteria: reliable manager contact, willingness to use QR-first flow, ability to train staff, average daily orders ≥ 20.

Pre-pilot setup (Week 0)

- Create tenant & location entries in admin console.
- Generate and print QR per table (session_code) and stick them on tables.
- Configure payment gateways (MoMo/VNPay/ZaloPay) or ensure staff can record cash payments.
- Provide quickstart PDF and 15-min training to manager + 1–2 staff.
- Instrument telemetry flags in staging for the pilot tenants (enable verbose event logging).

Pilot timeline (8 weeks recommended)

- Week 1: Onboard 2–3 earliest adopters for smoke testing.
- Week 2–3: Onboard remaining stores; monitor initial issues and tune TTL/polling.
- Week 4–5: Stabilize reconciliation job and support staff. Collect data for AI baseline.
- Week 6–8: Analyze KPIs, run early AI experiments (top-selling, slow-moving items), and collect merchant feedback.

Telemetry & metrics to collect

- session_created_count (per location/day)
- session_paid_count (per location/day)
- session_expired_count
- order_capture_rate = (sessions_paid + manual_reconciled) / expected_sales (where expected_sales approximated from POS/export or manager estimate)
- payment_match_rate = matched_payments / total_payments
- session_conversion_rate = paid_sessions / created_sessions
- duplicate_event_rate
- late_event_rate (>5 minutes delay)

Support & escalation

- Dedicated Slack channel for pilot merchants and operations.
- Daily check-in for Week 1, then every 2–3 days.
- Escalation criteria: payment_match_rate < 85% for any store for 3 consecutive days.

Merchant communication script (initial contact)

- Intro: "Chúng tôi muốn mời bạn tham gia thử nghiệm hệ thống menu QR của chúng tôi — giúp quản lý đơn hàng và cải thiện doanh thu bằng dữ liệu thông minh."
- What we need: printed QR per table, 15-min training, staff record cash payments in UI.
- Benefits: no hardware, immediate analytics, early access to AI recommendations.
- Commitment: 4–8 weeks pilot, daily summary email, 15-min weekly feedback call.

Pilot playbook for operations

1. Onboarding call + 15-min training.
2. Day 1: walk manager through reconciliation runbook and confirm test payments.
3. Day 3–7: daily health checks (payment_match_rate, duplicate events). Fix UX or TTL if needed.
4. Week 2: review data quality and run first "top-selling" report with manager.
5. Week 4: evaluate go/no-go for scaling. If KPIs met, prepare rollout checklist.

Rollback & safety

- If a store experiences >3 unresolved revenue mismatches in a week, pause automatic recommendations and increase manual reconciliation support.
- Keep a manual CSV import path for merchants to upload daily POS exports for audit/backfill.

Post-pilot deliverables

- Data quality report per store (order_capture_rate, payment_match_rate, duplicate_event_rate).
- Merchant feedback summary and prioritized bugs/UX items.
- Recommendation: scale, iterate, or pause investment in POS discovery based on pilot ROI.

# End of pilot checklist
