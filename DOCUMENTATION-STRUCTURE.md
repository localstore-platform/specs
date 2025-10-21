# Documentation Structure

**Last Updated:** October 22, 2025  
**Purpose:** Guide to navigating this documentation repository

---

## Core Strategy Documents (Start Here)

### 1. [README.md](./README.md)
**Purpose:** High-level project overview and value proposition  
**Audience:** New team members, stakeholders, executives  
**Content:** What we build, who we serve, product priorities

### 2. [planning/product-strategy-refined.md](./planning/product-strategy-refined.md)
**Purpose:** Single source of truth for product strategy  
**Audience:** Product team, engineering leads, investors  
**Content:** 
- Detailed target market analysis (both segments)
- Competitive advantages and data moat
- Product roadmap (Phases 1-3)
- Business model and pricing rationale
- Go-to-market strategy
- Success metrics

### 3. [planning/target-market-clarification.md](./planning/target-market-clarification.md)
**Purpose:** Quick reference for dual-segment strategy  
**Audience:** Marketing, sales, product managers  
**Content:** 
- Why both segments are equally important
- Key product and marketing implications
- Concise summary (detailed info in product-strategy-refined.md)

---

## Planning Documents

### Feature Development
- **[planning/feature-roadmap.md](./planning/feature-roadmap.md)** - Phased feature development (4 phases, Vietnam-only for Phases 1-3)
- **[planning/mvp-acceptance-criteria.md](./planning/mvp-acceptance-criteria.md)** - Detailed acceptance criteria for MVP features

### Business Strategy
- **[planning/monetization-strategy.md](./planning/monetization-strategy.md)** - Pricing tiers, revenue projections, ROI justification
- **[planning/analytics-ai-strategy.md](./planning/analytics-ai-strategy.md)** - Technical implementation of analytics and AI features

### Execution
- **[planning/implementation-roadmap.md](./planning/implementation-roadmap.md)** - 12-week sprint plan with task breakdown

---

## Architecture Documents

### Database Design
- **[architecture/database-schema.md](./architecture/database-schema.md)** - Core tables (19 tables + locations table)
- **[architecture/database-schema-analytics-extension.md](./architecture/database-schema-analytics-extension.md)** - Analytics tables for AI training (4 tables)

### System Design
- **[architecture/system-diagram.md](./architecture/system-diagram.md)** - Overall architecture and tech stack

---

## Market Research

### Vietnam Market (Primary Focus)
- **[research/vietnam-market-strategy.md](./research/vietnam-market-strategy.md)** - Go-to-market tactics for Vietnam
- **[research/competitive-analysis.md](./research/competitive-analysis.md)** - Competitor analysis

### Future Markets
- **[research/japan-market-strategy.md](./research/japan-market-strategy.md)** - Japan expansion strategy (deferred to Phase 4, Year 2+)

---

## Design & UX

- **[design/wireframes-ux-flow.md](./design/wireframes-ux-flow.md)** - Wireframes and user flows

---

## Knowledge Base

### Developer Guidelines
- **[.github/copilot-instructions.md](./.github/copilot-instructions.md)** - AI coding assistant instructions (Vietnam market focus)

### Team Resources
- **[knowledge-base/glossary.md](./knowledge-base/glossary.md)** - Vietnamese-English business term translations
- **[knowledge-base/communication-language-guidelines.md](./knowledge-base/communication-language-guidelines.md)** - Team communication standards

---

## Document Hierarchy & Cross-References

### How Documents Reference Each Other

```
README.md (overview)
    ↓ references
planning/product-strategy-refined.md (master strategy)
    ↓ referenced by
    ├── planning/target-market-clarification.md (quick reference)
    ├── planning/feature-roadmap.md (features)
    ├── planning/monetization-strategy.md (pricing)
    ├── planning/analytics-ai-strategy.md (AI implementation)
    ├── planning/implementation-roadmap.md (execution plan)
    └── research/vietnam-market-strategy.md (GTM tactics)
```

### Single Source of Truth (SSOT)

