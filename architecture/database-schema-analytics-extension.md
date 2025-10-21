# Database Schema - Analytics Tables Extension

**Related:** This extends `architecture/database-schema.md` with analytics tables for multi-location AI features.  
**Phase:** Vietnam MVP - Analytics & AI Focus  
**Date:** October 22, 2025

## Overview

These analytics tables support the **AI-first strategy** by collecting granular data per location. They enable:

- Real-time dashboards showing performance by location
- AI recommendations based on historical patterns
- Cross-location comparison and benchmarking
- Multi-tenant learning (aggregate insights from all chains)

All tables include `location_id` to support chain-level and location-level analytics.

---

## Analytics Tables

### `analytics_events`

Immutable event log - all business activities tracked for AI training.

```sql
CREATE TYPE analytics_event_type_enum AS ENUM (
  'menu_view',           -- Customer viewed menu
  'item_click',          -- Customer clicked item details
  'item_add_to_cart',    -- Customer added item to cart
  'order_placed',        -- Order submitted
  'order_completed',     -- Order fulfilled/paid
  'item_out_of_stock',   -- Item unavailable
  'price_change',        -- Item price updated
  'promotion_applied'    -- Discount/promo used
);

CREATE TYPE analytics_event_category_enum AS ENUM (
  'engagement',          -- User interaction (views, clicks)
  'transaction',         -- Commerce (orders, payments)
  'inventory',           -- Stock events
  'pricing'              -- Price/promotion changes
);

CREATE TABLE analytics_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    location_id UUID REFERENCES locations(id) ON DELETE SET NULL, -- Which location generated this event
    
    -- Event Classification
    event_type analytics_event_type_enum NOT NULL,
    event_category analytics_event_category_enum NOT NULL,
    
    -- Event Context
    menu_item_id UUID REFERENCES menu_items(id) ON DELETE SET NULL,
    category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    
    -- User Context (if available)
    user_id UUID REFERENCES users(id) ON DELETE SET NULL, -- For logged-in users
    session_id VARCHAR(255), -- For anonymous tracking
    
    -- Device & Location Context
    device_type VARCHAR(50), -- 'mobile', 'desktop', 'tablet'
    browser VARCHAR(100),
    ip_address INET,
    city VARCHAR(100),
    country_code CHAR(2),
    
    -- Transaction Details (for order events)
    order_id UUID, -- Reference to orders table (future)
    revenue_vnd DECIMAL(10, 2), -- Revenue generated (if transaction)
    quantity INT, -- Items ordered (if transaction)
    
    -- External Context (for AI training)
    weather_condition VARCHAR(50), -- 'sunny', 'rainy', 'cloudy' (from weather API)
    is_holiday BOOLEAN DEFAULT false, -- Tết, national holidays, etc.
    day_of_week SMALLINT, -- 0=Sunday, 6=Saturday
    hour_of_day SMALLINT, -- 0-23
    
    -- Metadata (flexible JSON for additional context)
    metadata JSONB DEFAULT '{}',
    
    -- Timestamp (immutable)
    event_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for common queries
CREATE INDEX idx_analytics_events_tenant_location ON analytics_events(tenant_id, location_id, event_timestamp DESC);
CREATE INDEX idx_analytics_events_type ON analytics_events(event_type, event_timestamp DESC);
CREATE INDEX idx_analytics_events_item ON analytics_events(menu_item_id, event_timestamp DESC) WHERE menu_item_id IS NOT NULL;
CREATE INDEX idx_analytics_events_session ON analytics_events(session_id, event_timestamp DESC) WHERE session_id IS NOT NULL;
CREATE INDEX idx_analytics_events_date ON analytics_events(tenant_id, location_id, DATE(event_timestamp));

-- Partition by month for performance (optional, for high-volume tenants)
-- CREATE TABLE analytics_events_2025_10 PARTITION OF analytics_events
-- FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');
```

---

### `daily_metrics`

Pre-aggregated daily statistics per location (for fast dashboard queries).

