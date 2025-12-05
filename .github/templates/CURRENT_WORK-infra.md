# Current Work – Infrastructure Repository

> **Last Updated:** 2025-01-XX  
> **Current Sprint:** Sprint 0.5 (Menu Demo)  
> **Sprint Spec:** [planning/sprint-0.5-menu-demo.md](https://github.com/localstore-platform/specs/blob/master/planning/sprint-0.5-menu-demo.md)  
> **Note:** AWS deployment POSTPONED - using localhost Docker for Sprint 0.5

---

## Sprint 0.5 Stories (This Repo)

| Story | Description | Status | Notes |
|-------|-------------|--------|-------|
| I0.1 | Docker Compose (Local Dev) | 🔴 Not Started | PostgreSQL, Redis containers |
| I0.2 | Environment Variables | 🔴 Not Started | .env templates |

**Status Legend:** 🔴 Not Started | 🟡 In Progress | ✅ Done | ⏸️ Blocked

---

## Future Stories (Post-Demo)

| Story | Description | Status | Notes |
|-------|-------------|--------|-------|
| I1.1 | AWS VPC Setup | ⏸️ Postponed | After demo |
| I1.2 | RDS PostgreSQL | ⏸️ Postponed | After demo |
| I1.3 | ElastiCache Redis | ⏸️ Postponed | After demo |
| I1.4 | ECS/Fargate Setup | ⏸️ Postponed | After demo |
| I1.5 | CI/CD Pipeline | ⏸️ Postponed | After demo |

---

## Spec References

| Story | Specification | Lines |
|-------|--------------|-------|
| I0.1 | [backend-setup-guide.md](https://github.com/localstore-platform/specs/blob/master/architecture/backend-setup-guide.md) | L200-L280 |
| I0.2 | [backend-setup-guide.md](https://github.com/localstore-platform/specs/blob/master/architecture/backend-setup-guide.md) | L50-L100 |
| I1.x | [system-diagram.md](https://github.com/localstore-platform/specs/blob/master/architecture/system-diagram.md) | L1-L200 |

---

## Current Focus

**Next Task:** Story I0.1 - Docker Compose (Local Dev)

**Requirements:**

- Create `docker-compose.yml` for local development
- PostgreSQL 14 container with volume
- Redis 7 container
- Network for service communication
- Health checks

---

## Session Notes

### Session: YYYY-MM-DD

- Started: [Story X.X]
- Completed: [What was done]
- Blockers: [Any issues]
- Next: [What to do next]

---

## Blockers

None for local Docker setup.

---

## Quick Commands

```bash
# Docker Compose
docker-compose up -d      # Start containers
docker-compose down       # Stop containers
docker-compose logs -f    # View logs

# Terraform (future)
terraform init            # Initialize
terraform plan            # Preview changes
terraform apply           # Apply changes
terraform destroy         # Destroy resources
```
