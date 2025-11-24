README Template — local-store-contracts

Purpose
Canonical API contract repository. Hosts OpenAPI specs, GraphQL SDL, schema versions, and generated contract SDKs (or codegen steps).

Quick summary

- Tech: YAML/SDL artifacts, optional codegen scripts
- Audience: Backend, frontend, QA, integration teams
- Responsibility: Single source of truth for API contracts and test vectors

Suggested layout

- /openapi/
- /graphql/
- /postman/
- /codegen/ (scripts to generate SDKs)
- /releases/ (versioned contract snapshots)

Versioning & Release

- Use semantic versioning for contract releases. Tag releases as `v1.0.0`, `v1.1.0`.
- Consumers should pin to a contract version. Breaking changes require a major version bump.

CI & Automation

- On contract change, run contract-validation and publish a new release.
- Trigger consumer repository contract tests in CI (via webhook or CI orchestration).

Ownership

- CODEOWNERS: API/platform lead

Notes

- Keep test vectors (webhook samples) here for QA.
