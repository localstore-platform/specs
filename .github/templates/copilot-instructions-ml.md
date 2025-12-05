# GitHub Copilot Instructions – ML Repository

This extends the global instructions from the [specs repository](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** ml
- **Purpose:** Python AI/ML service for demand forecasting, price optimization, and menu recommendations
- **Tech Stack:** Python 3.11, FastAPI, gRPC, Celery, scikit-learn
- **Status:** ⏸️ PAUSED (awaiting pilot data from real customers)
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
3. If repo is PAUSED → inform user and suggest alternative

### Step 3: Load Specifications

1. Check the "Spec References" section in CURRENT_WORK.md for the spec link
2. Fetch and read the relevant specification section
3. Primary spec: `planning/analytics-ai-strategy.md`

### Step 4: Implement

1. Follow the spec exactly
2. Apply code standards from this file (see below)
3. Python/FastAPI patterns, ML best practices

### Step 5: Update Progress

After implementation, **update `docs/CURRENT_WORK.md`**:

- Change story status from 🔴 to 🟡 (in progress) or ✅ (done)
- Add notes about what was implemented
- Add any blockers or follow-up items
- Update "Last Updated" timestamp

### Step 6: Git Workflow

1. Create/use feature branch: `feat/<story-description>`
2. Commit with conventional message
3. Run `pytest && black --check . && flake8`
4. Create PR when story is complete

### Step 7: Report

Tell the user:

- What story you worked on
- What was implemented
- Current status
- What's next

---

## Key Spec Files

| Spec File | Sections | Purpose |
|-----------|----------|---------|
| `planning/analytics-ai-strategy.md` | Entire file | ML model specifications |
| `planning/analytics-ai-strategy.md` | Lines 200-400 | Demand forecasting |
| `planning/analytics-ai-strategy.md` | Lines 400-600 | Price optimization |
| `planning/analytics-ai-strategy.md` | Lines 600-800 | Menu recommendations |
| `architecture/api-specification.md` | Lines 1500-1600 | gRPC service definitions |

Also check `docs/SPEC_LINKS.md` for curated links with line numbers.

---

## Build & Test Commands

```bash
# Virtual environment
python3 -m venv venv
source venv/bin/activate

# Dependencies
pip install -r requirements.txt

# Tests and linting
pytest                    # Run tests
flake8 src/               # Lint code
black src/ --check        # Check formatting
black src/                # Format code

# Run servers
python -m src.grpc_server                  # gRPC server
uvicorn src.main:app --reload              # FastAPI REST interface

# Training (when data available)
python -m src.training.train_all
```

---

## Project Structure

```text
src/
├── grpc/
│   ├── server.py         # gRPC server
│   ├── protos/           # Protocol buffer definitions
│   └── services/         # Service implementations
├── models/
│   ├── demand/           # Demand forecasting
│   ├── pricing/          # Price optimization
│   └── recommendations/  # Menu recommendations
├── training/
│   ├── train_all.py      # Training orchestrator
│   └── data_pipeline.py  # Data preprocessing
├── api/
│   └── main.py           # FastAPI REST interface
└── utils/
    ├── config.py         # Configuration
    └── logging.py        # Structured logging

models/                   # Saved model files
data/                     # Training data (gitignored)
```

---

## Code Standards

### Python Conventions

- Python 3.11+ features allowed
- Type hints for all functions
- Docstrings for all public functions (Google style)
- Use `black` for formatting, `flake8` for linting

### ML Best Practices

- Reproducible training with seed setting
- Model versioning with metadata
- Feature engineering in pipelines
- Proper train/validation/test splits

---

## Git Workflow

**CRITICAL:** Follow `docs/GIT_WORKFLOW.md`:

- Never commit directly to main branch
- Branch naming: `feat/<story-description>` (e.g., `feat/demand-forecast`)
- Use conventional commit messages
- Create PR after commits

---

## Why Paused?

ML models require real customer data to train effectively:

| Feature | Data Required |
|---------|---------------|
| Demand Forecasting | 3+ months order history |
| Price Optimization | Price change experiments |
| Menu Recommendations | Customer ordering patterns |

Rule-based recommendations will be used initially (in API repo).

---

## Dependencies

- **Communicates with:** `api` repo via gRPC
- **Requires:** Training data from production database
- **Model artifacts:** Deployed alongside API

---

## Related Documentation

- [Analytics AI Strategy](https://github.com/localstore-platform/specs/blob/v1.1-specs/planning/analytics-ai-strategy.md)
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
- [Backend Setup Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/backend-setup-guide.md)
