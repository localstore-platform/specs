# Implementation Roadmap

**Strategic Context:** See [product-strategy-refined.md](./product-strategy-refined.md) for complete business strategy  
**Target:** Launch MVP for 10 pilot customers (5 chains + 5 skilled operators) in 12 weeks  
**Last Updated:** October 22, 2025

---

## Overview

**What We're Building:** AI business intelligence platform for Vietnamese restaurant operators

**Focus Areas:**

1. **Multi-location infrastructure** (database, APIs, UI)
2. **AI-powered analytics dashboard** (core value proposition)
3. **Data collection tools** (menu management, manual entry)
4. **Basic storefront** (data collection endpoint only)

**Timeline:** 12 weeks (3 sprints × 4 weeks)  
**Team:** 2 backend, 1 frontend, 0.5 product manager, 0.5 designer  
**Budget:** Estimated $30-40k total cost

---

## Sprint 1: Foundation (Weeks 1-4)

**Goal:** Multi-tenant + multi-location infrastructure ready for chains

### Week 1: Database Schema & Backend Setup

**Backend Tasks:**

- [ ] **Database migration for multi-location support**
  - Add `locations` table (see `database-schema.md`)
  - Update `tenants` table with `is_chain`, `total_locations`
  - Update `menu_items` with `location_id`, `is_chain_wide`
  - Add analytics tables (see `database-schema-analytics-extension.md`)
  - Set up Row-Level Security policies for location scoping
  - **Estimate:** 3 days
  - **Owner:** Backend Lead

- [ ] **Location CRUD APIs**
  - `POST /api/tenants/:id/locations` - Create location
  - `GET /api/tenants/:id/locations` - List locations
  - `GET /api/tenants/:id/locations/:locationId` - Get location details
  - `PATCH /api/tenants/:id/locations/:locationId` - Update location
  - `DELETE /api/tenants/:id/locations/:locationId` - Soft delete location
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 1

- [ ] **Location selection middleware**
  - Extract `location_id` from request headers/query params
  - Validate user has access to location (belongs to same tenant)
  - Inject `location_id` into request context
  - Default to primary location if not specified
  - **Estimate:** 1 day
  - **Owner:** Backend Dev 1

- [ ] **Update existing APIs for location scoping**
  - Menu items: filter by `location_id` or show chain-wide
  - Categories: support location-specific categories
  - Ensure backward compatibility for single-location tenants
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 2

**Infrastructure:**

- [ ] Set up staging environment with PostgreSQL 14+
- [ ] Configure Redis for session storage
- [ ] Set up S3-compatible storage (AWS S3 or DigitalOcean Spaces)
- [ ] Docker Compose for local development
- **Estimate:** 1 day
- **Owner:** DevOps/Backend Lead

**Deliverables:**

- ✅ Database schema supports multiple locations
- ✅ APIs accept `location_id` parameter
- ✅ RLS policies enforce location isolation
- ✅ Local dev environment running

---

### Week 2: Frontend Foundation & Location UI

**Frontend Tasks:**

- [ ] **Location selector component**
  - Dropdown in header showing all tenant locations
  - Special option: "Tất cả địa điểm" (All Locations)
  - Persist selected location in session/localStorage
  - Update all API calls to include `location_id`
  - **Estimate:** 2 days
  - **Owner:** Frontend Dev

- [ ] **Location management screens**
  - Location list page with cards (name, address, status, manager)
  - Location create/edit form (see MVP acceptance criteria Feature 2.0)
  - Address autocomplete using Google Maps API
  - Operating hours configuration (per-day JSON editor)
  - **Estimate:** 3 days
  - **Owner:** Frontend Dev

- [ ] **Onboarding flow update**
  - Add "Bạn có nhiều địa điểm kinh doanh không?" question after business type
  - If YES → show location setup wizard
  - If NO → create default location automatically
  - **Estimate:** 1 day
  - **Owner:** Frontend Dev

**Design:**

