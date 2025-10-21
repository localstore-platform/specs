# Design: Wireframes & UX Flow

This document outlines the user experience and visual design direction.

## User Onboarding Flow

1. **Landing Page:** Clear value proposition, targeting either Vietnamese or Japanese audience based on browser language/region.
2. **Sign Up:** Social login is prioritized (Zalo/Facebook for VN, LINE/Google for JP).
3. **Business Profile:** Enter store name, type (restaurant, cafe, etc.), and address.
4. **Template Selection:** Showcases culturally relevant templates.
5. **Guided Setup:** A wizard walks the user through adding their logo, primary colors, and first few menu items.
6. **Publish:** One-click publish to a subdomain.

## Admin Dashboard

A mobile-first dashboard with large, touch-friendly controls.

- **Home:** Overview of new orders/reservations, quick stats.
- **Menu:** Manage items, categories, daily specials.
  - **Drag-and-drop reordering:** Visual interface to rearrange menu items, categories, and variants by updating their `display_order` field.
  - **SKU management:** Optional field for inventory tracking and POS integration (mã sản phẩm).
  - **Bulk actions:** Update pricing, availability, and ordering across multiple items.
- **Reservations/Orders:** View and manage incoming requests.
- **Website:** Customize appearance, pages.
- **Integrations:** Connect to Zalo, LINE, payment gateways.
- **Settings:** Account, billing, etc.

## Menu Management UX Patterns

### Item Ordering (Display Sequence)

**Purpose:** Controls visual sequence of items across all customer-facing surfaces (web, Zalo Mini App, QR menus).

#### For Store Owners (Admin Dashboard)

- **Drag-and-drop reordering:** When owners rearrange items, backend updates `display_order` values
- **Manual sorting:** Set "Today's Special" to appear first (`display_order = 0`) or last (`display_order = 999`)
- **Category arrangement:** Control which categories show first (e.g., "Phở" before "Đồ Uống")
- **Multi-level ordering:** Applies to menus → categories → items → variants → add-ons → images

#### For Customers (Storefront)

- Items are **always fetched with `ORDER BY display_order ASC`**
- Ensures consistent presentation across all platforms
- Featured items can be pinned to top by setting `display_order = 0` and `is_featured = true`

#### Technical Workflow

When an owner **reorders items** in the UI:

1. Frontend sends new positions: `[item_id_3, item_id_1, item_id_2]`
2. Backend updates: `item_id_3 → display_order = 0`, `item_id_1 → display_order = 1`, `item_id_2 → display_order = 2`
3. Next page load shows items in the new sequence

#### Default Behavior

- **New items:** Get `display_order = 0` by default
- **Smart gaps:** Allow increments of 10 (0, 10, 20, 30) for easier insertion without reordering all items
- **Batch updates:** Can SQL-update: `UPDATE menu_items SET display_order = id_sequence WHERE ...`

#### Query Pattern

```sql
-- Backend query example:
SELECT * FROM menu_items 
WHERE tenant_id = ? AND status = 'published'
ORDER BY display_order ASC, created_at DESC;
```

**Core UX control:** Lets owners curate menu presentation without writing code.

### SKU & Inventory Tracking

- **Optional field:** SKU (Stock Keeping Unit / mã sản phẩm) is nullable for small vendors who don't need formal inventory.
- **Auto-generation option:** Offer simple pattern like `{category-code}-{item-number}` (e.g., `PHO-001`, `CAFE-002`).
- **Variant-level SKUs:** Each size/type variation can have unique SKU for precise tracking.
- **Integration hooks:** SKUs enable sync with POS systems (KiotViet, Sapo) and delivery partners (GrabFood, ShopeeFood).
- **Low stock alerts:** When `track_inventory = true`, notify owners when `stock_quantity < low_stock_threshold`.
