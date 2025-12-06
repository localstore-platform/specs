# Repository Templates

Templates for setting up implementation repositories with AI-assisted development.

**Created:** 2025-12-06  
**Spec Version:** v1.1-specs

---

## Overview

Each implementation repository needs **2 files** for AI-assisted development:

| File | Purpose | Location in Impl Repo |
|------|---------|----------------------|
| `copilot-instructions.md` | Copilot instructions + **"continue work" trigger** + repo context | `.github/copilot-instructions.md` |
| `CURRENT_WORK.md` | Local progress tracker (Copilot reads AND updates) | `docs/CURRENT_WORK.md` |

---

## How "Continue Work" Works

Just say **"continue work"** (or "tiếp tục", "next task") and Copilot will:

1. Read `docs/CURRENT_WORK.md` to see current progress
2. Identify the next 🔴 Not Started task (or continue 🟡 In Progress)
3. Fetch the relevant specification from specs repo
4. Implement the feature
5. **Update `docs/CURRENT_WORK.md`** with new status
6. Follow git workflow (branch, commit, PR)
7. **Post event to Slack** `#agent-events` channel

No copy-paste needed!

---

## 🔄 Cross-Repo Communication (Slack)

Agents communicate via the `#agent-events` Slack channel (ID: `C0A1VSFQ9SS`).

### Event Types

| Emoji | Event | Source Repos | Description |
|-------|-------|--------------|-------------|
| 📤 | `API_READY` | api | API endpoint published |
| 📦 | `SCHEMA_UPDATED` | contracts | Types/interfaces updated |
| 📝 | `SPEC_CHANGED` | specs | Specification updated |
| ✅ | `STORY_DONE` | all | Story completed |
| 🏗️ | `INFRA_UPDATED` | infra | Infrastructure changed |
| 🤖 | `ML_READY` | ml | ML model ready |

### "Sync Events" Trigger

Say **"sync events"** (or "check events", "đồng bộ") and Copilot will:

1. Read recent messages from `#agent-events`
2. Filter for events affecting this repo
3. Report relevant updates with recommended actions

### Example Flow

```
api completes Menu API
    ↓
Posts: "📤 API_READY from api: Menu endpoints"
    ↓
You open menu repo, say "sync events"
    ↓
menu agent sees API_READY, integrates the endpoints
```

### MCP Setup Required

The Slack MCP server must be configured in VS Code:

```json
{
  "servers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-token",
        "SLACK_TEAM_ID": "T09M8DKS3HV"
      }
    }
  }
}
```

---

## Templates Location

```text
.github/templates/
├── copilot-instructions-{repo}.md   # 8 files - copy to .github/copilot-instructions.md
├── CURRENT_WORK-{repo}.md           # 8 files - copy to docs/CURRENT_WORK.md
└── README.md                        # This file
```

---

## Repository Quick Reference

### Priority Repos (Sprint 0.5)

| Repo | Type | Sprint 0.5 Stories |
|------|------|-------------------|
| `menu` | Next.js Public Menu | 1.1, 1.2, 3.1, 4.1, 4.2 |
| `api` | NestJS Backend | 2.1, 2.2, 2.3 |
| `contracts` | TypeScript Types | 1.2, 3.2 |

### Future Repos (Sprint 1+)

| Repo | Type | Status |
|------|------|--------|
| `mobile` | Flutter App | 🟡 Planned for Sprint 1 |
| `dashboard` | Next.js Admin | 🟡 Planned for Sprint 1 |
| `infra` | Terraform + Docker | 🔴 Postponed (using localhost) |
| `web-admin` | Internal Admin | 🔴 Low priority |
| `ml` | Python AI/ML | ⏸️ Paused (awaiting data) |

---

## How to Use Templates

### Initial Setup (New Repo)

```bash
# Example for menu repo
cp specs/.github/templates/copilot-instructions-menu.md menu/.github/copilot-instructions.md
cp specs/.github/templates/CURRENT_WORK-menu.md menu/docs/CURRENT_WORK.md
```

### Continue Work (Existing Repo)

1. Open the repository in VS Code
2. Say **"continue work"** to Copilot Chat
3. Done! Copilot handles the rest.

---

## What's in Each Template

### copilot-instructions.md

Contains everything Copilot needs to work in that repo:

- **🚀 "Continue Work" Trigger** - Step-by-step instructions for resuming work
- **🔄 "Sync Events" Trigger** - Check Slack for cross-repo updates
- **Repository Context** - Purpose, tech stack, deployment
- **Key Spec Files** - Table of spec files with line numbers
- **Build & Test Commands** - All commands needed
- **Project Structure** - Folder layout
- **Code Standards** - Patterns and conventions
- **Git Workflow** - Branch naming, commit messages
- **Dependencies** - What this repo depends on / provides

### CURRENT_WORK.md

Local progress tracker for the repo:

- **Sprint Stories** - Table of assigned stories with status
- **Spec References** - Links to relevant specs with line numbers
- **Current Focus** - Next task description
- **Session Notes** - Log of what was done
- **Blockers** - Any issues preventing progress
- **Quick Commands** - Common commands for the repo

---

## Workflow Example

```bash
# 1. Open menu repo
cd ~/code/localstore-platform/menu && code .

# 2. Say "continue work" to Copilot Chat

# 3. Copilot automatically:
#    - Reads docs/CURRENT_WORK.md
#    - Sees Story 1.1 (Menu Display Page) is 🔴 Not Started
#    - Fetches wireframes-ux-flow.md from specs repo
#    - Implements the feature
#    - Updates CURRENT_WORK.md: Story 1.1 → 🟡 In Progress
#    - Creates branch: feat/menu-display-page
#    - Commits code
#    - Creates PR
#    - Updates CURRENT_WORK.md: Story 1.1 → ✅ Done
#    - Posts event to #agent-events Slack channel
```

### Cross-Repo Sync Example

```bash
# 1. api repo completed Menu API, posted to Slack

# 2. Open menu repo
cd ~/code/localstore-platform/menu && code .

# 3. Say "sync events" to Copilot Chat

# 4. Copilot:
#    - Reads #agent-events channel
#    - Sees "📤 API_READY from api: Menu endpoints"
#    - Reports: "API repo published Menu endpoints. Recommend integrating."
#    - You say "continue work" and it integrates the API
```

---

## Related Documentation

- [REPO_INITIALIZATION_PROMPT.md](../REPO_INITIALIZATION_PROMPT.md) - Initial repo setup
- [IMPLEMENTATION_PROGRESS.md](../../IMPLEMENTATION_PROGRESS.md) - Global progress tracker
- [planning/sprint-0.5-menu-demo.md](../../planning/sprint-0.5-menu-demo.md) - Current sprint