- [ ] Wireframes for location selector (mobile + desktop)
- [ ] Wireframes for location management screens
- [ ] Design system: location status badges (active/closed)
- **Estimate:** 2 days
- **Owner:** Designer

**Deliverables:**

- ✅ Users can add/edit multiple locations
- ✅ Location selector works across all pages
- ✅ Onboarding detects chains vs single-location

---

### Week 3-4: Analytics Infrastructure & Basic Dashboard

**Backend Tasks:**

- [ ] **Analytics event tracking service**
  - `POST /api/analytics/track` endpoint
  - Insert events into `analytics_events` table
  - Include location_id, device context, weather (optional for MVP)
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 2

- [ ] **Manual sales entry API** (for MVP without POS)
  - `POST /api/tenants/:id/locations/:locationId/sales` - Log daily sales
  - Store as `order_completed` events in analytics_events
  - **Estimate:** 1 day
  - **Owner:** Backend Dev 2

- [ ] **Daily metrics aggregation worker**
  - Cron job runs at 1 AM daily
  - Aggregate yesterday's events into `daily_metrics` table
  - Calculate revenue, orders, AOV, top items
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 2

- [ ] **Dashboard API endpoints**
  - `GET /api/analytics/dashboard/summary?location_id=X&date=today`
  - `GET /api/analytics/dashboard/locations-comparison?date=today`
  - `GET /api/analytics/dashboard/top-items?location_id=X&period=7d`
  - Return data from `daily_metrics` (fast queries)
  - **Estimate:** 3 days
  - **Owner:** Backend Dev 1

**Frontend Tasks:**

- [ ] **Analytics dashboard homepage** (MVP version)
  - Summary cards: revenue, orders, AOV (with % change from yesterday)
  - Top 5 selling items list
  - Low stock alerts section
  - 7-day revenue chart (line chart)
  - **Estimate:** 3 days
  - **Owner:** Frontend Dev

- [ ] **Cross-location comparison view** (for chains)
  - Table showing all locations side-by-side
  - Metrics: revenue, orders, AOV, top item
  - Status indicators (🟢 good, 🟡 average, 🔴 needs attention)
  - Sort by any column
  - **Estimate:** 2 days
  - **Owner:** Frontend Dev

- [ ] **Manual sales entry form**
  - Simple form: "Hôm nay bán được bao nhiêu?" (How much did you sell today?)
  - Input: total revenue, number of orders (optional: per-item quantities)
  - Submit → creates analytics events
  - **Estimate:** 1 day
  - **Owner:** Frontend Dev

**Deliverables:**

- ✅ Basic analytics dashboard showing today's performance
- ✅ Cross-location comparison for chains
- ✅ Manual sales entry working (for MVP without POS)
- ✅ 7-day revenue chart

---

## Sprint 2: AI Recommendations & Menu Management (Weeks 5-8)

**Goal:** Basic AI insights working, location-aware menu CRUD

### Week 5: Rule-Based AI Recommendations (MVP)

**Backend Tasks:**

- [ ] **AI recommendation engine** (rule-based for MVP)
  - Detect slow-moving items (< 2 sales in 3 days)
  - Detect out-of-stock items
  - Identify performance gaps between locations
  - Flag unusual drops in revenue (>20% decline)
  - **Estimate:** 3 days
  - **Owner:** Backend Dev 2

- [ ] **Recommendations API**
  - `GET /api/analytics/recommendations?location_id=X`
  - Return array of recommendations with:
    - Type (slow_moving, stock_alert, performance_gap)
    - Priority (high, medium, low)
    - Message (Vietnamese)
    - Actionable (boolean)
    - Actions (["View details", "Create promotion"])
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 2

**Frontend Tasks:**

- [ ] **AI Recommendations section on dashboard**
  - Card-based layout with priority badges
  - Action buttons for each recommendation
  - Dismiss/snooze functionality
  - Notification badge when new recommendations available
  - **Estimate:** 2 days
  - **Owner:** Frontend Dev

**Content:**

