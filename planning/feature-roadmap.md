# Feature Roadmap

**Strategic Context:** AI-first business intelligence platform for Vietnamese restaurant operators (chains + skilled operators lacking business knowledge). See [product-strategy-refined.md](./product-strategy-refined.md) for complete strategy.

**Market Focus:** Vietnam only for Phases 1-3. Japan expansion deferred to Phase 4 (Year 2+).

---

## Phase 1: MVP - Foundation + AI Dashboard (Months 1-3)

**Goal:** Launch AI-powered analytics dashboard for 10 pilot customers (mix of chains + skilled operators) with proven ROI.

**Target Segments:**
- **Segment A:** 5 small chains (2-5 locations each)
- **Segment B:** 5 skilled operators (chefs/baristas, 1 location each)

### Core Features (Priority Order)

#### 1. Multi-Location Infrastructure (Weeks 1-4)
**Why First:** Foundation for everything else, serves both segments.

- [ ] **User authentication** (phone OTP primary, Facebook OAuth secondary)
  - SMS verification with 120s cooldown, 3/hour rate limit
  - Social login (Facebook) with phone verification required
  - Vietnamese-first UI and messaging

- [ ] **Multi-tenant + multi-location database**
  - `tenants` table with `is_chain` flag
  - `locations` table for chain support (Segment A) and growth path (Segment B)
  - Row-Level Security (RLS) for data isolation
  - Analytics tables: `analytics_events`, `daily_metrics`, `hourly_metrics`, `item_performance_snapshots`

- [ ] **Location management**
  - Location CRUD (create, edit, list, soft delete)
  - Location selector in header (for chains)
  - Default location for single-location operators
  - Operating hours, manager info, geo-coordinates

#### 2. AI Analytics Dashboard (Weeks 3-8) - **CORE VALUE (80% focus)**
**Why Critical:** This IS the product. Everything else feeds data into this.

- [ ] **Daily business overview (per location)**
  - Today's revenue, orders, average order value (with % change from yesterday)
  - Top 5 selling items by revenue
  - Low stock alerts
  - 7-day revenue trend chart
  - Mobile-first design (<1s load time on 4G)

- [ ] **Cross-location comparison (for chains - Segment A)**
  - Side-by-side performance table (all locations)
  - Status indicators (🟢 good, 🟡 average, 🔴 needs attention)
  - Sort by revenue, orders, AOV
  - Heatmap visualization
  - Export to CSV

- [ ] **AI Recommendations (rule-based for MVP)**
  - **Slow-moving items:** "Món [X] ế ẩm (chỉ bán 2 phần trong 3 ngày). Thử giảm giá 20%?"
  - **Stock alerts:** "Hết hàng: [món] tại [địa điểm]. Cần nhập thêm."
  - **Performance gaps (chains):** "Địa điểm A bán được nhiều [món] hơn 2 lần so với B. Xem xét chiến lược?"
  - **Profitability coaching (Segment B):** "Phở bò của bạn giá ₫35k nguyên liệu, bán ₫50k - chỉ 43% lời. Tăng giá lên ₫65k?"
  - Vietnamese messaging with actionable buttons

- [ ] **Manual sales entry (for MVP without POS)**
  - Simple daily form: "Hôm nay bán được bao nhiêu?" (total revenue, orders)
  - Optional: per-item quantity sold
  - Stores as analytics events for aggregation
  - Reminder notifications (end of day)

#### 3. Data Collection Tools (Weeks 5-8) - **15% focus**
**Why Important:** Feeds AI with training data. Not the end product.

- [ ] **Menu management (location-aware)**
  - Menu item CRUD (create, edit, list, delete)
  - Categories with smart defaults by business type
  - Location scoping: chain-wide OR location-specific items
  - Location-specific pricing (same item, different price per location)
  - Inventory tracking per location (stock levels, low stock threshold)
  - Bilingual support (Vietnamese/English)

- [ ] **Business type onboarding with smart defaults**
  - Detect business type: restaurant, café, street food, bakery, bubble tea
  - Auto-create 3-5 relevant categories
  - Optional: 2-3 sample menu items per category
  - Saves 30 minutes of setup time

- [ ] **Image upload & management**
  - Upload item photos (max 5MB, JPG/PNG/WebP)
  - Auto-resize and optimize (thumbnail, medium, large)
  - S3-compatible storage
  - Max 5 images per item (Free tier)

#### 4. Basic Storefront (Weeks 7-8) - **5% focus**
**Why Deprioritized:** Data collection endpoint only, not core value.

