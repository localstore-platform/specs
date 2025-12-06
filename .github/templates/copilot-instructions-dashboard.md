# GitHub Copilot Instructions – Dashboard Repository

This extends the global instructions from the [specs repository](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** dashboard
- **Purpose:** Next.js admin dashboard for restaurant owners (menu management, analytics, orders)
- **Tech Stack:** Next.js 14, React, TypeScript, Apollo Client, Tailwind CSS, WebSocket
- **Status:** 🟡 Planned for Sprint 1 (after Sprint 0.5 Menu Demo)
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
3. Primary specs: `graphql-schema.md` and `wireframes-ux-flow.md`

### Step 4: Implement

1. Follow the spec exactly
2. Apply code standards from this file (see below)
3. Mobile-first dashboard, Vietnamese locale, GraphQL integration

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
🖥️ STORY_DONE from dashboard: [Story Title]
---
Details: [What pages/features were implemented]
Affected: [api, specs]
Action: [Any API needs or spec updates required]
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
2. **Filter for events affecting this repo** (look for "dashboard" in Affected field)
3. **Report relevant events** to user with recommended actions
4. **If API_READY from api** → integrate new GraphQL endpoints
5. **If SCHEMA_UPDATED from contracts** → run `npm run codegen` to update types

---

## Key Spec Files

| Spec File | Sections | Purpose |
|-----------|----------|---------|
| `architecture/graphql-schema.md` | Entire file | GraphQL queries, mutations, subscriptions |
| `architecture/api-specification.md` | Lines 1800-2000 | WebSocket events |
| `architecture/multi-domain-user-management.md` | Entire file | Multi-location access control |
| `design/wireframes-ux-flow.md` | Lines 800-1500 | Admin dashboard views |

Also check `docs/SPEC_LINKS.md` for curated links with line numbers.

---

## Build & Test Commands

```bash
npm install          # Install dependencies
npm run dev          # Start development server (localhost:3000)
npm run test         # Run tests
npm run lint         # Lint code
npm run build        # Build for production
npm run start        # Start production server
npm run codegen      # Generate GraphQL types
```

---

## Project Structure

```text
app/
├── (auth)/
│   ├── login/            # Owner login
│   └── verify/           # OTP verification
├── (dashboard)/
│   ├── layout.tsx        # Dashboard layout with sidebar
│   ├── page.tsx          # Overview/metrics
│   ├── menu/             # Menu CRUD
│   ├── orders/           # Order management
│   ├── analytics/        # Charts and trends
│   └── settings/         # Location configuration
└── layout.tsx            # Root layout

lib/
├── apollo/               # Apollo Client setup
├── graphql/              # Queries, mutations, fragments
└── hooks/                # Custom hooks for data fetching

components/
├── dashboard/            # Dashboard-specific components
├── menu/                 # Menu management components
└── ui/                   # Shared UI components
```

---

## Code Standards

### Next.js Patterns

- Use App Router (`app/` directory)
- Server Components by default, Client Components for interactivity
- Route groups for layout: `(auth)`, `(dashboard)`
- Use middleware for authentication checks

### Apollo GraphQL

- Centralized client configuration in `lib/apollo/`
- Use generated types from `npm run codegen`
- Handle loading and error states consistently

### Vietnamese Localization

- Default locale: vi-VN
- Currency: VND with format 75.000₫
- All user-facing text in Vietnamese

---

## Git Workflow

**CRITICAL:** Follow `docs/GIT_WORKFLOW.md`:

- Never commit directly to main branch
- Branch naming: `feat/<story-description>` (e.g., `feat/menu-management`)
- Use conventional commit messages
- Create PR after commits

---

## Dependencies

- **Requires:** `api` repo for GraphQL endpoints
- **Requires:** `contracts` repo for shared TypeScript types
- **Deployment:** Independent (Vercel or similar)

---

## Related Documentation

- [GraphQL Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/graphql-schema.md)
- [Wireframes & UX Flow](https://github.com/localstore-platform/specs/blob/v1.1-specs/design/wireframes-ux-flow.md)
- [Multi-Domain User Management](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/multi-domain-user-management.md)
