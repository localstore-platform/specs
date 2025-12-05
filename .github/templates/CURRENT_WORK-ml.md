# Current Work – ML Repository

> **Last Updated:** 2025-01-XX  
> **Current Sprint:** N/A (Paused)  
> **Sprint Spec:** [planning/analytics-ai-strategy.md](https://github.com/localstore-platform/specs/blob/master/planning/analytics-ai-strategy.md)  
> **Note:** ⏸️ PAUSED - Awaiting pilot data from real customers

---

## Planned Stories (This Repo)

| Story | Description | Status | Notes |
|-------|-------------|--------|-------|
| ML1.1 | Project Setup | ⏸️ Paused | FastAPI + gRPC scaffold |
| ML1.2 | Demand Forecasting | ⏸️ Paused | Needs historical data |
| ML1.3 | Menu Optimization | ⏸️ Paused | Needs sales data |
| ML1.4 | Price Recommendations | ⏸️ Paused | Needs market data |

**Status Legend:** 🔴 Not Started | 🟡 In Progress | ✅ Done | ⏸️ Blocked/Paused

---

## Spec References

| Story | Specification | Lines |
|-------|--------------|-------|
| ML1.1 | [analytics-ai-strategy.md](https://github.com/localstore-platform/specs/blob/master/planning/analytics-ai-strategy.md) | L1-L100 |
| ML1.2 | [analytics-ai-strategy.md](https://github.com/localstore-platform/specs/blob/master/planning/analytics-ai-strategy.md) | L100-L200 |
| ML1.3 | [analytics-ai-strategy.md](https://github.com/localstore-platform/specs/blob/master/planning/analytics-ai-strategy.md) | L200-L300 |
| ML1.4 | [analytics-ai-strategy.md](https://github.com/localstore-platform/specs/blob/master/planning/analytics-ai-strategy.md) | L300-L400 |

---

## Current Focus

**Status:** ⏸️ PAUSED - Awaiting pilot data

ML features require real customer data for training. Resume after pilot launch.

---

## Session Notes

### Session: YYYY-MM-DD

- Started: [Story X.X]
- Completed: [What was done]
- Blockers: [Any issues]
- Next: [What to do next]

---

## Blockers

⏸️ **Awaiting Pilot Data** - Need 3+ months of transaction data for meaningful ML

---

## Quick Commands

```bash
# Python/Poetry
poetry install         # Install dependencies
poetry run pytest      # Run tests
poetry run black .     # Format code
poetry run flake8      # Lint code

# FastAPI
poetry run uvicorn main:app --reload  # Start dev server

# Docker
docker build -t ml-service .          # Build image
docker run -p 8000:8000 ml-service    # Run container
```
