# GitHub Copilot Instructions – Contracts Repository

This extends the global instructions from the [specs repository](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** contracts
- **Purpose:** Shared TypeScript types, interfaces, and utilities. Published as `@localstore/contracts` npm package
- **Tech Stack:** TypeScript 5, ESM + CommonJS, Jest
- **Spec Version:** v1.1-specs

---

## 🚀 "Continue Work" Trigger

When the user says **"continue work"**, **"tiếp tục"**, or **"next task"**:

### Step 1: Read Local Progress

Read `docs/CURRENT_WORK.md` in this repository to understand:

- Current sprint and focus
- Which stories are assigned to this repo
- Current status of each story (🔴 Not Started / 🟡 In Progress / ✅ Done)
- Any blockers or notes from previous session

### Step 2: Identify Next Task

From CURRENT_WORK.md:

1. If any story is 🟡 In Progress → continue that story
2. Otherwise, pick the first 🔴 Not Started story
3. If all stories are ✅ Done → report completion and suggest next sprint

### Step 3: Load Specifications

1. Check the "Spec References" section in CURRENT_WORK.md for the spec link
2. Fetch and read the relevant specification section
3. Also check `docs/SPEC_LINKS.md` for additional context if needed

### Step 4: Implement

1. Follow the spec exactly
2. Types MUST match database-schema.md and api-specification.md exactly
3. Apply code standards from this file (see below)

### Step 5: Update Progress

After implementation, **update `docs/CURRENT_WORK.md`**:

- Change story status from 🔴 to 🟡 (in progress) or ✅ (done)
- Add notes about what was implemented
- Add any blockers or follow-up items
- Update "Last Updated" timestamp

### Step 6: Git Workflow

1. Create/use feature branch: `feat/<story-description>`
2. Commit with conventional message
3. Run `npm run lint && npm run test && npm run typecheck`
4. Create PR when story is complete

### Step 7: Post Event to Slack

After completing a story, post an event to `#agent-events` (channel ID: `C0A1VSFQ9SS`):

```
📦 SCHEMA_UPDATED from contracts: [Story Title]
---
Details: [What types/interfaces were added or changed]
Affected: [api, menu, mobile, dashboard]
Action: [Update @localstore/contracts dependency, regenerate types]
Version: [New package version if published]
```

Use `slack_post_message` with channel_id `C0A1VSFQ9SS`.

### Step 8: Report

Tell the user:

- What story you worked on
- What was implemented
- Current status
- What's next
- Event posted to Slack

---

## 🔄 "Sync Events" Trigger

When the user says **"sync events"**, **"check events"**, or **"đồng bộ"**:

1. **Read recent messages** from `#agent-events` (channel ID: `C0A1VSFQ9SS`) using `slack_get_channel_history`
2. **Filter for events affecting this repo** (look for "contracts" in Affected field)
3. **Report relevant events** to user with recommended actions
4. **If SPEC_CHANGED from specs** → review and update types accordingly
5. **If API_READY from api** → ensure response types are defined

---

## Key Spec Files

| Spec File | Sections | Purpose |
|-----------|----------|---------|
| `architecture/api-specification.md` | Lines 80-1200 | REST DTOs |
| `architecture/graphql-schema.md` | Lines 1-400 | GraphQL types |
| `architecture/database-schema.md` | Lines 80-500 | Entity interfaces |
| `architecture/database-schema.md` | Lines 750-850 | Enums and constants |
| `documentation/contract-change-process.md` | Entire file | Versioning workflow |

Also check `docs/SPEC_LINKS.md` for curated links with line numbers.

---

## Build & Test Commands

```bash
npm install          # Install dependencies
npm run test         # Run tests
npm run lint         # Lint code
npm run build        # Build package
npm run typecheck    # Type check
npm publish          # Publish to npm (when ready)
```

---

## Project Structure

```text
src/
├── entities/
│   ├── menu.ts           # MenuItem, Category interfaces
│   ├── location.ts       # Location, Tenant interfaces
│   ├── user.ts           # User, Role interfaces
│   └── order.ts          # Order, OrderItem interfaces
├── dto/
│   ├── menu.dto.ts       # API request/response types
│   ├── auth.dto.ts       # Login, OTP types
│   └── analytics.dto.ts  # Metrics, recommendations
├── enums/
│   ├── status.ts         # OrderStatus, ItemStatus
│   └── priority.ts       # RecommendationPriority
├── utils/
│   ├── currency.ts       # formatVND()
│   └── date.ts           # Vietnamese date formatting
└── index.ts              # Main exports
```

---

## Code Standards

### TypeScript Conventions

- **Strict mode:** No `any` types in production code
- **Interfaces over types:** Use `interface` for object shapes
- **Readonly:** Mark immutable properties as `readonly`
- **Optional:** Use `?` for optional properties, document why

### Naming Conventions

- Interfaces: PascalCase, no `I` prefix (`MenuItem`, not `IMenuItem`)
- Enums: PascalCase with UPPER_CASE values
- Utilities: camelCase functions (`formatVND`)
- Files: kebab-case (`menu-item.ts`)

### Documentation

- JSDoc comments for all exported types
- Include usage examples in comments
- Reference spec file and line numbers

### Utility Functions

- Pure functions, no side effects
- 100% test coverage for utilities
- Handle edge cases (undefined, null, negative numbers)

---

## Git Workflow

**CRITICAL:** Follow `docs/GIT_WORKFLOW.md`:

### Never Commit Directly to Main

- **Always** create a new branch for changes
- If currently on `main`, checkout to a new branch before committing
- Branch naming: `<type>/<short-description>`

### Branch Naming Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `chore` | Maintenance, dependencies, tooling |
| `docs` | Documentation only |
| `refactor` | Code refactoring |
| `test` | Adding or updating tests |

### Logical Commits

- Group related changes into logical commits
- Each commit should represent a single logical change
- Use conventional commit format:

```text
<type>: <short description>

- Detail 1
- Detail 2
```

### Commit Granularity Principle

Each commit should answer ONE question: "What single feature/fix does this add?"

**Rule of thumb:** If you need "and" to describe the commit, split it.

- ❌ `Add docs and config files` → Split
- ✅ `Add GitHub PR template and CODEOWNERS` → OK (same purpose: GitHub config)

### Pull Request Workflow

After committing:

1. Push the branch to origin
2. Create a PR to `main` branch
3. If PR already exists, update the title and description
4. **Do not wait for confirmation** - push and create PR automatically

---

## Contract Change Protocol

1. Create PR with type changes
2. Update version in package.json (semver)
3. Document breaking changes in CHANGELOG.md
4. After merge, publish to npm
5. Update consuming repos to new version

---

## Dependencies

- **No external LocalStore dependencies** (this is the base package)
- **Consumed by:** `api`, `menu`, `dashboard`, `mobile` repos
- Changes require version bump and consumer updates

---

## Related Documentation

- [Database Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/database-schema.md)
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
- [Contract Change Process](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/contract-change-process.md)
