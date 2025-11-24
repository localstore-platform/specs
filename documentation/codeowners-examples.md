# CODEOWNERS Examples and Guidance

Place a `CODEOWNERS` file at the repository root or in `.github/` to automatically request reviews from responsible teams. Below are suggested entries for each implementation repo. Replace `@org/team-...` with your GitHub org/team handles.

## Example: `local-store-web` (Next.js)

```
# Frontend ownership
/.github/ @org/team-frontend
/src/ @org/team-frontend
/pages/ @org/team-frontend
/components/ @org/team-frontend

# Docs and UX
/public/ @org/team-ux
```

## Example: `local-store-api` (NestJS)

```
# Backend ownership
/.github/ @org/team-api
/src/controllers/ @org/team-api
/src/services/ @org/team-api
/migrations/ @org/team-api

# Security and infra touchpoints
/src/security/ @org/team-security
```

## Example: `local-store-graphql`

```
# GraphQL gateway
/.github/ @org/team-graphql
/schema/ @org/team-graphql
/src/gateway/ @org/team-graphql
```

## Example: `local-store-ml`

```
# ML team
/.github/ @org/team-ml
/notebooks/ @org/team-ml
/src/features/ @org/team-ml
```

## Example: `local-store-mobile` (Flutter)

```
# Mobile ownership
/.github/ @org/team-mobile
/lib/ @org/team-mobile
/android/ @org/team-mobile
/ios/ @org/team-mobile
```

## Example: `local-store-contracts`

```
# Contracts ownership
/.github/ @org/team-contracts
/openapi/ @org/team-contracts
/schema/ @org/team-contracts
```

## Example: `local-store-infra`

```
# Infra ownership
/.github/ @org/team-infra
/terraform/ @org/team-infra
/charts/ @org/team-infra
```

## Example: `local-store-ops`

```
# Ops and runbooks
/.github/ @org/team-ops
/runbooks/ @org/team-ops
/monitoring/ @org/team-ops
```

## Guidance

- Prefer team-level handles (e.g., `@org/team-api`) not individuals.
- Use directory-level ownership for large repos and file-level for special cases (migrations, secrets, infra).
- Keep `CODEOWNERS` minimal to avoid noisy review requests.
- Add a section in each repo README describing who to ping for urgent approvals.
