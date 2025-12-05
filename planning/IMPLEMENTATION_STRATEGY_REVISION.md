# Implementation Strategy Revision

**Date:** 2025-11-25  
**Reason:** Revised approach to prioritize demo-first development

---

## Key Changes

### 🎯 New Development Strategy

**OLD Strategy (Infrastructure-first):**

```
AWS Setup → Backend API → Mobile App → ML Service → Launch
```

**NEW Strategy (Demo-first):**

```
Local Docker → Menu Web Demo → Backend API → Mobile App → AWS Deploy
```

### 📋 Revised Priorities

| Priority | Component | Goal | Timeline |
|----------|-----------|------|----------|
| **1** | 🐳 Local Dev Setup | Docker Compose localhost | Week 1 |
| **1** | 🌐 Menu Web (Demo) | Customer-facing menu website | Week 1-2 |
| **2** | 🔧 Backend API | Menu CRUD endpoints | Week 3-4 |
| **3** | 📱 Mobile App | Owner menu management | Week 5-7 |
| **4** | 📊 Basic Analytics | View tracking, simple charts | Week 8-9 |
| **5** | 🏗️ AWS Infrastructure | Production deployment | Week 10-12 |
| **Future** | 🤖 ML Service | AI recommendations (when có data) | Post-MVP |

### 🎯 Phase 1 Focus: Menu Web Demo (Week 1-2)

**Goal:** Tạo menu website đơn giản để demo cho merchants

**Components:**

1. Docker Compose setup (PostgreSQL + Redis + Backend)
2. Backend API: `GET /public/menu/:slug` endpoint
3. Database: `menus`, `menu_items`, `locations` tables
4. Seed data: 2-3 Vietnamese menu examples
5. Next.js menu web: Responsive, Vietnamese formatting
6. QR code generation

**Success Criteria:**

- ✅ Có thể scan QR code → xem menu đẹp trên điện thoại
- ✅ Menu hiển thị giá VND format (75.000₫)
- ✅ Mobile-first responsive design
- ✅ Fast loading (< 2s)

**Postponed Items:**

- ❌ AWS infrastructure (use localhost)
- ❌ OTP authentication (use simple API key)
- ❌ ML Service (chưa có data)
- ❌ Mobile app (focus on web demo first)
- ❌ Analytics (add sau khi có demo)

### 🔄 Migration Path

**From local dev → production:**

1. **Week 1-2:** Develop on localhost
   - Docker Compose: `docker-compose up`
   - Access: `http://localhost:3000/menu/pho-ha-noi-24`

2. **Week 10:** Deploy to AWS
   - Setup EC2 + Docker
   - Domain: `https://menu.quanly.ai/pho-ha-noi-24`
   - Same codebase, different environment

### 💰 Cost Impact

**Development Phase (Week 1-9):**

- Cost: $0 (localhost only)
- Infrastructure: None

**Production Phase (Week 10+):**

- Cost: ~$20/month (AWS t2.small)
- Infrastructure: EC2 + RDS + CloudFront (optional)

### 📊 Updated Component Status

| Component | Old Priority | New Priority | Status |
|-----------|-------------|--------------|---------|
| AWS Infrastructure | 1 (first) | 5 (last) | ⏸️ Paused |
| Menu Web Demo | N/A (missing) | 1 (first) | 🔴 Not Started |
| Local Dev Setup | N/A | 1 (first) | 🔴 Not Started |
| Backend API | 2 | 2 | 🔴 Not Started |
| Mobile App | 3 | 3 | 🔴 Not Started |
| ML Service | 4 | Future | ⏸️ Paused |

### 🎓 Lessons Applied

**Why demo-first approach:**

1. **Faster feedback:** Show working product to merchants trong 2 tuần
2. **Lower risk:** Validate idea trước khi invest vào infrastructure
3. **Cost efficient:** $0 development cost, deploy khi đã validated
4. **Simpler stack:** Docker Compose đơn giản hơn AWS cho development
5. **Focus:** 1 goal rõ ràng (menu web) thay vì multiple components

**Reference:** Similar to how Basecamp, GitHub started - simple localhost demo first, scale later.

---

## Updated Timeline

### Phase 1: Menu Web Demo (Week 1-2) ← **CURRENT FOCUS**

- Setup Docker Compose local environment
- Build menu display website (Next.js)
- Create sample Vietnamese menus
- Generate QR codes
- **Deliverable:** Working demo on localhost

### Phase 2: Menu Management API (Week 3-4)

- Add CRUD endpoints for menu management
- Image upload functionality
- Simple API key authentication
- **Deliverable:** Can create/update menus via API

### Phase 3: Mobile Owner App (Week 5-7)

- Flutter app for shop owners
- OTP authentication
- Menu management screens
- **Deliverable:** Owners can manage menus from phone

### Phase 4: Basic Analytics (Week 8-9)

- Track menu views
- Simple dashboard (total views, popular items)
- **Deliverable:** Owners see value (view counts)

### Phase 5: Production Deploy (Week 10-12)

- AWS infrastructure setup
- Deploy to production
- Onboard 3-5 pilot shops
- **Deliverable:** Live platform with real users

### Future Phases (Post-MVP)

- AI recommendations (when có enough data)
- Payment integrations (MoMo, ZaloPay)
- Order management
- Multi-location support

---

## Action Items

### Immediate Next Steps

1. **Create `backend-api` repo:**
   - Initialize NestJS project
   - Setup Docker Compose
   - Create docs/SPEC_LINKS.md referencing v1.1-specs

2. **Create `menu-web` repo:**
   - Initialize Next.js 14 project
   - Setup Tailwind CSS
   - Create docs/SPEC_LINKS.md

3. **Update specs if needed:**
   - Add menu web specific wireframes
   - Define public menu API contract
   - Document QR code format

4. **Start development:**
   - Week 1, Day 1: Docker Compose setup
   - Week 1, Day 2-3: Database schema + seed data
   - Week 1, Day 4-5: Menu display API endpoint
   - Week 2, Day 1-3: Next.js menu pages
   - Week 2, Day 4-5: QR code + polish

---

## References

**Original Plan:** See [IMPLEMENTATION_PROGRESS.md](IMPLEMENTATION_PROGRESS.md)  
**Spec Version:** v1.1-specs  
**Decision Date:** 2025-11-25

**Related Documents:**

- `planning/implementation-roadmap.md` - Original roadmap
- `planning/sprint-1-implementation.md` - Will be updated
- `architecture/backend-setup-guide.md` - Dev environment setup
- `design/wireframes-ux-flow.md` - Menu UI designs
