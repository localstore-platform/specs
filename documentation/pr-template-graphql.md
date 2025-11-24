# Pull Request Template — GraphQL / Gateway

## Summary

- Short description of schema or gateway change.

## Related issue / ticket

- Issue: [JIRA/Issue #]

## Type of change

- [ ] Schema change
- [ ] Resolver / server logic
- [ ] Docs
- [ ] Chore

## GraphQL Schema Changes

- List types/queries/mutations/subscriptions added/changed/deprecated.
- If breaking, list impacted consumers and migration steps.

## Contract & Versioning

- Update GraphQL SDL and add schema diff. Follow `contract-change-process.md` and bump semantic version where appropriate.

## Tests

- [ ] Unit tests for resolvers
- [ ] Integration tests for schema
- [ ] Contract tests for consumers

## How to test locally

1. Start gateway: `pnpm --filter graphql dev`
2. Run schema lint and validation

## Checklist

- [ ] Schema documented
- [ ] Consumer owners notified (if breaking)
- [ ] Contract tests updated
