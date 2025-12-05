# Repository Initialization Prompt

Use this prompt when setting up a new implementation repository for the LocalStore Platform.

---

## Prompt Template

Copy and customize this prompt based on the repository type you're creating:

```markdown
I need to initialize this new LocalStore Platform implementation repository to work with our specification-driven development workflow.

**Repository Information:**
- Repository: [REPO_NAME] (e.g., `api`, `menu`, `mobile`, `contracts`)
- Type: [REPO_TYPE] (e.g., NestJS Backend, Next.js Frontend, Flutter Mobile, TypeScript Contracts)
- Current state: Fresh repository with only README.md, AGPL-3.0 LICENSE, and .gitignore
- Spec repository: https://github.com/localstore-platform/specs
- Spec version: v1.1-specs

**Required Setup:**

Please create the following structure to enable AI context loading and spec-driven development:

1. **Documentation Structure** (`docs/` directory):
   - `SPEC_LINKS.md` - Curated links to relevant specifications from the specs repo
   - `.github/copilot-instructions.md` - Repository-specific Copilot instructions

2. **GitHub Configuration** (`.github/` directory):
   - `CODEOWNERS` - Define code ownership
   - PR template appropriate for this repository type
   - Issue templates (bug report, feature request)

3. **Development Files**:
   - `.env.example` - Environment variable template (if applicable)
   - Development setup documentation

4. **Update README.md**:
   - Add proper description linking back to specs
   - Add tech stack information
   - Add quick start guide
   - Add link to specs repository

**Specification References:**

Based on the repository type, `docs/SPEC_LINKS.md` should reference these specifications:

[Select appropriate specs based on repository type - see details below]

**Additional Context:**

- Primary market: Vietnamese small businesses (restaurants, street food vendors)
- Language: Vietnamese-first approach (vi-VN locale, VND currency)
- Architecture: Multi-tenant SaaS platform
- Cost target: ~$20/month MVP deployment

Please create all necessary files following the patterns established in the specs repository, ensuring:
- All markdown files pass markdown linting (MD022, MD032, MD034, MD009, MD036)
- Vietnamese localization is properly configured
- Links to specs use the v1.1-specs tag
- Documentation is concise and actionable
```

---

## Repository-Specific Details

### For `api` Repository (NestJS Backend)

**Type:** `NestJS Backend + Python AI Service`

**Tech Stack:**

- NestJS 10 + TypeScript 5
- PostgreSQL 14 + Redis 7
- Python 3.11 + FastAPI (AI service)
- Bull Queue, Winston Logger
- TypeORM, Apollo GraphQL, Socket.io

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
  - REST endpoints (lines 80-1200)
  - GraphQL schema (lines 1200-1800)
  - gRPC service definitions (lines 1800-2000)
  
- [Backend Setup Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/backend-setup-guide.md)
  - Development environment (lines 1-200)
  - Docker Compose setup (lines 200-400)
  - Database migrations (lines 400-600)
  - Testing strategy (lines 800-1000)
  - Production deployment (lines 1200-1600)

- [Database Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/database-schema.md)
  - Core tables (lines 80-500)
  - RLS policies (lines 500-650)
  - Indexes (lines 650-750)
  - Analytics extension (see database-schema-analytics-extension.md)

- [GraphQL Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/graphql-schema.md)
  - Type definitions (lines 1-400)
  - Queries (lines 400-600)
  - Mutations (lines 600-800)
  - Subscriptions (lines 800-900)

