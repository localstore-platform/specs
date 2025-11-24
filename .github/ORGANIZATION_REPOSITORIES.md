# LocalStore Platform - GitHub Organization Repositories

**Organization:** `localstore-platform`  
**Website:** TBD  
**Primary Market:** Vietnamese small shops and street food vendors

---

## Repository Descriptions

### 1. **specs** (Current Repository)

**URL:** `github.com/localstore-platform/specs`  
**Description:**

```
📋 Technical specifications and architecture documentation for LocalStore Platform - 
AI-powered menu & analytics SaaS for Vietnamese SMBs. Complete specs for API, 
database, mobile app, and deployment. Single source of truth for all implementation repos.
```

**Topics/Tags:**

```
specifications, documentation, architecture, api-spec, database-schema, 
vietnamese-market, restaurant-tech, saas-platform, multi-tenant
```

**About:**

- Complete technical specifications (v1.0-specs)
- Architecture decisions and system diagrams
- Implementation roadmaps and sprint planning
- Vietnamese market research and strategy
- Spec-driven development guidelines

**Website:** Link to rendered docs (GitHub Pages)

---

### 2. **api**

**URL:** `github.com/localstore-platform/api`  
**Description:**

```
🔧 Backend API for LocalStore Platform - Hybrid NestJS + Python architecture with 
GraphQL, WebSocket, and REST endpoints. Multi-tenant SaaS backend supporting menu 
management, analytics, and AI recommendations for Vietnamese restaurants.
```

**Topics/Tags:**

```
nestjs, typescript, nodejs, graphql, websocket, postgresql, redis, 
multi-tenant, restaurant-api, vietnamese, python-ai, grpc
```

**About:**

- NestJS API Gateway with TypeORM
- GraphQL (Apollo) + WebSocket (Socket.io)
- Python FastAPI AI service integration
- Multi-tenancy with Row-Level Security
- Vietnamese localization (VND currency, vi-VN locale)
- Docker Compose for local development

**Tech Stack:**

- NestJS 10 + TypeScript 5
- PostgreSQL 14 + Redis 7
- Python 3.11 + FastAPI (AI service)
- Bull Queue, Winston Logger

---

### 3. **menu**

**URL:** `github.com/localstore-platform/menu`  
**Description:**

```
🌐 Public menu website for LocalStore Platform - Fast, mobile-first Next.js site 
for customer-facing restaurant menus. Powers {tenant}.lsp.menu domains with QR code 
support and Vietnamese formatting. Optimized for 4G on entry-level Android devices.
```

**Topics/Tags:**

```
nextjs, react, tailwind-css, menu-website, qr-code, vietnamese, 
restaurant-menu, mobile-first, static-site-generation, vercel
```

**About:**

- Next.js 14 with App Router
- Static site generation for fast loading (<2s TTI)
- Responsive design (mobile-first)
- Vietnamese currency formatting (75.000₫)
- QR code landing pages
- SEO optimized for local search

**Demo Priority:** Phase 1 implementation focus

---

### 4. **dashboard**

**URL:** `github.com/localstore-platform/dashboard`  
**Description:**

```
🎛️ Operations dashboard for LocalStore Platform - Next.js web portal for restaurant 
owners and staff. Menu management, sales tracking, and analytics. Powers 
{tenant}.localstoreplatform.com operations domains with GraphQL + real-time updates.
```

**Topics/Tags:**

```
nextjs, react, dashboard, graphql-client, apollo-client, restaurant-management, 
analytics-dashboard, vietnamese, multi-location, real-time
```

**About:**

- Next.js 14 + Tailwind CSS
- Apollo GraphQL Client
- Real-time updates via WebSocket subscriptions
- Multi-location support for chains
- Vietnamese UI with vi-VN locale
- Mobile-responsive admin interface

**Priority:** Phase 3 (Optional - mobile app can replace in early stages)

---

### 5. **mobile**