- [ ] Write Vietnamese recommendation templates
  - "Món [X] ế ẩm (chỉ bán [N] phần trong 3 ngày). Thử giảm giá 20%?"
  - "Địa điểm [A] bán được nhiều [món] hơn 2 lần so với [B]. Xem xét chiến lược tại [B]?"
  - "Hết hàng: [món] tại [địa điểm]. Cần nhập thêm."
  - **Estimate:** 1 day
  - **Owner:** Product Manager

**Deliverables:**

- ✅ 3-5 types of AI recommendations working
- ✅ Recommendations display on dashboard
- ✅ Vietnamese messaging for all recommendation types

---

### Week 6-7: Location-Aware Menu Management

**Backend Tasks:**

- [ ] **Update menu APIs for location scoping**
  - Add `location_id` filter to all menu queries
  - Support `is_chain_wide` flag (show at all locations vs specific)
  - Allow location-specific pricing (same item, different price per location)
  - **Estimate:** 3 days
  - **Owner:** Backend Dev 1

- [ ] **Location inventory tracking**
  - Separate stock quantities per location
  - Low stock alerts per location
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 1

**Frontend Tasks:**

- [ ] **Menu management with location context**
  - Location selector on menu pages
  - Toggle: "Áp dụng cho tất cả địa điểm" (Apply to all locations)
  - Show which locations have this item
  - Location-specific price overrides
  - **Estimate:** 3 days
  - **Owner:** Frontend Dev

- [ ] **Inventory management per location**
  - Stock levels shown per location
  - Bulk update across locations
  - Low stock warnings per location
  - **Estimate:** 2 days
  - **Owner:** Frontend Dev

**Deliverables:**

- ✅ Chains can create chain-wide or location-specific menu items
- ✅ Inventory tracked separately per location
- ✅ Location-specific pricing supported

---

### Week 8: Subscription Tiers & Upgrade Flow

**Backend Tasks:**

- [ ] **Update subscription plans**
  - Free: 1 location, 20 items, no AI
  - Growth (₫500k/month): 3 locations, unlimited items, basic AI
  - Pro (₫1.5M/month): 10 locations, advanced AI, consultant calls
  - **Estimate:** 1 day
  - **Owner:** Backend Dev 1

- [ ] **Subscription enforcement**
  - Block adding location #2 if Free tier
  - Show upgrade prompt when limit reached
  - **Estimate:** 1 day
  - **Owner:** Backend Dev 1

**Frontend Tasks:**

- [ ] **Pricing page redesign**
  - Emphasize multi-location + AI value
  - Comparison table (Free vs Growth vs Pro)
  - Highlight "Best for chains" badge on Growth/Pro
  - **Estimate:** 2 days
  - **Owner:** Frontend Dev

- [ ] **Upgrade flow (placeholder for MVP)**
  - Show modal: "Tính năng thanh toán đang phát triển. Liên hệ support@..."
  - Capture intent (which plan, contact info)
  - Send email to sales team
  - **Estimate:** 1 day
  - **Owner:** Frontend Dev

**Deliverables:**

- ✅ Subscription tiers reflect chain-focused pricing
- ✅ Free tier limited to 1 location
- ✅ Upgrade flow captures intent (payment integration future phase)

---

## Sprint 3: Polish, Testing & Launch Prep (Weeks 9-12)

**Goal:** Production-ready MVP, pilot customer onboarding

### Week 9: Storefront & Publishing (Minimal MVP)

**Backend Tasks:**

- [ ] **Update storefront API for location context**
  - Public API: `GET /api/public/storefronts/:slug?location=X`
  - If chain, show location selector on storefront
  - Menu items filtered by location
  - **Estimate:** 2 days
  - **Owner:** Backend Dev 1

**Frontend Tasks:**

- [ ] **Basic storefront template**
  - Mobile-first design
  - Location selector (if chain)
  - Menu items grouped by category
  - Prices in VND format
  - NO online ordering (not MVP priority)
  - **Estimate:** 2 days
  - **Owner:** Frontend Dev