### Implementation
- [Implementation Roadmap](https://github.com/localstore-platform/specs/blob/v1.1-specs/planning/implementation-roadmap.md)
- [Sprint 1 Plan](https://github.com/localstore-platform/specs/blob/v1.1-specs/planning/sprint-1-implementation.md)
- [MVP Acceptance Criteria](https://github.com/localstore-platform/specs/blob/v1.1-specs/planning/mvp-acceptance-criteria.md)
```

**Additional Files:**

- `docker-compose.yml` - PostgreSQL + Redis + API services
- `docker-compose.dev.yml` - Development overrides
- `.env.example` - Database URLs, JWT secrets, Redis config
- `migrations/` directory structure

---

### For `menu` Repository (Next.js Public Menu)

**Type:** `Next.js 14 Static Site Generation`

**Tech Stack:**

- Next.js 14 (App Router)
- React 18 + TypeScript
- Tailwind CSS 3
- Vercel deployment

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
  - Menu public endpoints (lines 400-600)
  - QR code session endpoints (lines 900-1000)

- [Database Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/database-schema.md)
  - Menu items schema (lines 250-350)
  - Categories schema (lines 200-250)

### Design
- [Wireframes & UX Flow](https://github.com/localstore-platform/specs/blob/v1.1-specs/design/wireframes-ux-flow.md)
  - Customer menu views (lines 200-400)
  - Mobile-first design (lines 100-200)

- [Flowchart](https://github.com/localstore-platform/specs/blob/v1.1-specs/design/flowchart.md)
  - Customer ordering flow (lines 100-300)

### Market Context
- [Vietnam Market Strategy](https://github.com/localstore-platform/specs/blob/v1.1-specs/research/vietnam-market-strategy.md)
  - Mobile optimization requirements
  - 4G performance targets (<2s TTI)
  - Currency formatting (75.000₫)
```

**Additional Files:**

- `next.config.js` - Static export configuration
- `.env.example` - API base URL
- `tailwind.config.js` - Vietnamese design tokens
- `public/` directory structure

---

### For `mobile` Repository (Flutter App)

**Type:** `Flutter 3.x Mobile App (iOS + Android)`

**Tech Stack:**

- Flutter 3.x + Dart 3.x
- Riverpod 2.4 (state management)
- Dio + Retrofit (networking)
- Hive (local storage)
- Firebase Cloud Messaging

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [Flutter Mobile App Spec](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/flutter-mobile-app-spec.md)
  - Complete app architecture (entire file - ~2500 lines)
  - Project structure (lines 100-300)
  - Feature specifications (lines 300-1800)
  - UI/UX guidelines (lines 1800-2200)
  - Performance optimization (lines 2200-2400)

- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
  - REST endpoints for mobile (lines 80-1200)
  - Authentication (lines 120-280)
  - Dashboard metrics (lines 500-750)

- [GraphQL Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/graphql-schema.md)
  - Queries for dashboard (lines 400-600)
  - Real-time subscriptions (lines 800-900)

### Design
- [Wireframes & UX Flow](https://github.com/localstore-platform/specs/blob/v1.1-specs/design/wireframes-ux-flow.md)
  - Mobile owner dashboard (lines 400-800)
  - Onboarding flow (lines 50-150)

### Implementation
- [Sprint 1 Plan](https://github.com/localstore-platform/specs/blob/v1.1-specs/planning/sprint-1-implementation.md)
  - Mobile app tasks (Week 4-6)
```

**Additional Files:**

- `pubspec.yaml` - Dependencies
- `.env.example` - API URLs, Firebase config
- `analysis_options.yaml` - Dart linting rules
- `lib/` directory structure (following Clean Architecture)

---

### For `dashboard` Repository (Next.js Admin Portal)

**Type:** `Next.js 14 Admin Dashboard`

**Tech Stack:**

- Next.js 14 + TypeScript
- Apollo GraphQL Client
- Tailwind CSS 3
- Socket.io client (WebSocket)

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [GraphQL Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/graphql-schema.md)
  - Complete schema (entire file)
  - Real-time subscriptions (lines 800-900)

- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
  - REST fallback endpoints (lines 80-1200)
  - WebSocket events (lines 1800-2000)

- [Multi-Domain User Management](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/multi-domain-user-management.md)
  - Multi-location access control

### Design
- [Wireframes & UX Flow](https://github.com/localstore-platform/specs/blob/v1.1-specs/design/wireframes-ux-flow.md)
  - Admin dashboard views (lines 800-1200)
  - Menu management UI (lines 1200-1500)
```

---

### For `ml` Repository (Python AI Service)

**Type:** `Python 3.11 FastAPI + gRPC`

**Tech Stack:**

- Python 3.11
- FastAPI 0.104
- gRPC + Protobuf
- Celery 5.x
- scikit-learn, pandas

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
  - gRPC service definitions (lines 1800-2000)
  - AI recommendation endpoints (lines 800-950)

- [Backend Setup Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/backend-setup-guide.md)
  - Python AI service setup (lines 600-800)

### Planning
- [Analytics & AI Strategy](https://github.com/localstore-platform/specs/blob/v1.1-specs/planning/analytics-ai-strategy.md)
  - ML model specifications (entire file)
  - Demand forecasting (lines 200-400)
  - Price optimization (lines 400-600)
  - Menu recommendations (lines 600-800)
```

---

### For `contracts` Repository (Shared Types)

**Type:** `TypeScript Shared Library`

**Tech Stack:**

- TypeScript 5
- GraphQL Code Generator
- Protobuf compiler

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [API Specification](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/api-specification.md)
  - REST DTOs (lines 80-1200)

- [GraphQL Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/graphql-schema.md)
  - Type definitions (lines 1-400)

- [Database Schema](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/database-schema.md)
  - Enums and constants (lines 750-850)

### Documentation
- [Contract Change Process](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/contract-change-process.md)
```

---

### For `infra` Repository (Infrastructure)

**Type:** `Terraform + Docker`

**Tech Stack:**

- Terraform 1.6+
- Docker Compose
- GitHub Actions
- AWS (EC2, RDS, ElastiCache)

**Required Specs in SPEC_LINKS.md:**

```markdown
## Core Specifications

### Architecture
- [Backend Setup Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/backend-setup-guide.md)
  - Production deployment (lines 1200-1600)
  - Docker Compose setup (lines 200-400)

- [System Diagram](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/system-diagram.md)
  - Infrastructure architecture (entire file)

- [Decision: Hybrid Architecture](https://github.com/localstore-platform/specs/blob/v1.1-specs/architecture/decision-hybrid-architecture.md)

### Implementation
- [Implementation Timeline](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/implementation-timeline.md)
  - Phase 5: Production deployment (lines 300-400)
```

---

## SPEC_LINKS.md Template

Here's a base template that should be customized for each repository:

```markdown
# Specification Links

This document maps features in this repository to their source specifications in the [specs repository](https://github.com/localstore-platform/specs).

**Spec Version:** [v1.1-specs](https://github.com/localstore-platform/specs/tree/v1.1-specs)

**Last Updated:** [DATE]

---

## How to Use This Document

1. When implementing a feature, find it in the table below
2. Click the spec link to view the authoritative specification
3. Use line number references to load only relevant sections
4. Follow the spec exactly for API contracts, schemas, and behaviors

---

## Core Specifications

[Repository-specific specification links - see examples above]

---

## Cross-Cutting Concerns

### Localization
- [Communication Language Guidelines](https://github.com/localstore-platform/specs/blob/v1.1-specs/knowledge-base/communication-language-guidelines.md)
- [Glossary](https://github.com/localstore-platform/specs/blob/v1.1-specs/knowledge-base/glossary.md)
  - Vietnamese/English term translations
  - Standard terminology

### Market Context
- [Vietnam Market Strategy](https://github.com/localstore-platform/specs/blob/v1.1-specs/research/vietnam-market-strategy.md)
  - Mobile-first requirements
  - Payment integrations (MoMo, ZaloPay, VNPay)
  - Social platform integrations (Zalo, Facebook)

### Monitoring & Operations
- [Monitoring Runbook](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/monitoring-runbook.md)
- [Launch Readiness Checklist](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/LAUNCH-READINESS.md)

---

## Spec Update Protocol

When the specs repository releases a new version:

1. Review [SPEC_CHANGELOG.md](https://github.com/localstore-platform/specs/blob/master/SPEC_CHANGELOG.md)
2. Identify breaking changes affecting this repository
3. Update this file to reference new spec version
4. Implement required changes
5. Update repository version and tag release

---

## AI Context Loading

AI assistants should:
- Load specs **on-demand** using the links above
- Use line number ranges to minimize context
- Reference this spec version consistently
- Follow [AI Context Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/AI_CONTEXT_GUIDE.md)

**Example:**

```

semantic_search(
  repo: "<https://github.com/localstore-platform/specs>",
  query: "OTP authentication endpoint"
)

```

Then read specific sections:

```

read_file(
  "architecture/api-specification.md",
  lines: 150-280
)

```
```

---

## Copilot Instructions Template

Create `.github/copilot-instructions.md` with repository-specific rules:

```markdown
# GitHub Copilot Instructions – [REPO_NAME]

Extends the global instructions from [specs/.github/copilot-instructions.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/copilot-instructions.md).

## Repository Context

- **Repository:** [REPO_NAME]
- **Type:** [REPO_TYPE]
- **Tech Stack:** [LIST_STACK]
- **Spec Version:** v1.1-specs
- **Spec Links:** See [docs/SPEC_LINKS.md](../docs/SPEC_LINKS.md)

## Specific Guidelines

### Code Style
[Repository-specific code style rules]

### Architecture Patterns
[Common patterns used in this repository]

### Testing Requirements
[Testing conventions and coverage requirements]

### Vietnamese Localization
[Repository-specific localization requirements]

## Common Tasks

### Adding a New Feature
1. Check SPEC_LINKS.md for relevant specifications
2. Load spec sections using semantic_search
3. Implement following spec exactly
4. Add tests matching spec examples
5. Update documentation

### Updating API Contracts
1. Ensure contracts repository is updated first
2. Update this repository to use new contract version
3. Run contract tests
4. Update API documentation

## Related Documentation

- [Main Specs Repository](https://github.com/localstore-platform/specs)
- [AI Context Guide](https://github.com/localstore-platform/specs/blob/v1.1-specs/.github/AI_CONTEXT_GUIDE.md)
- [Spec Changelog](https://github.com/localstore-platform/specs/blob/v1.1-specs/SPEC_CHANGELOG.md)
```

---

## CODEOWNERS Template

Create `.github/CODEOWNERS`:

```
# CODEOWNERS for [REPO_NAME]
# See: https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/codeowners-examples.md

# Global owners
* @[GITHUB_USERNAME]

# Documentation
/docs/ @[DOCS_OWNER]
README.md @[DOCS_OWNER]

# [Repository-specific ownership rules]
```

---

## PR Template

Copy appropriate template from specs repository:

- API: [pr-template-api.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/pr-template-api.md)
- Mobile: [pr-template-mobile.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/pr-template-mobile.md)
- Web: [pr-template-web.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/pr-template-web.md)
- Contracts: [pr-template-contracts.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/pr-template-contracts.md)
- Infrastructure: [pr-template-infra.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/pr-template-infra.md)
- ML: [pr-template-ml.md](https://github.com/localstore-platform/specs/blob/v1.1-specs/documentation/pr-template-ml.md)

---

## Complete Initialization Example

Here's a complete example for initializing the `api` repository:

### Step 1: Prepare Prompt

```markdown
I need to initialize the LocalStore Platform API repository for spec-driven development.

**Repository:** api
**Type:** NestJS Backend + Python AI Service
**Spec Version:** v1.1-specs
**Spec Repo:** https://github.com/localstore-platform/specs

Please create:
1. docs/SPEC_LINKS.md with API-relevant specification links
2. .github/copilot-instructions.md for NestJS/TypeScript conventions
3. .github/CODEOWNERS
4. .github/pull_request_template.md (using pr-template-api.md)
5. .env.example with DATABASE_URL, REDIS_URL, JWT_SECRET
6. Update README.md with proper description and links

Use the patterns from specs/.github/REPO_INITIALIZATION_PROMPT.md for the API repository type.
```

### Step 2: AI Creates Structure

The AI will create:

```
api/
├── .github/
│   ├── copilot-instructions.md
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── docs/
│   └── SPEC_LINKS.md
├── .env.example
├── .gitignore
├── LICENSE (AGPL-3.0)
└── README.md (updated)
```

### Step 3: Verify Setup

Check that:

- ✅ SPEC_LINKS.md references v1.1-specs with line numbers
- ✅ All markdown files pass linting
- ✅ README links back to specs repository
- ✅ .env.example has all required variables
- ✅ Copilot instructions extend global guidelines

---

## Troubleshooting

### Issue: AI doesn't know which specs to include

**Solution:** Explicitly specify the repository type in your prompt using the templates above.

### Issue: SPEC_LINKS.md has broken links

**Solution:** Verify spec version tag exists: `https://github.com/localstore-platform/specs/tree/v1.1-specs`

### Issue: Missing Vietnamese localization

**Solution:** Include this in your prompt:

```
Ensure Vietnamese localization is configured:
- vi-VN locale in configs
- VND currency formatting (75.000₫)
- Vietnamese error messages
- Link to knowledge-base/glossary.md for translations
```

---

## Next Steps After Initialization

1. **Review Generated Files:** Ensure all documentation is accurate
2. **Setup Local Development:** Follow README quick start
3. **Run First Implementation:** Pick a feature from SPEC_LINKS.md
4. **Test AI Context Loading:** Verify AI can load specs on-demand
5. **Iterate:** Update SPEC_LINKS.md as you discover useful spec sections

---

**Created:** 2025-11-25
**Spec Version:** v1.1-specs
**Last Updated:** This document
