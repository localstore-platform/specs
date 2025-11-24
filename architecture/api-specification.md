# REST API Specification

**Last Updated:** October 22, 2025  
**Purpose:** Complete REST API specification for mobile (Flutter) and web (Next.js) clients  
**Architecture:** NestJS backend with PostgreSQL, Redis cache, multi-tenant with location scoping

---

## Table of Contents

1. [API Design Principles](#api-design-principles)
2. [Authentication](#authentication)
3. [Common Patterns](#common-patterns)
4. [Endpoints by Module](#endpoints-by-module)
5. [Error Handling](#error-handling)
6. [Rate Limiting](#rate-limiting)

---

## API Design Principles

### RESTful Conventions

- Use HTTP verbs correctly: GET (read), POST (create), PUT/PATCH (update), DELETE (remove)
- Resource-based URLs: `/api/locations/{id}` not `/api/getLocation?id=123`
- Plural nouns for collections: `/api/locations` not `/api/location`
- Nested resources for relationships: `/api/tenants/{tenantId}/locations`

### Versioning

```
Current: /api/v1/*
Future: /api/v2/* (when breaking changes needed)
```

### Response Format (Consistent)

```typescript
// Success response
{
  "data": { /* actual payload */ },
  "meta": {
    "timestamp": "2025-10-22T14:30:00Z",
    "tenant_id": "uuid",
    "location_id": "uuid" // if applicable
  }
}

// Paginated response
{
  "data": [ /* items */ ],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8
  }
}

// Error response
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Số điện thoại không hợp lệ",
    "details": {
      "field": "phone",
      "constraint": "must be 10 digits"
    }
  },
  "meta": {
    "timestamp": "2025-10-22T14:30:00Z",
    "request_id": "uuid"
  }
}
```

### Performance Targets

- **Mobile (4G Vietnam):** <500ms response time for dashboard endpoints
- **Web (Desktop):** <300ms response time
- **Cache strategy:** Redis for hot data (60s TTL for metrics)
- **Database:** PostgreSQL with proper indexes, materialized views for aggregations

---

## Authentication

### Phone OTP Flow

**1. Request OTP**

```http
POST /api/v1/auth/otp/request
Content-Type: application/json

{
  "phone": "+84912345678",
  "country_code": "VN"
}

Response 200 OK:
{
  "data": {
    "session_id": "uuid",
    "expires_at": "2025-10-22T14:32:00Z", // 2 minutes
    "retry_after": 120 // seconds until can resend
  }
}

Response 429 Too Many Requests:
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Vui lòng đợi 2 phút trước khi gửi lại mã",
    "retry_after": 120
  }
}
```

**2. Verify OTP**

```http
POST /api/v1/auth/otp/verify
Content-Type: application/json

{
  "session_id": "uuid",
  "code": "123456"
}

Response 200 OK:
{
  "data": {
    "access_token": "jwt-token",
    "refresh_token": "refresh-token",
    "expires_in": 86400, // 24 hours
    "user": {
      "id": "uuid",
      "phone": "+84912345678",
      "role": "OWNER",
      "tenant_id": "uuid",
      "tenant": {
        "id": "uuid",
        "name": "Chuỗi Cà Phê Highlands Clone",
        "is_chain": true,
        "total_locations": 5
      }
    }
  }
}

Response 401 Unauthorized:
{
  "error": {
    "code": "INVALID_OTP",
    "message": "Mã OTP không đúng"
  }
}
```

**3. Refresh Token**

```http
POST /api/v1/auth/refresh
Content-Type: application/json
Authorization: Bearer {refresh_token}

Response 200 OK:
{
  "data": {
    "access_token": "new-jwt-token",
    "expires_in": 86400
  }
}
```

### Authorization Headers

```http
# All authenticated requests
Authorization: Bearer {access_token}

# Mobile app identifier (for analytics)
X-Client-Type: mobile
X-Client-Version: 1.0.0
X-Device-OS: iOS 17.0

# Location context (for chains)
X-Location-ID: uuid
```

---

## Common Patterns

### Date Range Filtering

```http
# All metric endpoints support these query params
?from=2025-10-15&to=2025-10-21  # ISO 8601 dates (YYYY-MM-DD)
?range=today                     # today, yesterday, last_7_days, last_30_days, this_month
```

### Location Scoping

```http
# Single location
GET /api/v1/locations/{locationId}/metrics/today

# All locations (chains only)
GET /api/v1/tenants/{tenantId}/metrics/today

# Specific locations (chains only)
GET /api/v1/tenants/{tenantId}/metrics/today?location_ids=uuid1,uuid2
```

### Pagination

```http
GET /api/v1/recommendations?page=1&per_page=20

# Cursor-based for real-time data
GET /api/v1/analytics/events?cursor=abc123&limit=50
```

---

## Endpoints by Module

### 1. Dashboard & Metrics (CORE - Mobile Priority)

#### Get Today's Metrics (Mobile Home Screen)

```http
GET /api/v1/locations/{locationId}/metrics/today
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "date": "2025-10-22",
    "location_id": "uuid",
    "location_name": "Quận 1",
    "revenue": {
      "current": 2150000,      // VND
      "previous": 1850000,     // Yesterday
      "change_percent": 16.2,
      "target": 2500000,       // Monthly target / 30
      "target_progress": 86.0
    },
    "orders": {
      "current": 43,
      "previous": 38,
      "change_percent": 13.2
    },
    "average_order_value": {
      "current": 50000,
      "previous": 48684,
      "change_percent": 2.7
    },
    "last_updated_at": "2025-10-22T14:30:00Z"
  }
}

Response 304 Not Modified (if cached with ETag)
```

#### Get Top Selling Items

```http
GET /api/v1/locations/{locationId}/items/top-selling
Authorization: Bearer {token}
Query Params:
  ?range=today (default) | yesterday | last_7_days | last_30_days
  ?limit=5 (default)

Response 200 OK:
{
  "data": {
    "items": [
      {
        "item_id": "uuid",
        "name": "Cà phê sữa đá",
        "quantity_sold": 28,
        "revenue": 560000,
        "percentage_of_total": 26.0,
        "image_url": "https://cdn.../coffee.jpg"
      },
      {
        "item_id": "uuid",
        "name": "Phở bò tái",
        "quantity_sold": 15,
        "revenue": 900000,
        "percentage_of_total": 41.9,
        "image_url": null
      }
      // ... 3 more items
    ],
    "total_items_sold": 120,
    "total_revenue": 2150000
  }
}
```

#### Get Cross-Location Comparison (Chains Only)

```http
GET /api/v1/tenants/{tenantId}/locations/compare
Authorization: Bearer {token}
Query Params:
  ?range=today (default) | yesterday | last_7_days
  ?metric=revenue (default) | orders | aov

Response 200 OK:
{
  "data": {
    "locations": [
      {
        "location_id": "uuid",
        "name": "Quận 1",
        "revenue": 3200000,
        "change_percent": 18.0,
        "status": "good",          // good, average, poor
        "rank": 1
      },
      {
        "location_id": "uuid",
        "name": "Quận 10",
        "revenue": 1800000,
        "change_percent": -15.0,
        "status": "poor",
        "rank": 5,
        "alert": "Doanh thu giảm bất thường"
      }
      // ... other locations
    ],
    "total_revenue": 12450000,
    "average_revenue": 2490000
  }
}
```

#### Get Revenue Trend (Chart Data)

```http
GET /api/v1/locations/{locationId}/metrics/trend
Authorization: Bearer {token}
Query Params:
  ?range=last_7_days (default) | last_30_days | this_month
  ?granularity=daily (default) | hourly

Response 200 OK:
{
  "data": {
    "points": [
      {
        "timestamp": "2025-10-22T00:00:00Z",
        "revenue": 1250000,
        "orders": 45
      },
      // ... more data points
    ],
    "summary": {
      "max": 1800000,
      "min": 850000,
      "average": 1100000,
      "trend": "increasing" // increasing, stable, decreasing
    }
  }
}
```

### 2. AI Recommendations (CORE - Mobile Priority)

#### Get Recommendations List

```http
GET /api/v1/locations/{locationId}/recommendations
Authorization: Bearer {token}
Query Params:
  ?status=pending (default) | approved | dismissed | all
  ?priority=all (default) | critical | high | normal | low
  ?page=1&per_page=20

Response 200 OK:
{
  "data": {
    "recommendations": [
      {
        "id": "uuid",
        "type": "DYNAMIC_PRICING",
        "priority": "high",
        "status": "pending",
        "created_at": "2025-10-22T12:00:00Z",
        "title": "Giảm giá Cà phê sữa đá 20%",
        "description": "Giờ sepi (14h-16h). Dự kiến tăng 12 cốc/ngày",
        "impact": {
          "metric": "revenue",
          "estimated_increase": 240000, // VND per day
          "confidence": 0.85
        },
        "action": {
          "type": "apply_discount",
          "params": {
            "item_ids": ["uuid"],
            "discount_percent": 20,
            "time_range": "14:00-16:00",
            "duration_days": 7
          }
        },
        "reasoning": [
          "Giờ 14h-16h bán trung bình chỉ 3 ly/ngày",
          "Các quán khác giảm giá giờ này và tăng 40% lượng bán",
          "Chi phí nguyên liệu thấp, vẫn lời 55% sau giảm giá"
        ]
      },
      {
        "id": "uuid",
        "type": "INVENTORY_ALERT",
        "priority": "critical",
        "status": "pending",
        "created_at": "2025-10-22T13:00:00Z",
        "title": "Phở bò sắp hết",
        "description": "Còn 8 phần. Dự kiến hết vào 16h hôm nay",
        "impact": {
          "metric": "missed_revenue",
          "estimated_loss": 720000,
          "confidence": 0.92
        },
        "action": {
          "type": "reorder_inventory",
          "params": {
            "item_ids": ["uuid"],
            "quantity": 30,
            "supplier_id": "uuid"
          }
        },
        "reasoning": [
          "Trung bình bán 12 tô/ngày vào T7",
          "Đã bán 7 tô sáng nay (cao hơn bình thường)",
          "Nhà cung cấp giao trong 2 giờ"
        ]
      }
    ]
  },
  "meta": {
    "page": 1,
    "total": 15,
    "pending_count": 8,
    "critical_count": 2
  }
}
```

#### Get Single Recommendation Detail

```http
GET /api/v1/recommendations/{recommendationId}
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "id": "uuid",
    "type": "DYNAMIC_PRICING",
    "priority": "high",
    "status": "pending",
    "created_at": "2025-10-22T12:00:00Z",
    "title": "Giảm giá Cà phê sữa đá 20%",
    "description": "Giờ sepi (14h-16h). Dự kiến tăng 12 cốc/ngày",
    "impact": {
      "metric": "revenue",
      "estimated_increase": 240000,
      "confidence": 0.85,
      "breakdown": {
        "current_avg_sales": 3,        // cups per day in that time range
        "predicted_sales": 15,         // with discount
        "current_price": 20000,
        "discounted_price": 16000,
        "net_increase": 240000         // (15 * 16000) - (3 * 20000)
      }
    },
    "action": {
      "type": "apply_discount",
      "params": {
        "item_ids": ["uuid"],
        "discount_percent": 20,
        "time_range": "14:00-16:00",
        "duration_days": 7,
        "start_date": "2025-10-23"
      }
    },
    "reasoning": [
      "Giờ 14h-16h bán trung bình chỉ 3 ly/ngày",
      "Các quán khác giảm giá giờ này và tăng 40% lượng bán",
      "Chi phí nguyên liệu ₫6,000, vẫn lời 62.5% sau giảm giá"
    ],
    "data_sources": [
      "30 ngày lịch sử bán hàng tại location này",
      "So sánh với 150 quán khác trong hệ thống",
      "Phân tích giá cạnh tranh trong bán kính 1km"
    ],
    "alternative_actions": [
      {
        "title": "Giảm 15% thay vì 20%",
        "impact": "Dự kiến tăng 8 ly/ngày (+₫128k)",
        "reasoning": "Ít rủi ro hơn, vẫn có hiệu quả"
      },
      {
        "title": "Combo: Cà phê + Bánh",
        "impact": "Tăng AOV 25%",
        "reasoning": "Khách thường mua kèm bánh vào giờ này"
      }
    ]
  }
}
```

#### Approve/Dismiss Recommendation

```http
POST /api/v1/recommendations/{recommendationId}/approve
Authorization: Bearer {token}
Content-Type: application/json

{
  "action": "approve",  // or "dismiss"
  "note": "OK, thử 7 ngày xem sao" // optional
}

Response 200 OK:
{
  "data": {
    "id": "uuid",
    "status": "approved",
    "approved_at": "2025-10-22T14:30:00Z",
    "approved_by": "uuid",
    "execution_status": "scheduled", // scheduled, in_progress, completed, failed
    "scheduled_at": "2025-10-23T14:00:00Z"
  }
}
```

### 3. Locations Management

#### List All Locations (Chains)

```http
GET /api/v1/tenants/{tenantId}/locations
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "locations": [
      {
        "id": "uuid",
        "name": "Quận 1",
        "address": "123 Nguyễn Huệ, Q1, HCMC",
        "phone": "+84912345678",
        "status": "active", // active, inactive, closed
        "manager_name": "Nguyễn Văn A",
        "created_at": "2025-01-01T00:00:00Z",
        "metrics_today": {
          "revenue": 3200000,
          "orders": 85,
          "status": "good"
        }
      }
      // ... more locations
    ]
  }
}
```

#### Get Location Details

```http
GET /api/v1/locations/{locationId}
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "id": "uuid",
    "name": "Quận 1",
    "address": "123 Nguyễn Huệ, Q1, HCMC",
    "phone": "+84912345678",
    "email": "quan1@example.com",
    "status": "active",
    "operating_hours": {
      "monday": "08:00-22:00",
      "tuesday": "08:00-22:00",
      "wednesday": "08:00-22:00",
      "thursday": "08:00-22:00",
      "friday": "08:00-23:00",
      "saturday": "08:00-23:00",
      "sunday": "08:00-22:00"
    },
    "manager": {
      "name": "Nguyễn Văn A",
      "phone": "+84987654321"
    },
    "coordinates": {
      "lat": 10.7769,
      "lng": 106.7009
    },
    "created_at": "2025-01-01T00:00:00Z"
  }
}
```

### 4. Push Notifications

#### Register Device Token

```http
POST /api/v1/notifications/devices
Authorization: Bearer {token}
Content-Type: application/json

{
  "token": "firebase-device-token",
  "platform": "ios", // ios, android
  "device_id": "unique-device-id",
  "app_version": "1.0.0",
  "os_version": "iOS 17.0"
}

Response 200 OK:
{
  "data": {
    "device_id": "uuid",
    "registered_at": "2025-10-22T14:30:00Z"
  }
}
```

#### Get Notification History

```http
GET /api/v1/notifications
Authorization: Bearer {token}
Query Params:
  ?status=unread (default) | read | all
  ?type=all | ai_recommendation | alert | daily_summary
  ?page=1&per_page=20

Response 200 OK:
{
  "data": {
    "notifications": [
      {
        "id": "uuid",
        "type": "ai_recommendation",
        "priority": "high",
        "title": "🤖 AI khuyến nghị mới",
        "body": "Giảm giá Cà phê sữa 20% từ 14h-16h",
        "data": {
          "recommendation_id": "uuid",
          "location_id": "uuid"
        },
        "read": false,
        "created_at": "2025-10-22T12:00:00Z",
        "action_url": "/recommendations/uuid"
      }
      // ... more notifications
    ]
  },
  "meta": {
    "unread_count": 5,
    "page": 1,
    "total": 50
  }
}
```

#### Mark Notification as Read

```http
PATCH /api/v1/notifications/{notificationId}/read
Authorization: Bearer {token}

Response 204 No Content
```

### 5. Menu & Items (Web Portal Priority)

#### List Menu Items

```http
GET /api/v1/locations/{locationId}/menu/items
Authorization: Bearer {token}
Query Params:
  ?category_id=uuid
  ?status=active | inactive | all
  ?search=cà phê

Response 200 OK:
{
  "data": {
    "items": [
      {
        "id": "uuid",
        "name": "Cà phê sữa đá",
        "description": "Cà phê truyền thống Việt Nam",
        "price": 20000,
        "cost": 6000,        // Cost of goods (for profit calculation)
        "margin_percent": 70,
        "category_id": "uuid",
        "category_name": "Đồ uống",
        "image_url": "https://cdn.../coffee.jpg",
        "status": "active",
        "is_chain_wide": false, // Only for this location
        "available": true,
        "inventory_count": 150, // If tracking inventory
        "performance_last_7_days": {
          "quantity_sold": 196,
          "revenue": 3920000,
          "rank": 1
        }
      }
      // ... more items
    ]
  }
}
```

### QR Sessions & Table Ordering (Mobile Priority)

This module supports the QR-per-table session model used by the storefront. Sessions are ephemeral per-table pages that persist until paid (or expired). Clients write session events (item add/remove, submits, payment attempts) via POST. Clients read deltas using a cursor-based GET. Push (SSE/WebSocket) is optional and recommended later for staff dashboards.

#### Create QR Session

```http
POST /api/v1/locations/{locationId}/qr-sessions
Content-Type: application/json
Authorization: Bearer {token}

{
  "table_id": "A12",            // optional: printed table code
  "ttl_minutes": 60              // optional, default 60
}

Response 201 Created:
{
  "data": {
    "session_id": "uuid",
    "qr_url": "https://store.ourplatform.com/s/abcd1234",
    "location_id": "uuid",
    "table_id": "A12",
    "status": "active",         // active | paid | cancelled | expired
    "expires_at": "2025-10-22T16:30:00Z"
  }
}
```

#### Append Session Event (writes)

Clients append events (item_add, item_remove, payment_attempt, payment_success, refund).

```http
POST /api/v1/locations/{locationId}/sessions/{sessionId}/events
Content-Type: application/json
Authorization: Bearer {token}

{
  "idempotency_key": "client-generated-uuid-or-hash",
  "event_type": "item_add",      // item_add | item_remove | quantity_update | submit_order | payment_attempt | payment_success | payment_failed | refund
  "client_ts": "2025-10-22T14:28:33Z",
  "device_id": "device-abc",
  "items": [
    {
      "item_id": "uuid",         // platform item_id (preferred)
      "sku": "SKU123",          // optional vendor SKU
      "name": "Cà phê sữa đá",
      "unit_price": 20000,
      "quantity": 2,
      "modifiers": [{"name":"Thêm đá","price":0}]
    }
  ],
  "metadata": {"note":"khách gọi thêm"}
}

Response 201 Created:
{
  "data": {
    "event_id": "uuid",
    "event_seq": 12345,          // monotonic per-session sequence (server assigned)
    "server_ts": "2025-10-22T14:28:34Z"
  }
}
```

Notes:
- Server deduplicates on `idempotency_key` and returns the same `event_id` for retries.
- Server assigns `event_seq` (bigserial) to guarantee ordering and supports cursor semantics.

#### Read Events (cursor-based incremental fetch)

```http
GET /api/v1/locations/{locationId}/sessions/{sessionId}/events?cursor={cursor}&limit=50
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "events": [ /* array of events ordered by event_seq */ ],
    "next_cursor": "base64(...)"  // opaque cursor token
  },
  "meta": {"limit":50}
}
```

Cursor semantics (recommended):
- Cursor encodes last seen `event_seq` and `server_ts` (e.g. base64 JSON). Clients pass cursor to fetch deltas only.
- If a cursor is expired (older than retention), server responds 410 and client should call the snapshot endpoint.

#### Session Snapshot (hydrate UI)

```http
GET /api/v1/locations/{locationId}/sessions/{sessionId}?include_latest_events=true
Authorization: Bearer {token}

Response 200 OK:
{
  "data": {
    "session_id": "uuid",
    "status": "active",
    "created_at": "...",
    "last_activity_at": "...",
    "items": [ /* current cart state computed from events */ ],
    "totals": {"subtotal": 40000, "tax": 0, "discount":0, "total":40000}
  }
}
```

#### Record / Bind Payment (gateway or manual)

```http
POST /api/v1/locations/{locationId}/sessions/{sessionId}/payments
Content-Type: application/json
Authorization: Bearer {token}

{
  "payment_method": "momo|vnpay|zalopay|cash|external_pos",
  "payment_reference": "gateway_txn_12345", // gateway or POS order id
  "amount": 40000,
  "status": "success", // success | pending | failed
  "operator_id": "staff-uuid" // optional for manual cash
}

Response 201 Created:
{
  "data": {
    "payment_id": "uuid",
    "matched": true,
    "server_ts": "..."
  }
}
```

Notes:
- Payment webhooks from gateways must be normalized and linked to `session_id` via `payment_reference` where possible. For cash/external POS, merchants use the reconciliation UI to bind payments to sessions.
- When a payment with `status: success` is recorded and validated (gateway webhook or manual confirmation), server marks session `status=paid` and emits a `payment_success` event.

#### Optional: Event Stream (SSE/WebSocket) for staff dashboards

This is optional for Phase 1 and can be added later. Example:

```http
GET /api/v1/locations/{locationId}/sessions/{sessionId}/events/stream
Authorization: Bearer {token}

# Server-sent events (SSE) stream of new events. Fallback to cursor polling if unsupported.
```

Implementation guidance:
- Phase 1: persist events to an append-only `session_events` table with `event_seq` bigserial and `idempotency_key` unique constraint.
- Use cursor GET + short polling during checkout (3–5s) and longer polling during browsing (30–60s).
- Add a background publisher to forward writes to pub/sub if/when SSE/WebSocket is enabled.

### 6. Sales Entry (Manual - Web Portal)

#### Create Sales Record

```http
POST /api/v1/locations/{locationId}/sales
Authorization: Bearer {token}
Content-Type: application/json

{
  "date": "2025-10-22",
  "time": "14:30:00",
  "items": [
    {
      "item_id": "uuid",
      "quantity": 2,
      "price": 20000,  // Actual selling price (may differ from menu price due to discounts)
      "discount": 0
    },
    {
      "item_id": "uuid",
      "quantity": 1,
      "price": 60000,
      "discount": 10000 // ₫10k discount applied
    }
  ],
  "payment_method": "cash", // cash, momo, zalopay, vnpay
  "note": "Khách quen"
}

Response 201 Created:
{
  "data": {
    "id": "uuid",
    "location_id": "uuid",
    "date": "2025-10-22",
    "time": "14:30:00",
    "total": 90000,
    "items_count": 3,
    "created_at": "2025-10-22T14:30:00Z"
  }
}
```

---

## Error Handling

### HTTP Status Codes

- `200 OK` - Success
- `201 Created` - Resource created
- `204 No Content` - Success with no response body
- `304 Not Modified` - Cached response is still valid
- `400 Bad Request` - Invalid input
- `401 Unauthorized` - Missing or invalid token
- `403 Forbidden` - Valid token but insufficient permissions
- `404 Not Found` - Resource not found
- `409 Conflict` - Resource conflict (e.g., duplicate)
- `422 Unprocessable Entity` - Validation failed
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Temporary maintenance

### Error Response Format

```typescript
{
  "error": {
    "code": "ERROR_CODE",           // Machine-readable
    "message": "User-friendly message in Vietnamese",
    "details": {                    // Optional, for debugging
      "field": "phone",
      "constraint": "must be valid Vietnamese phone number"
    },
    "help_url": "https://docs.quanly.ai/errors/ERROR_CODE" // Optional
  },
  "meta": {
    "timestamp": "2025-10-22T14:30:00Z",
    "request_id": "uuid"            // For support/debugging
  }
}
```

### Common Error Codes

```typescript
// Authentication
"AUTH_TOKEN_INVALID"      // Invalid or expired token
"AUTH_TOKEN_MISSING"      // No Authorization header
"AUTH_OTP_INVALID"        // Wrong OTP code
"AUTH_OTP_EXPIRED"        // OTP expired (>2 minutes)
"AUTH_RATE_LIMIT"         // Too many OTP requests

// Authorization
"FORBIDDEN"               // User doesn't have permission
"OWNER_ONLY"              // Endpoint only for OWNER role

// Validation
"INVALID_INPUT"           // Generic validation error
"INVALID_PHONE"           // Phone number format wrong
"INVALID_DATE_RANGE"      // from > to, or invalid dates

// Resources
"LOCATION_NOT_FOUND"      // Location ID doesn't exist
"RECOMMENDATION_NOT_FOUND"
"TENANT_NOT_FOUND"

// Business Logic
"LOCATION_LIMIT_EXCEEDED" // Tenant has reached max locations for their plan
"RECOMMENDATION_ALREADY_ACTIONED" // Can't approve an already approved recommendation
```

---

## Rate Limiting

### Limits by Endpoint Type

```typescript
// Authentication (per IP)
"/api/v1/auth/otp/request":   3 requests / 10 minutes

// Dashboard (per user)
"/api/v1/locations/*/metrics/*": 60 requests / minute

// AI Recommendations (per user)
"/api/v1/recommendations/*":  100 requests / minute

// Write operations (per user)
"POST /api/v1/sales":         30 requests / minute
"POST /api/v1/menu/items":    20 requests / minute
```

### Rate Limit Headers

```http
Response Headers:
X-RateLimit-Limit: 60          // Max requests per window
X-RateLimit-Remaining: 45      // Requests left
X-RateLimit-Reset: 1698064800  // Unix timestamp when limit resets

# When limit exceeded (429 response)
Retry-After: 30                // Seconds until can retry
```

---

## Caching Strategy

### Client-Side Caching (ETag)

```http
# First request
GET /api/v1/locations/{id}/metrics/today

Response 200 OK:
ETag: "abc123"
Cache-Control: max-age=60

# Subsequent request
GET /api/v1/locations/{id}/metrics/today
If-None-Match: "abc123"

Response 304 Not Modified (if data unchanged)
OR
Response 200 OK with new ETag (if data changed)
```

### Server-Side Caching (Redis)

```typescript
// Dashboard metrics: 60 seconds
cache.set(`metrics:location:${id}:today`, data, 60);

// Top selling items: 120 seconds (less volatile)
cache.set(`items:top:location:${id}:today`, data, 120);

// Recommendations: 300 seconds (generated by AI, expensive)
cache.set(`recommendations:location:${id}:pending`, data, 300);

// User profile: 600 seconds (rarely changes)
cache.set(`user:${id}:profile`, data, 600);
```

---

## Next Steps

1. **NestJS Implementation:** Create controllers, services, DTOs for each module
2. **Database Migrations:** Ensure schema matches API contracts
3. **Flutter SDK:** Generate Dart models and API client from this spec
4. **API Documentation:** Auto-generate Swagger/OpenAPI docs
5. **Integration Tests:** Test all endpoints with realistic data
6. **Performance Testing:** Load test dashboard endpoints for <500ms target

**Related Documents:**
- [Database Schema](./database-schema.md) - Data model
- [Database Analytics Extension](./database-schema-analytics-extension.md) - Analytics tables
- [System Diagram](./system-diagram.md) - Overall architecture
