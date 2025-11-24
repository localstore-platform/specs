# System Architecture - Hybrid NestJS + Python

**Last Updated:** October 22, 2025  
**Architecture Type:** Hybrid Microservices  
**Status:** In Implementation

## Overview

AI-first, real-time business intelligence platform for Vietnamese restaurant operators. Hybrid architecture optimizes for GraphQL/WebSocket performance (NestJS) while leveraging Python's ML ecosystem for AI recommendations.

### Multi-Domain Architecture

**Platform Domains:**

- `app.localstoreplatform.com` - Main platform (owners only): Cross-location analytics, staff management, billing
- `{tenant-slug}.localstoreplatform.com` - Store operations (owner + staff): Daily operations, sales entry, location-specific dashboard
- `{tenant-slug}.lsp.menu` - Public storefront (customers, Phase 2): Menu viewing, online ordering

**Examples:**

- `pho-bo-hanoi.localstoreplatform.com` - Phở Bò Hà Nội operations portal
- `banh-mi-saigon.localstoreplatform.com` - Bánh Mì Sài Gòn operations portal

**Key Benefits:**

- ✅ Tenant isolation by subdomain (can't accidentally access other businesses)
- ✅ Clear separation: strategic (main platform) vs operational (tenant subdomain)
- ✅ Scalable onboarding: Phone invites (small) + permanent codes (chains)

## High-Level Architecture Diagram

```text
┌─────────────────────────────────────────────────────────────────────┐
│                      CLIENT APPLICATIONS                             │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Flutter    │  │  Next.js     │  │   Electron   │              │
│  │   Mobile     │  │  Web Portal  │  │   Desktop    │              │
│  │ (Owner Only) │  │ (Staff/Mgr)  │  │   (Future)   │              │
│  │              │  │              │  │              │              │
│  │ GraphQL ◄────┼──┼─ GraphQL     │  │  GraphQL     │              │
│  │ WebSocket    │  │ WebSocket    │  │  WebSocket   │              │
│  │ FCM Push     │  │              │  │              │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS (443)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    NESTJS API GATEWAY (Port 3000)                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐        │
│  │ GraphQL Server │  │ WebSocket      │  │ REST Endpoints │        │
│  │ (Apollo)       │  │ (Socket.io)    │  │                │        │
│  │                │  │                │  │ /auth/*        │        │
│  │ • queries      │  │ • subscriptions│  │ /webhooks/*    │        │
│  │ • mutations    │  │ • pub/sub      │  │ /uploads/*     │        │
│  │ • dataloaders  │  │ • rooms        │  │                │        │
│  └────────────────┘  └────────────────┘  └────────────────┘        │
│                                                                       │
│  ┌────────────────────────────────────────────────────────┐         │
│  │              BUSINESS LOGIC LAYER                      │         │
│  │  • AuthModule (JWT, OTP, Refresh Tokens)              │         │
│  │  • TenantsModule (Multi-tenancy enforcement)          │         │
│  │  • LocationsModule (Multi-location scoping)           │         │
│  │  • DashboardModule (Metrics aggregation)              │         │
│  │  • MenuModule (Items, categories, pricing)            │         │
│  │  • SalesModule (Transactions, analytics)              │         │
│  │  • NotificationsModule (FCM, WebSocket broadcast)     │         │
│  │  • PaymentsModule (MoMo/ZaloPay/VNPay webhooks)       │         │
│  │  • IntegrationsModule (AI service gRPC client)        │         │
│  └────────────────────────────────────────────────────────┘         │
│                                                                       │
│  ┌────────────────────────────────────────────────────────┐         │
│  │              INFRASTRUCTURE SERVICES                   │         │
│  │  • TypeORM (PostgreSQL connection pool)               │         │
│  │  • Redis Client (Cache + Pub/Sub)                     │         │
│  │  • S3 Client (Image uploads)                          │         │
│  │  • gRPC Client (AI service communication)             │         │
│  │  • Bull Queue (Background jobs)                       │         │
│  │  • Winston Logger (Structured logging)                │         │
│  └────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ gRPC (Binary Protocol)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  PYTHON AI SERVICE (Port 5000)                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐        │
│  │  gRPC Server   │  │  FastAPI HTTP  │  │ Celery Workers │        │
│  │                │  │  (Admin API)   │  │                │        │
│  │ • GetRecommend │  │                │  │ • Train models │        │
│  │ • PredictDemand│  │ /admin/models  │  │ • Process data │        │
│  │ • AnalyzePricing│ │ /admin/metrics │  │ • Generate     │        │
│  │ • StreamData   │  │                │  │   reports      │        │
│  └────────────────┘  └────────────────┘  └────────────────┘        │
│                                                                       │
│  ┌────────────────────────────────────────────────────────┐         │
│  │                  ML ENGINE                             │         │
│  │  • RecommendationService (scikit-learn)               │         │
│  │  • ForecastingService (Prophet, ARIMA)                │         │
│  │  • PricingService (Dynamic pricing rules)             │         │
│  │  • AnomalyDetector (Isolation Forest)                 │         │
│  │  • DataPipeline (pandas, numpy transformations)       │         │
│  └────────────────────────────────────────────────────────┘         │
│                                                                       │
│  ┌────────────────────────────────────────────────────────┐         │
│  │            DATA ACCESS LAYER                           │         │
│  │  • PostgreSQL Client (asyncpg)                        │         │
│  │  • Redis Client (Cache + Feature store)              │         │
│  │  • Model Registry (MLflow - future)                  │         │
│  └────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                  │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ PostgreSQL   │  │ Redis        │  │ S3 Storage   │              │
│  │    14+       │  │    6+        │  │              │              │
│  │              │  │              │  │              │              │
│  │ • 23 tables  │  │ • Sessions   │  │ • Images     │              │
│  │ • Multi-     │  │ • Cache      │  │ • Exports    │              │
│  │   tenant RLS │  │   (60s TTL)  │  │ • ML models  │              │
│  │ • Location   │  │ • Pub/Sub    │  │   (future)   │              │
│  │   scoping    │  │ • Job queue  │  │              │              │
│  │ • Materialized│ │ • Feature    │  │              │              │
│  │   views      │  │   store      │  │              │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      EXTERNAL SERVICES                               │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ SMS Provider │  │ Payment APIs │  │ Firebase FCM │              │
│  │              │  │              │  │              │              │
│  │ • Twilio     │  │ • MoMo       │  │ • iOS Push   │              │
│  │ • Viettel    │  │ • ZaloPay    │  │ • Android    │              │
│  │   SMS        │  │ • VNPay      │  │   Push       │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

## Technology Stack

### Client Applications

- **Mobile (Owner):** Flutter 3.x (Dart) - iOS + Android native
  - State Management: Riverpod 2.x
  - GraphQL: graphql_flutter
  - WebSocket: socket_io_client
  - Offline: Hive 2.x
  - Push: firebase_messaging
- **Web Portal (Staff):** Next.js 14 (React + TypeScript) + Tailwind CSS
  - GraphQL: Apollo Client
  - WebSocket: socket.io-client
  - State: Zustand or Redux Toolkit
  - UI: shadcn/ui + Radix UI

### Backend Services

- **API Gateway:** NestJS 10 (TypeScript/Node.js 20)
  - Framework: @nestjs/core
  - GraphQL: @nestjs/graphql + Apollo Server
  - WebSocket: @nestjs/websockets + Socket.io
  - ORM: TypeORM 0.3
  - Auth: @nestjs/jwt + Passport
  - Cache: @nestjs/cache-manager + Redis
  - Queue: @nestjs/bull + Bull
  - Validation: class-validator + class-transformer
  
- **AI Service:** Python 3.11 + FastAPI 0.104
  - gRPC: grpcio + grpcio-tools
  - ML: scikit-learn 1.3, pandas 2.1, numpy 1.26
  - Forecasting: Prophet 1.1 (optional: statsmodels)
  - Jobs: Celery 5.3 + Redis broker
  - DB: asyncpg (PostgreSQL), redis-py
  - API: FastAPI (for admin endpoints)

### Data Layer

- **Database:** PostgreSQL 14+ with TimescaleDB extension (time-series)
  - Multi-tenant: Row-Level Security (RLS)
  - Location scoping: Composite indexes
  - Materialized views: Real-time refresh for analytics
  
- **Cache:** Redis 6+
  - Sessions (60min TTL)
  - Metrics cache (60s TTL)
  - Recommendations (5min TTL)
  - Pub/Sub for WebSocket broadcast
  - Bull queue for background jobs
  
- **Storage:** S3-compatible (AWS S3 or MinIO)
  - Images: Optimized with CloudFront CDN
  - Exports: CSV/PDF reports
  - ML models: Versioned artifacts (future: MLflow)

### Infrastructure

- **Container:** Docker + Docker Compose (dev), Kubernetes (prod)
- **Reverse Proxy:** Nginx or Traefik
- **Monitoring:** Prometheus + Grafana
- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)
- **CI/CD:** GitHub Actions
- **Hosting:**
  - NestJS: DigitalOcean/AWS ECS
  - Python AI: DigitalOcean/AWS ECS
  - Flutter: App Store + Google Play
  - Next.js: Vercel

## Architecture Rationale

### Why Hybrid (NestJS + Python)?

**NestJS Strengths (Client-Facing API):**

- Full-stack TypeScript: Type safety across frontend/backend
- Built-in GraphQL: Decorators, automatic schema generation, DataLoader
- WebSocket native: @nestjs/websockets with Socket.io
- Fast iteration: Vietnamese developer availability, rapid prototyping
- Ecosystem: Rich libraries for auth, validation, caching, queues

**Python Strengths (AI/ML Service):**

- ML ecosystem: scikit-learn, pandas, Prophet, TensorFlow (future)
- Data processing: Fast numpy/pandas operations
- Proven patterns: Jupyter notebooks → production services
- Future-proof: Easy to add advanced ML (PyTorch, Transformers)

**gRPC Communication (Backend ↔ AI):**

- 10x faster than REST JSON for large data transfers
- Binary protocol: Efficient serialization (protobuf)
- Streaming: Bi-directional streams for real-time predictions
- Type safety: Auto-generated clients from .proto definitions

### Why GraphQL (Not REST)?

**Flexible Queries:**

- Mobile needs: Minimal data (battery, bandwidth)
- Web needs: Rich data (detailed analytics)
- Single endpoint: No versioning hell (/api/v1, /api/v2)

**Example:**

```graphql
# Mobile (minimal)
query DashboardMobile {
  location(id: "123") {
    todayRevenue
    todayOrders
  }
}

# Web (rich)
query DashboardWeb {
  location(id: "123") {
    todayRevenue
    todayOrders
    hourlyTrends { hour revenue }
    topItems { name quantity revenue }
    recommendations { title priority }
  }
}
```

### Why WebSocket (Not Polling)?

**Real-Time Requirements:**

- Payment webhooks: <500ms notification to owner
- Live metrics: Dashboard updates every 30s
- Multi-device sync: Changes propagate instantly
- Battery efficient: No polling overhead

**Example Flow:**

```
MoMo webhook → NestJS processes → PostgreSQL insert → 
Redis publish → WebSocket broadcast → All clients update (<500ms)
```

## Communication Patterns

### Client → NestJS (GraphQL)

```
Flutter/Next.js --HTTPS--> GraphQL Endpoint (/graphql)
  - Queries: Read data
  - Mutations: Write data
  - Subscriptions: Real-time updates via WebSocket
```

### Client → NestJS (WebSocket)

```
Flutter/Next.js --WSS--> Socket.io (/socket.io)
  - Join rooms (tenant ID + location ID)
  - Receive broadcasts (new order, payment, AI alert)
  - Auto-reconnect with exponential backoff
```

### NestJS → Python AI (gRPC)

```
NestJS --gRPC--> Python AI Service (Port 5000)
  - GetRecommendations(location_id, date_range)
  - PredictDemand(location_id, item_id, forecast_days)
  - AnalyzePricing(location_id, competitor_prices)
  - StreamSalesData (bidirectional stream)
```

### External Webhooks → NestJS (REST)

```
MoMo/ZaloPay --HTTPS--> /api/v1/webhooks/payment
  - Verify signature
  - Process payment
  - Broadcast to WebSocket
  - Send FCM push notification
```

## Data Flow Examples

### Scenario 1: Owner Views Dashboard

```
1. Flutter app opens → GraphQL query
   query { location(id: "123") { todayRevenue todayOrders } }

2. NestJS checks Redis cache (key: dashboard:123:2025-10-22)
   - Cache HIT (60s TTL) → Return cached data (10ms response)
   - Cache MISS → Query PostgreSQL + cache result

3. WebSocket subscription active
   subscription { dashboardUpdates(locationId: "123") }

4. New sale occurs → NestJS updates PostgreSQL
   → Redis publish("dashboard:123", newData)
   → WebSocket broadcast to all connected clients
   → Flutter updates UI reactively (Riverpod auto-refresh)
```

### Scenario 2: AI Recommendation Generation

```
1. Daily cron job (12 AM) → NestJS triggers background job

2. NestJS → gRPC call to Python AI service
   GetRecommendations(location_id="123", date_range="2025-10-15:2025-10-21")

3. Python AI service:
   - Queries PostgreSQL (last 7 days sales via asyncpg)
   - Runs ML model (scikit-learn rules engine)
   - Calculates recommendations (pricing, inventory, promo ideas)
   - Returns gRPC response (protobuf binary)

4. NestJS receives recommendations → Saves to PostgreSQL
   → Sends FCM push notification to owner
   → Broadcasts to WebSocket (if owner online)

5. Owner opens app → Sees recommendations → Approves/Dismisses
   → GraphQL mutation → NestJS updates status
```

### Scenario 3: Payment Webhook Processing

```
1. Customer pays via MoMo → MoMo calls NestJS webhook
   POST /api/v1/webhooks/payment
   { transaction_id, amount, status: "success", signature }

2. NestJS validates signature → Creates transaction record
   → Updates daily revenue in PostgreSQL

3. NestJS publishes to Redis Pub/Sub
   PUBLISH dashboard:123 '{"event":"new_payment","amount":50000}'

4. WebSocket server subscribes to Redis → Broadcasts to clients
   - Owner's Flutter app: Real-time notification + dashboard update
   - Manager's web portal: Live transaction log update

5. NestJS sends FCM push notification (if owner offline)
   "Thanh toán mới: 50.000₫ - Cà phê sữa đá"
```

## Security Architecture

### Authentication Flow (Phone OTP)

```
1. User enters phone → POST /api/v1/auth/otp/request
2. NestJS generates 6-digit OTP → Sends SMS → Stores in Redis (120s TTL)
3. User enters OTP → POST /api/v1/auth/otp/verify
4. NestJS validates OTP → Issues JWT access token (15min) + refresh token (30d)
5. Client stores tokens securely (iOS Keychain, Android Keystore)
6. GraphQL requests include Authorization: Bearer <access_token>
7. Access token expires → POST /api/v1/auth/refresh with refresh token
```

### Multi-Tenancy Enforcement

- **Database:** All tables include `tenant_id` column
- **ORM:** TypeORM global filter: `WHERE tenant_id = :currentTenantId`
- **GraphQL:** Context includes authenticated user's tenant ID
- **Validation:** Every query/mutation checks tenant ownership

### Location Scoping

- **Chains:** User can manage multiple locations under same tenant
- **Queries:** Include `location_id` parameter
- **Authorization:** Verify user has access to requested location
- **Dashboard:** Default to user's primary location, allow switching

## Performance Targets

### API Response Times (p95)

- GraphQL queries: <200ms (cached), <500ms (uncached)
- GraphQL mutations: <300ms
- WebSocket message delivery: <100ms
- gRPC calls (NestJS → Python): <150ms

### Mobile App (4G Network)

- Cold start: <3s
- Dashboard load: <2s (with cache)
- Screen transitions: <300ms
- Image loading: <1s (with CDN + cache)

### Database Performance

- Simple queries: <10ms
- Aggregations: <100ms
- Materialized view refresh: <5s (background)
- Connection pool: 20 connections (NestJS), 10 connections (Python)

### Cache Hit Rates

- Dashboard metrics: >80% (60s TTL)
- User sessions: 100% (in-memory Redis)
- Recommendations: >70% (5min TTL)

## Scalability Strategy

### Horizontal Scaling

- **NestJS:** Stateless, scale to N instances behind load balancer
- **Python AI:** Stateless gRPC service, scale independently
- **PostgreSQL:** Read replicas for analytics queries
- **Redis:** Redis Cluster for high availability

### Vertical Scaling (Initial)

- **NestJS:** 2 CPU, 4GB RAM per instance (DigitalOcean Droplet $24/mo)
- **Python AI:** 4 CPU, 8GB RAM (ML workloads, $48/mo)
- **PostgreSQL:** 4 CPU, 8GB RAM ($48/mo)
- **Redis:** 2 CPU, 2GB RAM ($12/mo)
- **Total:** ~$150/mo for <1000 users

### Future Optimizations

- CDN for static assets (CloudFront, Cloudflare)
- Database sharding by tenant_id (when >10k tenants)
- Redis Cluster (when >5k concurrent users)
- Kubernetes auto-scaling (when traffic unpredictable)
