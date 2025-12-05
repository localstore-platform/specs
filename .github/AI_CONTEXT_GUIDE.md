# AI Context Guide for Implementation Repositories

This document explains how AI coding assistants should work with this documentation repository when building implementation repositories.

## Overview

This repository (`local-store-platform`) is a **specification-only repository** that serves as the single source of truth for the entire Local Store Platform project. Implementation code lives in separate repositories that reference these specs.

**Key Principle:** AI assistants should load specifications **on-demand** rather than loading the entire documentation repository upfront, to avoid context window overload.

---

## For AI Coding Assistants in Implementation Repos

### Initial Setup

When you start working in an implementation repository (e.g., `backend-api`, `mobile-app`), follow this workflow:

1. **Read `docs/SPEC_LINKS.md`** in the implementation repo
   - This file contains curated links to relevant specification sections
   - It specifies which spec version (git tag) the repo is compatible with

2. **Note the spec version** (e.g., `v1.1-specs`)
   - All spec references should use this pinned version
   - Ensures consistency across development

3. **Load specs on-demand** as needed for each feature
   - Use `semantic_search` to find relevant sections
   - Use `read_file` with line ranges to load only needed portions
   - Never load the entire spec repository into context

### Context Loading Strategy

**❌ DON'T:**

- Load all spec files upfront (causes context overload)
- Work across multiple implementation repos simultaneously
- Make changes to spec repo and implementation repo in same session

**✅ DO:**

- Work in one implementation repo at a time
- Load only the specific spec sections relevant to current task
- Reference spec version consistently (use git tags)
- Use semantic search to discover relevant specs

### Example Workflows

#### Workflow 1: Implementing a New Feature

**User Request:** "Implement the OTP authentication endpoint"

**AI Steps:**

1. **Check spec reference:**

   ```
   read_file("docs/SPEC_LINKS.md")
   → Notes: Spec version is v1.1-specs
   → Relevant specs: api-specification.md, database-schema.md
   ```

2. **Search for feature details:**

   ```
   semantic_search(
     repo: "https://github.com/donggiangthai/local-store-platform",
     query: "OTP authentication endpoint request response"
   )
   → Found: architecture/api-specification.md lines 150-280
   ```

3. **Load specific sections:**

   ```
   read_file("architecture/api-specification.md", lines: 150-280)
   → Loads ~3k tokens with OTP endpoint specs
   
   read_file("architecture/database-schema.md", lines: 85-125)
   → Loads ~1k tokens with users/otp_sessions tables
   ```

4. **Implement based on specs:**

   ```typescript
   // Generate auth.controller.ts following spec exactly
   @Post('/auth/otp/request')
   async requestOtp(@Body() body: OtpRequestDto) {
     // Implementation matches spec request/response schemas
   }
   ```

5. **Total context:** Current repo code (20k) + Spec sections (5k) = 25k tokens ✅

#### Workflow 2: Understanding Existing Code

**User Request:** "Why is this database query scoped by tenant_id?"

**AI Steps:**

1. **Check architecture decisions:**

   ```
   semantic_search(
     repo: "https://github.com/donggiangthai/local-store-platform",
     query: "multi-tenant tenant_id row level security"
   )
   ```

2. **Load relevant spec:**

   ```
   read_file("architecture/database-schema.md", lines: 50-100)
   → Explains RLS policies and multi-tenancy strategy
   ```

3. **Explain to user:**

   ```
   According to the database schema spec, all queries must be scoped
   by tenant_id to enforce multi-tenancy isolation...
   ```

#### Workflow 3: API Contract Validation

**User Request:** "Does this API response match the spec?"

**AI Steps:**

1. **Find endpoint spec:**

   ```
   semantic_search(
     repo: "https://github.com/donggiangthai/local-store-platform",
     query: "dashboard metrics endpoint response schema"
   )
   ```

2. **Load spec section:**

   ```
   read_file("architecture/api-specification.md", lines: 500-600)
   ```