- [ ] **Minimal storefront publishing**
  - Subdomain: `{slug}.ourplatform.com`
  - Mobile-first menu display grouped by category
  - Location selector (if chain)
  - QR code generation per location
  - Template selection (1-2 templates only for MVP)
  - NO online ordering (not MVP priority)

- [ ] **Branding customization (basic)**
  - Upload logo
  - Set primary/secondary colors
  - Site title and tagline

#### 5. Subscription Management (Week 8)

- [ ] **Subscription tiers**
  - Free: 1 location, 20 items, no AI
  - Starter (₫200k): 1 location, unlimited items, basic AI (for Segment B)
  - Growth (₫500k): 3 locations, basic AI, cross-location analytics (for Segment A)
  - Pro (₫1.5M): 10 locations, advanced AI, consultant calls

- [ ] **Tier enforcement**
  - Block features based on tier (location limit, AI access)
  - Upgrade prompts when limits reached
  - Placeholder payment flow (manual for MVP, collect intent)

### Success Metrics (End of Phase 1)

- ✅ 10 pilot customers onboarded (5 chains + 5 skilled operators)
- ✅ 80% log sales data at least 3x/week
- ✅ 60% check dashboard daily
- ✅ 1 customer reports 10%+ revenue increase OR 20%+ waste reduction
- ✅ NPS >40
- ✅ Dashboard loads <1s (p95)

---

## Phase 2: AI Enhancement + Automation (Months 4-6)

**Goal:** Prove AI value with measurable ROI. Convert 3-5 pilot customers to paid subscriptions.

### Features

#### 1. POS Integration (automate data collection)

- [ ] **Integrate with popular Vietnamese POS systems**
  - Identify top 3 POS vendors (Magestore, KiotViet, Sapo)
  - Build webhook receivers for real-time sales sync
  - Eliminate manual data entry
  - Automatic inventory updates

#### 2. Advanced AI Recommendations (ML-powered)

- [ ] **Demand forecasting**
  - Predict tomorrow's sales by item, by location
  - "Ngày mai dự đoán bán ₫2.5M, tăng 15% so với hôm nay" (Tomorrow predicted ₫2.5M, +15%)
  - Optimal inventory ordering suggestions

- [ ] **Dynamic pricing suggestions**
  - "Cà phê sữa đá: quán bên cạnh bán ₫28k, bạn bán ₫25k. Tăng lên ₫27k?" (Competitors charge ₫28k, you charge ₫25k. Raise to ₫27k?)
  - Promotional timing: "Thứ 2 sáng thường ế, thử giảm 10% để thu hút khách" (Monday mornings slow, try 10% off)

- [ ] **Customer segmentation (per location)**
  - "Địa điểm A: khách thích đồ cao cấp (AOV ₫85k)" (Location A: customers prefer premium, AOV ₫85k)
  - "Địa điểm B: khách nhạy giá (nhiều combo)" (Location B: price-sensitive, order combos)

#### 3. Automated Reporting

- [ ] **Daily/weekly email summaries**
  - "Tuần này: doanh thu ₫18.5M (+12%), món bán chạy nhất: phở bò" (This week: revenue ₫18.5M +12%, top item: phở bò)
  - Location-specific insights for chains
  - PDF export option

- [ ] **WhatsApp/Zalo notifications**
  - Critical alerts: low stock, unusual drop in sales
  - Daily summary at 8 PM
  - Opt-in/opt-out per notification type

#### 4. Payment Gateway Integration

- [ ] **Subscription billing automation**
  - MoMo, ZaloPay, VNPay for recurring payments
  - Automated invoicing
  - Grace period for failed payments (3 days)

### Success Metrics (End of Phase 2)

- ✅ 50 total customers (30 chains + 70 skilled operators)
- ✅ ₫29M MRR (₫348M ARR)
- ✅ 3 published case studies (10-20% revenue increase OR 30% waste reduction)
- ✅ 60% of AI recommendations accepted by users
- ✅ <5% monthly churn
- ✅ NPS >50

---

## Phase 3: Scale + Advanced Features (Months 7-12)

**Goal:** Scale to 200 customers, expand feature set based on customer feedback.

### Features

#### 1. Online Ordering (Revenue Expansion)

- [ ] **Customer-facing order placement**
  - Add to cart, checkout flow
  - Order confirmation via Zalo/SMS
  - Integration with storefront
  - Commission-based revenue model (5% per transaction)

