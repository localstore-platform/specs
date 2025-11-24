README Template — local-store-web

Purpose
This repository contains the Next.js web application(s) for storefronts and merchant/admin dashboards.

Quick summary

- Tech: Next.js, TypeScript, Tailwind CSS
- Audience: Frontend engineers, product designers
- Responsibility: UI features, storefront templates, client-side analytics hooks

Suggested layout

- /apps/
  - /merchant-dashboard/
  - /storefront/
- /components/ (shared UI components)
- /public/
- /styles/
- /integration-tests/
- /docs/ (frontend-specific guides)

Onboarding (short)

1. Install Node 18+, pnpm
2. pnpm install
3. pnpm dev

CI & Deploy

- CI should run lint, unit tests, and basic accessibility checks.
- Deploy previews on PRs; production deploy via `main` -> Vercel or `local-store-infra` pipeline.

Contracts & API

- Depend on `local-store-contracts` for API contract references. Do not hardcode API shapes; use generated clients where possible.

Ownership

- CODEOWNERS: frontend lead
- Contacts: frontend@ourplatform

Notes

- Keep i18n strings external (vi and en). Use `vi-VN` and VND formatting by default.
- Performance: TTI < 2s target for mobile-first UX.