```sql
CREATE TABLE daily_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    location_id UUID REFERENCES locations(id) ON DELETE SET NULL, -- NULL = chain-wide aggregate
    metric_date DATE NOT NULL,
    
    -- Revenue Metrics
    total_revenue_vnd DECIMAL(12, 2) DEFAULT 0,
    total_orders INT DEFAULT 0,
    avg_order_value_vnd DECIMAL(10, 2) DEFAULT 0,
    
    -- Customer Metrics
    unique_visitors INT DEFAULT 0, -- Unique session_ids
    returning_customers INT DEFAULT 0,
    new_customers INT DEFAULT 0,
    
    -- Menu Performance
    total_items_sold INT DEFAULT 0,
    unique_items_sold INT DEFAULT 0, -- How many different items sold
    top_selling_item_id UUID REFERENCES menu_items(id),
    top_selling_item_quantity INT DEFAULT 0,
    
    -- Inventory Metrics
    out_of_stock_events INT DEFAULT 0,
    items_restocked INT DEFAULT 0,
    
    -- Engagement Metrics
    menu_views INT DEFAULT 0,
    item_clicks INT DEFAULT 0,
    add_to_cart_rate DECIMAL(5, 4), -- clicks / views
    conversion_rate DECIMAL(5, 4), -- orders / views
    
    -- External Context (for AI pattern recognition)
    weather_condition VARCHAR(50), -- Most common weather that day
    is_holiday BOOLEAN DEFAULT false,
    day_of_week SMALLINT,
    
    -- Comparison Metrics (vs previous day)
    revenue_change_pct DECIMAL(6, 2),
    orders_change_pct DECIMAL(6, 2),
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_tenant_location_date UNIQUE (tenant_id, location_id, metric_date)
);

CREATE INDEX idx_daily_metrics_tenant_date ON daily_metrics(tenant_id, metric_date DESC);
CREATE INDEX idx_daily_metrics_location_date ON daily_metrics(location_id, metric_date DESC) WHERE location_id IS NOT NULL;
CREATE INDEX idx_daily_metrics_revenue ON daily_metrics(tenant_id, total_revenue_vnd DESC);

-- View for chain-wide aggregates (across all locations)
CREATE OR REPLACE VIEW chain_daily_metrics AS
SELECT 
  tenant_id,
  metric_date,
  SUM(total_revenue_vnd) as total_revenue_vnd,
  SUM(total_orders) as total_orders,
  AVG(avg_order_value_vnd) as avg_order_value_vnd,
  SUM(unique_visitors) as unique_visitors,
  SUM(total_items_sold) as total_items_sold,
  COUNT(DISTINCT location_id) as active_locations,
  MAX(revenue_change_pct) as best_location_growth,
  MIN(revenue_change_pct) as worst_location_growth
FROM daily_metrics
WHERE location_id IS NOT NULL
GROUP BY tenant_id, metric_date;
```

---

### `hourly_metrics`

Hourly aggregation for real-time dashboards and intra-day pattern analysis.

```sql
CREATE TABLE hourly_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    location_id UUID REFERENCES locations(id) ON DELETE SET NULL,
    metric_hour TIMESTAMP NOT NULL, -- Truncated to hour (e.g., 2025-10-21 14:00:00)
    
    -- Hourly Metrics
    revenue_vnd DECIMAL(10, 2) DEFAULT 0,
    orders_count INT DEFAULT 0,
    items_sold INT DEFAULT 0,
    unique_visitors INT DEFAULT 0,
    
    -- Peak Detection
    is_peak_hour BOOLEAN DEFAULT false, -- Auto-detected if revenue > 1.5x daily average
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_tenant_location_hour UNIQUE (tenant_id, location_id, metric_hour)
);

CREATE INDEX idx_hourly_metrics_tenant_hour ON hourly_metrics(tenant_id, metric_hour DESC);
CREATE INDEX idx_hourly_metrics_location_hour ON hourly_metrics(location_id, metric_hour DESC) WHERE location_id IS NOT NULL;
CREATE INDEX idx_hourly_metrics_peak ON hourly_metrics(tenant_id, location_id) WHERE is_peak_hour = true;
```

---

### `item_performance_snapshots`

Track menu item performance over time for trend analysis and AI forecasting.

```sql
CREATE TABLE item_performance_snapshots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    location_id UUID REFERENCES locations(id) ON DELETE SET NULL,
    menu_item_id UUID NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
    
    -- Time Period
    snapshot_date DATE NOT NULL,
    
    -- Performance Metrics
    times_viewed INT DEFAULT 0,
    times_ordered INT DEFAULT 0,
    total_quantity_sold INT DEFAULT 0,
    total_revenue_vnd DECIMAL(10, 2) DEFAULT 0,
    
    -- Conversion Metrics
    view_to_order_rate DECIMAL(5, 4), -- orders / views
    avg_price_vnd DECIMAL(10, 2), -- May change due to promotions
    
    -- Inventory
    times_out_of_stock INT DEFAULT 0,
    stock_at_end_of_day INT,
    
    -- Ranking (within location)
    revenue_rank INT, -- 1 = best seller
    quantity_rank INT,
    
    -- Trend Indicators
    is_trending_up BOOLEAN DEFAULT false, -- Sales increasing week-over-week
    is_slow_moving BOOLEAN DEFAULT false, -- <2 sales in last 3 days
    
    -- AI Features (future)
    predicted_demand INT, -- Forecast for next day
    confidence_score DECIMAL(3, 2), -- 0.00-1.00
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT unique_tenant_location_item_date UNIQUE (tenant_id, location_id, menu_item_id, snapshot_date)
);

CREATE INDEX idx_item_performance_tenant_date ON item_performance_snapshots(tenant_id, snapshot_date DESC);
CREATE INDEX idx_item_performance_location_item ON item_performance_snapshots(location_id, menu_item_id, snapshot_date DESC);
CREATE INDEX idx_item_performance_rank ON item_performance_snapshots(tenant_id, location_id, revenue_rank) WHERE revenue_rank <= 10;
CREATE INDEX idx_item_performance_slow_moving ON item_performance_snapshots(tenant_id, location_id) WHERE is_slow_moving = true;
```