**URL:** `github.com/localstore-platform/mobile`  
**Description:**

```
📱 Owner mobile app for LocalStore Platform - Flutter app for iOS & Android. 
Daily monitoring, menu management, and AI-powered business insights for Vietnamese 
restaurant owners. Offline-first with FCM push notifications.
```

**Topics/Tags:**

```
flutter, dart, mobile-app, ios, android, riverpod, offline-first, 
restaurant-app, vietnamese, firebase-messaging, owner-app
```

**About:**

- Flutter 3.x (iOS + Android)
- Riverpod state management
- Clean Architecture (data/domain/presentation)
- Offline support with Hive local storage
- Push notifications (Firebase Cloud Messaging)
- Vietnamese UI/UX with accessibility standards
- Biometric authentication support

**Tech Stack:**

- Flutter 3.x, Dart 3.x
- Riverpod 2.4, Dio, Retrofit
- Hive, Firebase, Local Auth

---

### 6. **ml**

**URL:** `github.com/localstore-platform/ml`  
**Description:**

```
🤖 AI/ML service for LocalStore Platform - Python FastAPI + gRPC service providing 
demand forecasting, price optimization, and menu recommendations. Celery workers 
for model training and batch processing. Integrates with main API via gRPC.
```

**Topics/Tags:**

```
python, machine-learning, fastapi, grpc, celery, scikit-learn, 
recommendation-engine, demand-forecasting, restaurant-ai, vietnamese-market
```

**About:**

- FastAPI HTTP API + gRPC server
- Celery workers for model training
- Demand forecasting (time series)
- Dynamic pricing optimization
- Menu recommendations (collaborative filtering)
- MLflow model versioning
- Vietnamese business logic

**Tech Stack:**

- Python 3.11, FastAPI, gRPC
- scikit-learn, pandas, Prophet
- Celery, Redis, PostgreSQL

**Priority:** Phase 6+ (Post-MVP, when có enough data)

---

### 7. **infra**

**URL:** `github.com/localstore-platform/infra`  
**Description:**

```
🏗️ Infrastructure as Code for LocalStore Platform - Terraform configurations for 
AWS deployment, Docker Compose for local dev, and CI/CD pipelines. Single-server 
MVP setup (~$20/month) with scaling path to production architecture.
```

**Topics/Tags:**

```
terraform, infrastructure-as-code, aws, docker, docker-compose, 
devops, ci-cd, github-actions, cost-optimization, startup-infra
```

**About:**

- Terraform for AWS infrastructure
- Docker Compose for local development
- GitHub Actions CI/CD pipelines
- Cost-optimized MVP deployment (EC2 t2.small)
- Production scaling configurations
- Monitoring and alerting setup (CloudWatch)

**Environments:**

- Local: Docker Compose (PostgreSQL + Redis)
- Production: AWS EC2 + RDS + ElastiCache

**Priority:** Phase 5 (Production deployment)

---

### 8. **contracts**

**URL:** `github.com/localstore-platform/contracts`  
**Description:**

```
📦 Shared types and contracts for LocalStore Platform - TypeScript definitions, 
GraphQL schemas, and Protobuf specs shared across all repos. Published as 
@localstore/types npm package for contract-driven development.
```

**Topics/Tags:**

```
typescript, graphql, protobuf, api-contracts, shared-types, 
monorepo, contract-testing, npm-package
```

**About:**

- TypeScript API types (REST DTOs)
- GraphQL schema definitions
- Protobuf definitions (gRPC contracts)
- Shared enums and constants
- Vietnamese locale types
- Published as `@localstore/types` npm package

**Purpose:**

- Single source of truth for types
- Contract testing support
- Type safety across repos

---

### 9. **web-admin** (Optional)

**URL:** `github.com/localstore-platform/web-admin`  
**Description:**

```
👨‍💼 Platform admin dashboard - Internal tool for LocalStore Platform operators. 
Tenant management, system monitoring, and support tools. Not customer-facing. 
Separate from tenant operations dashboard.
```

