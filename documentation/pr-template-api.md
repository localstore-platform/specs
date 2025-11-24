# Pull Request Template — API (NestJS / REST)

## Summary

- One-line summary of the API change.

## Related issue / ticket

- Issue: [JIRA/Issue #]

## Type of change

- [ ] Bugfix
- [ ] Feature
- [ ] Breaking change
- [ ] Docs
- [ ] Chore

## Endpoints / DTOs changed

- List endpoints added/changed/removed and DTOs affected.

## Database / Migrations

- Describe DB migrations required. Add migration file path and required downtime (if any).

## Contract / OpenAPI

- Did this change the OpenAPI or clients? (yes/no). If yes:
  - Add OpenAPI diff and point to `local-store-contracts` PR/branch.
  - Update contract tests and client generation steps.

## Idempotency / Concurrency

- Note any idempotency or ordering considerations (e.g., event_seq, idempotency_key).

## How to test locally

1. Run migrations: `pnpm --filter api migrate` (example)
2. Start server and run smoke tests

## Checklist

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Contract tests updated (if contract changed)
- [ ] Swagger / OpenAPI updated
- [ ] API docs updated
- [ ] Security review performed for input validation/auth

## Rollout

- Backwards compatibility notes and migration plan
- Consumer notification required? (list owners)

Reviewer tips: highlight the places reviewers should focus on (migrations, contract changes, auth scopes).
