Monitoring & Alerts Runbook — QR-session Pilot

Purpose
Define key metrics, dashboards, alert thresholds, and an on-call runbook for pilot operations.

Key metrics (recommended)

- order_capture_rate = sessions_with_order / sessions_created (target > 30% initially)
- payment_match_rate = payments_linked_to_sessions / payments_total (target > 90%)
- duplicate_event_rate = duplicate_events / session_events (target < 0.1%)
- late_event_rate = events_received_after_5m_of_action / events_total (monitor trend)
- reconciliation_issue_count = rows in reconciliation_issues (alert on spike)

Dashboards

- Overview: total sessions, active sessions, paid sessions, revenue (VND)
- Health: order_capture_rate (7d, 24h), payment_match_rate (7d), reconciliation_issue_count
- Events: event throughput (per-second), duplicate_event_rate, avg event processing latency

Alerting thresholds (suggested)

- payment_match_rate < 85% for 30m → P1 alert to on-call
- order_capture_rate drop > 30% vs baseline for 60m → P1
- reconciliation_issue_count increase > 3x baseline in 24h → P2
- duplicate_event_rate > 1% in 10m → P2

Runbook actions (on-call)

1) Payment match failure
   - Check ingestion logs for webhook errors (signature verification failures).
   - Query `session_payments` for recent payments; compare payment_reference and session totals.
   - If webhooks failing, reroute to backlog and notify integrations team; create incident.

2) Orders dropping
   - Check client error logs for increased 4xx/5xx on POST events.
   - Confirm backend latency and DB connection pool health.
   - Check for recent deploys and rollback if needed.

3) Reconciliation spike
   - Open reconciliation UI (if available) and inspect top issues.
   - Run ad-hoc matching script to correlate payments by payment_reference.

Operational notes

- Nightly reconciliation job runs at 02:00 local; results are emailed to ops and visible in the dashboard.
- Retention: keep event logs for 1 year; payments for 2 years per policy.

Contacts

- On-call Backend: backend-oncall@ourplatform (rotating)
- Payments integration: payments@ourplatform
- Ops escalation: ops-lead@ourplatform

How to add a new alert

- Add to monitoring system (Grafana/Datadog) with a runbook link and clear paging instructions.

This runbook should be included in the pilot pack distributed to Ops and Merchant Success teams.