---

## Data Pipeline Architecture

### 1. Event Collection

**Frontend Tracking (JavaScript SDK):**

```javascript
// Track menu view
analyticsTracker.track('menu_view', {
  tenant_id: 'uuid',
  location_id: 'uuid',
  session_id: 'session-uuid',
  device_type: 'mobile'
});

// Track item click
analyticsTracker.track('item_click', {
  tenant_id: 'uuid',
  location_id: 'uuid',
  menu_item_id: 'uuid',
  category_id: 'uuid'
});
```

**Backend Events (Server-Side):**

```typescript
// Track order completion
await analyticsService.trackEvent({
  event_type: 'order_completed',
  tenant_id: order.tenant_id,
  location_id: order.location_id,
  order_id: order.id,
  revenue_vnd: order.total_amount,
  quantity: order.items.length
});
```

### 2. Aggregation Workers

**Daily Aggregation (Runs at 1 AM daily):**

```sql
INSERT INTO daily_metrics (tenant_id, location_id, metric_date, total_revenue_vnd, total_orders, ...)
SELECT 
  tenant_id,
  location_id,
  DATE(event_timestamp) as metric_date,
  SUM(revenue_vnd) as total_revenue_vnd,
  COUNT(DISTINCT order_id) as total_orders,
  AVG(revenue_vnd) as avg_order_value_vnd,
  ...
FROM analytics_events
WHERE event_type IN ('order_completed')
  AND DATE(event_timestamp) = CURRENT_DATE - INTERVAL '1 day'
GROUP BY tenant_id, location_id, DATE(event_timestamp);
```

**Hourly Aggregation (Runs every hour):**

```sql
INSERT INTO hourly_metrics (tenant_id, location_id, metric_hour, revenue_vnd, orders_count, ...)
SELECT 
  tenant_id,
  location_id,
  DATE_TRUNC('hour', event_timestamp) as metric_hour,
  SUM(revenue_vnd) as revenue_vnd,
  COUNT(DISTINCT order_id) as orders_count,
  ...
FROM analytics_events
WHERE event_timestamp >= DATE_TRUNC('hour', CURRENT_TIMESTAMP - INTERVAL '1 hour')
  AND event_timestamp < DATE_TRUNC('hour', CURRENT_TIMESTAMP)
GROUP BY tenant_id, location_id, DATE_TRUNC('hour', event_timestamp);
```

### 3. AI Feature Engineering

**Rule-Based Recommendations (MVP):**

```typescript
// Detect slow-moving items
const slowMovingItems = await db.query(`
  SELECT menu_item_id, name_vi
  FROM item_performance_snapshots
  WHERE location_id = $1
    AND snapshot_date >= CURRENT_DATE - INTERVAL '3 days'
    AND total_quantity_sold < 2
`);

// Generate recommendation
recommendations.push({
  type: 'slow_moving',
  priority: 'medium',
  message: `Món "${item.name_vi}" ế ẩm (chỉ bán ${item.total_quantity_sold} phần trong 3 ngày). Thử giảm giá 20%?`,
  actionable: true,
  actions: ['Create promotion', 'View details']
});
```

**Cross-Location Comparison (MVP):**

```typescript
// Identify performance gaps
const locationComparison = await db.query(`
  SELECT 
    l.location_name,
    dm.total_revenue_vnd,
    dm.revenue_change_pct,
    CASE 
      WHEN dm.total_revenue_vnd > avg_revenue * 1.2 THEN 'good'
      WHEN dm.total_revenue_vnd < avg_revenue * 0.8 THEN 'needs_attention'
      ELSE 'average'
    END as status
  FROM daily_metrics dm
  JOIN locations l ON l.id = dm.location_id
  CROSS JOIN (
    SELECT AVG(total_revenue_vnd) as avg_revenue
    FROM daily_metrics
    WHERE tenant_id = $1 AND metric_date = CURRENT_DATE
  ) avg
  WHERE dm.tenant_id = $1 AND dm.metric_date = CURRENT_DATE