3. **Compare:**

   ```
   Spec expects:
   {
     "revenue": { "current": number, "change_percent": number },
     "orders": { "current": number, "change_percent": number }
   }
   
   Code returns:
   {
     "revenue": { "current": 250000, "changePercent": 15.5 },
     ...
   }
   
   ❌ Field name mismatch: "change_percent" vs "changePercent"
   → Should be snake_case per API spec conventions
   ```

---

## Specification File Map

### By Implementation Area

| Implementation Repo | Required Specifications | Priority |
|---------------------|------------------------|----------|
| **backend-api** (NestJS) | • `architecture/api-specification.md`<br>• `architecture/backend-setup-guide.md`<br>• `architecture/database-schema.md`<br>• `architecture/graphql-schema.md` | HIGH |
| **ml-service** (Python/FastAPI) | • `architecture/api-specification.md` (gRPC section)<br>• `planning/analytics-ai-strategy.md`<br>• `architecture/backend-setup-guide.md` (AI service) | HIGH |
| **mobile-app** (Flutter) | • `architecture/flutter-mobile-app-spec.md`<br>• `architecture/api-specification.md` (REST endpoints)<br>• `design/wireframes-ux-flow.md` | HIGH |
| **web-admin** (Next.js) | • `architecture/graphql-schema.md`<br>• `design/wireframes-ux-flow.md`<br>• `architecture/api-specification.md` (REST fallback) | MEDIUM |
| **infra** (Terraform) | • `architecture/backend-setup-guide.md` (Deployment section)<br>• `architecture/decision-hybrid-architecture.md`<br>• `architecture/system-diagram.md` | MEDIUM |
| **contracts** (Shared types) | • `architecture/graphql-schema.md`<br>• `architecture/database-schema.md`<br>• `architecture/api-specification.md` | LOW |

### By Feature Type

**Authentication Feature:**

- `architecture/api-specification.md` (lines 120-280: OTP, JWT endpoints)
- `architecture/database-schema.md` (lines 85-140: users, otp_sessions)
- `architecture/backend-setup-guide.md` (lines 450-550: JWT config)

**Dashboard Metrics:**

- `architecture/api-specification.md` (lines 500-750: Metrics endpoints)
- `architecture/database-schema.md` (lines 200-380: orders, analytics views)
- `architecture/flutter-mobile-app-spec.md` (lines 250-450: Dashboard UI)

**AI Recommendations:**

- `architecture/api-specification.md` (lines 800-950: Recommendations endpoints)
- `architecture/database-schema.md` (lines 520-620: ai_recommendations table)
- `planning/analytics-ai-strategy.md` (ML model specs)
- `architecture/flutter-mobile-app-spec.md` (lines 600-780: Recommendations UI)

**Multi-Location Management:**

- `architecture/api-specification.md` (lines 350-480: Locations endpoints)
- `architecture/database-schema.md` (lines 140-200: locations, tenant_locations)
- `architecture/multi-domain-user-management.md`

---

## Spec Update Protocol

### When Specs Change

1. **Spec repository updated** (PR merged in `local-store-platform`)
2. **New version tagged** (e.g., `v1.1-specs`)
3. **SPEC_CHANGELOG.md updated** with changes
4. **Implementation repos notified** (GitHub issue or Slack)
5. **Implementation repos update:**
   - Review `SPEC_CHANGELOG.md` for breaking changes
   - Update `docs/SPEC_LINKS.md` to reference new version
   - Update `.github/copilot-instructions.md` if needed
   - Implement changes to match new specs
   - Run integration tests
   - Update repo version (e.g., `v0.2.0`)

### Handling Breaking Changes

**Breaking changes include:**

- API endpoint path changes
- Request/response schema changes (field renames, type changes)
- Database schema migrations
- Authentication flow changes

**Process:**

1. Spec repo tags breaking change as **major version** (e.g., `v2.0-specs`)
2. Implementation repo creates migration plan:
   - API versioning (e.g., `/v1/` → `/v2/`)
   - Database migrations
   - Backward compatibility period
3. All dependent repos coordinate update timing
4. Mobile app releases new version with updated API client

---

## Best Practices for AI Assistants

### 1. Verify Spec Version Match

Before implementing, confirm:

```
Current repo spec version: v1.1-specs
Spec being referenced: v1.1-specs
✅ Match confirmed
```

If mismatch:

```
⚠️ Warning: Repo uses v1.1-specs, but you're referencing v1.1-specs
→ Either update repo's SPEC_LINKS.md or use v1.1-specs
```

### 2. Follow Spec Exactly

When generating code:

- **Field names:** Match spec exactly (snake_case for API, camelCase for UI)
- **Types:** Use TypeScript/Dart types that match spec schemas
- **Validation rules:** Implement all constraints mentioned in spec
- **Error messages:** Use Vietnamese messages from spec examples

### 3. Vietnamese Localization

All user-facing content must be in Vietnamese:

```typescript
// ✅ Correct (from spec)
throw new BadRequestException('Không tìm thấy cửa hàng');

// ❌ Wrong
throw new BadRequestException('Store not found');
```

Check `knowledge-base/glossary.md` for standard translations.

### 4. Multi-Tenancy Scoping

Always scope database queries by `tenant_id`:

```typescript
// ✅ Correct (from database-schema.md spec)
await this.repository.find({
  where: { tenant_id: user.tenantId, location_id: locationId }
});

// ❌ Wrong (security vulnerability)
await this.repository.find({
  where: { location_id: locationId }
});
```

### 5. Reference Spec in Code Comments

When implementing complex logic, reference spec:

```typescript
/**
 * Approve AI recommendation
 * 
 * Spec: architecture/api-specification.md lines 850-880
 * Updates recommendation status to 'approved' and triggers execution
 */
async approveRecommendation(id: string, userId: string) {
  // Implementation follows spec
}
```

---

## Tool Usage Patterns

### Semantic Search

**When to use:**

- You don't know which spec file contains the information
- You need to find all mentions of a concept across specs
- You're exploring a new feature area

**Example:**

```javascript
semantic_search({
  repo: "donggiangthai/local-store-platform",
  query: "Vietnamese currency formatting 75.000₫"
})
// Returns: flutter-mobile-app-spec.md, api-specification.md
```

### Read File with Line Ranges

**When to use:**

- You know the specific spec file and section
- You want to minimize context window usage
- You need just one specific endpoint/schema

**Example:**

```javascript
read_file({
  filePath: "architecture/api-specification.md",
  offset: 150,
  limit: 50
})
// Loads lines 150-200 only (~2k tokens vs 50k for full file)
```

### Grep Search

**When to use:**

- You need to find exact string matches
- You want to see all occurrences of a field name
- You're tracking down inconsistencies

**Example:**

```javascript
grep_search({
  query: "tenant_id",
  includePattern: "architecture/*.md",
  isRegexp: false
})
// Shows all places where tenant_id is mentioned in architecture specs
```

---

## Context Window Management

### Estimated Token Usage

