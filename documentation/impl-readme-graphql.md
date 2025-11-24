README Template — local-store-graphql

Purpose
GraphQL gateway and schema orchestration repository. Hosts the GraphQL SDL, federation setup, and gateway glue code.

Quick summary

- Tech: Apollo Federation (or GraphQL Yoga), TypeScript
- Audience: API architects, backend engineers
- Responsibility: Compose services, provide GraphQL entrypoint for clients, schema governance

Suggested layout

- /schema/ (SDL, federation configs)
- /gateway/ (runtime gateway code)
- /services/ (if hosting federated microservices)
- /tests/schema-contracts/

Onboarding

1. pnpm install
2. pnpm start:dev (local gateway; requires local instances of dependent services)

Schema governance

- All schema changes must be proposed as PRs with schema diff and compatibility notes.
- Use automated schema validation in CI (check for breaking changes against published schema versions).

Contracts

- Source-of-truth for GraphQL SDL may be published to `local-store-contracts`.

Ownership

- CODEOWNERS: API/platform lead

Notes

- Prefer schema additions over breaking modifications; deprecate fields with clear timelines.
