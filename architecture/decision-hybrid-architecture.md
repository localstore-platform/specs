# Architecture Decision Record: Hybrid NestJS + Python Backend

**Date:** October 22, 2025  
**Status:** Accepted  
**Deciders:** Technical Team, Product Lead

---

## Context and Problem Statement

We need to select a backend architecture that supports:

1. **Real-time features:** Dashboard updates, payment webhooks, multi-device sync (<500ms latency)
2. **AI/ML capabilities:** Recommendation engine, demand forecasting, pricing optimization
3. **Vietnamese market:** Local developer availability, rapid iteration, cost-effectiveness
4. **Scalability:** Multi-tenant SaaS supporting 100+ restaurants initially, 10k+ long-term
5. **Multi-platform clients:** Flutter mobile (iOS/Android), Next.js web portal

## Decision Drivers

- **Performance:** API response times <500ms (p95), real-time updates <100ms
- **Developer Productivity:** Fast prototyping, strong typing, Vietnamese talent pool
- **AI/ML Ecosystem:** Access to mature ML libraries (scikit-learn, pandas, Prophet)
- **Cost Efficiency:** Minimize infrastructure costs during MVP phase (<$500/month)
- **Future-Proofing:** Easy to add advanced ML (PyTorch, TensorFlow) later
- **Communication:** Flexible client APIs (mobile needs minimal data, web needs rich data)

---

## Considered Options

### Option 1: NestJS (TypeScript) Only

**Pros:**

- Full-stack TypeScript (type safety across frontend/backend)
- Built-in GraphQL + WebSocket support
- Fast iteration with decorators and CLI
- Large Vietnamese developer community
- Rich ecosystem (auth, validation, caching, queues)

**Cons:**

- ML libraries immature (TensorFlow.js limited compared to Python)
- Node.js single-threaded (not ideal for CPU-intensive ML)
- No native data science libraries (pandas equivalent lacking)

**Verdict:** ❌ Strong for API but weak for AI/ML

---

### Option 2: Go (Golang) Only

**Pros:**

- Blazing fast (10x faster than Node.js)
- Low memory footprint (20-50MB vs 200-500MB)
- Built-in concurrency (goroutines)
- gRPC native support
- Great for microservices

**Cons:**

- GraphQL ecosystem less mature (gqlgen vs Apollo Server)
- Smaller Vietnamese developer pool (harder to hire)
- ML libraries very limited (no equivalent to scikit-learn)
- Slower development velocity (more boilerplate)

**Verdict:** ❌ Excellent performance but weak ecosystem for our use case

---

### Option 3: Python (FastAPI) Only

**Pros:**

- Best ML ecosystem (scikit-learn, pandas, Prophet, TensorFlow, PyTorch)
- Single codebase (API + AI in one language)
- Fast for prototyping
- Great for data processing

**Cons:**

- Slower than Node.js/Go (5k req/sec vs 10k-100k)
- GIL limits true parallelism (CPU-bound tasks)
- GraphQL less mature (Strawberry/Graphene vs Apollo Server)
- WebSocket support not as robust as Socket.io
- Type hints not as strong as TypeScript

**Verdict:** ❌ Strong for ML but weak for real-time client APIs

---

### Option 4: Hybrid - NestJS + Python (SELECTED ✅)

**Architecture:**

- **NestJS API Gateway:** Client-facing APIs (GraphQL, WebSocket, REST)
- **Python AI Service:** ML models, recommendation engine, data processing
- **Communication:** gRPC (binary, streaming, 10x faster than REST JSON)

**Pros:**

- ✅ **Best of both worlds:** Fast APIs + powerful ML
- ✅ **Separation of concerns:** API logic separate from ML logic
- ✅ **Independent scaling:** Scale NestJS and Python separately
- ✅ **Language-specific strengths:** TypeScript for APIs, Python for data science
- ✅ **Vietnamese talent:** Easy to find both Node.js and Python developers
- ✅ **Future-proof:** Can add more microservices (e.g., Go for analytics)
- ✅ **gRPC efficiency:** 10x faster than REST for backend-to-backend communication

**Cons:**

- ⚠️ **Complexity:** Two services to maintain (but isolated failures)
- ⚠️ **DevOps overhead:** More containers to deploy (mitigated by Docker Compose)
- ⚠️ **Network latency:** gRPC calls add 10-50ms (acceptable for our use case)

**Verdict:** ✅ **SELECTED - Optimal trade-offs for our requirements**

---

## Decision Outcome

### Chosen Option: Hybrid - NestJS + Python

**Why this works for us:**

1. **Real-Time Requirements Met:**
   - NestJS handles GraphQL + WebSocket natively (Apollo Server + Socket.io)
   - Python AI service focuses on ML, not blocking client requests
   - gRPC streaming keeps AI responses fast (<150ms)

