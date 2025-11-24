# Sprint 1: Foundation Implementation

**Duration:** 4 weeks (October 22 - November 19, 2025)  
**Team:** Backend Lead, Mobile Developer, Data Scientist  
**Goal:** Launch MVP with authentication, basic dashboard, and AI recommendations

---

## Overview

Sprint 1 establishes the foundation for the Local Store Platform by implementing:

- ✅ Hybrid backend architecture (NestJS + Python) with gRPC communication
- ✅ Phone OTP authentication flow
- ✅ Real-time dashboard with GraphQL + WebSocket
- ✅ Basic AI recommendation engine (rule-based)
- ✅ Flutter mobile app (authentication + dashboard)
- ✅ PostgreSQL database with 23 tables
- ✅ Docker development environment

---

## Success Criteria

### Functional Requirements

- [ ] Users can sign up/login via phone OTP
- [ ] Dashboard displays today's metrics (revenue, orders, customers)
- [ ] AI generates 3+ recommendations daily
- [ ] Owners can approve/dismiss recommendations
- [ ] Mobile app receives push notifications (FCM)
- [ ] Real-time updates when new sales occur (<500ms)
- [ ] Multi-location support (view/switch locations)

### Technical Requirements

- [ ] NestJS API Gateway operational with GraphQL + WebSocket
- [ ] Python AI Service responding to gRPC requests
- [ ] PostgreSQL database with all 23 tables migrated
- [ ] Redis caching with 60s TTL for dashboard metrics
- [ ] Docker Compose for local development
- [ ] 80%+ code coverage for critical paths
- [ ] <500ms API response time (p95)

### Business Requirements

- [ ] Owner can onboard in <5 minutes
- [ ] Demo-able to first 10 pilot customers
- [ ] Analytics tracking for onboarding funnel
- [ ] Vietnamese language throughout

---

## Week 1: Backend Foundation (Oct 22-28)

### Day 1-2: Project Setup (Backend Lead)

**Tasks:**

- [ ] Initialize NestJS project with dependencies
- [ ] Configure TypeORM + PostgreSQL connection
- [ ] Setup Redis client with cache manager
- [ ] Create Docker Compose development environment
- [ ] Initialize Python AI service project
- [ ] Generate gRPC code from proto definitions
- [ ] Setup environment variables (.env files)
- [ ] Create GitHub Actions CI workflow (linting, tests)

**Deliverables:**

- `backend/` project structure with working `npm run start:dev`
- `ai-service/` project structure with working `python -m app.grpc_server`
- `docker-compose.yml` running all services (Postgres, Redis, NestJS, Python)
- `.github/workflows/ci.yml` for automated checks

**Acceptance Criteria:**

- `docker-compose up` starts all services without errors
- Health check endpoints return 200 OK
- Hot reload works for both NestJS and Python

---

### Day 3-5: Database & Authentication (Backend Lead)

**Tasks:**

- [ ] Create TypeORM entities for all 23 tables
- [ ] Generate initial database migration
- [ ] Seed test data (1 tenant, 2 locations, 10 menu items, 50 transactions)
- [ ] Implement AuthModule (OTP request, verify, refresh)
- [ ] Implement JWT strategy with Passport
- [ ] Create auth guards for protected routes
- [ ] Setup SMS provider (Twilio or Vietnamese provider)
- [ ] Write unit tests for AuthService (80% coverage)

**Deliverables:**

- Database schema deployed with migrations
- `/api/v1/auth/otp/request` endpoint (POST)
- `/api/v1/auth/otp/verify` endpoint (POST)
- `/api/v1/auth/refresh` endpoint (POST)
- `@UseGuards(JwtAuthGuard)` decorator working

**Acceptance Criteria:**

- User can request OTP via phone number
- OTP is sent via SMS (or logged in dev mode)
- Verifying correct OTP returns JWT access + refresh tokens
- Invalid/expired OTP returns appropriate error message (Vietnamese)
- Refresh token endpoint successfully renews access token
- Rate limiting prevents >3 OTP requests per 10 minutes

**Database Migration SQL:**

