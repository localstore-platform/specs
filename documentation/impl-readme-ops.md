README Template — local-store-ops

Purpose
Operational and on-call runbooks repository. Companion to the orchestration docs but focused on runbooks, dashboards, and incident playbooks.

Quick summary

- Tech: Markdown runbooks, Grafana dashboards, alerting configs
- Audience: Ops, support, merchant success
- Responsibility: On-call runbooks, monitoring dashboards, escalation paths

Suggested layout

- /runbooks/
- /dashboards/
- /alerts/
- /roster/ (on-call roster)

On-call

- Document playbooks for common incidents (payment webhook failure, reconciliation spike).

Ownership

- CODEOWNERS: ops lead

Notes

- Keep runbooks brief with clear steps and expected outcomes. Link to orchestration docs for business context.