- [ ] **QR code generation per location**
  - Generate QR code pointing to storefront with location param
  - Downloadable from location settings
  - **Estimate:** 1 day
  - **Owner:** Frontend Dev

**Note:** Storefront is 5% effort in this pivot. It's a data collection endpoint, not core value.

**Deliverables:**

- ✅ Basic storefront working for chains (location-aware)
- ✅ QR codes generated per location
- ✅ NO ordering system (future phase)

---

### Week 10: Vietnamese Localization & Content

**Tasks:**

- [ ] **Full Vietnamese translation review**
  - All UI text reviewed by native speaker
  - Currency formatting (₫1,250,000)
  - Date formatting (dd/mm/yyyy)
  - Number formatting (Vietnamese style)
  - **Estimate:** 2 days
  - **Owner:** Product Manager + Freelance Translator

- [ ] **Create sample data for 3 business types**
  - Restaurant chain (3 locations, full menu)
  - Café chain (2 locations, drinks menu)
  - Street food (single location)
  - **Estimate:** 1 day
  - **Owner:** Product Manager

- [ ] **Write user documentation**
  - "Cách thêm địa điểm mới" (How to add locations)
  - "Cách xem báo cáo phân tích" (How to view analytics)
  - "Cách tạo thực đơn theo địa điểm" (How to create location-specific menus)
  - **Estimate:** 2 days
  - **Owner:** Product Manager

**Deliverables:**

- ✅ All text in Vietnamese, reviewed by native
- ✅ Sample data ready for demos
- ✅ Basic user documentation in Vietnamese

---

### Week 11: Testing & Bug Fixes

**Backend Testing:**

- [ ] **Unit tests for critical paths**
  - Location CRUD (>80% coverage)
  - Analytics aggregation logic
  - Recommendation engine
  - Multi-tenant isolation
  - **Estimate:** 3 days
  - **Owner:** Backend Devs

- [ ] **Integration tests**
  - End-to-end location creation flow
  - Analytics tracking → aggregation → dashboard
  - Multi-location menu queries
  - **Estimate:** 2 days
  - **Owner:** Backend Lead

**Frontend Testing:**

- [ ] **Manual testing on real devices**
  - Android 4G (low-end phone)
  - iPhone Safari
  - Desktop Chrome
  - **Estimate:** 1 day
  - **Owner:** Frontend Dev + Product Manager

- [ ] **Performance testing**
  - Dashboard load time <1s
  - Location selector <100ms switch time
  - Mobile TTI <2s on 4G
  - **Estimate:** 1 day
  - **Owner:** Frontend Dev

**Load Testing:**

- [ ] **Simulate 50 chains (150 locations)**
  - Seed database with realistic data
  - Run artillery load tests
  - Target: <500ms p95 API response time
  - **Estimate:** 2 days
  - **Owner:** Backend Lead

**Deliverables:**

- ✅ >80% test coverage for backend
- ✅ Performance benchmarks met
- ✅ Load testing passed (50 chains)

---

### Week 12: Launch Prep & Pilot Onboarding

**Sales/Marketing Prep:**

- [ ] **Identify 50 target chains** in HCMC + Hanoi
  - Scrape Google Maps for chains with 2-5 locations
  - Compile contact info (Facebook pages, phone)
  - **Estimate:** 1 day
  - **Owner:** Product Manager

- [ ] **Create sales deck** (Vietnamese)
  - Slide 1: Problem (chain management is hard)
  - Slide 2: Solution (AI business intelligence)
  - Slide 3: Demo (screenshots of dashboard)
  - Slide 4: Pricing & ROI calculator
  - Slide 5: Testimonials (after pilot)
  - **Estimate:** 2 days
  - **Owner:** Product Manager + Designer

- [ ] **Cold outreach message template**
  - Subject: "Phân tích MIỄN PHÍ cho [Chain Name] - AI giúp tăng doanh thu"
  - Body: Personalized analysis (we noticed X locations, Y growth)
  - CTA: "Đăng ký dùng thử 1 tháng miễn phí"
  - **Estimate:** 1 day
  - **Owner:** Product Manager