```sql
-- See database/migrations/001_initial_schema.sql
-- Contains all 23 tables:
-- tenants, locations, users, categories, menu_items,
-- transactions, transaction_items, recommendations,
-- notifications, notification_devices, etc.
```

---

### Day 6-7: GraphQL Setup (Backend Lead)

**Tasks:**

- [ ] Configure Apollo Server with GraphQL module
- [ ] Create base resolvers (User, Tenant, Location)
- [ ] Implement GraphQL context (user, tenantId from JWT)
- [ ] Setup DataLoader for N+1 query prevention
- [ ] Create custom scalars (DateTime, JSON)
- [ ] Implement pagination (relay-style connections)
- [ ] Setup GraphQL Playground for development
- [ ] Write integration tests for queries

**Deliverables:**

- GraphQL endpoint `/graphql` operational
- Queries: `me`, `locations`, `location(id)`
- Mutations: `createLocation`, `updateLocation`
- GraphQL Playground accessible at `http://localhost:3000/graphql`
- Auto-generated schema file `src/schema.gql`

**Acceptance Criteria:**

- GraphQL queries require valid JWT token
- `me` query returns authenticated user + tenant
- `locations` query returns only locations for user's tenant (multi-tenancy enforced)
- DataLoader batches database queries efficiently
- Pagination works with `first`, `after` cursor args
- GraphQL Playground shows all available queries/mutations

**Example Query:**

```graphql
query {
  me {
    id
    phone
    role
    tenant {
      name
      subscriptionTier
      locations {
        id
        name
        status
      }
    }
  }
}
```

---

## Week 2: Python AI Service & gRPC (Oct 29 - Nov 4)

### Day 1-3: Python AI Service Foundation (Data Scientist)

**Tasks:**

- [ ] Setup FastAPI project structure
- [ ] Implement asyncpg database client
- [ ] Create Redis client for caching
- [ ] Build RecommendationService (rule-based logic)
- [ ] Implement data fetching queries (sales, items, transactions)
- [ ] Create helper functions (normalize data, calculate stats)
- [ ] Write unit tests for recommendation logic (70% coverage)
- [ ] Setup pytest fixtures for test data

**Deliverables:**

- `app/services/recommendation_service.py` with `generate_recommendations()` method
- `app/database/queries.py` with SQL queries for sales data
- FastAPI HTTP endpoints for admin (optional: `/api/admin/recommendations`)
- Pytest test suite passing

**Acceptance Criteria:**

- Service can fetch last 7 days of sales data from PostgreSQL
- Generates 3+ recommendations based on rules:
  - Low-selling items → Pricing recommendation
  - High-volume items → Inventory recommendation
  - Low-traffic hours → Promotion recommendation
- Recommendations include Vietnamese title, description, impact, action JSON
- Insufficient data (<10 transactions) returns default onboarding recommendation
- Unit tests cover all recommendation types

**Recommendation Rules (Phase 1):**

1. **Pricing**: Items in bottom 20% quantity → suggest 10-15% discount
2. **Inventory**: Items in top 20% quantity → suggest 3x daily avg stock
3. **Promotion**: Hours in bottom 30% revenue → suggest time-based discount
4. **Cross-sell**: (Placeholder for Phase 2 - ML-based)

---

### Day 4-5: gRPC Server Implementation (Data Scientist)

**Tasks:**

- [ ] Implement gRPC server with AIServiceServicer
- [ ] Create `GetRecommendations` RPC method
- [ ] Convert Python dict recommendations to protobuf messages
- [ ] Setup gRPC health check
- [ ] Handle errors and return proper gRPC status codes
- [ ] Log all gRPC requests with timing
- [ ] Write integration tests (gRPC client → server)

**Deliverables:**

- `app/grpc_server.py` running on port 5000
- `GetRecommendations` RPC responding to requests
- gRPC reflection enabled for debugging
- Integration tests using grpcio testing framework

**Acceptance Criteria:**

- gRPC server starts successfully in Docker
- `GetRecommendations(tenant_id, location_id, date_range_days)` returns recommendations
- Response includes protobuf `Recommendation` objects with all fields
- Server handles errors gracefully (e.g., database connection failure)
- Average RPC latency <100ms for typical requests
- Health check RPC returns healthy status

