# GitHub Copilot Instructions – Infrastructure Repository

This extends the global instructions from the [specs repository](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** infra
- **Purpose:** Terraform configurations and Docker Compose for LocalStore Platform infrastructure
- **Tech Stack:** Terraform 1.6+, Docker Compose, AWS, GitHub Actions
- **Status:** 🔴 POSTPONED (using localhost Docker for Sprint 0.5)
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
3. Primary specs: `backend-setup-guide.md` and `system-diagram.md`

### Step 4: Implement

1. Follow the spec exactly
2. Apply code standards from this file (see below)
3. Focus on Docker Compose for local dev first

### Step 5: Update Progress

After implementation, **update `docs/CURRENT_WORK.md`**:

- Change story status from 🔴 to 🟡 (in progress) or ✅ (done)
- Add notes about what was implemented
- Add any blockers or follow-up items
- Update "Last Updated" timestamp

### Step 6: Git Workflow

1. Create/use feature branch: `feat/<story-description>`
2. Commit with conventional message
3. Run `terraform validate && terraform fmt -check`
4. Create PR when story is complete

### Step 7: Post Event to Slack

After completing a story, post an event to `#agent-events` (channel ID: `C0A1VSFQ9SS`):

```
🏗️ INFRA_UPDATED from infra: [Story Title]
---
Details: [What infrastructure was provisioned or changed]
Affected: [api, menu, dashboard, mobile]
Action: [Update env vars, redeploy services, etc.]
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
2. **Filter for events affecting this repo** (look for "infra" in Affected field)
3. **Report relevant events** to user with recommended actions
4. **If STORY_DONE from api/menu** → may need deployment pipeline updates
5. **If SPEC_CHANGED from specs** → review infrastructure requirements

---

## Key Spec Files

| Spec File | Sections | Purpose |
|-----------|----------|---------|
| `architecture/backend-setup-guide.md` | Lines 200-400 | Docker Compose setup |
| `architecture/backend-setup-guide.md` | Lines 1200-1600 | Production deployment |
| `architecture/backend-setup-guide.md` | Lines 2250-2700 | AWS infrastructure |
| `architecture/system-diagram.md` | Entire file | Infrastructure architecture |
| `documentation/monitoring-runbook.md` | Entire file | Monitoring setup |

Also check `docs/SPEC_LINKS.md` for curated links with line numbers.

---

## Build & Test Commands

```bash
# Docker Compose (development)
docker-compose up -d       # Start containers
docker-compose down        # Stop containers
docker-compose logs -f     # View logs

# Terraform
terraform init             # Initialize
terraform validate         # Validate configuration
terraform plan             # Preview changes
terraform apply            # Apply changes
terraform destroy          # Destroy resources
terraform fmt              # Format files
```

---

## Project Structure

```text
terraform/
├── environments/
│   ├── dev/              # Development environment
│   ├── staging/          # Staging environment
│   └── prod/             # Production environment
├── modules/
│   ├── vpc/              # Network configuration
│   ├── ec2/              # Compute instances
│   ├── rds/              # Database (future)
│   └── elasticache/      # Redis (future)
└── main.tf               # Root module

docker/
├── docker-compose.yml           # Development setup
├── docker-compose.prod.yml      # Production setup
└── nginx/
    └── nginx.conf               # Reverse proxy config

github/
└── workflows/
    ├── deploy-api.yml           # API deployment
    └── deploy-menu.yml          # Menu website deployment
```

---

## Code Standards

### Terraform Conventions

- Use modules for reusable components
- Environment-specific configs in `environments/`
- Variable naming: snake_case
- Always include `description` for variables

### Docker Compose

- Use named volumes for data persistence
- Health checks for all services
- Environment variables via `.env` files

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

## Current Approach

| Environment | Approach | Cost |
|-------------|----------|------|
| Development | Docker Compose on localhost | $0/month |
| Sprint 0.5 Demo | Vercel for menu (free tier) | $0/month |
| Production (future) | AWS EC2 t2.small | ~$17/month |

---

## Dependencies

- **Requires:** AWS account with IAM permissions
- **Requires:** CloudFlare account for DNS
- **All repos depend on this** for production deployment

---

## Related Documentation

- [Backend Setup Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/backend-setup-guide.md)
- [System Diagram](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/system-diagram.md)
- [Monitoring Runbook](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/monitoring-runbook.md)