**Pilot Customer Onboarding:**

- [ ] **Reach out to 10 customers for free trial (mixed segments)**
  - **Segment A:** 5 chains (3-5 locations each)
  - **Segment B:** 5 skilled operators (chefs/baristas, 1 location each)
  - Offer: 1 month free (Pro tier for chains, Growth tier for singles)
  - Goal: 5 paying customers after trial (3 chains + 2 skilled operators)
  - **Estimate:** Ongoing (weeks 10-12)
  - **Owner:** Product Manager + CEO

- [ ] **Onboarding support**
  - Schedule 30-min kickoff calls
  - Help import existing menus
  - Train on dashboard usage
  - Set up manual sales logging
  - **Estimate:** 1-2 hours per chain
  - **Owner:** Product Manager

**Deliverables:**

- ✅ Sales deck ready
- ✅ 10 pilot chains signed up for free trial
- ✅ Cold outreach campaign started

---

## Success Metrics (3 Months Post-Launch)

**Customer Acquisition:**

- [ ] 10 pilot chains onboarded (free trial)
- [ ] 3-5 converting to paid (₫500k-1.5M/month)
- [ ] <10% churn in first 3 months

**Product Usage:**

- [ ] 80% of chains log sales daily
- [ ] 60% check dashboard at least 3x/week
- [ ] 50% of AI recommendations accepted

**Customer Impact:**

- [ ] 1 customer reports 10%+ revenue increase
- [ ] 1 customer reports 20%+ waste reduction
- [ ] NPS >40 (acceptable for early product)

**Technical:**

- [ ] Dashboard loads <1s (p95)
- [ ] 99.5% uptime
- [ ] Zero data breaches or multi-tenant leaks

---

## Budget Estimate

### Development Costs (12 weeks)

- Backend developers (2 × $4k/month × 3 months): **$24,000**
- Frontend developer (1 × $3.5k/month × 3 months): **$10,500**
- Designer (0.5 × $3k/month × 3 months): **$4,500**
- Product Manager (0.5 × $4k/month × 3 months): **$6,000**

**Total Development:** ~$45,000

### Infrastructure Costs (3 months)

- DigitalOcean Droplets (staging + production): **$50/month**
- PostgreSQL managed database: **$60/month**
- Redis managed instance: **$30/month**
- S3-compatible storage (Spaces): **$10/month**
- Domain + SSL: **$20/month**
- SMS OTP (1000 messages): **$50/month**

**Total Infrastructure (3 months):** ~$660

### Third-Party Services

- Google Maps API (address autocomplete): **$50/month**
- Sentry (error tracking): **$29/month**
- Analytics (Plausible or similar): **$19/month**

**Total Third-Party (3 months):** ~$294

### Marketing (Pilot Phase)

- Freelance translator (Vietnamese review): **$300**
- LinkedIn outreach tool: **$50/month × 3**
- Facebook Ads (small budget): **$200/month × 3**

**Total Marketing:** ~$1,050

---

**Grand Total (12 weeks):** ~$47,000

*(Can reduce by 30-40% with offshore developers or part-time roles)!*

---

## Risks & Contingency Plans

### Risk 1: Development Takes Longer (14-16 weeks instead of 12)

**Mitigation:**

- Cut scope: Remove storefront (focus 100% on dashboard)
- Hire contractor for specific tasks (UI polish, translations)
- Extend timeline, communicate transparently to pilot customers

### Risk 2: No Chains Sign Up for Pilot

**Mitigation:**

- Offer aggressive incentives (3 months free Pro + $100 credit)
- Target single-location owners preparing to expand
- Pivot messaging: "Prepare for growth now"
- Partner with POS vendor for customer referrals

### Risk 3: AI Recommendations Aren't Valuable

**Mitigation:**

- Gather qualitative feedback in week 10-12
- Iterate recommendation types based on feedback
- Focus on "quick wins" (low stock alerts, slow items)
- Delay advanced forecasting to Phase 2

### Risk 4: Technical Issues (Performance, Bugs)

