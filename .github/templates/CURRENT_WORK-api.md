# Current Work – API Repository

> **Last Updated:** 2025-01-XX  
> **Current Sprint:** Sprint 0.5 (Menu Demo)  
> **Sprint Spec:** [planning/sprint-0.5-menu-demo.md](https://github.com/localstore-platform/specs/blob/master/planning/sprint-0.5-menu-demo.md)

---

## Sprint 0.5 Stories (This Repo)

| Story | Description | Status | Notes |
|-------|-------------|--------|-------|
| 2.1 | Menu API Endpoints | 🔴 Not Started | GET /api/v1/menu/:tenantId |
| 2.2 | Mock Data Seeder | 🔴 Not Started | Vietnamese sample menu |
| 2.3 | Health Check & CORS | 🔴 Not Started | Allow menu frontend origin |

**Status Legend:** 🔴 Not Started | 🟡 In Progress | ✅ Done | ⏸️ Blocked

---

## Spec References

| Story | Specification | Lines |
|-------|--------------|-------|
| 2.1 | [api-specification.md](https://github.com/localstore-platform/specs/blob/master/architecture/api-specification.md) | L200-L280 |
| 2.2 | [database-schema.md](https://github.com/localstore-platform/specs/blob/master/architecture/database-schema.md) | L50-L150 |
| 2.3 | [api-specification.md](https://github.com/localstore-platform/specs/blob/master/architecture/api-specification.md) | L50-L80 |

---

## Current Focus

**Next Task:** Story 2.1 - Menu API Endpoints

**Requirements:**

- Create `src/modules/menu/` module
- GET `/api/v1/menu/:tenantId` - Public menu endpoint
- GET `/api/v1/menu/:tenantId/categories` - Categories list
- Response includes items grouped by category
- Price in VND (integer, formatted on client)

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
npm run start:dev  # Start dev server (localhost:3000)
npm run test       # Run tests
npm run test:e2e   # Run e2e tests
npm run lint       # Lint code
npx prisma migrate dev  # Run migrations
npx prisma db seed      # Seed database
```