2. **AI/ML Capabilities Unlocked:**
   - Python's ecosystem is unmatched (scikit-learn, pandas, Prophet, statsmodels)
   - Easy to add advanced ML later (PyTorch, TensorFlow, Transformers)
   - Data scientists can work in Jupyter notebooks → Production services seamlessly

3. **Vietnamese Developer Availability:**
   - Node.js/TypeScript: Very common in Vietnam (easier to hire)
   - Python: Standard for data science roles
   - Full-stack TypeScript developers can maintain NestJS
   - Data scientists maintain Python AI service (clear ownership)

4. **Cost-Effective Scaling:**
   - Start small: 1 NestJS instance ($24/mo) + 1 Python instance ($48/mo) = $72/mo
   - Scale independently: Add more NestJS instances for API traffic, Python instances for ML workload
   - Total initial infrastructure: ~$150/mo (vs $100-200/mo for single-service)

5. **Proven Pattern:**
   - Netflix, Uber, Spotify use similar hybrid architectures
   - gRPC is industry standard for microservice communication
   - Mature tooling (Docker, Kubernetes, Prometheus)

---

## Communication Pattern

### Client → NestJS (GraphQL + WebSocket)

```
Flutter/Next.js --HTTPS--> NestJS API Gateway
  - GraphQL queries (flexible data fetching)
  - GraphQL mutations (write operations)
  - WebSocket subscriptions (real-time updates)
```

**Why GraphQL?**

- Mobile needs minimal data (battery, bandwidth)
- Web needs rich data (detailed analytics)
- Single endpoint, no versioning hell

**Example:**

```graphql
# Mobile (minimal)
query { location(id: "123") { todayRevenue todayOrders } }

# Web (rich)
query { location(id: "123") { todayRevenue todayOrders hourlyTrends { hour revenue } topItems { name quantity } } }
```

### NestJS → Python AI (gRPC)

```
NestJS --gRPC--> Python AI Service
  - GetRecommendations(location_id, date_range)
  - PredictDemand(item_id, forecast_days)
  - AnalyzePricing(competitor_prices)
```

**Why gRPC?**

- 10x faster than REST JSON (binary protocol)
- Streaming support (bidirectional)
- Type-safe (protobuf definitions)
- Efficient for large data transfers (millions of sales records)

**Performance:**

- REST JSON: 500ms to transfer 1MB sales data
- gRPC Protobuf: 50ms for same data (10x faster)

---

## Implementation Details

### NestJS API Gateway Responsibilities

- Authentication (JWT, phone OTP)
- Multi-tenancy enforcement (tenant_id scoping)
- GraphQL server (queries, mutations, subscriptions)
- WebSocket server (real-time broadcasting)
- REST endpoints (payment webhooks, file uploads)
- Redis caching (dashboard metrics 60s TTL)
- Bull queue (background jobs)
- TypeORM (PostgreSQL database access)
- gRPC client (calls to Python AI service)

### Python AI Service Responsibilities

- gRPC server (ML predictions)
- Recommendation engine (rule-based → ML-based)
- Demand forecasting (Prophet, ARIMA, seasonal trends)
- Pricing optimization (dynamic pricing rules)
- Anomaly detection (Isolation Forest for outliers)
- Data pipeline (ETL, feature engineering, pandas/numpy)
- Celery workers (model training, batch processing)
- asyncpg (direct PostgreSQL access for large queries)

### Technology Stack Summary

| Component | Technology | Reason |
|-----------|-----------|--------|
| API Gateway | NestJS 10 (TypeScript) | Built-in GraphQL, WebSocket, Vietnamese devs |
| AI Service | Python 3.11 + FastAPI | ML ecosystem (scikit-learn, pandas, Prophet) |
| Communication | gRPC (protobuf) | 10x faster than REST, streaming support |
| Database | PostgreSQL 14+ | ACID, multi-tenant RLS, mature |
| Cache | Redis 6+ | Fast, pub/sub for WebSocket, Bull queue |
| Client APIs | GraphQL + WebSocket | Flexible queries, real-time updates |
| Mobile | Flutter 3.x | Single codebase (iOS + Android) |
| Web | Next.js 14 | React + TypeScript, Vercel deployment |

---

## Risks and Mitigations

### Risk 1: gRPC Communication Complexity

**Mitigation:**

- Use gRPC reflection for debugging
- Implement health checks on both services
- Add circuit breaker pattern (fallback to basic recommendations if Python service down)
- Monitor gRPC latency with Prometheus

### Risk 2: Two Services to Maintain

**Mitigation:**

- Docker Compose for local development (single `docker-compose up`)
- Kubernetes for production (auto-scaling, health checks, rollbacks)
- Clear service boundaries (API vs ML logic)
- Separate teams (backend devs vs data scientists)

### Risk 3: Network Latency (NestJS ↔ Python)