| Item | Tokens | When to Load |
|------|--------|--------------|
| Full spec repo | ~150k | ❌ Never |
| Single spec file (complete) | ~30-50k | ❌ Rarely |
| Spec file section (100 lines) | ~2-5k | ✅ Yes |
| Implementation repo (complete) | ~30-80k | ✅ Yes (you're working here) |
| SPEC_LINKS.md | ~1-2k | ✅ Yes (always load first) |
| Code examples from spec | ~500-2k | ✅ Yes (per example) |

### Target Context Budget

Aim for this distribution:

```
Total context: ~50k tokens (well under 200k limit)
├─ Implementation repo code: ~35k (70%)
├─ Loaded spec sections: ~10k (20%)
├─ Conversation history: ~5k (10%)
└─ Buffer for responses: ~150k remaining
```

### If Context Grows Too Large

1. **Invalidate old spec sections:** Remove from context after implementation
2. **Summarize conversation:** Condense earlier parts of chat
3. **Focus on current feature:** Remove unrelated code from view
4. **Split task:** Break large feature into smaller subtasks

---

## Cross-Repository Coordination

### Scenario: API Change Affects Multiple Repos

**Example:** Add new field to Dashboard Metrics API

**Workflow:**

1. **Spec repo** (`local-store-platform`):

   ```
   ├─ Update architecture/api-specification.md
   ├─ Update architecture/database-schema.md
   ├─ Update SPEC_CHANGELOG.md
   ├─ Create PR, review, merge
   └─ Tag v1.1-specs
   ```

2. **Backend repo** (`backend-api`):

   ```
   ├─ AI reads new spec section (v1.1-specs)
   ├─ AI generates database migration
   ├─ AI updates API endpoint
   ├─ AI updates tests
   ├─ Update SPEC_LINKS.md to v1.1-specs
   └─ Deploy
   ```

3. **Mobile repo** (`mobile-app`):

   ```
   ├─ AI reads new spec section (v1.1-specs)
   ├─ AI updates data models
   ├─ AI updates UI to show new field
   ├─ AI updates tests
   ├─ Update SPEC_LINKS.md to v1.1-specs
   └─ Release new version
   ```

**Key:** Each AI session works in its own repo, loading specs on-demand.

---

## Troubleshooting

### Problem: Spec Not Found

**Error:** "Cannot find specification for feature X"

**Solution:**

1. Check `SPEC_CHANGELOG.md` for spec version you're using
2. Search semantic: `semantic_search(query: "feature X")`
3. Check if feature was added in later spec version
4. Consider updating to newer spec version

### Problem: Spec Conflicts with Code

**Error:** "Existing code doesn't match spec"

**Solution:**

1. Check spec version: Is code using older spec?
2. Check `SPEC_CHANGELOG.md`: Was there a breaking change?
3. If intentional deviation: Document in code comments with reason
4. If spec is wrong: Create issue in spec repo

### Problem: Context Window Full

**Error:** "AI cannot load more context"

**Solution:**

1. Close and restart AI session (clears context)
2. Load fewer spec sections (use line ranges)
3. Focus on one feature at a time
4. Use `grep_search` instead of loading full files

---

## Examples from Real Development

### Example 1: Backend API Development

**Context loaded:**

```
Implementation repo: backend-api/ (~25k tokens)
├─ src/auth/auth.controller.ts (3k)
├─ src/auth/auth.service.ts (4k)
├─ src/common/decorators/ (2k)
└─ test/auth/auth.e2e-spec.ts (3k)

Specs loaded on-demand: (~8k tokens)
├─ api-specification.md lines 150-280 (OTP endpoints) (3k)
├─ database-schema.md lines 85-140 (users table) (2k)
└─ backend-setup-guide.md lines 450-520 (JWT config) (3k)

Total: 33k tokens (67% remaining)
```

**Workflow:**

1. User: "Implement OTP verification endpoint"
2. AI searches spec for OTP endpoint details
3. AI loads specific lines from api-specification.md
4. AI generates controller + service + tests matching spec
5. AI validates against database schema
6. Implementation complete, spec sections unloaded

### Example 2: Mobile App Development

**Context loaded:**

```
Implementation repo: mobile-app/ (~30k tokens)
├─ lib/features/dashboard/ (12k)
├─ lib/core/widgets/ (8k)
└─ lib/core/utils/formatters.dart (2k)

Specs loaded on-demand: (~12k tokens)
├─ flutter-mobile-app-spec.md lines 250-450 (Dashboard) (5k)
├─ flutter-mobile-app-spec.md lines 900-1100 (UI/UX Guidelines) (4k)
└─ api-specification.md lines 500-600 (Metrics API) (3k)

Total: 42k tokens (58% remaining)
```

**Workflow:**

1. User: "Add new metric card to dashboard"
2. AI loads Dashboard section from Flutter spec
3. AI loads UI/UX Guidelines for styling
4. AI generates MetricCard widget following design system
5. AI adds Vietnamese formatting from spec examples
6. Implementation complete

---

## Related Documents

- [SPEC_CHANGELOG.md](../SPEC_CHANGELOG.md) - Track spec version changes
- [copilot-instructions.md](copilot-instructions.md) - Global coding guidelines
- `docs/SPEC_LINKS.md` (in each impl repo) - Curated spec references
- `knowledge-base/glossary.md` - Vietnamese/English term translations

---

## Questions?

If you encounter issues with this workflow:

1. Check if spec version matches between repo and references
2. Review SPEC_CHANGELOG.md for recent changes
3. Create issue in spec repo if specs are unclear or conflicting
4. Update this guide if you discover better workflows
