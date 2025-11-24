# Pull Request Template — Web (Next.js)

## Summary

- Short description of the change (1-2 sentences).

## Related issue / ticket

- Issue: [JIRA/Issue #]

## Type of change

- [ ] Bugfix
- [ ] Feature
- [ ] Refactor
- [ ] Docs
- [ ] Chore

## Changes

- List key changes made (components, pages, hooks, styles).

## Screenshots / Recordings

- Attach screenshots or short screen recordings demonstrating the UI change.

## How to test locally

1. Branch & install: `pnpm install` (or `npm install`)
2. Start dev server: `pnpm dev`
3. Steps to reproduce the change

## Cross-cutting concerns

- Accessibility: keyboard & screen reader pass? (yes/no)
- Responsiveness: mobile / tablet / desktop verified? (yes/no)
- Performance: bundle size impact / lighthouse notes
- i18n: Vietnamese (`vi-VN`) strings included/updated

## Checklist

- [ ] Unit / integration tests added or updated
- [ ] Lint passed (pre-commit hooks)
- [ ] Storybook / visual snapshots updated (if applicable)
- [ ] E2E checks (if relevant) updated
- [ ] Documentation updated (`/docs` or `README.md`)

## Deployment / Rollout notes

- Any environment feature flags or gating required?
- Rollout plan (canary/100%)

---

Optional notes for reviewers: suggest the most relevant reviewers and highlight any risky areas (data, auth, payments).