**Test with grpcurl:**

```bash
grpcurl -plaintext \
  -d '{"tenant_id":"tenant-123","location_id":"loc-456","date_range_days":7}' \
  localhost:5000 \
  ai_service.AIService/GetRecommendations
```

---

### Day 6-7: NestJS ↔ Python Integration (Backend Lead + Data Scientist)

**Tasks:**

- [ ] Create IntegrationsModule in NestJS
- [ ] Implement GrpcAiService client
- [ ] Create RecommendationsModule in NestJS
- [ ] Implement GraphQL resolver for recommendations
- [ ] Create scheduled job (cron) for daily recommendation generation
- [ ] Save recommendations to PostgreSQL via TypeORM
- [ ] Implement recommendation approval/dismissal mutations
- [ ] Write end-to-end integration test (NestJS → gRPC → Python)

**Deliverables:**

- `backend/src/modules/integrations/grpc-ai.service.ts`
- `backend/src/modules/recommendations/recommendations.module.ts`
- GraphQL queries: `recommendations(locationId)`, `recommendation(id)`
- GraphQL mutations: `approveRecommendation(id)`, `dismissRecommendation(id)`
- Cron job: Run daily at 12 AM to generate recommendations for all locations

**Acceptance Criteria:**

- GraphQL `recommendations` query fetches data from PostgreSQL
- `generateRecommendations` mutation triggers gRPC call to Python service
- Recommendations are saved to database with status `PENDING`
- Owner can approve recommendation → status changes to `APPROVED`, `approvedAt` timestamp set
- Owner can dismiss recommendation → status changes to `DISMISSED`, `dismissedAt` timestamp set
- Cron job runs successfully and generates recommendations for all active locations
- End-to-end test: Mock sales data → Generate recommendations → Verify database

**GraphQL Example:**

```graphql
query GetRecommendations($locationId: ID!) {
  recommendations(locationId: $locationId, status: PENDING, limit: 10) {
    edges {
      node {
        id
        type
        priority
        title
        description
        impact
        action
        generatedAt
      }
    }
  }
}

mutation Approve($id: ID!) {
  approveRecommendation(id: $id) {
    id
    status
    approvedAt
  }
}
```

---

## Week 3: Dashboard & Real-Time (Nov 5-11)

### Day 1-3: Dashboard Metrics Module (Backend Lead)

**Tasks:**

- [ ] Create DashboardModule in NestJS
- [ ] Implement metrics aggregation service
- [ ] Create GraphQL resolver for dashboard metrics
- [ ] Implement Redis caching with 60s TTL
- [ ] Calculate today's metrics (revenue, orders, customers, AOV)
- [ ] Calculate % changes (vs yesterday)
- [ ] Fetch hourly trends (revenue by hour)
- [ ] Fetch top-selling items (quantity, revenue)
- [ ] Implement cross-location comparison query
- [ ] Write unit tests for calculations

**Deliverables:**

- `backend/src/modules/dashboard/dashboard.module.ts`
- GraphQL query: `locationMetrics(locationId)`
- GraphQL query: `crossLocationComparison(tenantId, startDate, endDate)`
- GraphQL query: `revenueTrends(locationId, startDate, endDate, groupBy)`
- Redis cache keys: `dashboard:{locationId}:{date}`

**Acceptance Criteria:**

- `locationMetrics` query returns accurate today's metrics
- Metrics calculated from `transactions` table with proper aggregations
- Redis cache hit rate >80% for repeated queries within 60s
- Yesterday comparison shows correct % change (positive/negative)
- Hourly trends show 24 data points (0-23 hours)
- Top items limited to top 5 by revenue
- Cross-location query supports multi-tenant scenarios
- Response time <200ms (cached), <500ms (uncached)

**SQL Aggregation Example:**

```sql
SELECT
  COALESCE(SUM(final_amount), 0) as today_revenue,
  COUNT(DISTINCT id) as today_orders,
  COUNT(DISTINCT customer_phone) as today_customers,
  COALESCE(AVG(final_amount), 0) as average_order_value
FROM transactions
WHERE location_id = $1
  AND DATE(transaction_date) = CURRENT_DATE
  AND status = 'COMPLETED'
  AND deleted_at IS NULL;
```

