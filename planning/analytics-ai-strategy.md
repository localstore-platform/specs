# Analytics & AI Strategy

**Last Updated:** October 22, 2025  
**Purpose:** Technical implementation guide for analytics and AI features  
**Context:** AI dashboard is our core product (80% of effort). See [product-strategy-refined.md](./product-strategy-refined.md) for business strategy.

## Table of Contents

1. [MVP Analytics Features](#mvp-analytics-features)
2. [Data Collection Strategy](#data-collection-strategy)
3. [Database Schema for Analytics](#database-schema-for-analytics)
4. [Analytics Dashboard UI](#analytics-dashboard-ui)
5. [AI/ML Roadmap](#aiml-roadmap)
6. [Technical Architecture](#technical-architecture)
7. [Privacy & Data Governance](#privacy--data-governance)

---

## Development Phases

**Phase 1 (Months 1-3):** Data collection + basic analytics dashboard + rule-based recommendations  
**Phase 2 (Months 4-6):** ML-powered forecasting and dynamic pricing  
**Phase 3 (Months 7-12):** Advanced AI with automation and predictive analytics

---

## MVP Analytics Features

### Core Metrics Dashboard

**Target User:** Store owner checking performance at end of day or during slow hours

**Key Questions to Answer:**

1. ✅ "Hôm nay bán được bao nhiêu tiền?" (How much revenue today?)
2. ✅ "Món nào bán chạy nhất?" (Which items are selling best?)
3. ✅ "Món nào ế ẩm cần khuyến mãi?" (Which items need promotion?)
4. ✅ "So với hôm qua thế nào?" (How does it compare to yesterday?)
5. ✅ "Tồn kho còn bao nhiêu?" (Current inventory levels)
6. ✅ "Giờ nào đông khách nhất?" (Peak hours analysis)

### MVP Dashboard Layout

```text
┌─────────────────────────────────────────┐
│  📊 Tổng Quan - Hôm Nay                  │
├─────────────────────────────────────────┤
│                                          │
│  💰 Doanh Thu Hôm Nay                    │
│     ₫1,250,000                           │
│     ↑ 15% so với hôm qua                 │
│                                          │
│  📦 Đơn Hàng                              │
│     45 đơn                                │
│     ↓ 5% so với hôm qua                  │
│                                          │
│  🎯 Giá Trị Trung Bình                   │
│     ₫27,778 / đơn                        │
│     ↑ 20% so với hôm qua                 │
│                                          │
├─────────────────────────────────────────┤
│  🔥 Top 5 Món Bán Chạy                   │
├─────────────────────────────────────────┤
│  1. Cà phê sữa đá    18 ly   ₫450,000   │
│  2. Phở bò tái       12 tô   ₫720,000   │
│  3. Bạc xỉu          10 ly   ₫280,000   │
│  4. Bánh mì thịt      8 ổ    ₫120,000   │
│  5. Sinh tố bơ        6 ly   ₫180,000   │
│                                          │
├─────────────────────────────────────────┤
│  ⚠️ Món Cần Chú Ý                        │
├─────────────────────────────────────────┤
│  🔻 Ế ẩm (cần khuyến mãi)                │
│     • Trà đào cam sả (0 ly hôm nay)      │
│     • Bánh flan (1 phần trong 3 ngày)    │
│                                          │
│  📉 Tồn kho thấp                         │
│     • Phở bò (còn 5 tô)                  │
│     • Cà phê sữa đá (còn 10 ly)          │
│                                          │
├─────────────────────────────────────────┤
│  📈 Biểu Đồ Doanh Thu Theo Giờ           │
├─────────────────────────────────────────┤
│  (Bar chart showing sales by hour)       │
│  Peak: 12:00-13:00 (₫450,000)            │
│  Slow: 15:00-16:00 (₫50,000)             │
│                                          │
├─────────────────────────────────────────┤
│  📅 So Sánh 7 Ngày Qua                   │
├─────────────────────────────────────────┤
│  (Line chart: revenue last 7 days)       │
│  Trung bình: ₫1,100,000/ngày             │
│  Cao nhất: Chủ nhật ₫1,800,000           │
│  Thấp nhất: Thứ 3 ₫850,000               │
│                                          │
└─────────────────────────────────────────┘
```

### Date Range Filters

- **Hôm nay** (Today) - Default
- **Hôm qua** (Yesterday)
- **7 ngày qua** (Last 7 days)
- **30 ngày qua** (Last 30 days)
- **Tùy chọn** (Custom range picker)

---

## Data Collection Strategy

### Events to Track

#### 1. Storefront View Events

```typescript
{
  event: 'storefront_view',
  tenant_id: 'uuid',
  timestamp: '2025-10-21T14:30:00Z',
  visitor_id: 'anonymous-hash', // Cookie-based
  device_type: 'mobile', // mobile/desktop
  source: 'qr_code', // qr_code/direct/social/search
  session_id: 'uuid',
  page_path: '/',
  user_agent: '...'
}
```

#### 2. Menu Item View Events

```typescript
{
  event: 'menu_item_view',
  tenant_id: 'uuid',
  menu_item_id: 'uuid',
  item_name: 'Cà phê sữa đá',
  category_id: 'uuid',
  timestamp: '2025-10-21T14:31:00Z',
  visitor_id: 'anonymous-hash',
  session_id: 'uuid'
}
```

#### 3. Order Events (Future - when ordering enabled)

```typescript
{
  event: 'order_created',
  tenant_id: 'uuid',
  order_id: 'uuid',
  timestamp: '2025-10-21T14:35:00Z',
  items: [
    { menu_item_id: 'uuid', quantity: 2, price: 25000 }
  ],
  subtotal: 50000,
  total: 50000,
  payment_method: 'cash', // cash/momo/zalopay
  source: 'qr_code',
  customer_type: 'new' // new/returning
}
```

#### 4. Item Stock Changes

```typescript
{
  event: 'stock_changed',
  tenant_id: 'uuid',
  menu_item_id: 'uuid',
  timestamp: '2025-10-21T14:40:00Z',
  old_quantity: 50,
  new_quantity: 48,
  change_type: 'sale', // sale/adjustment/restock
  changed_by_user_id: 'uuid'
}
```

#### 5. Price Changes

```typescript
{
  event: 'price_changed',
  tenant_id: 'uuid',
  menu_item_id: 'uuid',
  timestamp: '2025-10-21T10:00:00Z',
  old_price: 25000,
  new_price: 22000,
  changed_by_user_id: 'uuid'
}
```

### Collection Methods

**For MVP (Phase 1):**

- Frontend JavaScript tracking (storefront views)
- Backend API events (stock changes, price changes)
- Manual order entry by owner (if no POS integration yet)

**Future (Phase 2+):**

- POS system integration (real-time sales data)
- Payment gateway webhooks (automated order tracking)
- Delivery partner APIs (GrabFood/ShopeeFood orders)

---

## Database Schema for Analytics

### Approach: Hybrid (Transactional + Time-Series)

**Transactional Database (PostgreSQL):**

- Stores raw events for audit and compliance
- Supports complex queries for custom reports
- Good for: detailed drill-downs, historical analysis

**Time-Series Database (Optional - Phase 2):**

- TimescaleDB (PostgreSQL extension) or InfluxDB
- Optimized for aggregations and time-range queries
- Good for: real-time dashboards, trend analysis

### New Analytics Tables (PostgreSQL)

#### `analytics_events`

Immutable event log for all tracked activities.

```sql
CREATE TYPE event_type_enum AS ENUM (
  'storefront_view',
  'menu_item_view',
  'order_created',
  'order_completed',
  'stock_changed',
  'price_changed',
  'item_published',
  'item_unpublished'
);

CREATE TABLE analytics_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    
    -- Event Info
    event_type event_type_enum NOT NULL,
    event_data JSONB NOT NULL, -- Flexible schema for different event types
    
    -- Context
    visitor_id VARCHAR(255), -- Anonymous visitor tracking (cookie-based)
    session_id UUID,
    user_id UUID REFERENCES users(id), -- If authenticated
    
    -- Timestamp (immutable)
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Metadata
    device_type VARCHAR(50), -- mobile/desktop/tablet
    source VARCHAR(100), -- qr_code/direct/social/search
    user_agent TEXT,
    ip_address INET
);

-- Indexes for fast querying
CREATE INDEX idx_analytics_events_tenant_time ON analytics_events(tenant_id, created_at DESC);
CREATE INDEX idx_analytics_events_type ON analytics_events(event_type, created_at DESC);
CREATE INDEX idx_analytics_events_visitor ON analytics_events(visitor_id, created_at DESC);
CREATE INDEX idx_analytics_events_session ON analytics_events(session_id);

-- Partial index for today's events (hot data)
CREATE INDEX idx_analytics_events_today ON analytics_events(tenant_id, event_type) 
WHERE created_at >= CURRENT_DATE;

-- GIN index for JSONB queries
CREATE INDEX idx_analytics_events_data ON analytics_events USING GIN(event_data);
```

#### `daily_metrics`

Pre-aggregated daily statistics for fast dashboard loading.

```sql
CREATE TABLE daily_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    metric_date DATE NOT NULL,
    
    -- Revenue Metrics
    total_revenue_vnd DECIMAL(12, 2) DEFAULT 0,
    total_orders INT DEFAULT 0,
    avg_order_value_vnd DECIMAL(10, 2) DEFAULT 0,
    
    -- Traffic Metrics
    storefront_views INT DEFAULT 0,
    unique_visitors INT DEFAULT 0,
    menu_item_views INT DEFAULT 0,
    
    -- Item Performance (JSONB for flexibility)
    top_items JSONB, -- [{ item_id, name, quantity_sold, revenue }]
    slow_items JSONB, -- [{ item_id, name, days_since_last_sale }]
    
    -- Hourly Breakdown (for peak hours analysis)
    hourly_revenue JSONB, -- { "07": 50000, "08": 120000, ... }
    
    -- Stock Metrics
    low_stock_items JSONB, -- [{ item_id, name, current_stock, threshold }]
    
    -- Computed Metrics
    conversion_rate DECIMAL(5, 2), -- (orders / unique_visitors) * 100
    
    -- Comparison
    revenue_vs_yesterday_pct DECIMAL(5, 2),
    orders_vs_yesterday_pct DECIMAL(5, 2),
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_tenant_date UNIQUE (tenant_id, metric_date)
);

CREATE INDEX idx_daily_metrics_tenant_date ON daily_metrics(tenant_id, metric_date DESC);
```

#### `hourly_metrics`

Real-time aggregations for intraday analysis.

```sql
CREATE TABLE hourly_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    metric_hour TIMESTAMP NOT NULL, -- Rounded to hour: 2025-10-21 14:00:00
    
    -- Revenue
    revenue_vnd DECIMAL(10, 2) DEFAULT 0,
    order_count INT DEFAULT 0,
    
    -- Traffic
    storefront_views INT DEFAULT 0,
    unique_visitors INT DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_tenant_hour UNIQUE (tenant_id, metric_hour)
);

CREATE INDEX idx_hourly_metrics_tenant_time ON hourly_metrics(tenant_id, metric_hour DESC);
```

#### `item_performance_snapshots`

Track individual menu item performance over time.

```sql
CREATE TABLE item_performance_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    menu_item_id UUID NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
    snapshot_date DATE NOT NULL,
    
    -- Sales Metrics
    units_sold INT DEFAULT 0,
    revenue_vnd DECIMAL(10, 2) DEFAULT 0,
    views INT DEFAULT 0,
    
    -- Inventory
    stock_at_start INT,
    stock_at_end INT,
    stock_changes INT, -- Net change (sales + adjustments)
    
    -- Performance Indicators
    view_to_order_rate DECIMAL(5, 2), -- (units_sold / views) * 100
    days_since_last_sale INT,
    
    -- Ranking
    sales_rank INT, -- 1 = best seller
    revenue_rank INT,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_item_date UNIQUE (tenant_id, menu_item_id, snapshot_date)
);

CREATE INDEX idx_item_snapshots_tenant_date ON item_performance_snapshots(tenant_id, snapshot_date DESC);
CREATE INDEX idx_item_snapshots_item ON item_performance_snapshots(menu_item_id, snapshot_date DESC);
```

### Data Retention Policy

**Hot Data (Fast Access):**

- Last 7 days: Keep in `hourly_metrics`
- Last 30 days: Keep in `daily_metrics`
- Last 90 days: Keep in `analytics_events` (raw events)

**Cold Data (Archival):**

- 90+ days: Compress and move to archive storage (S3/Glacier)
- Keep aggregated data in `daily_metrics` indefinitely
- Delete raw `analytics_events` after 1 year (GDPR compliance)

---

## Analytics Dashboard UI

### Feature 7.1: Analytics Overview (MVP)

**User Story:** As a store owner, I want to see today's business performance so that I can understand what's working and make quick decisions.

**Acceptance Criteria:**

**Today's Summary Cards:**

- [ ] Total revenue today (VND) with comparison to yesterday (% change, up/down arrow)
- [ ] Total orders today with comparison to yesterday
- [ ] Average order value with comparison to yesterday
- [ ] Peak hour indicator (e.g., "Giờ đông nhất: 12:00-13:00")

**Top Performing Items:**

- [ ] List top 5 best-selling items by quantity
- [ ] Show: item name, quantity sold, total revenue
- [ ] Visual indicator (medal icons: 🥇🥈🥉 for top 3)

**Items Needing Attention:**

- [ ] Section: "Món ế ẩm" (Slow-moving items)
  - Items with 0 sales today or < 2 sales in last 3 days
  - Suggest action: "Giảm giá 10%?" button (quick discount creator)
- [ ] Section: "Tồn kho thấp" (Low stock)
  - Items where `stock_quantity < low_stock_threshold`
  - Show current stock, threshold, suggest restock quantity

**Revenue Chart (7 Days):**

- [ ] Line chart showing daily revenue for last 7 days
- [ ] Highlight today (different color or bold)
- [ ] Show average line for comparison
- [ ] Mobile-responsive (horizontal scroll on small screens)

**Hourly Breakdown (Today):**

- [ ] Bar chart showing revenue by hour (07:00 - 22:00)
- [ ] Highlight peak hours
- [ ] Show "quiet hours" for potential promotions

**Date Range Selector:**

- [ ] Quick filters: Hôm nay / Hôm qua / 7 ngày / 30 ngày
- [ ] Custom date range picker
- [ ] All charts and metrics update when range changes

**Export Options:**

- [ ] "Xuất báo cáo" button
- [ ] Export as PDF (for printing/sharing)
- [ ] Export as CSV (for Excel analysis)

**Technical Requirements:**

```typescript
// API Endpoint - Get Dashboard Metrics
GET /api/tenants/:tenantId/analytics/dashboard?date=2025-10-21

// Response
{
  "summary": {
    "date": "2025-10-21",
    "revenue": {
      "today": 1250000,
      "yesterday": 1087000,
      "change_pct": 15.0,
      "trend": "up"
    },
    "orders": {
      "today": 45,
      "yesterday": 47,
      "change_pct": -4.3,
      "trend": "down"
    },
    "avg_order_value": {
      "today": 27778,
      "yesterday": 23128,
      "change_pct": 20.1,
      "trend": "up"
    },
    "peak_hour": {
      "hour": "12:00-13:00",
      "revenue": 450000,
      "orders": 18
    }
  },
  
  "top_items": [
    {
      "menu_item_id": "uuid",
      "name_vi": "Cà phê sữa đá",
      "quantity_sold": 18,
      "revenue": 450000,
      "rank": 1
    },
    // ... top 5
  ],
  
  "slow_moving_items": [
    {
      "menu_item_id": "uuid",
      "name_vi": "Trà đào cam sả",
      "days_since_last_sale": 3,
      "last_sale_date": "2025-10-18"
    }
  ],
  
  "low_stock_items": [
    {
      "menu_item_id": "uuid",
      "name_vi": "Phở bò",
      "current_stock": 5,
      "threshold": 10,
      "suggested_restock": 45
    }
  ],
  
  "revenue_chart": {
    "labels": ["15/10", "16/10", "17/10", "18/10", "19/10", "20/10", "21/10"],
    "data": [950000, 1100000, 850000, 1300000, 1050000, 1087000, 1250000],
    "average": 1084000
  },
  
  "hourly_breakdown": {
    "labels": ["07:00", "08:00", "09:00", ..., "22:00"],
    "revenue": [50000, 120000, 180000, ..., 80000],
    "orders": [2, 5, 8, ..., 3]
  }
}
```

**Performance Requirements:**

- [ ] Dashboard loads in <1s (use cached `daily_metrics`)
- [ ] Charts render smoothly on mobile
- [ ] Real-time updates every 5 minutes (WebSocket or polling)
- [ ] Offline support: show last cached data with "Cập nhật lúc XX:XX" timestamp

---

## AI/ML Roadmap (Future Phases)

### Phase 2: AI Recommendations (Month 3-6)

**Goal:** Provide actionable insights, not just data.

#### Feature: Smart Promotions

**Input Data:**

- Item performance history
- Inventory levels
- Weather data (rainy days = hot soup sales)
- Day of week patterns
- Competitor pricing (if available)

**AI Model:** Time-series forecasting + rule-based recommendations

**Output:**

```text
🤖 Đề Xuất Hôm Nay:

1. Khuyến mãi "Giảm giá 15% Trà Đào Cam Sả"
   Lý do: Món này ế 3 ngày, thời tiết nắng nóng
   Dự đoán: Tăng 8-12 đơn hàng nếu giảm giá
   [Áp dụng ngay]

2. Tăng giá "Phở Bò Tái" lên ₫60,000
   Lý do: Bán hết stock mỗi ngày, có thể tăng lợi nhuận
   Dự đoán: Không ảnh hưởng doanh số, tăng 20% lợi nhuận
   [Xem chi tiết]

3. Nhập thêm "Cà Phê Sữa Đá"
   Lý do: Sắp hết hàng, món bán chạy nhất
   Đề xuất: Nhập 100 ly, đủ cho 5 ngày
   [Tạo đơn nhập hàng]
```

#### Feature: Demand Forecasting

**Model:** ARIMA or Prophet (Facebook's time-series library)

**Predictions:**

- Tomorrow's expected revenue (±10% accuracy)
- Expected sales by item (for inventory planning)
- Peak hours forecast (staff scheduling)
- Weekly trend (plan promotions for slow days)

**UI:**

```text
📊 Dự Báo Tuần Tới:

Thứ 2: ₫950,000 (thấp - đề xuất khuyến mãi)
Thứ 3: ₫1,100,000
Thứ 4: ₫1,200,000
Thứ 5: ₫1,350,000
Thứ 6: ₫1,800,000 (cao - chuẩn bị thêm hàng)
Thứ 7: ₫2,100,000 (đỉnh tuần)
Chủ nhật: ₫1,900,000
```

### Phase 3: Advanced AI (Month 6-12)

#### Feature: Customer Behavior Prediction

**Goal:** Understand customer segments and preferences.

**Models:**

- Clustering (K-means) for customer segmentation
- Collaborative filtering for item recommendations
- Churn prediction (returning vs one-time customers)

**Output:**

```text
👥 Phân Khúc Khách Hàng:

Nhóm 1: "Dân văn phòng trưa" (35% khách)
- Thời gian: 11:30-13:00
- Món ưa thích: Cơm, Phở
- Giá trị trung bình: ₫45,000
- Đề xuất: Combo cơm trưa giảm giá

Nhóm 2: "Hội bạn trẻ chiều" (25% khách)
- Thời gian: 15:00-18:00
- Món ưa thích: Trà sữa, Bánh ngọt
- Giá trị trung bình: ₫35,000
- Đề xuất: Promotion "Mua 2 tặng 1"
```

#### Feature: Automated Campaign Creator

**Goal:** AI writes promotion copy and schedules posts.

**Flow:**

1. AI detects opportunity (slow sales, excess inventory)
2. Generates promotion idea (discount %, bundle offer)
3. Creates social media post (Vietnamese copy + image template)
4. Owner reviews → approves → auto-posts to Facebook/Zalo

**Example:**

```text
🤖 Chiến Dịch Tự Động:

"Flash Sale Chiều Nay! ☀️"

Nội dung:
"Trời nóng quá, giải nhiệt với Trà Đào Cam Sả 
chỉ còn ₫25,000 (giảm 20%) từ 15h-18h hôm nay! 
Số lượng có hạn nhé!"

Kênh: Facebook, Zalo OA
Thời gian: Đăng lúc 14:30, hết hạn 18:00
Ngân sách: Miễn phí (organic post)

[Xem trước] [Đăng ngay]
```

---

## Technical Architecture

### Data Pipeline (MVP)

```text
┌──────────────┐
│  Storefront  │ ──── JavaScript Tracker ───┐
│   (Next.js)  │                             │
└──────────────┘                             ▼
                                      ┌──────────────┐
┌──────────────┐                      │   Events     │
│ Admin Panel  │ ──── API Events ──▶  │   Ingestion  │
│   (React)    │                      │   Service    │
└──────────────┘                      └──────┬───────┘
                                             │
┌──────────────┐                             ▼
│  POS System  │ ──── Webhooks ──────▶ ┌──────────────┐
│ (Future)     │                       │ PostgreSQL   │
└──────────────┘                       │ analytics_   │
                                       │ events       │
                                       └──────┬───────┘
                                              │
                                              ▼
                                       ┌──────────────┐
                                       │  Aggregation │
                                       │  Worker      │
                                       │  (Cron job)  │
                                       └──────┬───────┘
                                              │
                                              ▼
                                       ┌──────────────┐
                                       │  daily_      │
                                       │  metrics     │
                                       │  hourly_     │
                                       │  metrics     │
                                       └──────┬───────┘
                                              │
                                              ▼
                                       ┌──────────────┐
                                       │  Dashboard   │
                                       │  API         │
                                       └──────────────┘
```

### Event Ingestion Service (NestJS)

```typescript
// Service: analytics.service.ts
export class AnalyticsService {
  
  async trackEvent(event: AnalyticsEvent): Promise<void> {
    // 1. Validate event schema
    // 2. Enrich with metadata (IP, user agent, etc.)
    // 3. Write to analytics_events table (async)
    // 4. Publish to Redis queue for real-time processing
    
    await this.db.analytics_events.create({
      tenant_id: event.tenant_id,
      event_type: event.event_type,
      event_data: event.data,
      created_at: new Date()
    });
    
    // Real-time update for today's metrics
    await this.updateHourlyMetrics(event);
  }
  
  async updateHourlyMetrics(event: AnalyticsEvent): Promise<void> {
    const currentHour = new Date();
    currentHour.setMinutes(0, 0, 0);
    
    await this.db.hourly_metrics.upsert({
      where: {
        tenant_id: event.tenant_id,
        metric_hour: currentHour
      },
      update: {
        revenue_vnd: { increment: event.data.amount },
        order_count: { increment: 1 }
      },
      create: {
        tenant_id: event.tenant_id,
        metric_hour: currentHour,
        revenue_vnd: event.data.amount,
        order_count: 1
      }
    });
  }
}
```

### Aggregation Worker (Daily Cron)

```typescript
// Run daily at 00:01 to aggregate previous day
export class MetricsAggregationWorker {
  
  @Cron('1 0 * * *') // Every day at 00:01
  async aggregateDailyMetrics() {
    const yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);
    yesterday.setHours(0, 0, 0, 0);
    
    const tenants = await this.db.tenants.findMany({
      where: { status: 'active' }
    });
    
    for (const tenant of tenants) {
      await this.computeDailyMetrics(tenant.id, yesterday);
    }
  }
  
  async computeDailyMetrics(tenantId: string, date: Date) {
    // Aggregate from analytics_events
    const metrics = await this.db.analytics_events.aggregate({
      where: {
        tenant_id: tenantId,
        created_at: {
          gte: date,
          lt: new Date(date.getTime() + 86400000) // +1 day
        },
        event_type: 'order_created'
      },
      _sum: { 'event_data.total': true },
      _count: true
    });
    
    // Get top items
    const topItems = await this.getTopItems(tenantId, date);
    const slowItems = await this.getSlowMovingItems(tenantId, date);
    
    // Save to daily_metrics
    await this.db.daily_metrics.upsert({
      where: {
        tenant_id: tenantId,
        metric_date: date
      },
      update: {
        total_revenue_vnd: metrics._sum['event_data.total'] || 0,
        total_orders: metrics._count,
        top_items: topItems,
        slow_items: slowItems,
        updated_at: new Date()
      },
      create: {
        tenant_id: tenantId,
        metric_date: date,
        total_revenue_vnd: metrics._sum['event_data.total'] || 0,
        total_orders: metrics._count,
        top_items: topItems,
        slow_items: slowItems
      }
    });
  }
}
```

### Frontend Tracking (Storefront)

```typescript
// lib/analytics.ts
export class Analytics {
  
  static trackStorefrontView(tenantId: string) {
    const visitorId = this.getOrCreateVisitorId();
    const sessionId = this.getOrCreateSessionId();
    
    fetch('/api/analytics/track', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        event: 'storefront_view',
        tenant_id: tenantId,
        visitor_id: visitorId,
        session_id: sessionId,
        device_type: this.detectDevice(),
        source: this.detectSource()
      })
    });
  }
  
  static trackMenuItemView(tenantId: string, itemId: string) {
    // Similar to above, track item views
  }
  
  private static getOrCreateVisitorId(): string {
    // Check cookie, create if doesn't exist
    let visitorId = Cookies.get('visitor_id');
    if (!visitorId) {
      visitorId = crypto.randomUUID();
      Cookies.set('visitor_id', visitorId, { expires: 365 });
    }
    return visitorId;
  }
  
  private static detectSource(): string {
    const params = new URLSearchParams(window.location.search);
    if (params.get('source') === 'qr') return 'qr_code';
    if (document.referrer.includes('facebook')) return 'facebook';
    if (document.referrer.includes('zalo')) return 'zalo';
    return 'direct';
  }
}

// Usage in storefront page
useEffect(() => {
  Analytics.trackStorefrontView(tenantId);
}, []);
```

---

## Privacy & Data Governance

### GDPR & Vietnam Data Protection Compliance

**Data Minimization:**

- Only collect necessary data for analytics
- Anonymous visitor tracking (no PII without consent)
- IP addresses hashed for privacy

**User Rights:**

- Store owners can export all their analytics data
- Can request data deletion (keeps aggregated metrics, deletes raw events)
- Clear privacy policy explaining data collection

**Data Retention:**

- Raw events: 90 days
- Aggregated metrics: Indefinite (no PII)
- Customer data (when ordering enabled): Follow GDPR guidelines

**Security:**

- Encrypt sensitive fields in `event_data` JSONB
- Row-level security on all analytics tables
- Audit log for data access

### Ethical AI Principles

**Transparency:**

- Always explain WHY AI suggests something
- Show confidence levels ("Độ tin cậy: 85%")
- Owner has final decision, AI assists only

**Fairness:**

- Don't discriminate based on customer demographics
- Don't exploit vulnerable customers (e.g., price gouging)

**Accountability:**

- Track AI recommendation outcomes
- Improve models based on feedback
- Human review for high-impact decisions (e.g., major price changes)

---

## Implementation Roadmap

### Month 1 (MVP Foundation)

- ✅ Design database schema
- ✅ Implement event ingestion service
- ✅ Create basic dashboard UI (today's metrics only)
- ✅ Frontend tracking (storefront views)
- ✅ Daily aggregation worker

**Deliverable:** Store owners can see today's revenue, orders, and top items.

### Month 2 (Enhanced Analytics)

- 📊 Add 7-day and 30-day views
- 📊 Hourly breakdown charts
- 📊 Item performance tracking
- 📊 Low stock alerts
- 📊 Export to PDF/CSV

**Deliverable:** Full analytics dashboard with historical trends.

### Month 3-6 (AI Recommendations - Phase 2)

- 🤖 Demand forecasting model
- 🤖 Smart promotion suggestions
- 🤖 Inventory optimization
- 🤖 Automated alerts (Zalo/SMS notifications)

**Deliverable:** AI actively helps owners make better decisions.

### Month 6-12 (Advanced AI - Phase 3)

- 🤖 Customer segmentation
- 🤖 Campaign automation
- 🤖 Competitor analysis
- 🤖 Voice assistant ("Hôm nay bán được bao nhiêu?")

**Deliverable:** Fully autonomous business intelligence platform.

---

## Success Metrics

**For Store Owners:**

- Time saved on manual tracking: 30 min/day → 0
- Revenue increase from optimized pricing: 10-15%
- Inventory waste reduction: 20-30%
- User engagement: 80% check dashboard daily

**For Platform:**

- Competitive differentiation: Only platform with AI insights
- Retention: Analytics users have 2x lower churn
- Upsell opportunity: Premium analytics tier
- Data moat: More usage = better AI = more value

---

**Document Owner:** Data Team + Product Manager
**Next Steps:**

1. Get stakeholder buy-in on MVP scope
2. Finalize database schema design
3. Set up event tracking infrastructure
4. Build MVP dashboard UI
5. Launch beta with 10 pilot stores

**Questions?** Open an issue with label `analytics-ai-strategy`
