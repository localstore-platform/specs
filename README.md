# Local Store Platform

**AI-powered business intelligence platform** for Vietnamese restaurant operators. We help chain owners and skilled operators make data-driven business decisions through AI recommendations.

This repository contains all strategic planning, architecture, and design documents for the project.

## 🚀 What We Build

**We're not a menu builder—we're an AI business manager.** Menu creation and online ordering are data collection tools that feed our AI models to deliver actionable business recommendations.

### Our Customers (Equal Priority)

**Segment A:** Chain owners (3-10 locations) who lack TIME to manage all locations  
**Segment B:** Skilled operators (chefs/baristas) who lack BUSINESS KNOWLEDGE

**Unified Value Proposition:** "AI quản lý kinh doanh cho chủ quán không có thời gian hoặc kiến thức về kinh tế"

### Product Priorities

1. **AI Analytics Dashboard (80%)** - Core product delivering business insights
2. **Data Collection Tools (15%)** - Menu, ordering, POS integration
3. **Customer-Facing Tools (5%)** - Public storefronts, QR menus

**Market:** Vietnam only (Phases 1-3). Deep integrations with MoMo, ZaloPay, GrabFood, Zalo.

> **📖 For complete strategy, see [planning/product-strategy-refined.md](./planning/product-strategy-refined.md)**

## 📁 Document Structure

- **/planning**: Contains the feature roadmap and monetization strategy.
- **/research**: Contains market analysis and go-to-market strategies for Vietnam and Japan.
- **/design**: Contains wireframes, UX flows, and UI mockups.
- **/architecture**: Contains the system architecture diagram and technology stack decisions.
- **/knowledge-base**: Living playbooks for market-specific execution (currently focused on Vietnam).

### Quick Links

- [Language guidelines (VN/EN)](./knowledge-base/communication-language-guidelines.md)
- [Glossary](./knowledge-base/glossary.md)
- [Documentation structure guide](./DOCUMENTATION-STRUCTURE.md)

<!-- Pointer to implementation repo -->
_Developer artifacts (OpenAPI for codegen, seeds, CI) live in the implementation repository — see `documentation/implementation-repo-README-template.md` for expected layout._

## 🛠 Tech Stack

### Client Applications
- **Mobile (Owner):** Flutter (iOS + Android) - Real-time dashboard, push notifications
- **Web Portal (Staff):** Next.js + Tailwind CSS - Admin operations, data entry

### Backend Services (Hybrid Architecture)
- **API Gateway:** NestJS (TypeScript) - GraphQL, WebSocket, REST
- **AI Service:** Python + FastAPI - ML models, recommendations engine
- **Communication:** gRPC (backend ↔ AI), GraphQL + WebSocket (client ↔ backend)

### Data & Infrastructure
- **Database:** PostgreSQL 14+ (multi-tenant with Row-Level Security)
- **Cache:** Redis 6+ (metrics, sessions, pub/sub)
- **Storage:** S3-compatible (images, exports)
- **Deployment:** Docker + Docker Compose (dev), Kubernetes (prod)

> **📖 See [architecture/system-diagram.md](./architecture/system-diagram.md) for complete architecture details**

## 🚀 Getting Started

### For Developers

**Quick Start (30 minutes):**
```bash
# Prerequisites: Node.js 20+, Python 3.11+, Docker

# 1. Start database services
docker-compose up -d

# 2. Setup backend (NestJS)
cd backend
npm install
npm run start:dev

# 3. Setup AI service (Python)
cd ../ai-service
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m app.grpc_server

# 4. Setup mobile app (Flutter)
cd ../mobile
flutter pub get
flutter run
```

**📖 Full setup guide:** [QUICK-START.md](./QUICK-START.md)

### Implementation Resources
- **Backend Setup:** [architecture/backend-setup-guide.md](./architecture/backend-setup-guide.md)
- **GraphQL Schema:** [architecture/graphql-schema.md](./architecture/graphql-schema.md)
- **Flutter App Spec:** [architecture/flutter-mobile-app-spec.md](./architecture/flutter-mobile-app-spec.md)
- **Sprint 1 Plan:** [planning/sprint-1-implementation.md](./planning/sprint-1-implementation.md) (4 weeks to MVP)

### For Stakeholders

**Current Status:** Implementation Phase - Sprint 1 (Oct 22 - Nov 19, 2025)

**Sprint 1 Deliverables:**
- ✅ Phone OTP authentication
- ✅ Real-time dashboard (revenue, orders, AI recommendations)
- ✅ Push notifications (iOS + Android)
- ✅ Multi-location support
- ✅ AI recommendation engine (rule-based)