- [ ] **Order management dashboard**
  - Real-time order tracking
  - Kitchen display system (KDS) integration
  - Order history and analytics

#### 2. Delivery Partner Integration

- [ ] **GrabFood, ShopeeFood, Gojek APIs**
  - Unified order dashboard (all platforms)
  - Automatic menu sync
  - Webhook-based order notifications
  - Richer data for AI (delivery patterns, peak times)

#### 3. Zalo Mini App (Viral Distribution)

- [ ] **One-click Zalo Mini App generator**
  - Auto-convert menu to Zalo Mini App
  - Ordering within Zalo chat
  - Payment via ZaloPay
  - Social sharing (customers share with friends)

#### 4. Advanced Analytics

- [ ] **Customer analytics**
  - Returning vs new customers
  - Customer lifetime value (CLV)
  - Cohort analysis
  - RFM segmentation (Recency, Frequency, Monetary)

- [ ] **Predictive inventory**
  - Auto-generate purchase orders
  - Supplier integration (future)
  - Waste prediction and prevention

#### 5. Business Consultant Calls (Pro Tier)

- [ ] **Monthly 1-on-1 calls**
  - Review dashboard with customer
  - Identify optimization opportunities
  - Answer questions about "kiến thức kinh tế"
  - Especially valuable for Segment B (skilled operators)

### Success Metrics (End of Phase 3)

- ✅ 200 customers (60 chains + 140 skilled operators)
- ✅ ₫60M+ MRR
- ✅ 10 case studies published
- ✅ 15-20% proven revenue increase for customers
- ✅ 30%+ proven waste reduction
- ✅ Launch in 2-3 additional Vietnamese cities (Hanoi, Da Nang)

---

## Phase 4: International Expansion (Year 2+)

**Goal:** Expand to Thailand, Indonesia, or Japan after proving Vietnam model.

**Decision Criteria Before Expanding:**
- ✅ Vietnam: 500+ customers, ₫100M+ MRR
- ✅ Product-market fit proven (NPS >60, <3% churn)
- ✅ Unit economics positive (LTV:CAC > 3:1)
- ✅ Team scaled (10+ employees)

### Potential Markets (in priority order)

#### 1. Thailand (Highest Priority)
**Why:** Similar to Vietnam, large F&B market, LINE dominance (like Zalo).
- Adapt for Thai language
- Integrate PromptPay (like MoMo)
- LINE integration (like Zalo Mini App)
- Estimated TAM: ~80,000 restaurants

#### 2. Indonesia
**Why:** Massive market (270M people), growing middle class, mobile-first.
- Adapt for Bahasa Indonesia
- Integrate GoPay, OVO (e-wallets)
- Partnership with Gojek/Grab
- Estimated TAM: ~150,000 restaurants

#### 3. Japan (Lowest Priority)
**Why:** High-value market but very different culture, saturated with incumbents.
- Requires complete product redesign (reservation systems, LINE Pay)
- Strong local competition (Tabelog, Gurunavi)
- High customer acquisition cost
- Only pursue if specific partnership opportunity arises

**Features for Japan (if pursued):**
- Reservation system (critical for Japan)
- LINE integration
- Japanese payment gateways (PayPay, LINE Pay)
- Japanese templates and UX patterns
- Localized customer support

---

## Out of Scope (Permanently)

The following features will **NOT** be built:

- ❌ Custom mobile apps (iOS/Android) - web is sufficient, lower cost
- ❌ Loyalty programs - too complex for MVP, revisit in Phase 4
- ❌ Email marketing automation - not core value, use Mailchimp integration instead
- ❌ Multi-language storefronts (beyond Vietnamese/English) - not needed in Vietnam
- ❌ White-label platform - not scalable for small team
- ❌ Franchise management tools - too niche, focus on chains first

---

## Feature Prioritization Framework

**Evaluate every feature request against:**

1. **Does it improve AI recommendations?** (data collection or model improvement)
2. **Does it serve BOTH segments?** (chains AND skilled operators)
3. **Can we prove ROI?** (revenue increase or cost reduction for customer)
4. **Is it Vietnam-specific?** (deep local integration = competitive moat)
5. **Can we build it in <4 weeks?** (speed to market)

**If 3+ answers are "yes" → prioritize. If not → defer or decline.**

---

**Document Owner:** Product Manager  
**Review Cadence:** Monthly after customer interviews  
**Next Review:** End of Phase 1 (Month 3)  
**Feedback:** Open issue with label `feature-roadmap`
