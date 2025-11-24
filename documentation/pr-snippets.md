# PR-ready Snippets and Copy-Paste Templates

Below are short, PR-ready description snippets you can paste into a PR body and then fill the placeholders. Use the repo-specific snippet that matches the change. Place these into the new implementation repo README or copy into the PR when creating a change.

---

## Web (Next.js) — PR snippet

Summary

- [short summary] Add support for {feature} on the storefront (Next.js).

Why

- [brief justification] Improves checkout flow for QR sessions and reduces latency for browsing on low-end Android.

What changed

- Added: `/pages/checkout`, `components/Cart`, `hooks/useSessionPoll`
- Updated: i18n strings for `vi-VN`, responsive layout for mobile

How to test

1. Install and run: `pnpm install && pnpm dev`
2. Open <http://localhost:3000> on a mobile viewport
3. Walk through: open menu → add item → view cart → simulate payment

Notes

- Storybook screenshots attached. Review accessibility (focus order) and Vietnamese copy.

---

## API (NestJS / REST) — PR snippet

Summary

- [short summary] Implement POST `/api/v1/sessions` to create QR sessions.

Why

- Needed so the client can obtain a `session_id` and `qr_url` for table-based checkout.

What changed

- Added controller: `src/controllers/sessions.controller.ts`
- Added service + DTOs + migration for `qr_sessions` table

DB / Migrations

- Migration: `migrations/20251101_create_qr_sessions.sql`
- No downtime expected. Consumers: `local-store-web` will call new endpoint.

How to test

1. Apply migrations: `pnpm --filter api migrate`
2. Run unit tests for sessions
3. Curl endpoint: `POST /api/v1/locations/{locationId}/qr-sessions` with sample payload

Notes

- If OpenAPI / contract changed, bump `local-store-contracts` and run contract tests.

---

## GraphQL — PR snippet

Summary

- [short summary] Add `sessionSnapshot` query to gateway schema.

What changed

- SDL updated: `schema/session.graphql`
- Resolver added in gateway to stitch data from API and cache

How to test

1. Start gateway: `pnpm --filter graphql dev`
2. Run schema lint: `yarn graphql:validate`

Notes

- This is non-breaking for current consumers; if you rename fields, follow the contract-change-process.

---

## ML — PR snippet

Summary

- [short summary] Train ranking model v0.2 for menu recommendations.

What changed

- Training script + notebooks and updated feature transforms
- Model artifact stored at `s3://local-store-models/ranking/v0.2`

How to validate

1. Run a small local training using provided sample dataset
2. Validate metrics are >= baseline

Notes

- Include model card and tag artifact for reproducibility.

---

## Mobile (Flutter) — PR snippet

Summary

- [short summary] Add session snapshot view and cart flow.

What changed

- New pages and integration with the JS/TS client helper

How to test

1. Run on emulator: `flutter run -d <device>`
2. Test on Android low-end emulator (512MB) for performance

Notes

- Ensure both Android and iOS builds succeed in CI.

---

## Contracts — PR snippet

Summary

- [short summary] Add new endpoint to OpenAPI: `POST /api/v1/sessions`.

Why

- Required by `local-store-web` to create QR sessions.

What changed

- Updated `openapi/session.yaml` and examples.

Contract Process

- This is a non-breaking addition. If you mark it breaking, follow `contract-change-process.md` and add consumer approvers.

How to test

1. Run contract tests: `pnpm test:contracts`
2. Regenerate clients and smoke-test.

---

## Infra — PR snippet

Summary

- [short summary] Add new S3 bucket + IAM policy for model artifacts.

Plan output

- Attach Terraform plan and list resources to be created.

Rollback

- Provide rollback steps and state handling notes.

---

## Ops — PR snippet

Summary

- [short summary] Update reconciliation runbook and add alert for `payment_match_rate < 95%`.

What changed

- Runbook steps, dashboards and alert rules updated

How to validate

1. Deploy alert to staging and run a simulated incident tabletop

---

## Tips

- Keep PR bodies short and use the snippets above. Attach diffs for migrations and plan outputs where appropriate. For breaking changes, always link the `local-store-contracts` PR and list consumer approvers.