**Topics/Tags:**

```
nextjs, admin-dashboard, internal-tool, tenant-management, 
system-monitoring, support-tools
```

**About:**

- Platform operator tools
- Tenant/subscription management
- System health monitoring
- Support ticket handling
- Analytics across all tenants

**Priority:** Post-MVP (internal tooling)

---

## Organization Structure

```
localstore-platform/
├── specs/              (This repo - Documentation)
├── api/                (Backend - NestJS + Python)
├── menu/               (Customer menu web - Next.js)
├── dashboard/          (Operations portal - Next.js)
├── mobile/             (Owner app - Flutter)
├── ml/                 (AI service - Python)
├── infra/              (Infrastructure - Terraform)
├── contracts/          (Shared types - TypeScript)
└── web-admin/          (Platform admin - Next.js) [Future]
```

---

## Repository Relationships

```
specs (v1.0-specs)
    ↓ (references in SPEC_LINKS.md)
    ├─ api → contracts
    ├─ menu → contracts
    ├─ dashboard → contracts
    ├─ mobile → contracts
    └─ ml → contracts

api ←→ ml (gRPC communication)
menu ← api (REST API)
dashboard ← api (GraphQL + WebSocket)
mobile ← api (GraphQL + REST)
```

---

## Getting Started

### For New Contributors

1. **Start with specs repo:** Understand architecture and planning

   ```bash
   git clone https://github.com/localstore-platform/specs.git
   cd specs
   # Read README.md and architecture/*.md files
   ```

2. **Check implementation priority:**
   - Phase 1: `menu` + `api` (local dev)
   - Phase 2: `api` CRUD endpoints
   - Phase 3: `mobile` owner app
   - Phase 4: `ml` service (post-MVP)

3. **Setup local environment:**

   ```bash
   git clone https://github.com/localstore-platform/api.git
   cd api
   docker-compose up  # PostgreSQL + Redis + API
   ```

4. **Read SPEC_LINKS.md** in each repo for specific spec references

### For AI Assistants

- Follow guidelines in `specs/.github/AI_CONTEXT_GUIDE.md`
- Load specs on-demand via semantic_search
- Reference spec version (v1.0-specs) in each repo's SPEC_LINKS.md

---

## Tech Stack Summary

| Repo | Primary Languages | Key Frameworks | Priority |
|------|------------------|----------------|----------|
| **specs** | Markdown | - | Complete ✅ |
| **api** | TypeScript, Python | NestJS, FastAPI | Phase 1 🔴 |
| **menu** | TypeScript, React | Next.js 14 | Phase 1 🔴 |
| **dashboard** | TypeScript, React | Next.js 14 | Phase 3 🔴 |
| **mobile** | Dart | Flutter 3.x | Phase 3 🔴 |
| **ml** | Python | FastAPI, scikit-learn | Phase 6 ⏸️ |
| **infra** | HCL | Terraform, Docker | Phase 5 ⏸️ |
| **contracts** | TypeScript | - | Phase 2 🔴 |

---

## Cost Estimates

### Development Phase (Week 1-9)

- Cost: **$0** (localhost with Docker Compose)
- Infrastructure: None

### Production Phase (Week 10+)

- MVP: **~$20/month** (AWS EC2 t2.small)
- Growth: **~$60-80/month** (RDS, ElastiCache, load balancer)
- Scale: **~$120-150/month** (Multi-AZ, backups, monitoring)

---

## License

TBD (MIT recommended for maximum adoption)

---

## Contact

- **Organization:** LocalStore Platform
- **Primary Market:** Vietnam
- **Stage:** Pre-MVP Development
- **Docs:** [specs repository](https://github.com/localstore-platform/specs)

---

**Generated:** 2025-11-25  
**Spec Version:** v1.0-specs  
**Last Updated:** This document
