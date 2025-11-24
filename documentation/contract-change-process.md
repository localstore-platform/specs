Contract change process — safe API/Schema evolution

Purpose
Short, enforceable process for changing API contracts (OpenAPI / GraphQL) so consumers and CI gates remain reliable.

Principles

- Backwards compatibility by default. Breaking changes must be deliberate and coordinated.
- Contracts are versioned and published from `local-store-contracts`.
- Contract changes trigger automated contract-validation and consumer contract tests.

Process

1. Create a draft PR in `local-store-contracts` with the proposed change and a clear description: breaking or non-breaking, rationale, migration notes.
2. Add a `compatibility` section in the PR body: explain how clients should be updated and whether a feature flag or dual-version approach is required.
3. CI actions on PR:
   - Run schema lint and style checks
   - Run automated compatibility checks (OpenAPI diff or GraphQL schema check)
   - If change is breaking, require explicit approval from API owners and consumer owners (CODEOWNERS)
4. Publish a snapshot release (e.g., v1.2.0-beta) upon PR merge to `main` in contracts repo.
5. Consumers must update CI to run contract tests against the new snapshot. If tests fail, coordinate fixes or roll forward a compatibility shim.
6. For breaking changes, follow a deprecation timeline in the PR and documentation (example: remove field after 4 release cycles).

Automation

- Use a contract-diff tool (e.g., OpenAPI-diff, graphql-inspector) in CI.
- On publish, notify consumers via CI webhook and create a maintainer task template for each consumer repo.

Emergency rollback

- If a published contract causes widespread failures, revert the contract change and issue a patch release. Consumers should pin to a safe contract version during incident resolution.

Communications

- Add a clear changelog entry for each contract release.
- Notify via Slack #api-updates and tag service owners.

Acceptance

- Contract changes are traceable, tested, and have an owner. Consumers are notified and CI gates exist to catch regressions.