---

### Day 4-5: WebSocket Real-Time Updates (Backend Lead)

**Tasks:**

- [ ] Setup WebSocket gateway with Socket.io
- [ ] Create NotificationsModule
- [ ] Implement room management (join by `location_id`)
- [ ] Subscribe to Redis Pub/Sub for dashboard updates
- [ ] Broadcast dashboard metrics to connected clients
- [ ] Implement GraphQL subscription: `dashboardUpdates(locationId)`
- [ ] Implement GraphQL subscription: `newTransaction(locationId)`
- [ ] Test with multiple connected clients
- [ ] Implement reconnection handling

**Deliverables:**

- `backend/src/modules/notifications/notifications.gateway.ts`
- Socket.io WebSocket server on `/socket.io`
- GraphQL subscriptions endpoint (WebSocket upgrade)
- Room-based broadcasting by `location_id`

**Acceptance Criteria:**

- Client can connect to WebSocket at `ws://localhost:3000/socket.io`
- Client joins room automatically based on JWT user's locations
- When new transaction created → Broadcast to all clients in that location's room
- GraphQL subscription receives updates in real-time (<500ms latency)
- Dashboard metrics update automatically without polling
- Multiple clients receive same broadcast simultaneously
- Disconnected clients can reconnect and resume subscriptions

**WebSocket Event Flow:**

```
1. New transaction created (payment webhook or manual entry)
2. NestJS publishes to Redis: PUBLISH dashboard:loc-123 '{"event":"new_transaction",...}'
3. NotificationsGateway subscribed to Redis → Receives event
4. Gateway emits to Socket.io room: io.to("location:loc-123").emit("dashboard_update", data)
5. All connected clients in room receive event → Update UI
```

**GraphQL Subscription:**

```graphql
subscription DashboardLive($locationId: ID!) {
  dashboardUpdates(locationId: $locationId) {
    todayRevenue
    todayOrders
    todayCustomers
    averageOrderValue
  }
}
```

---

### Day 6-7: Push Notifications (FCM) (Backend Lead)

**Tasks:**

- [ ] Setup Firebase Admin SDK
- [ ] Create NotificationDevice entity (FCM tokens)
- [ ] Implement device registration mutation
- [ ] Create notification sending service
- [ ] Implement notification types (payment, recommendation, alert)
- [ ] Store notification history in database
- [ ] Create GraphQL queries for notification history
- [ ] Test push notifications on iOS + Android emulators

**Deliverables:**

- Firebase Admin SDK configured with service account
- GraphQL mutation: `registerDevice(fcmToken, platform)`
- `NotificationsService.sendPush(userId, title, body, data)`
- Database table: `notification_devices`, `notifications`
- GraphQL query: `notifications(isRead, limit, offset)`

**Acceptance Criteria:**

- Device registration stores FCM token in database
- Service can send push notification to single device
- Service can send push notification to all user's devices
- Notifications include Vietnamese title and body
- High-priority notifications use FCM priority flag
- Notification history persisted to database
- User can mark notification as read via GraphQL mutation
- Test notification successfully received on iOS simulator

**FCM Payload Example:**

```typescript
{
  notification: {
    title: 'Thanh toán mới',
    body: '50.000₫ - Cà phê sữa đá',
  },
  data: {
    type: 'payment',
    transactionId: 'txn-123',
    locationId: 'loc-456',
    amount: '50000',
  },
  apns: {
    payload: {
      aps: {
        sound: 'default',
        badge: 1,
      },
    },
  },
  android: {
    priority: 'high',
    notification: {
      sound: 'default',
      clickAction: 'FLUTTER_NOTIFICATION_CLICK',
    },
  },
}
```

---

## Week 4: Flutter Mobile App (Nov 12-18)

### Day 1-2: Project Setup & Authentication (Mobile Developer)

**Tasks:**

