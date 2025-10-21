# Knowledge Base: Vietnam Market Playbook

Last updated: October 21, 2025.

## Overview

The Vietnam track focuses on empowering hyper-local merchants—cafés, street food vendors, family restaurants—to create mobile-ready menu sites in minutes. Success hinges on deep integrations with Zalo/Facebook, frictionless payments, and a mobile-first admin experience that works on entry-level Android phones over 4G.

## Customer Insights

- Owners manage their business from smartphones; laptops are rare.
- Primary communication channels: Zalo (personal + official accounts) and Facebook Pages.
- Payments preference: e-wallets (MoMo, ZaloPay, VNPay) alongside cash-on-delivery.
- Need quick menu updates (daily specials), QR codes for in-store ordering, and social commerce hooks.

## Product Pillars

1. **Guided Onboarding**
   - Auto-detect locale → Zalo/Facebook social sign-in → business profile → template pick → starter menu.
   - Vietnamese language copy by default; provide English toggle when needed.
2. **Mobile-First Admin Dashboard**
   - Tailwind-based responsive layout with thumb-friendly controls.
   - Key modules: Home snapshot, Menu CRUD (supports variants/add-ons), Website appearance, Integrations, Settings.
   - Offline-friendly drafts with local storage fallbacks.
3. **Integrated Commerce & Social Presence**
   - Generate Zalo Mini App + Messenger bot from the same menu schema.
   - Publish to `store.ourplatform.com` subdomains; custom domains are Growth-tier and above.
   - Webhooks for GrabFood/ShopeeFood/Gojek to sync catalog and handle order events.
4. **Localized Payments & Compliance**
   - Abstracted payment layer supporting MoMo, ZaloPay, VNPay; design for future providers.
   - Settlement reporting in VND with downloadable statements.

## Technical Blueprint

- **Frontend:** Next.js app (admin + storefront), Tailwind CSS, TypeScript, SWR/React Query for data fetching, PWA baseline.
- **Backend:** NestJS monolith (modular structure) with multi-tenant context middleware.
- **Data Layer:** PostgreSQL (tenancy via `tenant_id`), Redis cache for menus & sessions, S3-compatible storage for media with auto-resize Lambda/worker.
- **Integrations:**
  - Zalo OAuth & Mini App APIs.
  - Facebook Messenger webhook + auto-responder template.
  - Payment gateways via individual provider SDKs wrapped in unified service.
  - Delivery partners via REST webhooks (catalog sync + order status updates).
- **Observability:** Structured logging (JSON), metrics via OpenTelemetry, analytics events for onboarding completion, menu publish, payment init/success.

## Delivery Roadmap (Vietnam Scope)

- **Phase 0 (Weeks 1-4):**
  - Finalize tenant data model & infra scaffolding.
  - Draft Figma components and onboarding wizard prototype.
  - Secure sandbox credentials (Zalo, MoMo, ZaloPay, VNPay, GrabFood).
- **Phase 1 MVP (Weeks 5-16):**
  - Zalo/email auth, tenant provisioning, menu CRUD, template engine, SSG publishing, QR generator.
  - Growth-tier gating, billing integration with localized pricing.
  - Beta testing with ≥10 pilot stores; ensure <2s load on 4G devices.
- **Phase 2 Enhancements (Weeks 17-28):**
  - Launch Zalo Mini App builder, Messenger bot automation.
  - Implement payment orchestration, delivery partner webhooks.
  - Advanced analytics dashboard, custom domain automation.
- **Phase 3 Scaling (Ongoing):**
  - Full ecommerce (cart, checkout, promotions), loyalty features, SEO tooling, data warehousing.

## KPIs & Monitoring

- Onboarding completion rate ≥90%.
- Time-to-first-menu publish ≤15 minutes median.
- Mobile dashboard TTI <2s on low-end Android (Lighthouse score ≥85 mobile).
- Payment success rate ≥95% across integrated gateways.
- Churn <5% monthly in Growth tier once live.

## Testing & QA Guidelines

- Simulate 4G latency/bandwidth in automated tests (e.g., Playwright network throttling).
- Include contract tests for each third-party integration with sandbox fixtures.
- Validate VND currency formatting, date localization, and bilingual toggles.
- Run accessibility audits (WCAG AA) especially for mobile contrast.

## Content & Localization Assets

- Maintain Vietnamese copy deck (CTA, onboarding steps, notifications) with tone: professional, warm, nationwide friendly.
- Provide sample menu dataset featuring dishes like phở bò, bún chả, bánh mì, cà phê sữa đá.
- Use template imagery that highlights vibrant street scenes, fresh ingredients, and modern street-food aesthetics.

## Open Questions & Follow-Ups

- Confirm regulatory requirements for storing transaction data in Vietnam (data residency laws evolving).
- Explore partnerships for POS integration (e.g., KiotViet, Sapo) for inventory sync.
- Determine customer support operating hours & staffing for Vietnamese merchants.
- Evaluate need for in-app learning center or video walkthroughs for onboarding.

Keep this page updated as features ship, customer feedback emerges, or market assumptions change.