**Mitigation:**

- Co-locate services (same datacenter, <1ms network latency)
- Use gRPC streaming (not request/response for large data)
- Cache AI recommendations in NestJS (5min TTL)
- Async processing (don't block client requests on AI calls)

### Risk 4: Cost (Running Two Services)

**Mitigation:**

- Share PostgreSQL and Redis (no duplication)
- Python service only scales for ML workload (not API traffic)
- Start with minimal resources (2 CPU, 4GB RAM each)
- Optimize later (add CDN, database read replicas, connection pooling)

---

## Performance Benchmarks (Expected)

### API Response Times (p95)

- GraphQL queries (cached): <100ms
- GraphQL queries (uncached): <300ms
- GraphQL mutations: <200ms
- WebSocket message delivery: <50ms
- gRPC calls (NestJS → Python): <100ms

### Throughput

- NestJS API Gateway: 10,000 req/sec (single instance)
- Python AI Service: 1,000 recommendations/sec (single instance)
- PostgreSQL: 5,000 queries/sec (properly indexed)

### Resource Usage (Single Instance)

- NestJS: 2 CPU, 4GB RAM (DigitalOcean $24/mo)
- Python AI: 4 CPU, 8GB RAM (ML workloads, $48/mo)
- PostgreSQL: 4 CPU, 8GB RAM ($48/mo)
- Redis: 2 CPU, 2GB RAM ($12/mo)
- **Total:** ~$150/mo for <1000 users

---

## Alternatives Considered and Rejected

### GraphQL Federation (Apollo Federation)

**Why rejected:** Over-engineering for MVP. Federation adds complexity (gateway, schema stitching) without clear benefit. We only have 2 services, not 10+. Simpler to use gRPC for backend-to-backend and GraphQL for client-to-backend.

### Kafka/RabbitMQ for Communication

**Why rejected:** Overkill for our message volume. Redis Pub/Sub + gRPC are sufficient. Event streaming adds operational complexity (broker management, topic config, consumer groups). Can add later if needed for event sourcing.

### Monolithic Architecture (All in NestJS)

**Why rejected:** ML in Node.js is painful. TensorFlow.js is immature, no pandas equivalent, single-threaded. Would block API requests during ML processing. Separating concerns (API vs ML) is cleaner.

### Serverless Functions (AWS Lambda)

**Why rejected:** Cold starts (500-2000ms) unacceptable for real-time requirements. WebSocket support limited. gRPC not well-supported. State management complex. Better for event-driven workloads, not API servers.

---

## Validation and Success Metrics

### Technical Metrics (Sprint 1)

- [ ] API response time <500ms (p95)
- [ ] gRPC latency <150ms (average)
- [ ] WebSocket message delivery <100ms
- [ ] Dashboard cache hit rate >80%
- [ ] Zero data leakage (multi-tenancy enforced)
- [ ] 80%+ test coverage (critical paths)

### Business Metrics (MVP Launch)

- [ ] Onboarding completion <5 minutes
- [ ] Dashboard load time <2s (4G mobile)
- [ ] AI recommendation accuracy >70% (user approval rate)
- [ ] Push notification delivery rate >95%
- [ ] System uptime >99.5%

### Developer Experience Metrics

- [ ] New developer setup time <30 minutes
- [ ] Code review turnaround <4 hours
- [ ] Build success rate >95%
- [ ] Hot reload working for both services

---

## Decision Review Date

**Next Review:** March 2026 (after 1000+ active users)

**Trigger for Re-evaluation:**

- API response times consistently >500ms (need Go rewrite?)
- Python AI service becomes bottleneck (need horizontal scaling?)
- gRPC complexity hurts developer productivity (simplify to REST?)
- Cost exceeds budget (optimize or consolidate services?)

---

## References

- [Backend Setup Guide](./backend-setup-guide.md)
- [GraphQL Schema](./graphql-schema.md)
- [System Architecture](./system-diagram.md)
- [Sprint 1 Plan](../planning/sprint-1-implementation.md)

---

## Appendix: Real-World Examples

### Netflix (Hybrid Polyglot)

- Java for API gateway (high throughput)
- Python for ML recommendations (scikit-learn, TensorFlow)
- gRPC for service-to-service communication

### Uber (Microservices)

- Go for high-performance services (routing, matching)
- Python for data science (surge pricing, ETA prediction)
- Node.js for real-time features (driver tracking)

### Spotify (Similar to Our Approach)

- Java/Scala for API backend
- Python for ML pipeline (music recommendations)
- gRPC + REST for communication

**Lesson:** Hybrid architectures are proven at scale. Use the right tool for each job.

---

**Decision Status:** ✅ **ACCEPTED**  
**Implementation Start:** October 22, 2025  
**Expected Completion:** November 19, 2025 (Sprint 1)