- [ ] Initialize Flutter project (iOS + Android)
- [ ] Setup folder structure (clean architecture)
- [ ] Install dependencies (Riverpod, Dio, GraphQL, Hive, FCM)
- [ ] Configure iOS + Android native settings
- [ ] Create API client with GraphQL configuration
- [ ] Implement phone OTP screens (request, verify)
- [ ] Implement biometric authentication wrapper
- [ ] Setup secure storage for tokens (Keychain, Keystore)
- [ ] Implement auto-login on app restart
- [ ] Write widget tests for auth screens

**Deliverables:**

- `lib/core/` with API client, storage, constants
- `lib/features/auth/` with OTP screens
- Biometric prompt (Face ID/Fingerprint)
- Token refresh logic on 401 errors
- Splash screen with auto-login check

**Acceptance Criteria:**

- User can enter phone number and request OTP
- OTP input screen with 6-digit boxes
- Correct OTP → JWT tokens stored securely → Navigate to dashboard
- Invalid OTP → Show Vietnamese error message
- Biometric prompt appears after first login
- App remembers user and auto-logins on restart
- Refresh token automatically renews access token before expiration
- All text in Vietnamese

**Screens:**

1. `PhoneInputScreen` - Phone number entry with Vietnamese validator
2. `OtpVerificationScreen` - 6-digit OTP input with countdown timer (120s)
3. `BiometricSetupScreen` - Optional biometric enrollment

---

### Day 3-5: Dashboard Implementation (Mobile Developer)

**Tasks:**

- [ ] Create DashboardScreen with AppBar + location switcher
- [ ] Implement metrics cards (revenue, orders, customers, AOV)
- [ ] Add change indicators (% vs yesterday) with color coding
- [ ] Create hourly trend chart (fl_chart line chart)
- [ ] Implement top items list with images
- [ ] Add AI recommendations preview (3 cards)
- [ ] Setup GraphQL query with Riverpod FutureProvider
- [ ] Implement pull-to-refresh
- [ ] Add shimmer loading effect
- [ ] Configure auto-refresh every 60s (when app in foreground)
- [ ] Write widget tests for dashboard components

**Deliverables:**

- `lib/features/dashboard/presentation/dashboard_screen.dart`
- GraphQL query hook for dashboard metrics
- Riverpod providers for state management
- Reusable metric card widget
- Chart widget with hourly revenue trend

**Acceptance Criteria:**

- Dashboard loads within 2s on 4G network
- Metrics display with proper Vietnamese formatting (₫, thousands separator)
- % change shows green (positive) or red (negative) with arrow icons
- Line chart renders smoothly with 24 data points
- Top items show item name, category, quantity, revenue, image
- Pull-to-refresh triggers API call and updates UI
- Auto-refresh every 60s when dashboard visible (pauses when app backgrounded)
- Shimmer effect shows during loading
- Error handling with retry button

**Dashboard Layout:**

```
┌─────────────────────────────────────┐
│ [☰] Cửa hàng ABC    [🔔] [👤]      │ AppBar
├─────────────────────────────────────┤
│ 📊 Hôm nay                          │
│ ┌─────────┐ ┌─────────┐            │
│ │50.000₫  │ │15 đơn   │            │ Metrics
│ │↑ +12%   │ │↓ -3%    │            │
│ └─────────┘ └─────────┘            │
│ ┌───────────────────────────────┐  │
│ │  📈 Doanh thu theo giờ        │  │ Chart
│ │  [Line chart 0h-23h]          │  │
│ └───────────────────────────────┘  │
│ 💡 Gợi ý AI (3)                    │
│ [Recommendation card 1]             │
│ [Recommendation card 2]             │
│ ...                                 │
│ 🏆 Món bán chạy                    │
│ [Top item 1]                        │
│ [Top item 2]                        │
│ ...                                 │
└─────────────────────────────────────┘
```

---

### Day 6: Recommendations Screen (Mobile Developer)

**Tasks:**

- [ ] Create RecommendationsScreen with filtering (type, priority)
- [ ] Implement recommendation card with priority badge
- [ ] Add approve/dismiss action buttons
- [ ] Show confirmation dialog on actions
- [ ] Implement GraphQL mutations (approve, dismiss)
- [ ] Add haptic feedback on button press
- [ ] Create detail modal for full recommendation view
- [ ] Update list after action (optimistic update)
- [ ] Write widget tests

