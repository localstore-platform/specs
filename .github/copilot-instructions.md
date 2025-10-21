# GitHub Copilot Instructions – Vietnam Market Focus

These instructions tailor GitHub Copilot to the immediate implementation track: launching the Local Store Platform for Vietnamese small businesses. Apply them whenever generating code, docs, or tests for this repository until the Japan roadmap is reactivated.

## Product Vision
- Deliver a mobile-first SaaS builder that lets Vietnamese shops publish localized menu sites quickly.
- Embrace hyper-localization: assume owners operate from low-cost Android phones, rely on Zalo/Facebook, and need effortless payment + delivery integrations.

## Technical Guardrails
- **Primary stack:** Next.js (React) + Tailwind CSS frontend, NestJS backend (Node.js), PostgreSQL, Redis cache, S3-compatible storage.
- **Architecture:** Multi-tenant SaaS. Always scope database queries, storage keys, and caches by `tenant_id`.
- **Infrastructure:** Prefer serverless/static delivery (Vercel) for storefronts, containerized services for APIs. Provide IaC hooks when adding new infrastructure pieces.
- **Code quality:** Enforce TypeScript on both frontend and backend. Aim for strong typing, reusable domain services, and unit/integration coverage for critical paths.

## Vietnam-Specific Functional Priorities
1. **Onboarding wizard**
   - Steps: landing locale detection → social signup (Zalo/Facebook) → business profile → template pick (VN set) → menu starter.
   - Must support Vietnamese language copy and currency formatting in VND.
2. **Admin dashboard**
   - Mobile-first layout. Large tap targets, minimal typing. Offline-friendly drafts when possible.
   - Core modules (in order): Home metrics, Menu CRUD, Website appearance, Integrations, Settings.
3. **Integrations**
   - Authentication: Zalo OAuth (primary) with email fallback.
   - Payments: MoMo, ZaloPay, VNPay (abstract behind integration layer for later expansion).
   - Social/delivery: Zalo Mini App generator, Facebook Messenger bot bootstrap, connectors for GrabFood/ShopeeFood/Gojek (webhook-based).
4. **Publishing**
   - Subdomain issuance like `store.ourplatform.com` with CDN caching. Include QR code generation for each menu.

## UX & Content Guidelines
- Design language: vibrant, street-food friendly palettes, culturally familiar imagery.
- Always use Vietnamese locale defaults (vi-VN), including dates, currency (₫), and numbering.
- Copy tone: friendly but professional; avoid slang that may not generalize nationwide.
- Provide bilingual hooks (vi/en) in data models, but English content can remain placeholder for now.

## Collaboration & Language Use (VN/EN)
- Team may mix Vietnamese and English in issues/PRs. Default assistant replies in English.
- On first use of a Vietnamese term, propose an English equivalent in parentheses and keep the original (e.g., "khuyến mãi (promotion)").
- Keep brand/product names as-is (Zalo Mini App, MoMo, VNPay). Clarify ambiguities briefly when needed.
- Maintain a lightweight glossary in `knowledge-base/glossary.md` and update when new recurring terms appear.

## Non-Functional Requirements
- Optimize for 4G on entry-level Android: target <2s TTI for admin dashboard and storefront pages.
- Image handling: automatic compression/resizing; keep uploads under 2 MB unless flagged otherwise.
- Security: JWT-based sessions, per-tenant data isolation, rate limiting on public APIs.
- Observability: instrument key flows (onboarding step completion, menu publish, payment initiation) with analytics hooks.

## Testing Expectations
- Cover happy paths plus low-bandwidth scenarios (simulate slow network in tests/mocks when feasible).
- Include contract tests around third-party integrations with fixtures for sandbox responses.
- Validate localization data (currency, date formatting) in unit tests.

## Deliverables Checklist
- Update relevant documentation (README, knowledge base) when introducing new VN-specific features.
- Provide sample data or seeds tailored to Vietnamese cuisine (e.g., phở, cà phê sữa đá) when demonstrating features.
- After significant changes, log open questions or follow-ups in the knowledge base.

Use this as your default instruction set until broader multi-market support is resumed.
