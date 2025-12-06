# GitHub Copilot Instructions – Web-Admin Repository

This extends the global instructions from the [specs repository](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** web-admin
- **Purpose:** Internal admin tool for LocalStore Platform operators (tenant management, monitoring)
- **Tech Stack:** Next.js 14, React, TypeScript, Tailwind CSS
- **Status:** 🔴 LOW PRIORITY (internal tool, not customer-facing)
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
3. Note: Internal admin specs are TBD

### Step 4: Implement

1. Follow the spec exactly
2. Apply code standards from this file (see below)
3. Internal admin patterns (not customer-facing standards)

### Step 5: Update Progress

After implementation, **update `docs/CURRENT_WORK.md`**:

- Change story status from 🔴 to 🟡 (in progress) or ✅ (done)
- Add notes about what was implemented
- Add any blockers or follow-up items
- Update "Last Updated" timestamp

### Step 6: Git Workflow

1. Create/use feature branch: `feat/<story-description>`
2. Commit with conventional message
3. Run `npm run lint && npm run test`
4. Create PR when story is complete

### Step 7: Post Event to Slack

After completing a story, post an event to `#agent-events` (channel ID: `C0A1VSFQ9SS`):

```
🔧 STORY_DONE from web-admin: [Story Title]
---
Details: [What admin features were implemented]
Affected: [specs]
Action: [Any documentation updates needed]
Spec: [Link to relevant spec section]
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
2. **Filter for events affecting this repo** (look for "web-admin" in Affected field)
3. **Report relevant events** to user with recommended actions
4. **If INFRA_UPDATED from infra** → may need deployment updates
5. **If SPEC_CHANGED from specs** → review admin requirements

---

## Key Spec Files

| Spec File | Sections | Purpose |
|-----------|----------|---------|
| Internal specifications | TBD | Admin tool specs pending |
| `architecture/database-schema.md` | Tenant tables | Multi-tenant management |
| `documentation/monitoring-runbook.md` | Entire file | System health monitoring |

Also check `docs/SPEC_LINKS.md` for curated links with line numbers.

---

## Build & Test Commands

```bash
npm install          # Install dependencies
npm run dev          # Start development server (localhost:3001)
npm run test         # Run tests
npm run lint         # Lint code
npm run build        # Build for production
npm run start        # Start production server
```

---

## Project Structure

```text
app/
├── (auth)/
│   └── login/            # Platform operator login
├── (admin)/
│   ├── layout.tsx        # Admin layout
│   ├── tenants/          # Tenant management
│   ├── monitoring/       # System health dashboards
│   ├── support/          # Support ticket tools
│   └── settings/         # Platform configuration
└── layout.tsx            # Root layout

lib/
├── db/                   # Direct database access
└── auth/                 # Admin authentication
```

---

## Code Standards

### Next.js Patterns

- Use App Router (`app/` directory)
- Server Components by default
- Route groups: `(auth)`, `(admin)`
- Use middleware for platform operator authentication

### Access Control

- Internal network only, not publicly accessible
- Separate auth from customer authentication
- Direct database queries for admin operations

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

## Dependencies

- **Requires:** `api` repo for some endpoints (or direct DB access)
- **Deployment:** Internal network only
- **No dependency on:** `contracts` repo (internal tool)

---

## Related Documentation

- [Database Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/database-schema.md)
- [Monitoring Runbook](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/monitoring-runbook.md)