**Deliverables:**

- `lib/features/recommendations/presentation/recommendations_screen.dart`
- GraphQL mutation hooks (approve, dismiss)
- Reusable recommendation card widget
- Filter dropdown (all, pricing, inventory, promotion)

**Acceptance Criteria:**

- Screen shows list of pending recommendations
- Cards display type icon, priority badge, title, description, impact
- Tapping card opens bottom sheet with full details + action JSON
- Approve button → Confirmation dialog → GraphQL mutation → Card removed from list
- Dismiss button → Reason input dialog → GraphQL mutation → Card removed from list
- Haptic feedback on approve/dismiss
- Optimistic UI update (immediate removal) with rollback on error
- Filter dropdown works (client-side filtering)

**Recommendation Card:**

```
┌─────────────────────────────────────┐
│ [HIGH] 🏷️ Giảm giá Cà phê sữa đá   │
│                                     │
│ Món này bán chậm (chỉ 8 phần       │
│ trong 7 ngày). Thử giảm giá        │
│ 10-15% để tăng doanh số.            │
│                                     │
│ 💰 Tăng 20-30% lượng bán           │
│                                     │
│ [✓ Áp dụng]  [✗ Bỏ qua]           │
└─────────────────────────────────────┘
```

---

### Day 7: Push Notifications & Polish (Mobile Developer)

**Tasks:**

- [ ] Configure Firebase Cloud Messaging (iOS + Android)
- [ ] Request notification permissions on app start
- [ ] Register FCM token via GraphQL mutation
- [ ] Implement foreground notification handler
- [ ] Implement background notification handler
- [ ] Handle notification tap → Navigate to relevant screen
- [ ] Show in-app notification banner (when app active)
- [ ] Add app icon badge count (iOS)
- [ ] Test notifications on physical devices
- [ ] Polish UI (spacing, colors, fonts, animations)
- [ ] Add loading states and error handling throughout
- [ ] Implement logout functionality
- [ ] Final testing on iOS + Android

**Deliverables:**

- Firebase configured in iOS (`GoogleService-Info.plist`) + Android (`google-services.json`)
- FCM service with token registration
- Notification handlers (foreground, background, terminated)
- Deep linking to dashboard on notification tap
- Polished UI matching design

**Acceptance Criteria:**

- App requests notification permission on first launch (iOS) or auto-granted (Android)
- FCM token registered to backend via GraphQL mutation
- Foreground notification shows as banner at top of app
- Background notification appears in notification center
- Tapping notification opens app and navigates to dashboard
- Payment notification shows amount and item name in Vietnamese
- App badge shows unread notification count (iOS)
- All screens have proper loading and error states
- UI matches Vietnamese design preferences (vibrant colors, clear hierarchy)
- App passes manual QA checklist

**FCM Setup:**

```dart
// Request permissions
await FirebaseMessaging.instance.requestPermission(
  alert: true,
  badge: true,
  sound: true,
  criticalAlert: true,
);

// Get token
final fcmToken = await FirebaseMessaging.instance.getToken();

// Register to backend
await apiClient.registerDevice(fcmToken: fcmToken, platform: 'ios');

// Foreground handler
FirebaseMessaging.onMessage.listen((RemoteMessage message) {
  showInAppNotification(message.notification?.title, message.notification?.body);
});

// Background handler
FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
```

---

## Week 5: Testing & Polish (Nov 19-25)

### Integration Testing (All)

**Tasks:**

- [ ] End-to-end test: Onboarding flow (OTP → Dashboard)
- [ ] End-to-end test: Recommendation generation → Approval
- [ ] End-to-end test: Real-time dashboard update
- [ ] End-to-end test: Push notification delivery
- [ ] Load testing: 100 concurrent dashboard requests
- [ ] Load testing: gRPC recommendation generation (10 locations in parallel)
- [ ] Security audit: SQL injection, XSS, JWT expiration
- [ ] Performance profiling: Identify slow queries (>1s)
- [ ] Fix critical bugs discovered during testing

**Deliverables:**

- E2E test suite passing (Supertest for NestJS, pytest for Python)
- Load test reports (Apache JMeter or k6)
- Security audit report with fixes applied
- Performance optimization PRs merged

