# Current Work – Contracts Repository

> **Last Updated:** 2025-01-XX  
> **Current Sprint:** Sprint 0.5 (Menu Demo)  
> **Sprint Spec:** [planning/sprint-0.5-menu-demo.md](https://github.com/localstore-platform/specs/blob/master/planning/sprint-0.5-menu-demo.md)

---

## Sprint 0.5 Stories (This Repo)

| Story | Description | Status | Notes |
|-------|-------------|--------|-------|
| 1.2 | Menu Types & Schemas | 🔴 Not Started | MenuItem, Category, etc. |
| 3.2 | API Response Types | 🔴 Not Started | Menu endpoint contracts |

**Status Legend:** 🔴 Not Started | 🟡 In Progress | ✅ Done | ⏸️ Blocked

---

## Spec References

| Story | Specification | Lines |
|-------|--------------|-------|
| 1.2 | [database-schema.md](https://github.com/localstore-platform/specs/blob/master/architecture/database-schema.md) | L50-L150 |
| 3.2 | [api-specification.md](https://github.com/localstore-platform/specs/blob/master/architecture/api-specification.md) | L200-L280 |

---

## Current Focus

**Next Task:** Story 1.2 - Menu Types & Schemas

**Requirements:**

- Create `src/types/menu.ts` with MenuItem, Category interfaces
- Create `src/schemas/menu.ts` with Zod schemas
- Price as number (VND integer)
- Support Vietnamese text fields
- Export from index

---

## Session Notes

### Session: YYYY-MM-DD

- Started: [Story X.X]
- Completed: [What was done]
- Blockers: [Any issues]
- Next: [What to do next]

---

## Blockers

None currently.

---

## Quick Commands

```bash
npm install        # Install dependencies
npm run build      # Build types
npm run test       # Run tests
npm run lint       # Lint code
npm run typecheck  # Type checking
```
