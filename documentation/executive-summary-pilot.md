Executive Summary — QR-session Pilot (10 stores)

Objective

Validate a fast QR-per-table data collection path to power AI analytics (demand forecasting, item performance) and prove early revenue lift with minimal merchant friction. Deliverables: pilot data, reconciliation flows, UX learnings, and go/no-go recommendation for POS investment.

Why now

- Fastest path to collect item-level sales signals without POS contracts or hardware.
- Mobile-first Vietnam context: QR + e-wallets (MoMo, VNPay, ZaloPay) are widely used.
- Early pilots reduce risk and provide the labeled data AI models need to show ROI.

Pilot scope

- 10 stores, mixed composition: 4 coffee shops, 3 restaurants, 3 small chains.
- Timeline: 8 weeks (onboard + stabilize + analyze).
- Core systems: QR sessions API (POST events + cursor GET), payment webhooks, nightly reconciliation job, merchant reconciliation UI, telemetry and alerts.

Success criteria (go/no-go)

- Order capture rate ≥ 80% across pilot stores after stabilization (2 weeks).
- Payment match rate ≥ 90% for e-wallet transactions.
- Business impact: at least one store shows measurable A/B uplift (≥5% revenue or operations time saved) from AI recommendations or workflows during pilot.

Key risks & mitigations

- Risk: merchants continue to use legacy POS and don't record cash in platform → Mitigation: merchant training + quick reconciliation UI; allow manual cash record.
- Risk: duplicate or abandoned sessions pollute training data → Mitigation: TTL expiry, event dedup via idempotency_key, offline event handling and heuristics to exclude abandoned carts from revenue training.
- Risk: payment gateway integration issues → Mitigation: staged webhook testing and manual payment path for cash.

Estimated effort & high-level costs (docs-only)

- PM/ops: 0.2 FTE during pilot (onboarding, support)
- Engineering: 1.0 backend (API + reconciliation + monitoring) for 4 weeks of initial work (docs & migrations already present), 0.6 frontend for merchant UI scaffolding.
- Support: ad-hoc merchant ops during first 2 weeks (part-time).
- Infrastructure: low for Phase 1 (Postgres, Redis, S3); add pub/sub only if push-based real-time needed.

Pilot playbook (summary)

1. Week 0: Select stores, set up tenants, print QR, configure gateways.
2. Week 1: Smoke test with 2 stores, fix critical issues.
3. Week 2–3: Full onboarding for remaining stores, monitor KPIs daily.
4. Week 4–6: Stabilize reconciliation and run first AI experiments.
5. Week 7–8: Analyze results, produce data quality report and merchant feedback, present go/no-go to leadership.

Decision gates

- Gate 1 (end of Week 3): If order_capture_rate < 60% overall, pause rollout and invest in solving UX or reconciliation issues.
- Gate 2 (end of Week 8): If success criteria met, proceed to scale and prioritize POS discovery funding; else iterate or pause.

Next steps (recommended immediate actions)

- Approve pilot plan and assign PM/ops lead.
- Prepare merchant packs and training material (done: `documentation/qr-session-quickstart.md`).
- Enable verbose telemetry for pilot tenants and schedule daily health-checks for first week.

Contact

Product lead: Product Manager (TBD)
Technical lead: Backend Lead (TBD)

# End