**Acceptance Criteria:**

- All critical user flows work end-to-end
- API handles 100 concurrent requests without errors
- Average response time <500ms under load
- Zero SQL injection vulnerabilities
- JWT tokens properly validated and refreshed
- Database queries optimized with indexes

---

### Documentation & Deployment Prep (Backend Lead)

**Tasks:**

- [ ] Write API documentation (Swagger/OpenAPI)
- [ ] Update README with setup instructions
- [ ] Create Docker Compose production config
- [ ] Setup environment variables for production
- [ ] Create database backup strategy
- [ ] Write deployment runbook
- [ ] Setup monitoring (Prometheus + Grafana)
- [ ] Configure logging (ELK Stack or CloudWatch)
- [ ] Create demo tenant with realistic data
- [ ] - Prepare demo script for pilot customers

**Deliverables:**

- `docs/API.md` with all endpoints documented
- `README.md` with quick start guide
- `docker-compose.prod.yml` for production deployment
- `.env.production.example` with required variables
- `docs/DEPLOYMENT.md` runbook
- Monitoring dashboards configured
- Demo environment accessible at `https://demo.localstoreplatform.com`

**Acceptance Criteria:**

- API documentation covers all GraphQL queries/mutations/subscriptions
- New developer can setup project in <30 minutes following README
- Production Docker Compose runs successfully on staging server
- Monitoring shows API response times, error rates, database connections
- Logs are centralized and searchable
- Demo tenant has 7 days of realistic sales data
- Demo script highlights key features (onboarding, dashboard, AI recommendations)

---

## Definition of Done

### Code Quality

- [ ] All TypeScript/Python code passes linting (ESLint, Flake8)
- [ ] Code reviewed and approved by peer
- [ ] Unit tests written and passing (80%+ coverage for critical paths)
- [ ] Integration tests passing
- [ ] No TypeScript `any` types in production code
- [ ] Documentation updated (inline comments, README)

### Functionality

- [ ] Feature works on development environment
- [ ] Feature works on staging environment
- [ ] Feature tested on iOS and Android (mobile)
- [ ] Feature meets acceptance criteria
- [ ] Edge cases handled (errors, empty states, loading)
- [ ] Vietnamese language throughout

### Performance

- [ ] API response time <500ms (p95)
- [ ] Mobile app screen load <2s (4G network)
- [ ] Database queries optimized (no N+1, proper indexes)
- [ ] Images optimized and cached
- [ ] No memory leaks detected

### Security

- [ ] Authentication required for protected routes
- [ ] Multi-tenancy enforced (no data leakage)
- [ ] Input validation on all endpoints
- [ ] SQL injection prevented (parameterized queries)
- [ ] Secrets stored securely (env variables, not code)
- [ ] HTTPS enforced in production

---

## Risks & Mitigation

### Technical Risks

#### Risk 1: gRPC communication issues between NestJS and Python

- **Impact:** High (blocks AI recommendations)
- **Probability:** Medium
- **Mitigation:**
  - Test gRPC early (Day 4-5, Week 2)
  - Use gRPC reflection for debugging
  - Implement fallback: NestJS can generate basic recommendations if Python service unavailable
  - Monitor gRPC latency and error rates

#### Risk 2: WebSocket scaling challenges

- **Impact:** Medium (affects real-time updates)
- **Probability:** Low (not an issue for MVP scale)
- **Mitigation:**
  - Use Redis Pub/Sub for horizontal scaling
  - Implement connection limits per server
  - Test with 100 concurrent connections
  - Document scaling strategy for future

#### Risk 3: Database performance with complex aggregations

- **Impact:** Medium (slow dashboard)
- **Probability:** Medium
- **Mitigation:**
  - Create materialized views for analytics
  - Add database indexes on frequently queried columns
  - Implement aggressive caching (Redis 60s TTL)
  - Profile queries with `EXPLAIN ANALYZE`

#### Risk 4: FCM push notification delivery issues

