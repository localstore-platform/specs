# Database Schema Design

**Last Updated:** October 21, 2025  
**Target:** Vietnam MVP (Phase 1)  
**Database:** PostgreSQL 14+  
**Multi-Tenancy Pattern:** Shared database with `tenant_id` isolation

## Table of Contents

1. [Core Principles](#core-principles)
2. [Entity Relationship Diagram](#entity-relationship-diagram)
3. [Table Definitions](#table-definitions)
4. [Indexes & Performance](#indexes--performance)
5. [Migration Strategy](#migration-strategy)
6. [Seed Data](#seed-data)

---

## Core Principles

### Multi-Tenant Isolation

- Every tenant-scoped table includes `tenant_id UUID NOT NULL`
- Row-Level Security (RLS) policies enforce isolation at database level
- Application middleware validates `tenant_id` on every request
- Indexes include `tenant_id` as first column for query performance

### Naming Conventions

- Tables: plural snake_case (e.g., `menu_items`, `subscriptions`)
- Primary keys: `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`
- Foreign keys: `{table_singular}_id` (e.g., `tenant_id`, `category_id`)
- Timestamps: `created_at`, `updated_at`, `deleted_at` (soft deletes where applicable)
- Enums: `{context}_enum` (e.g., `subscription_status_enum`)

### Data Integrity

- Foreign keys with `ON DELETE CASCADE` or `ON DELETE SET NULL` based on semantics
- Check constraints for business rules (e.g., price ≥ 0)
- Unique constraints to prevent duplicates
- NOT NULL constraints on required fields

---

## Entity Relationship Diagram

```text
┌─────────────┐
│   tenants   │
└──────┬──────┘
       │
       ├──────────────┬──────────────┬──────────────┬──────────────┐
       │              │              │              │              │
       ▼              ▼              ▼              ▼              ▼
 ┌─────────┐   ┌───────────┐  ┌────────────┐ ┌────────────┐ ┌────────────┐
 │  users  │   │   menus   │  │   sites    │ │subscription│ │integration │
 └────┬────┘   └─────┬─────┘  └─────┬──────┘ └────────────┘ └────────────┘
      │              │              │
      │         ┌────┴────┐         │
      │         ▼         ▼         │
      │   ┌──────────┐ ┌──────────┐ │
      │   │categories│ │menu_items│ │
      │   └─────┬────┘ └────┬─────┘ │
      │         │            │      │
      │         └────┬───────┘      │
      │              ▼              │
      │         ┌─────────────┐     │
      │         │item_variants│     │
      │         └─────────────┘     │
      │         ┌─────────────┐     │
      │         │item_add_ons │     │
      │         └─────────────┘     │
      │         ┌─────────────┐     │
      │         │item_images  │     │
      │         └─────────────┘     │
      │                             │
      └────────────┬────────────────┘
                   ▼
            ┌──────────────┐
            │audit_logs    │
            └──────────────┘
```

---

## Table Definitions

### 1. Core Tables

#### `tenants`

Primary entity representing a business/brand (can manage multiple locations for chains).

```sql
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Business Information
    business_name VARCHAR(255) NOT NULL, -- Brand/chain name (e.g., "Highlands Coffee")
    business_type VARCHAR(100), -- 'restaurant', 'cafe', 'street_food', 'bakery', etc.
    slug VARCHAR(100) UNIQUE NOT NULL, -- for subdomain: {slug}.ourplatform.com
    
    -- Chain Configuration
    is_chain BOOLEAN DEFAULT false, -- true if managing multiple locations
    total_locations INT DEFAULT 1, -- count of active locations
    
    -- Contact & Location (headquarters for chains)
    phone VARCHAR(50),
    email VARCHAR(255),
    address TEXT,
    city VARCHAR(100),
    province VARCHAR(100),
    postal_code VARCHAR(20),
    country_code CHAR(2) DEFAULT 'VN',
    
    -- Locale & Currency
    locale VARCHAR(10) DEFAULT 'vi-VN',
    timezone VARCHAR(50) DEFAULT 'Asia/Ho_Chi_Minh',
    currency_code CHAR(3) DEFAULT 'VND',
    
    -- Status & Metadata
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'deleted')),
    onboarding_completed_at TIMESTAMP,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    
    CONSTRAINT slug_format CHECK (slug ~ '^[a-z0-9-]+$')
);

CREATE INDEX idx_tenants_slug ON tenants(slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenants_status ON tenants(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenants_is_chain ON tenants(is_chain) WHERE deleted_at IS NULL;
```

#### `locations`

**NEW:** Individual store locations for chains. Single-location tenants have 1 default location.

```sql
CREATE TABLE locations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    
    -- Location Identity
    location_name VARCHAR(255) NOT NULL, -- e.g., "Highlands Coffee - Nguyễn Huệ"
    location_code VARCHAR(50), -- Internal code: "HCF-HCM-001"
    slug VARCHAR(100) NOT NULL, -- URL-friendly: "nguyen-hue"
    
    -- Address
    address TEXT NOT NULL,
    city VARCHAR(100),
    district VARCHAR(100), -- Quận/Huyện
    ward VARCHAR(100), -- Phường/Xã
    province VARCHAR(100),
    postal_code VARCHAR(20),
    
    -- Geo-coordinates (for mapping, delivery radius)
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    
    -- Contact
    phone VARCHAR(50),
    manager_name VARCHAR(255),
    manager_phone VARCHAR(50),
    
    -- Operating Hours (JSON for flexibility)
    operating_hours JSONB DEFAULT '{
      "monday": {"open": "08:00", "close": "22:00"},
      "tuesday": {"open": "08:00", "close": "22:00"},
      "wednesday": {"open": "08:00", "close": "22:00"},
      "thursday": {"open": "08:00", "close": "22:00"},
      "friday": {"open": "08:00", "close": "22:00"},
      "saturday": {"open": "08:00", "close": "23:00"},
      "sunday": {"open": "08:00", "close": "23:00"}
    }',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    is_primary BOOLEAN DEFAULT false, -- Flagship/headquarters location
    opened_at DATE, -- Store opening date
    closed_at DATE, -- If permanently closed
    
    -- Display
    display_order INT DEFAULT 0, -- For sorting in location selector
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    
    CONSTRAINT unique_tenant_slug UNIQUE (tenant_id, slug),
    CONSTRAINT unique_location_code UNIQUE (tenant_id, location_code)
);

CREATE INDEX idx_locations_tenant_id ON locations(tenant_id, is_active) WHERE deleted_at IS NULL;
CREATE INDEX idx_locations_slug ON locations(tenant_id, slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_locations_geo ON locations(latitude, longitude) WHERE deleted_at IS NULL;
CREATE INDEX idx_locations_city ON locations(city, province) WHERE deleted_at IS NULL;

-- Trigger to update tenant.total_locations count
CREATE OR REPLACE FUNCTION update_tenant_location_count()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE tenants 
  SET total_locations = (
    SELECT COUNT(*) 
    FROM locations 
    WHERE tenant_id = NEW.tenant_id 
      AND is_active = true 
      AND deleted_at IS NULL
  )
  WHERE id = NEW.tenant_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_location_count
AFTER INSERT OR UPDATE OR DELETE ON locations
FOR EACH ROW EXECUTE FUNCTION update_tenant_location_count();
```

#### `users`

Platform users (tenant owners, staff).

```sql
CREATE TYPE user_role_enum AS ENUM ('owner', 'admin', 'staff');

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    
    -- Authentication (phone is primary)
    phone VARCHAR(50) NOT NULL, -- Primary authentication method
    phone_verified_at TIMESTAMP NOT NULL,
    password_hash VARCHAR(255), -- nullable for social-only auth
    email VARCHAR(255), -- Optional, can be null
    email_verified_at TIMESTAMP,
    
    -- Social Auth
    zalo_id VARCHAR(255) UNIQUE,
    facebook_id VARCHAR(255) UNIQUE,
    
    -- Profile
    full_name VARCHAR(255) NOT NULL,
    avatar_url TEXT,
    locale VARCHAR(10) DEFAULT 'vi-VN',
    
    -- Role & Status
    role user_role_enum DEFAULT 'owner',
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'suspended')),
    
    -- Security
    last_login_at TIMESTAMP,
    last_login_ip INET,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    
    CONSTRAINT unique_phone UNIQUE (phone)
);

CREATE INDEX idx_users_tenant_id ON users(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_phone ON users(phone) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_email ON users(email) WHERE email IS NOT NULL AND deleted_at IS NULL;
CREATE INDEX idx_users_zalo_id ON users(zalo_id) WHERE zalo_id IS NOT NULL;
CREATE INDEX idx_users_facebook_id ON users(facebook_id) WHERE facebook_id IS NOT NULL;
```

#### `refresh_tokens`

JWT refresh token management.

```sql
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    token_hash VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    revoked_at TIMESTAMP,
    
    -- Device tracking
    device_info TEXT,
    ip_address INET,
    user_agent TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_expires ON refresh_tokens(expires_at) WHERE revoked_at IS NULL;
```

### 2. Menu & Catalog Tables

#### `menus`

Container for organizing menu items (e.g., "Lunch Menu", "Drinks").

```sql
CREATE TABLE menus (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    
    -- Content (bilingual support)
    name_vi VARCHAR(255) NOT NULL,
    name_en VARCHAR(255),
    description_vi TEXT,
    description_en TEXT,
    
    -- Display & Ordering
    display_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    
    -- Availability (optional scheduling)
    available_days SMALLINT[], -- Array: [0=Sun, 1=Mon, ..., 6=Sat]
    available_from TIME,
    available_until TIME,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_menus_tenant_id ON menus(tenant_id, display_order) WHERE deleted_at IS NULL;
```

#### `categories`

Menu item categories within a menu (e.g., "Appetizers", "Main Dishes").

```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    menu_id UUID REFERENCES menus(id) ON DELETE SET NULL,
    
    -- Hierarchy (optional parent for nesting)
    parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    
    -- Content
    name_vi VARCHAR(255) NOT NULL,
    name_en VARCHAR(255),
    description_vi TEXT,
    description_en TEXT,
    
    -- Display
    display_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_categories_tenant_id ON categories(tenant_id, display_order) WHERE deleted_at IS NULL;
CREATE INDEX idx_categories_menu_id ON categories(menu_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_categories_parent_id ON categories(parent_id) WHERE deleted_at IS NULL;
```

#### `menu_items`

Individual menu items (dishes, drinks, products).

```sql
CREATE TYPE item_status_enum AS ENUM ('draft', 'published', 'archived', 'out_of_stock');

CREATE TABLE menu_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    location_id UUID REFERENCES locations(id) ON DELETE CASCADE, -- NEW: Location-specific items
    category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    
    -- Location Scope
    is_chain_wide BOOLEAN DEFAULT false, -- true = available at all locations, false = specific location only
    
    -- Content
    name_vi VARCHAR(255) NOT NULL,
    name_en VARCHAR(255),
    description_vi TEXT,
    description_en TEXT,
    
    -- Pricing (can vary by location)
    base_price DECIMAL(10, 2) NOT NULL CHECK (base_price >= 0),
    compare_at_price DECIMAL(10, 2) CHECK (compare_at_price >= base_price), -- for discounts
    currency_code CHAR(3) DEFAULT 'VND',
    
    -- Inventory (location-specific for chains)
    sku VARCHAR(100), -- Stock Keeping Unit (mã sản phẩm): unique code for inventory tracking, POS integration
    track_inventory BOOLEAN DEFAULT false,
    stock_quantity INT, -- Per-location inventory count
    low_stock_threshold INT,
    
    -- Display & Media
    display_order INT DEFAULT 0,
    thumbnail_url TEXT,
    is_featured BOOLEAN DEFAULT false,
    is_spicy BOOLEAN DEFAULT false,
    is_vegetarian BOOLEAN DEFAULT false,
    is_vegan BOOLEAN DEFAULT false,
    
    -- Status
    status item_status_enum DEFAULT 'draft',
    published_at TIMESTAMP,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_menu_items_tenant_id ON menu_items(tenant_id, status, display_order) WHERE deleted_at IS NULL;
CREATE INDEX idx_menu_items_category_id ON menu_items(category_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_menu_items_sku ON menu_items(tenant_id, sku) WHERE sku IS NOT NULL;
CREATE INDEX idx_menu_items_featured ON menu_items(tenant_id) WHERE is_featured = true AND deleted_at IS NULL;
```

#### `item_variants`

Size/type variations (e.g., "Small", "Medium", "Large").

```sql
CREATE TABLE item_variants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    menu_item_id UUID NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
    
    -- Variant Info
    name_vi VARCHAR(255) NOT NULL,
    name_en VARCHAR(255),
    price_adjustment DECIMAL(10, 2) DEFAULT 0, -- relative to base_price
    sku VARCHAR(100), -- Stock Keeping Unit (mã sản phẩm): unique code for this variant, useful for size/type tracking
    
    -- Inventory
    track_inventory BOOLEAN DEFAULT false,
    stock_quantity INT,
    
    -- Display
    display_order INT DEFAULT 0,
    is_available BOOLEAN DEFAULT true,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_item_variants_tenant_id ON item_variants(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_item_variants_menu_item_id ON item_variants(menu_item_id) WHERE deleted_at IS NULL;
```

#### `item_add_ons`

Optional add-ons/modifiers (e.g., "Extra cheese", "Topping").

```sql
CREATE TABLE item_add_ons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    menu_item_id UUID NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
    
    -- Add-on Info
    name_vi VARCHAR(255) NOT NULL,
    name_en VARCHAR(255),
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    
    -- Constraints
    is_required BOOLEAN DEFAULT false,
    max_selections INT DEFAULT 1,
    
    -- Display
    display_order INT DEFAULT 0,
    is_available BOOLEAN DEFAULT true,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_item_add_ons_tenant_id ON item_add_ons(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_item_add_ons_menu_item_id ON item_add_ons(menu_item_id) WHERE deleted_at IS NULL;
```

#### `item_images`

Multiple images per menu item.

```sql
CREATE TABLE item_images (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    menu_item_id UUID NOT NULL REFERENCES menu_items(id) ON DELETE CASCADE,
    
    -- Image Info
    original_url TEXT NOT NULL,
    thumbnail_url TEXT,
    medium_url TEXT,
    large_url TEXT,
    alt_text_vi TEXT,
    alt_text_en TEXT,
    
    -- Display
    display_order INT DEFAULT 0,
    is_primary BOOLEAN DEFAULT false,
    
    -- Metadata
    file_size_bytes BIGINT,
    width_px INT,
    height_px INT,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_item_images_tenant_id ON item_images(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_item_images_menu_item_id ON item_images(menu_item_id, display_order) WHERE deleted_at IS NULL;
```

### 3. Storefront & Publishing Tables

#### `sites`

Published storefront configuration.

```sql
CREATE TYPE site_status_enum AS ENUM ('draft', 'published', 'suspended');

CREATE TABLE sites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID UNIQUE NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    
    -- Domain Configuration
    subdomain VARCHAR(100) UNIQUE NOT NULL, -- {subdomain}.ourplatform.com
    custom_domain VARCHAR(255) UNIQUE, -- for Growth+ tiers
    custom_domain_verified BOOLEAN DEFAULT false,
    
    -- Site Content
    site_title_vi VARCHAR(255),
    site_title_en VARCHAR(255),
    tagline_vi TEXT,
    tagline_en TEXT,
    description_vi TEXT,
    description_en TEXT,
    
    -- Branding
    logo_url TEXT,
    favicon_url TEXT,
    primary_color VARCHAR(7), -- hex color
    secondary_color VARCHAR(7),
    
    -- Template & Theme
    template_id VARCHAR(50) DEFAULT 'default_vn',
    custom_css TEXT,
    
    -- SEO
    meta_title_vi VARCHAR(70),
    meta_title_en VARCHAR(70),
    meta_description_vi VARCHAR(160),
    meta_description_en VARCHAR(160),
    
    -- Features
    show_platform_branding BOOLEAN DEFAULT true,
    enable_qr_menu BOOLEAN DEFAULT true,
    qr_code_url TEXT,
    
    -- Status & Publishing
    status site_status_enum DEFAULT 'draft',
    published_at TIMESTAMP,
    last_published_at TIMESTAMP,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    
    CONSTRAINT subdomain_format CHECK (subdomain ~ '^[a-z0-9-]+$')
);

CREATE INDEX idx_sites_tenant_id ON sites(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_sites_subdomain ON sites(subdomain) WHERE deleted_at IS NULL;
CREATE INDEX idx_sites_custom_domain ON sites(custom_domain) WHERE custom_domain IS NOT NULL AND deleted_at IS NULL;
```

### 4. Subscription & Billing Tables

#### `subscription_plans`

Available pricing tiers (Free, Growth, Pro).

```sql
CREATE TYPE billing_interval_enum AS ENUM ('monthly', 'yearly');

CREATE TABLE subscription_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Plan Info
    name VARCHAR(100) UNIQUE NOT NULL, -- 'starter', 'growth', 'pro'
    display_name_vi VARCHAR(255) NOT NULL,
    display_name_en VARCHAR(255),
    description_vi TEXT,
    description_en TEXT,
    
    -- Pricing
    price_vnd DECIMAL(10, 2) NOT NULL CHECK (price_vnd >= 0),
    price_usd DECIMAL(10, 2) CHECK (price_usd >= 0),
    billing_interval billing_interval_enum DEFAULT 'monthly',
    trial_days INT DEFAULT 0,
    
    -- Limits & Features
    max_menu_items INT, -- NULL = unlimited
    max_images_per_item INT DEFAULT 5,
    max_storage_mb INT,
    custom_domain_enabled BOOLEAN DEFAULT false,
    remove_branding BOOLEAN DEFAULT false,
    advanced_analytics BOOLEAN DEFAULT false,
    priority_support BOOLEAN DEFAULT false,
    
    -- Feature Flags (JSON for flexibility)
    features JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    display_order INT DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_subscription_plans_active ON subscription_plans(is_active, display_order);
```

#### `subscriptions`

Tenant subscription tracking.

```sql
CREATE TYPE subscription_status_enum AS ENUM ('trialing', 'active', 'past_due', 'canceled', 'expired');

CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    plan_id UUID NOT NULL REFERENCES subscription_plans(id),
    
    -- Subscription Period
    status subscription_status_enum DEFAULT 'trialing',
    trial_ends_at TIMESTAMP,
    current_period_start TIMESTAMP NOT NULL,
    current_period_end TIMESTAMP NOT NULL,
    
    -- Billing
    payment_provider VARCHAR(50), -- 'momo', 'zalopay', 'vnpay', 'stripe' (future)
    provider_subscription_id VARCHAR(255), -- Provider's subscription reference
    cancel_at_period_end BOOLEAN DEFAULT false,
    canceled_at TIMESTAMP,
    ended_at TIMESTAMP,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_subscriptions_tenant_id ON subscriptions(tenant_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status, current_period_end);
CREATE INDEX idx_subscriptions_provider_id ON subscriptions(payment_provider, provider_subscription_id) WHERE provider_subscription_id IS NOT NULL;
```

#### `invoices`

Billing history.

```sql
CREATE TYPE invoice_status_enum AS ENUM ('draft', 'open', 'paid', 'void', 'uncollectible');

CREATE TABLE invoices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    subscription_id UUID REFERENCES subscriptions(id),
    
    -- Invoice Details
    invoice_number VARCHAR(100) UNIQUE NOT NULL,
    status invoice_status_enum DEFAULT 'draft',
    
    -- Amounts
    subtotal_vnd DECIMAL(10, 2) NOT NULL,
    tax_vnd DECIMAL(10, 2) DEFAULT 0,
    total_vnd DECIMAL(10, 2) NOT NULL,
    currency_code CHAR(3) DEFAULT 'VND',
    
    -- Payment
    payment_provider VARCHAR(50), -- 'momo', 'zalopay', 'vnpay', 'stripe' (future)
    provider_invoice_id VARCHAR(255), -- Provider's invoice/transaction reference
    provider_transaction_id VARCHAR(255), -- Provider's payment transaction ID
    paid_at TIMESTAMP,
    
    -- Period
    period_start TIMESTAMP NOT NULL,
    period_end TIMESTAMP NOT NULL,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoices_tenant_id ON invoices(tenant_id, created_at DESC);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_provider_id ON invoices(payment_provider, provider_invoice_id) WHERE provider_invoice_id IS NOT NULL;
CREATE INDEX idx_invoices_transaction_id ON invoices(provider_transaction_id) WHERE provider_transaction_id IS NOT NULL;
```

### 5. Integration Tables

#### `integrations`

Third-party service connections (Zalo, Facebook, payments, delivery).

```sql
CREATE TYPE integration_type_enum AS ENUM ('zalo_oauth', 'facebook_oauth', 'payment_gateway', 'delivery_partner', 'analytics');
CREATE TYPE integration_status_enum AS ENUM ('pending', 'connected', 'error', 'disconnected');

CREATE TABLE integrations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    
    -- Integration Info
    type integration_type_enum NOT NULL,
    provider VARCHAR(100) NOT NULL, -- 'zalo', 'facebook', 'momo', 'zalopay', 'vnpay', 'grabfood', etc.
    status integration_status_enum DEFAULT 'pending',
    
    -- Credentials (encrypted at application layer)
    config JSONB DEFAULT '{}', -- provider-specific config
    credentials_encrypted TEXT, -- store encrypted tokens/secrets
    
    -- OAuth specific
    access_token_encrypted TEXT,
    refresh_token_encrypted TEXT,
    token_expires_at TIMESTAMP,
    
    -- Status
    last_sync_at TIMESTAMP,
    last_error TEXT,
    is_active BOOLEAN DEFAULT true,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    
    CONSTRAINT unique_tenant_provider UNIQUE (tenant_id, provider, type)
);

CREATE INDEX idx_integrations_tenant_id ON integrations(tenant_id, type) WHERE deleted_at IS NULL;
CREATE INDEX idx_integrations_status ON integrations(status) WHERE is_active = true;
```

#### `webhooks`

Webhook endpoint management for integrations.

```sql
CREATE TYPE webhook_status_enum AS ENUM ('active', 'paused', 'failed');

CREATE TABLE webhooks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    integration_id UUID REFERENCES integrations(id) ON DELETE CASCADE,
    
    -- Webhook Config
    event_type VARCHAR(100) NOT NULL, -- 'order.created', 'payment.succeeded', etc.
    endpoint_url TEXT NOT NULL,
    secret VARCHAR(255), -- for signature verification
    
    -- Status & Monitoring
    status webhook_status_enum DEFAULT 'active',
    last_triggered_at TIMESTAMP,
    last_success_at TIMESTAMP,
    last_failure_at TIMESTAMP,
    failure_count INT DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_webhooks_tenant_id ON webhooks(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_webhooks_integration_id ON webhooks(integration_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_webhooks_event_type ON webhooks(event_type, status);
```

### 6. Audit & Analytics Tables

#### `audit_logs`

Activity tracking for compliance and debugging.

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE SET NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    
    -- Event Info
    action VARCHAR(100) NOT NULL, -- 'menu_item.created', 'user.login', etc.
    resource_type VARCHAR(100), -- 'menu_item', 'user', 'site', etc.
    resource_id UUID,
    
    -- Context
    ip_address INET,
    user_agent TEXT,
    changes JSONB, -- before/after for updates
    metadata JSONB,
    
    -- Timestamp
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_tenant_id ON audit_logs(tenant_id, created_at DESC);
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id, created_at DESC);
CREATE INDEX idx_audit_logs_action ON audit_logs(action, created_at DESC);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource_type, resource_id);
```

---

## Indexes & Performance

### Composite Indexes for Common Queries

```sql
-- Menu browsing performance
CREATE INDEX idx_menu_items_tenant_category_status 
ON menu_items(tenant_id, category_id, status, display_order) 
WHERE deleted_at IS NULL;

-- User authentication
CREATE INDEX idx_users_email_lower ON users(LOWER(email)) WHERE deleted_at IS NULL;

-- Subscription queries
CREATE INDEX idx_subscriptions_active 
ON subscriptions(tenant_id, status, current_period_end) 
WHERE status IN ('active', 'trialing');

-- Integration health checks
CREATE INDEX idx_integrations_active_sync 
ON integrations(tenant_id, is_active, last_sync_at) 
WHERE deleted_at IS NULL;
```

### Partial Indexes for Filtered Queries

```sql
-- Only index active/published records
CREATE INDEX idx_menu_items_published 
ON menu_items(tenant_id, category_id, display_order) 
WHERE status = 'published' AND deleted_at IS NULL;

-- Custom domains requiring verification
CREATE INDEX idx_sites_unverified_domains 
ON sites(tenant_id, custom_domain) 
WHERE custom_domain IS NOT NULL AND custom_domain_verified = false;
```

---

## Migration Strategy

### Development Workflow

1. **Version Control**: All migrations in `migrations/` directory, timestamped

   ```text
   migrations/
   ├── 001_create_core_tables.sql
   ├── 002_create_menu_tables.sql
   ├── 003_create_integration_tables.sql
   └── ...
   ```

2. **Migration Tool**: Use TypeORM, Prisma, or Knex.js for NestJS

   ```bash
   npm run migration:generate -- -n CreateCoreTables
   npm run migration:run
   ```

3. **Rollback Strategy**: Every migration includes `UP` and `DOWN` scripts

   ```sql
   -- UP
   CREATE TABLE tenants (...);
   
   -- DOWN
   DROP TABLE IF EXISTS tenants CASCADE;
   ```

4. **Test Migrations**: Run against local dev DB, staging, then production

   ```bash
   npm run migration:run -- --dry-run  # Preview changes
   npm run migration:run                # Execute
   npm run migration:revert             # Rollback last migration
   ```

### Row-Level Security (RLS) Setup

Enable RLS for multi-tenant isolation:

```sql
-- Enable RLS on tenant-scoped tables
ALTER TABLE menu_items ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their tenant's data
CREATE POLICY tenant_isolation ON menu_items
FOR ALL
TO authenticated_user
USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Set tenant_id in application middleware
SET app.current_tenant_id = 'abc-123-def-456';
```

---

## Seed Data

### Initial Plans

```sql
INSERT INTO subscription_plans (id, name, display_name_vi, display_name_en, price_vnd, price_usd, max_menu_items, custom_domain_enabled, remove_branding) VALUES
(gen_random_uuid(), 'starter', 'Gói Miễn Phí', 'Free Tier', 0, 0, 20, false, false),
(gen_random_uuid(), 'growth', 'Gói Phát Triển', 'Growth Tier', 300000, 12, NULL, true, true),
(gen_random_uuid(), 'pro', 'Gói Chuyên Nghiệp', 'Pro Tier', 800000, 35, NULL, true, true);
```

### Sample Vietnamese Menu Data

```sql
-- Sample tenant
INSERT INTO tenants (id, business_name, slug, business_type, city, province) VALUES
('550e8400-e29b-41d4-a716-446655440000', 'Phở Hà Nội 24', 'pho-ha-noi-24', 'restaurant', 'Ho Chi Minh City', 'Ho Chi Minh');

-- Sample menu
INSERT INTO menus (tenant_id, name_vi, name_en) VALUES
('550e8400-e29b-41d4-a716-446655440000', 'Thực Đơn Chính', 'Main Menu');

-- Sample categories
INSERT INTO categories (tenant_id, menu_id, name_vi, name_en, display_order) VALUES
('550e8400-e29b-41d4-a716-446655440000', (SELECT id FROM menus LIMIT 1), 'Phở', 'Pho', 1),
('550e8400-e29b-41d4-a716-446655440000', (SELECT id FROM menus LIMIT 1), 'Đồ Uống', 'Drinks', 2);

-- Sample menu items
INSERT INTO menu_items (tenant_id, category_id, name_vi, name_en, description_vi, base_price, status) VALUES
('550e8400-e29b-41d4-a716-446655440000', 
 (SELECT id FROM categories WHERE name_vi = 'Phở' LIMIT 1),
 'Phở Bò Tái', 
 'Rare Beef Pho',
 'Phở bò tái mềm, nước dùng thanh ngọt',
 75000,
 'published'),
('550e8400-e29b-41d4-a716-446655440000',
 (SELECT id FROM categories WHERE name_vi = 'Đồ Uống' LIMIT 1),
 'Cà Phê Sữa Đá',
 'Iced Milk Coffee',
 'Cà phê phin truyền thống với sữa đặc',
 25000,
 'published');
```

---

## Next Steps

1. ✅ **Schema Reviewed** → Architecture team sign-off
2. ⏳ **Create Initial Migrations** → Backend team generates TypeORM migrations
3. ⏳ **Set Up Local DB** → Docker Compose with PostgreSQL 14+
4. ⏳ **Seed Development Data** → Vietnamese sample menus for testing
5. ⏳ **RLS Implementation** → Row-level security policies for isolation
6. ⏳ **Integration Tests** → Validate multi-tenant queries and constraints

---

**Document Ownership:** Tech Lead
**Review Cadence:** Update after each major feature addition
**Questions?** Open an issue or ping in #tech-vietnam channel
