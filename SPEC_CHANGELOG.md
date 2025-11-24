# Specification Changelog

All notable changes to specifications in this repository will be documented here.

This changelog helps implementation repositories track which spec versions they're compatible with and understand what changed between versions.

## Format

Each version follows this structure:

- **Version Tag**: Git tag identifier (e.g., `v1.0-specs`)
- **Date**: Release date
- **Added**: New specifications or sections
- **Changed**: Updates to existing specifications
- **Deprecated**: Specs marked for future removal
- **Removed**: Deleted specifications
- **Fixed**: Corrections to existing specs
- **Implementation Repos Compatible**: Which impl repo versions work with this spec version

---

## [v1.0-specs] - 2025-11-25

### Added

**Architecture Specifications:**

- Complete Flutter mobile app specification (`architecture/flutter-mobile-app-spec.md`)
  - App architecture with Clean Architecture layers
  - Complete project structure with Vietnamese-specific implementations
  - State management using Riverpod
  - Core features: Authentication (OTP), Dashboard, AI Recommendations
  - UI/UX guidelines with Vietnamese design system and accessibility standards
  - Push notifications with Firebase Cloud Messaging
  - Offline strategy with Hive local storage
  - Performance optimization patterns
  - Build & deployment configuration

- Backend setup guide (`architecture/backend-setup-guide.md`)
  - Hybrid NestJS + Python architecture
  - Database setup with TypeORM migrations and RLS policies
  - Testing strategy with Jest/pytest examples
  - AWS EC2 single-server MVP deployment (~$20/month)
  - Docker Compose production configuration
  - Vietnamese seed data examples

- API specification (`architecture/api-specification.md`)
  - REST API endpoints with request/response schemas
  - GraphQL schema and queries
  - gRPC service definitions for AI service
  - Authentication flows (OTP, JWT)
  - Vietnamese response messages

- Database schema (`architecture/database-schema.md`)
  - Multi-tenant PostgreSQL schema
  - Row-Level Security (RLS) policies
  - Indexes and performance optimizations
  - Analytics extension with materialized views
  - QR session database schema
  - Vietnamese content fields

- GraphQL schema (`architecture/graphql-schema.md`)
  - Type definitions for admin dashboard
  - Queries and mutations
  - Subscriptions for real-time updates

- System architecture diagrams (`architecture/system-diagram.md`)
  - Hybrid microservices architecture
  - Multi-domain user management
  - Decision documentation for architecture choices

**Design Specifications:**

- UX flowcharts (`design/flowchart.md`)
- Wireframes and UX flows (`design/wireframes-ux-flow.md`)

**Planning Documents:**

- Feature roadmap (`planning/feature-roadmap.md`)
- Implementation roadmap (`planning/implementation-roadmap.md`)
- Sprint 1 implementation plan (`planning/sprint-1-implementation.md`)
- MVP acceptance criteria (`planning/mvp-acceptance-criteria.md`)
- Product strategy refined (`planning/product-strategy-refined.md`)
- Analytics & AI strategy (`planning/analytics-ai-strategy.md`)
- Monetization strategy (`planning/monetization-strategy.md`)
- Target market clarification (`planning/target-market-clarification.md`)

**Research:**

- Vietnam market strategy (`research/vietnam-market-strategy.md`)
- Competitive analysis (`research/competitive-analysis.md`)
- Japan market strategy (future) (`research/japan-market-strategy.md`)

**Documentation:**

- Implementation timeline (`documentation/implementation-timeline.md`)
- Sprint ticket breakdown (`documentation/sprint-ticket-breakdown.md`)
- Launch readiness checklist (`documentation/LAUNCH-READINESS.md`)
- Pilot checklist (`documentation/pilot-checklist.md`)
- Monitoring runbook (`documentation/monitoring-runbook.md`)
- Reconciliation runbook (`documentation/reconciliation-runbook.md`)
- Merchant FAQ (`documentation/merchant-faq.md`)
- PR templates for all implementation areas
- Implementation README templates for all repos
- CODEOWNERS examples (`documentation/codeowners-examples.md`)
- Contract change process (`documentation/contract-change-process.md`)

**Knowledge Base:**

- Glossary of Vietnamese/English terms (`knowledge-base/glossary.md`)
- Communication language guidelines (`knowledge-base/communication-language-guidelines.md`)

**GitHub Configuration:**

- Global Copilot instructions (`.github/copilot-instructions.md`)
- Vietnam market focus with localization guidelines
- Technical guardrails and coding standards
- Markdown documentation standards

### Implementation Repos Compatible

This initial spec release is intended for:

- `backend-api` v0.1.0+ (NestJS + Python backend)
- `mobile-app` v0.1.0+ (Flutter mobile app)
- `web-admin` v0.1.0+ (Admin dashboard)
- `ml-service` v0.1.0+ (Python AI/ML service)
- `infra` v0.1.0+ (Terraform infrastructure)

### Notes

- Focus: Vietnamese market (small shops, street food vendors)
- Tech stack: NestJS, Python/FastAPI, Flutter, PostgreSQL, Redis
- Deployment: AWS Free Tier single-server MVP
- Localization: Vietnamese-first approach
- Cost target: ~$20/month for MVP (<100 users)

### Breaking Changes

None (initial release)

---

## How to Use This Changelog

### For Implementation Repository Maintainers

1. **Check your current spec version**: Look at `docs/SPEC_LINKS.md` in your repo
2. **Review changelog entries**: Understand what changed between versions
3. **Update dependencies**: Ensure your implementation matches new specs
4. **Update SPEC_LINKS.md**: Reference new spec version tag
5. **Test compatibility**: Run integration tests against new specs

### For Documentation Contributors

When updating specs:

1. **Make changes**: Update relevant .md files
2. **Update this changelog**: Add entries under a new version section
3. **Create PR**: Get review from implementation teams
4. **Tag release**: After merge, create git tag (e.g., `v1.1-specs`)
5. **Notify teams**: Let implementation repos know about new version

### Versioning Scheme

- **Major version** (v2.0-specs): Breaking changes to APIs, database schema, or architecture
- **Minor version** (v1.1-specs): New features, non-breaking additions
- **Patch version** (v1.0.1-specs): Bug fixes, clarifications, typo corrections

---

## Upcoming Changes (Unreleased)

_No unreleased changes yet!_

<!-- Template for future releases:

## [vX.Y-specs] - YYYY-MM-DD

### Added
- New feature or spec file

### Changed
- Updated existing spec

### Deprecated
- Spec marked for removal

### Removed
- Deleted spec file

### Fixed
- Corrections to specs

### Implementation Repos Compatible
- repo-name vX.Y.Z+

### Breaking Changes
- Description of breaking changes

-->