`);
```

---

## Performance Considerations

### Partitioning Strategy

For high-volume tenants (1M+ events/month):

```sql
-- Partition analytics_events by month
CREATE TABLE analytics_events (
  ...
) PARTITION BY RANGE (event_timestamp);

CREATE TABLE analytics_events_2025_10 PARTITION OF analytics_events
FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');

CREATE TABLE analytics_events_2025_11 PARTITION OF analytics_events
FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');
```

### Data Retention

```sql
-- Archive old events to cold storage (S3) after 90 days
-- Keep aggregated metrics (daily_metrics) forever
DELETE FROM analytics_events 
WHERE event_timestamp < CURRENT_DATE - INTERVAL '90 days';
```

### Query Optimization

```sql
-- Always query daily_metrics for dashboards (not raw events)
-- Bad (slow):
SELECT SUM(revenue_vnd) FROM analytics_events WHERE DATE(event_timestamp) = '2025-10-21';

-- Good (fast):
SELECT total_revenue_vnd FROM daily_metrics WHERE metric_date = '2025-10-21';
```

---

## Migration Strategy

### Step 1: Add Tables (Week 1)

```bash
# Generate TypeORM migration
npm run migration:generate -- -n AddAnalyticsTables

# Review generated SQL
cat src/migrations/timestamp-AddAnalyticsTables.ts

# Run migration on dev
npm run migration:run
```

### Step 2: Add Tracking Code (Week 2)

```typescript
// Add analytics tracking to menu views
@Get('/menu')
async getMenu(@Req() req) {
  await this.analyticsService.track({
    event_type: 'menu_view',
    tenant_id: req.tenant.id,
    location_id: req.location?.id,
    session_id: req.session.id
  });
  
  return this.menuService.findAll(req.tenant.id);
}
```

### Step 3: Build Aggregation Workers (Week 3)

```typescript
// Cron job: Run daily at 1 AM
@Cron('0 1 * * *')
async aggregateDailyMetrics() {
  const yesterday = new Date();
  yesterday.setDate(yesterday.getDate() - 1);
  
  await this.analyticsService.aggregateDailyMetrics(yesterday);
}
```

### Step 4: Build Dashboard (Week 4)

```typescript
// API endpoint for dashboard
@Get('/analytics/dashboard')
async getDashboard(@Req() req) {
  const metrics = await this.analyticsService.getDailyMetrics({
    tenant_id: req.tenant.id,
    location_id: req.location?.id,
    date: new Date()
  });
  
  return metrics;
}
```

---

## Testing Strategy

### Unit Tests

```typescript
describe('AnalyticsService', () => {
  it('should track menu view event', async () => {
    await analyticsService.track({
      event_type: 'menu_view',
      tenant_id: 'uuid',
      location_id: 'uuid'
    });
    
    const events = await db.analytics_events.find();
    expect(events).toHaveLength(1);
    expect(events[0].event_type).toBe('menu_view');
  });
  
  it('should aggregate daily metrics', async () => {
    // Seed events
    await seedEvents(tenant_id, location_id, '2025-10-21');
    
    // Run aggregation
    await analyticsService.aggregateDailyMetrics('2025-10-21');
    
    // Verify metrics
    const metrics = await db.daily_metrics.findOne({
      metric_date: '2025-10-21'
    });
    expect(metrics.total_revenue_vnd).toBe(1250000);
  });
});
```

### Load Testing

```bash
# Simulate 1000 concurrent users viewing menus
artillery run load-test-analytics.yml

# Target: <500ms p95 latency for event tracking
# Target: <1s p95 latency for dashboard queries
```

---

## Monitoring & Alerts

### Key Metrics to Monitor

```typescript
// Prometheus metrics
analytics_events_total{tenant_id, location_id, event_type}
analytics_aggregation_duration_seconds{type}
analytics_dashboard_query_duration_seconds
analytics_events_dropped_total
```

### Alerts

```yaml
# AlertManager rules
- alert: AnalyticsEventsDropped
  expr: rate(analytics_events_dropped_total[5m]) > 10
  annotations:
    summary: "Analytics events being dropped"
    
- alert: AggregationJobFailed
  expr: analytics_aggregation_last_success_timestamp < time() - 86400
  annotations:
    summary: "Daily aggregation job hasn't run in 24h"
```

---

## Next Steps

1. **Review schema** with backend team
2. **Prototype event tracking** SDK (JavaScript)
3. **Build aggregation workers** (NestJS + cron)
4. **Design dashboard wireframes** (Figma)
5. **Load test** with realistic data (100k events/day)

---

**Document Owner:** Backend Team + Data Engineer  
**Review Cadence:** Weekly during implementation  
**Questions?** Open issue with label `analytics-schema`