- **Impact:** Medium (affects user engagement)
- **Probability:** Low
- **Mitigation:**
  - Test on physical devices early (Day 7, Week 4)
  - Implement retry logic for failed deliveries
  - Monitor FCM success rates via Firebase Console
  - Fallback: In-app notification banner always works

### Timeline Risks

#### Risk 5: Backend development takes longer than 2 weeks

- **Impact:** High (delays mobile development)
- **Probability:** Medium
- **Mitigation:**
  - Prioritize core features (auth, dashboard queries)
  - Mobile dev can start with mock API responses
  - Daily standups to identify blockers early
  - Consider reducing scope (e.g., defer cross-location comparison to Sprint 2)

#### Risk 6: Integration testing reveals critical bugs in Week 5

- **Impact:** High (delays launch)
- **Probability:** Medium
- **Mitigation:**
  - Write integration tests throughout (not just Week 5)
  - Manual QA testing in parallel with development
  - Maintain bug backlog and prioritize by severity
  - Accept minor bugs if non-blocking for MVP

---

## Team Communication

### Daily Standup (15 minutes, 10 AM VN time)

- What did you complete yesterday?
- What will you work on today?
- Any blockers or dependencies?

### Weekly Review (1 hour, Friday 4 PM VN time)

- Demo completed features
- Review sprint progress vs plan
- Discuss technical challenges
- Plan next week's priorities

### Tools

- **Slack:** Real-time communication (#local-store-platform)
- **GitHub:** Code reviews, issue tracking, PRs
- **Notion:** Documentation, meeting notes
- **Figma:** Design references (mobile screens)

---

## Success Metrics (Sprint 1)

### Development Velocity

- Sprint completion rate: 90%+ tasks completed
- Code review turnaround: <4 hours
- Bug fix time: Critical <24h, Medium <3 days

### Technical Metrics

- API uptime: 99%+ (dev environment)
- Average API response time: <300ms
- Test coverage: 80%+ (critical paths)
- Build success rate: 95%+

### User Experience (Demo Testing)

- Onboarding completion rate: 100% (all demo users)
- Dashboard load time: <2s
- Recommendation accuracy: 70%+ deemed useful (subjective)
- Push notification delivery rate: 95%+

---

## Post-Sprint 1: Sprint 2 Preview

### Planned Features (Nov 26 - Dec 23)

- [ ] Manual sales entry (web portal)
- [ ] Payment gateway integration (MoMo, ZaloPay webhooks)
- [ ] Advanced analytics (revenue trends, forecasting)
- [ ] Menu management (CRUD operations, image upload)
- [ ] User management (invite staff, role permissions)
- [ ] Subscription billing (Stripe integration)
- [ ] Multi-language support (Vietnamese + English)
- [ ] Performance optimizations (CDN, database sharding)

---

## Getting Started

### Prerequisites

- Node.js 20+
- Python 3.11+
- Docker + Docker Compose
- PostgreSQL 14+ (or use Docker)
- Redis 6+ (or use Docker)
- Flutter 3.x (for mobile development)
- iOS/Android development environment

### Setup Instructions

```bash
# Clone repository
git clone https://github.com/your-org/local-store-platform.git
cd local-store-platform

# Start backend services with Docker
docker-compose up -d

# Verify services are running
curl http://localhost:3000/health  # NestJS
curl http://localhost:5000/health  # Python AI

# Setup database (run migrations)
cd backend
npm run migration:run

# Seed test data
npm run seed

# Start mobile app (separate terminal)
cd ../mobile
flutter pub get
flutter run
```

### Verify Setup

1. Open `http://localhost:3000/graphql` → GraphQL Playground loads
2. Request OTP for phone `0901234567` → SMS sent (logged in console)
3. Verify OTP `123456` → Receive JWT tokens
4. Query `me` → Returns user + tenant data
5. Mobile app: Login with same phone → Dashboard shows test data

---

Ready to start Sprint 1! 🚀

**Next Actions:**

1. Kick off meeting (Oct 22, 10 AM) - Review sprint plan, assign initial tasks
2. Backend Lead: Initialize NestJS project
3. Data Scientist: Initialize Python AI service project
4. Mobile Developer: Initialize Flutter project
5. Daily standups at 10 AM VN time

Good luck! 💪