| Topic | SSOT Document | Other Documents |
|-------|--------------|-----------------|
| **Target Market** | `product-strategy-refined.md` | `target-market-clarification.md` (summary) |
| **Pricing** | `monetization-strategy.md` | `product-strategy-refined.md` (overview) |
| **Features** | `feature-roadmap.md` | `mvp-acceptance-criteria.md` (detailed specs) |
| **Database** | `database-schema.md` | `database-schema-analytics-extension.md` (analytics tables) |
| **GTM Tactics** | `vietnam-market-strategy.md` | `product-strategy-refined.md` (strategy) |

---

## Recent Changes (October 2025)

### Documents Removed (Redundant)
- ❌ `planning/STRATEGIC-REALIGNMENT-SUMMARY.md` - Transition document, no longer needed
- ❌ `knowledge-base/vietnam-market.md` - Outdated (pre-pivot strategy), conflicted with current docs

### Documents Consolidated
- ✅ `target-market-clarification.md` - Slimmed to concise summary, removed redundant details
- ✅ `README.md` - High-level overview only, detailed strategy moved to product-strategy-refined.md
- ✅ `vietnam-market-strategy.md` - Removed strategy duplication, kept GTM tactics only
- ✅ `analytics-ai-strategy.md` - Removed business context, kept technical implementation only

### Cross-Reference System
- All documents now reference the master strategy document instead of duplicating content
- Clear document hierarchy established
- Single source of truth (SSOT) defined for each topic

---

## How to Use This Documentation

### For New Team Members
1. Start with [README.md](./README.md) - Understand what we're building
2. Read [planning/product-strategy-refined.md](./planning/product-strategy-refined.md) - Understand why and how
3. Review [planning/feature-roadmap.md](./planning/feature-roadmap.md) - Understand the timeline

### For Engineers
1. Review [architecture/database-schema.md](./architecture/database-schema.md) - Understand data model
2. Check [planning/implementation-roadmap.md](./planning/implementation-roadmap.md) - Understand sprint tasks
3. Reference [planning/mvp-acceptance-criteria.md](./planning/mvp-acceptance-criteria.md) - Understand requirements

### For Product/Marketing
1. Read [planning/product-strategy-refined.md](./planning/product-strategy-refined.md) - Complete strategy
2. Check [planning/target-market-clarification.md](./planning/target-market-clarification.md) - Quick segment reference
3. Review [planning/monetization-strategy.md](./planning/monetization-strategy.md) - Pricing and ROI

### For Sales/GTM
1. Read [planning/target-market-clarification.md](./planning/target-market-clarification.md) - Customer segments
2. Review [research/vietnam-market-strategy.md](./research/vietnam-market-strategy.md) - GTM tactics
3. Check [planning/monetization-strategy.md](./planning/monetization-strategy.md) - Pricing objection handling

---

## Keeping Documentation Current

### When to Update Documents

**Always update:**
- **Master strategy document** (`product-strategy-refined.md`) - Any strategic changes
- **Feature roadmap** (`feature-roadmap.md`) - When priorities shift
- **Database schema** (`database-schema.md`) - When data model changes

**Update if relevant:**
- **GTM strategy** (`vietnam-market-strategy.md`) - When tactics change
- **Implementation roadmap** (`implementation-roadmap.md`) - Weekly sprint updates

**Rarely update:**
- **Quick references** (`target-market-clarification.md`) - Only if fundamentals change
- **README** - Only for major pivots

### Avoiding Redundancy

✅ **Do:**
- Reference the master document: "See product-strategy-refined.md for details"
- Keep summaries concise with cross-references
- Update one source of truth, not multiple files

❌ **Don't:**
- Copy-paste full sections between documents
- Duplicate detailed analysis in multiple files
- Create new summary documents without removing old content

---

## Questions?

If you can't find what you're looking for:
1. Check this structure guide first
2. Use file search for keywords
3. Start with the master strategy document (product-strategy-refined.md)
4. Ask the team if information is missing or unclear