**Mitigation:**

- Allocate 20% sprint time to bug fixes
- Prioritize ruthlessly (analytics > storefront)
- Use feature flags to disable problematic features
- Over-communicate issues to pilot customers

---

## Post-MVP Roadmap (Months 4-6)

### Phase 2 Features (After Pilot Success)

1. **POS Integration** (automate data collection)
   - Integrate with popular Vietnamese POS systems
   - Real-time sales sync (no manual entry)

2. **ML-Powered Forecasting** (advanced AI)
   - Demand prediction (tomorrow's sales by item)
   - Dynamic pricing suggestions
   - Optimal inventory recommendations

3. **Payment Gateway Integration** (monetization)
   - MoMo/ZaloPay/VNPay for subscriptions
   - Automated billing and invoicing

4. **Online Ordering** (data + revenue)
   - Customer-facing order placement
   - Integrated with storefront
   - Commission-based revenue model

5. **Delivery Partner Integration** (data)
   - GrabFood/ShopeeFood webhooks
   - Unified order dashboard

6. **Zalo Mini App** (viral distribution)
   - One-click publish to Zalo
   - Ordering within Zalo chat

---

## Team Roles & Responsibilities

### Backend Lead

- Database schema design and migrations
- API architecture and security (RLS)
- Analytics aggregation workers
- Performance optimization

### Backend Dev

- Location CRUD APIs
- Menu management updates
- Integration with third-party services
- Unit/integration tests

### Frontend Dev

- Location selector and management UI
- Analytics dashboard (charts, tables)
- Menu management screens
- Mobile-responsive design

### Product Manager (Part-Time)

- Requirements refinement
- User story writing
- Pilot customer outreach
- Vietnamese content creation

### Designer (Part-Time)

- Wireframes for key screens
- UI design system
- Sales deck design
- User documentation visuals

---

## Communication Plan

### Daily Standups (15 min)

- What did you do yesterday?
- What will you do today?
- Any blockers?

### Weekly Sprint Planning (2 hours, Monday)

- Review last week's progress
- Plan this week's tasks
- Update roadmap if needed

### Bi-Weekly Stakeholder Updates (30 min)

- Demo progress to CEO/co-founder
- Review metrics and customer feedback
- Adjust priorities if needed

### Monthly Retrospectives (1 hour)

- What went well?
- What could be improved?
- Action items for next sprint

---

## Definition of Done

A feature is complete when:

- ✅ Code implemented and reviewed (PR approved)
- ✅ Unit tests written (>80% coverage for backend)
- ✅ Manual testing on staging (frontend)
- ✅ Vietnamese translation reviewed
- ✅ Performance benchmarks met (<1s dashboard, <500ms API)
- ✅ Deployed to production
- ✅ Documentation updated (if user-facing)

---

## Launch Checklist (Week 12)

### Technical

- [ ] Database backed up daily (automated)
- [ ] SSL certificate installed and auto-renewing
- [ ] Error tracking (Sentry) configured
- [ ] Analytics (Plausible) tracking pageviews
- [ ] Monitoring alerts set up (uptime, errors)
- [ ] Load balancer configured (if needed)
- [ ] DNS records updated (www, api subdomains)

### Legal/Compliance

- [ ] Privacy policy posted (Vietnamese)
- [ ] Terms of service posted (Vietnamese)
- [ ] GDPR/data retention policy documented
- [ ] User data export functionality (future, not MVP)

### Marketing

- [ ] Landing page live (Vietnamese)
- [ ] Demo video recorded (2-3 min, Vietnamese)
- [ ] Social media accounts created (Facebook, LinkedIn)
- [ ] Email templates ready (onboarding, support)

### Support

- [ ] Support email (<support@platform.com>) configured
- [ ] FAQ page published (top 10 questions)
- [ ] Internal troubleshooting docs for team

---

**Document Owner:** Product Manager + Tech Lead  
**Review Cadence:** Weekly during sprints, monthly post-launch  
**Questions?** Open issue with label `implementation-roadmap`
