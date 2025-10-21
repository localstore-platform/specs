# MVP Acceptance Criteria

**Last Updated:** October 21, 2025  
**Phase:** Vietnam MVP (Phase 1)  
**Status:** Draft - Pending Team Review

## Table of Contents

**Note:** This MVP is designed for **restaurant chains and AI-first intelligence**. See [product-strategy-refined.md](./product-strategy-refined.md) for strategic positioning.

1. [User Authentication & Tenant Management](#user-authentication--tenant-management)
2. [Multi-Location Setup (Chains)](#multi-location-setup-chains)
3. [Admin Dashboard](#admin-dashboard)
4. [Menu Management](#menu-management)
5. [Analytics Dashboard (MVP Priority)](#analytics-dashboard-mvp-priority)
6. [Storefront Publishing](#storefront-publishing)
7. [Image Upload & Management](#image-upload--management)
8. [Subscription Management](#subscription-management)

**Related Documents:**

- 📊 [Analytics & AI Strategy](./analytics-ai-strategy.md) - Comprehensive analytics roadmap and AI features

---

## User Authentication & Tenant Management

### Feature 1.1: Phone Number Registration with OTP Verification

**User Story:** As a Vietnamese store owner, I want to sign up with my phone number so that I can create my menu website quickly.

**Acceptance Criteria:**

- [ ] Registration form collects: phone number, password (min 8 chars), full name, business name, business type
- [ ] Phone validation: must be valid Vietnamese format (+84 prefix or 0 prefix), unique in system
- [ ] Password requirements: minimum 8 characters, at least 1 number, 1 letter
- [ ] Form displays in Vietnamese by default (vi-VN locale)
- [ ] On submit, system sends 6-digit OTP via SMS to phone number
- [ ] OTP verification screen shows with:
  - Phone number display (masked: +84 90 *** **67)
  - 6-digit input field
  - Resend OTP button (enabled after 120 seconds countdown)
  - OTP expires in 5 minutes
- [ ] Resending OTP:
  - Invalidates/expires the old OTP immediately
  - Generates new OTP with fresh 5-minute expiry
  - Resets attempt counter to 0
  - Starts new 120-second countdown
- [ ] Correct OTP → creates both `user` and `tenant` records, auto-login
- [ ] Wrong OTP → show error "Mã OTP không đúng" (max 5 attempts, then require new OTP)
- [ ] Email field optional (can add later in profile settings)
- [ ] Error messages display in Vietnamese with clear guidance

**Technical Requirements:**

```typescript
// Step 1: Request OTP
POST /api/auth/register/request-otp
{
  "phone": "+84901234567",
  "fullName": "Nguyễn Văn A",
  "password": "SecurePass123",
  "businessName": "Phở Hà Nội 24",
  "businessType": "restaurant"
}

// Response
{
  "sessionId": "uuid",
  "phone": "+84901234567",
  "phoneMasked": "+84 90 *** **67",
  "otpExpiresAt": "2025-10-21T10:35:00Z",
  "message": "Mã OTP đã được gửi đến số điện thoại của bạn"
}

// Step 2: Verify OTP
POST /api/auth/register/verify-otp
{
  "sessionId": "uuid",
  "otp": "123456"
}

// Response (success)
{
  "userId": "uuid",
  "tenantId": "uuid",
  "phone": "+84901234567",
  "accessToken": "jwt-token",
  "refreshToken": "jwt-token",
  "message": "Đăng ký thành công!"
}

// Step 3: Resend OTP (if needed)
POST /api/auth/register/resend-otp
{
  "sessionId": "uuid"
}

// Response
{
  "sessionId": "uuid",  // Same session ID
  "phoneMasked": "+84 90 *** **67",
  "otpExpiresAt": "2025-10-21T10:40:00Z",  // New expiry time
  "canResendAt": "2025-10-21T10:37:00Z",   // 120 seconds from now
  "message": "Mã OTP mới đã được gửi"
}
```

**Database Operations:**

1. Store OTP session in Redis with 5-minute TTL:

   ```typescript
   otp:{sessionId} = {
     phone: "+84901234567",
     otp_hash: "bcrypt_hash",
     attempts: 0,
     user_data: {...},
     last_sent_at: timestamp,
     can_resend_at: timestamp + 120s
   }
   TTL: 5 minutes (refreshed on each resend)
   ```

2. On resend OTP:
   - Expire old OTP by generating new hash (old OTP no longer valid)
   - Update `last_sent_at` to current timestamp
   - Update `can_resend_at` to current timestamp + 120 seconds
   - Reset `attempts` to 0
   - Extend Redis TTL to 5 minutes from now
3. On successful verification:
   - Create `tenants` record with auto-generated `slug` from `businessName`
   - Create `users` record with `role='owner'`, `tenant_id`, hashed password, phone (no email required)
   - Delete OTP session from Redis
   - Generate JWT tokens

**SMS Integration:**

- Use SMS provider (e.g., Twilio, SMSAPI.vn, Esms.vn)
- Message template: "Mã OTP của bạn là: {otp}. Có hiệu lực trong 5 phút. Không chia sẻ mã này với ai."
- Cost consideration: ~500-1000 VND per SMS

**Rate Limiting Implementation:**

To prevent abuse and control SMS costs, implement hourly rate limiting per phone number:

```typescript
// Redis key structure for rate limiting
rate_limit:otp:{phone} = {
  count: number,        // Number of OTP requests in current hour
  window_start: timestamp
}
TTL: 1 hour

// Before sending OTP, check rate limit:
async function checkOTPRateLimit(phone: string): Promise<void> {
  const key = `rate_limit:otp:${phone}`;
  const limit = await redis.get(key);
  
  if (!limit) {
    // First request in this hour
    await redis.set(key, JSON.stringify({
      count: 1,
      window_start: Date.now()
    }), 'EX', 3600); // 1 hour TTL
    return;
  }
  
  const data = JSON.parse(limit);
  
  if (data.count >= 3) {
    const resetIn = Math.ceil((data.window_start + 3600000 - Date.now()) / 60000);
    throw new Error(`Bạn đã vượt quá giới hạn gửi OTP. Vui lòng thử lại sau ${resetIn} phút.`);
  }
  
  // Increment counter
  data.count += 1;
  await redis.set(key, JSON.stringify(data), 'EX', 3600);
}

// Call this before sending any OTP (initial request or resend)
await checkOTPRateLimit(phone);
await sendOTPViaSMS(phone, otp);
```

**Rate Limit Behavior:**

- Counter includes ALL OTP requests: initial registration + resends
- Example: User requests OTP at 10:00 AM
  - Request 1 (10:00): ✓ Allowed
  - Request 2 (10:05): ✓ Allowed (resend)
  - Request 3 (10:10): ✓ Allowed (resend)
  - Request 4 (10:15): ✗ Blocked - "Vui lòng thử lại sau 45 phút"
  - Request 5 (11:01): ✓ Allowed (new hour window)
- Redis TTL automatically resets counter after 1 hour
- No manual cleanup needed

**Edge Cases:**

- Duplicate phone → return "Số điện thoại đã được đăng ký"
- Invalid phone format → "Số điện thoại không hợp lệ"
- OTP expired → "Mã OTP đã hết hạn. Vui lòng gửi lại."
- Max OTP attempts exceeded (5) → "Quá số lần thử. Vui lòng gửi lại mã OTP mới."
- Resend clicked before 120s cooldown → "Vui lòng đợi {remaining} giây để gửi lại"
- Resend OTP → old OTP immediately invalidated, new OTP generated
- Rate limit exceeded (3 requests/hour) → "Bạn đã vượt quá giới hạn gửi OTP. Vui lòng thử lại sau {minutes} phút."
- SMS delivery failure → retry up to 3 times, then show error with support contact
- Invalid business name (special chars) → sanitize and suggest valid slug
- Rate limiting: max 3 OTP requests per phone per hour (includes initial + all resends)

---

### Feature 1.2: Social Authentication (Facebook OAuth)

**User Story:** As a store owner, I want to sign up with Facebook so that I don't need to remember another password.

**Acceptance Criteria:**

- [ ] "Đăng nhập với Facebook" button on signup/login pages
- [ ] Clicking button initiates OAuth flow (Facebook Login dialog)
- [ ] On first login: prompt for business name and phone number (required), email auto-filled from Facebook (optional)
- [ ] Phone verification required even for social auth:
  - Send OTP to entered phone number
  - Verify OTP before completing registration
- [ ] On subsequent logins: redirect directly to dashboard
- [ ] System stores `facebook_id` in users table
- [ ] User can later link password to same account (for phone + password login)
- [ ] If Facebook account's phone matches existing phone user → prompt to link accounts

**Technical Requirements:**

```typescript
// OAuth Callback
GET /api/auth/facebook/callback?code=xxx

// If new user, show onboarding:
{
  "isNewUser": true,
  "facebookProfile": {
    "id": "fb-123456",
    "email": "owner@gmail.com",
    "name": "Nguyễn Văn A"
  }
}

// Complete onboarding (Step 1: Request OTP):
POST /api/auth/facebook/complete-profile
{
  "businessName": "Phở Hà Nội 24",
  "businessType": "restaurant",
  "phone": "+84901234567"
}

// Response
{
  "sessionId": "uuid",
  "phoneMasked": "+84 90 *** **67",
  "otpExpiresAt": "2025-10-21T10:40:00Z",
  "canResendAt": "2025-10-21T10:37:00Z",  // 120 seconds from now
  "message": "Mã OTP đã được gửi"
}

// Step 2: Verify OTP
POST /api/auth/facebook/verify-otp
{
  "sessionId": "uuid",
  "otp": "123456"
}

// Step 3: Resend OTP (if needed - same endpoint as regular registration)
POST /api/auth/register/resend-otp
{
  "sessionId": "uuid"
}
```

**Database Operations:**

1. Check if `facebook_id` exists → existing user login
2. If new: verify phone via OTP, then create tenant + user with `facebook_id`, verified phone
3. Store Facebook access token encrypted in `users` table (for future Graph API calls)

---

### Feature 1.3: Business Type Selection & Smart Defaults

**User Story:** As a store owner, I want the system to create a starter menu based on my business type so that I can launch quickly without building everything from scratch.

**Acceptance Criteria:**

- [ ] During registration, owner selects business type from dropdown:
  - **Nhà hàng** (Restaurant)
  - **Quán cà phê** (Café)
  - **Quán ăn vặt / Ăn vặt đường phố** (Street Food / Snacks)
  - **Tiệm bánh** (Bakery)
  - **Quán trà sữa** (Bubble Tea Shop)
  - **Quán ăn chay** (Vegetarian Restaurant)
  - **Khác** (Other - no defaults)
- [ ] After successful registration, system automatically creates:
  - Default categories (3-5 categories based on business type)
  - Sample menu items (2-3 items per category with Vietnamese names, optional)
  - Default menu named "Thực Đơn Chính" / "Main Menu"
- [ ] Owner sees onboarding tour showing:
  - "Chúng tôi đã tạo sẵn danh mục cho bạn!" (We've created starter categories for you!)
  - Quick tutorial: Edit items → Add photos → Publish
- [ ] All defaults are fully editable:
  - Can rename/delete categories
  - Can edit/delete sample items
  - Can add more categories/items
- [ ] Defaults are marked with `is_default: true` flag (for analytics/tracking)

**Default Category Templates:**

```typescript
const BUSINESS_TYPE_DEFAULTS = {
  restaurant: {
    menu_name: "Thực Đơn Chính",
    categories: [
      { name_vi: "Khai vị", name_en: "Appetizers", display_order: 0 },
      { name_vi: "Món chính", name_en: "Main Dishes", display_order: 10 },
      { name_vi: "Món phụ", name_en: "Side Dishes", display_order: 20 },
      { name_vi: "Món tráng miệng", name_en: "Desserts", display_order: 30 },
      { name_vi: "Đồ uống", name_en: "Beverages", display_order: 40 }
    ],
    sample_items: [
      { 
        category: "Món chính",
        name_vi: "Cơm rang dương châu",
        name_en: "Yangzhou Fried Rice",
        base_price: 45000
      }
      // More samples...
    ]
  },
  
  cafe: {
    menu_name: "Thực Đơn",
    categories: [
      { name_vi: "Cà phê", name_en: "Coffee", display_order: 0 },
      { name_vi: "Trà", name_en: "Tea", display_order: 10 },
      { name_vi: "Sinh tố & Nước ép", name_en: "Smoothies & Juices", display_order: 20 },
      { name_vi: "Bánh ngọt", name_en: "Pastries", display_order: 30 }
    ],
    sample_items: [
      {
        category: "Cà phê",
        name_vi: "Cà phê sữa đá",
        name_en: "Iced Milk Coffee",
        base_price: 25000
      },
      {
        category: "Cà phê",
        name_vi: "Bạc xỉu",
        name_en: "Bac Xiu",
        base_price: 28000
      }
      // More samples...
    ]
  },
  
  street_food: {
    menu_name: "Thực Đơn",
    categories: [
      { name_vi: "Phở", name_en: "Pho", display_order: 0 },
      { name_vi: "Bún", name_en: "Rice Noodles", display_order: 10 },
      { name_vi: "Bánh mì", name_en: "Banh Mi", display_order: 20 },
      { name_vi: "Đồ uống", name_en: "Drinks", display_order: 30 }
    ],
    sample_items: [
      {
        category: "Phở",
        name_vi: "Phở bò tái",
        name_en: "Rare Beef Pho",
        base_price: 50000
      }
      // More samples...
    ]
  },
  
  bakery: {
    menu_name: "Thực Đơn",
    categories: [
      { name_vi: "Bánh mì", name_en: "Bread", display_order: 0 },
      { name_vi: "Bánh ngọt", name_en: "Pastries", display_order: 10 },
      { name_vi: "Bánh sinh nhật", name_en: "Birthday Cakes", display_order: 20 },
      { name_vi: "Đồ uống", name_en: "Beverages", display_order: 30 }
    ]
  },
  
  bubble_tea: {
    menu_name: "Thực Đơn",
    categories: [
      { name_vi: "Trà sữa trân châu", name_en: "Bubble Milk Tea", display_order: 0 },
      { name_vi: "Trà trái cây", name_en: "Fruit Tea", display_order: 10 },
      { name_vi: "Sinh tố", name_en: "Smoothies", display_order: 20 },
      { name_vi: "Topping", name_en: "Toppings", display_order: 30 }
    ]
  },
  
  vegetarian: {
    menu_name: "Thực Đơn Chay",
    categories: [
      { name_vi: "Món khai vị", name_en: "Appetizers", display_order: 0 },
      { name_vi: "Món chính", name_en: "Main Dishes", display_order: 10 },
      { name_vi: "Món canh", name_en: "Soups", display_order: 20 },
      { name_vi: "Đồ uống", name_en: "Beverages", display_order: 30 }
    ]
  },
  
  other: {
    menu_name: "Thực Đơn",
    categories: [] // No defaults
  }
};
```

**Technical Requirements:**

```typescript
// After successful OTP verification and tenant creation:
POST /api/tenants/:tenantId/setup-defaults
{
  "businessType": "cafe"
}

// Response
{
  "menu": {
    "id": "uuid",
    "name_vi": "Thực Đơn",
    "categories": [
      {
        "id": "uuid",
        "name_vi": "Cà phê",
        "name_en": "Coffee",
        "items": [
          {
            "id": "uuid",
            "name_vi": "Cà phê sữa đá",
            "base_price": 25000,
            "status": "draft",
            "is_default": true
          }
        ]
      }
      // More categories...
    ]
  },
  "message": "Đã tạo danh mục mẫu thành công!"
}
```

**Database Operations:**

1. Create default menu: `INSERT INTO menus (tenant_id, name_vi, name_en)`
2. Create categories: `INSERT INTO categories (tenant_id, menu_id, name_vi, name_en, display_order)`
3. Create sample items (optional): `INSERT INTO menu_items (tenant_id, category_id, name_vi, base_price, status='draft', is_default=true)`
4. All defaults are editable - no special protection

**Onboarding Flow:**

1. User completes OTP verification → account created
2. System creates defaults based on business type
3. Redirect to dashboard with welcome modal:
   - "Chào mừng đến với [Platform Name]! 🎉"
   - "Chúng tôi đã tạo sẵn danh mục cho [Quán cà phê] của bạn"
   - "Bạn có thể chỉnh sửa, xóa, hoặc thêm món mới bất cứ lúc nào"
   - [Button: "Xem thực đơn của tôi"]
4. Owner can immediately start editing/adding photos/publishing

**Benefits:**

- ⚡ **Time to value:** Owner sees populated menu in 30 seconds (vs 30 minutes of manual work)
- 🎯 **Relevant defaults:** Categories match their business type
- ✏️ **Fully customizable:** Nothing is locked, all defaults are suggestions
- 📊 **Learning by example:** Sample items show proper formatting
- 🚀 **Lower abandonment:** Reduces "blank canvas syndrome"

---

### Feature 1.4: Tenant Slug Generation

**User Story:** As a system, I want to auto-generate unique subdomains so that each store has a web address.

**Acceptance Criteria:**

- [ ] Slug auto-generated from `businessName` during registration
- [ ] Conversion rules: lowercase, diacritics removed, spaces→hyphens, special chars removed
- [ ] Examples:
  - "Phở Hà Nội 24" → `pho-ha-noi-24`
  - "Cà Phê Sài Gòn!" → `ca-phe-sai-gon`
  - "Bánh Mì 123" → `banh-mi-123`
- [ ] If slug exists, append incrementing number: `pho-ha-noi-24-2`
- [ ] Slug validation: 3-50 characters, only `a-z0-9-`
- [ ] Owner can edit slug once during onboarding (before first publish)
- [ ] After publish, slug is locked (requires support ticket to change)

**Technical Requirements:**

```typescript
// Slug generation function
function generateSlug(businessName: string): Promise<string> {
  // 1. Convert to lowercase
  // 2. Remove Vietnamese diacritics (à→a, ô→o, etc.)
  // 3. Replace spaces with hyphens
  // 4. Remove special characters
  // 5. Check uniqueness, append number if needed
}

// Database check
SELECT slug FROM tenants WHERE slug LIKE 'pho-ha-noi-24%' AND deleted_at IS NULL;
```

---

## Multi-Location Setup (Chains)

### Feature 2.0: Location Management for Chains

**User Story:** As a chain owner, I want to add my multiple locations so that I can manage them separately and compare performance.

**Acceptance Criteria:**

**During Onboarding:**

- [ ] After business type selection, ask: "Bạn có nhiều địa điểm kinh doanh không?" (Do you have multiple locations?)
- [ ] If YES:
  - Mark tenant as `is_chain = true`
  - Show location setup wizard
  - Prompt to add first location: name, address, phone, manager
  - Option to add more locations or skip (can add later)
- [ ] If NO:
  - Create default location with business name
  - Continue to regular onboarding

**Location Management Screen:**

- [ ] Accessible from Settings > Locations
- [ ] List all locations with:
  - Location name
  - Address
  - Status (Active/Closed)
  - Manager name
  - Actions: Edit, View Analytics, Deactivate
- [ ] "Thêm địa điểm mới" button
- [ ] Location form fields:
  - **Tên địa điểm:** Required (e.g., "Highlands Coffee - Nguyễn Huệ")
  - **Mã địa điểm:** Optional internal code (e.g., "HCF-HCM-001")
  - **Địa chỉ:** Required full address
  - **Quận/Huyện:** Dropdown
  - **Tỉnh/Thành phố:** Dropdown
  - **Tọa độ:** Auto-filled from address (Google Maps API) or manual
  - **Số điện thoại:** Location phone
  - **Quản lý:** Manager name & phone
  - **Giờ mở cửa:** Per-day configuration (Mon-Sun)
  - **Trạng thái:** Active/Temporarily Closed

**Location Selector (Global):**

- [ ] If chain (`is_chain = true`), show location dropdown in header
- [ ] Dropdown lists all active locations
- [ ] Special option: "Tất cả địa điểm" (All Locations) - for chain-wide view
- [ ] Selected location persists across pages (stored in session)
- [ ] Analytics, menu, inventory scoped to selected location

**Technical Requirements:**

```typescript
// API Endpoint - List Locations
GET /api/tenants/:tenantId/locations

// Response
{
  "locations": [
    {
      "id": "uuid",
      "location_name": "Highlands Coffee - Nguyễn Huệ",
      "location_code": "HCF-HCM-001",
      "address": "123 Nguyễn Huệ, Quận 1, TP.HCM",
      "city": "Ho Chi Minh City",
      "district": "Quận 1",
      "phone": "+84 28 1234 5678",
      "manager_name": "Nguyễn Văn A",
      "is_active": true,
      "is_primary": true,
      "opened_at": "2020-05-15"
    }
    // ... more locations
  ]
}

// API Endpoint - Create Location
POST /api/tenants/:tenantId/locations
{
  "location_name": "Highlands Coffee - Lê Lợi",
  "location_code": "HCF-HCM-002",
  "address": "456 Lê Lợi, Quận 1, TP.HCM",
  "city": "Ho Chi Minh City",
  "district": "Quận 1",
  "phone": "+84 28 9876 5432",
  "manager_name": "Trần Thị B",
  "operating_hours": {...}
}

// Response
{
  "id": "uuid",
  "location_name": "Highlands Coffee - Lê Lợi",
  "slug": "le-loi",
  "created_at": "2025-10-21T10:00:00Z"
}
```

**Database Operations:**

1. Check if tenant has `is_chain = true`
2. Insert into `locations` table
3. Auto-generate `slug` from location name
4. Trigger updates `tenants.total_locations` count
5. If first location for new tenant, set `is_primary = true`

**Edge Cases:**

- Single-location tenant wants to add 2nd location → automatically upgrade to `is_chain = true`
- Location limit based on subscription tier (Free: 1, Growth: 3, Pro: 10)
- Cannot delete last active location
- Closing location (not deletion) preserves historical analytics data

---

## Admin Dashboard

### Feature 3.1: Dashboard Home Screen (Multi-Location Aware)

**User Story:** As a chain owner, I want to see performance across all my locations so that I can identify which stores need attention.

**Updated Acceptance Criteria (Chain-Focused):**

- [ ] Mobile-first responsive layout (works on 375px screens)
- [ ] **Location selector in header** (if chain): dropdown to switch between locations or view "All Locations"
- [ ] Displays tenant business name and logo at top
- [ ] **New: Cross-Location Comparison Cards** (if chain, viewing "All Locations"):
  - Best performing location (by revenue today)
  - Worst performing location (needs attention)
  - Total revenue across all locations
  - Average order value comparison (highest vs lowest location)
- [ ] **Single Location View:**
  - Today's revenue with yesterday comparison (% change, ↑/↓)
  - Total orders today
  - Top 3 selling items
  - Low stock alerts
  - Quick actions (Add Menu Item, View Analytics, Log Sales)
- [ ] **AI Recommendations Section** (prioritized):
  - "Location A sells 2x more cà phê sữa đá in afternoons - adjust inventory"
  - "Location B has high waste on rainy days - reduce phở orders"
  - "Item X hasn't sold in 3 days - try 20% discount"
  - Note: Basic rule-based for MVP, ML-powered in Phase 2
- [ ] Navigation menu with icons: Home, **Analytics** (new priority), Menu, Locations, Settings
- [ ] User profile dropdown: Account Settings, Logout
- [ ] All text in Vietnamese by default

**Technical Requirements:**

```typescript
// API Endpoint
GET /api/dashboard/overview

// Response
{
  "tenant": {
    "id": "uuid",
    "businessName": "Phở Hà Nội 24",
    "slug": "pho-ha-noi-24",
    "logoUrl": "https://cdn.../logo.jpg"
  },
  "stats": {
    "menuItemsPublished": 15,
    "menuItemsDraft": 3,
    "websiteStatus": "published",
    "websiteUrl": "https://pho-ha-noi-24.ourplatform.com"
  },
  "subscription": {
    "planName": "Gói Miễn Phí",
    "currentPeriodEnd": "2026-01-21T00:00:00Z",
    "maxMenuItems": 20,
    "usedMenuItems": 18
  }
}
```

**UI Requirements:**

- Touch targets minimum 44x44px
- Loading states for all API calls
- Offline indicator when network unavailable
- Vietnamese number formatting (e.g., `18` items, not `18` món)

---

## Menu Management

### Feature 3.1: Create Menu Item

**User Story:** As a store owner, I want to add a new dish to my menu so that customers can see it on my website.

**Acceptance Criteria:**

- [ ] Form fields (all in Vietnamese):
  - **Tên món (Vietnamese):** Required, max 255 chars, text input
  - **Tên món (English):** Optional, max 255 chars, text input
  - **Mô tả (Vietnamese):** Optional, textarea, max 1000 chars
  - **Mô tả (English):** Optional, textarea, max 1000 chars
  - **Giá:** Required, number input, min 0, max 999,999,999 (VND) - Current selling price
  - **Giá so sánh (Giá gốc):** Optional (for showing discounts), must be ≥ base price
    - Purpose: Original/regular price before discount
    - If set, display as strikethrough above current price
    - Example: ~~₫75,000~~ **₫65,000** (save ₫10,000)
  - **Danh mục:** Dropdown select from existing categories
  - **SKU (Mã sản phẩm):** Optional, alphanumeric, max 100 chars
    - **Purpose:** Internal inventory tracking only (NOT shown to customers)
    - Used for: POS integration, delivery partner order matching, accounting reports
    - Example: `PHO-001`, `CAFE-SDA-M` (cafe sữa đá - medium)
  - **Theo dõi tồn kho:** Checkbox (default unchecked)
    - If checked: show "Số lượng" (number) and "Cảnh báo tồn kho thấp" (number) fields
  - **Trạng thái:** Radio buttons (Nháp / Xuất bản)
  - **Nổi bật:** Checkbox
  - **Tags:** Checkboxes (Cay / Chay / Thuần Chay)
- [ ] Validation on submit:
  - Name (vi) and Price are required
  - Price must be positive number
  - If compare_at_price set, must be ≥ base_price (cannot show "discount" where compare price is lower)
  - If track_inventory true, stock_quantity is required
- [ ] Success: redirect to menu list with toast "Đã thêm món thành công"
- [ ] Error: show field-specific error messages inline

**Technical Requirements:**

```typescript
// API Endpoint
POST /api/tenants/:tenantId/menu-items

{
  "name_vi": "Phở Bò Tái",
  "name_en": "Rare Beef Pho",
  "description_vi": "Phở bò tái mềm, nước dùng thanh ngọt",
  "description_en": "Tender rare beef pho with savory broth",
  "base_price": 65000,        // Current selling price (discounted)
  "compare_at_price": 75000,  // Original price (shows as strikethrough)
  "category_id": "uuid-category",
  "sku": "PHO-001",
  "track_inventory": false,
  "status": "published",
  "is_featured": false,
  "is_spicy": false,
  "is_vegetarian": false,
  "is_vegan": false
}

// Response
{
  "id": "uuid",
  "name_vi": "Phở Bò Tái",
  "base_price": 75000,
  "status": "published",
  "display_order": 0,
  "created_at": "2025-10-21T10:30:00Z"
}
```

**Database Operations:**

1. Validate tenant_id matches authenticated user
2. Verify category_id belongs to same tenant
3. Check if tenant has reached max_menu_items limit (subscription constraint)
4. Insert into `menu_items` with `display_order = MAX(display_order) + 10` for category
5. If status='published', set `published_at = NOW()`

**Edge Cases:**

- Free tier limit reached (20 items) → show upgrade prompt
- Duplicate SKU within tenant → return "SKU đã tồn tại"
- No categories exist → prompt to create category first

---

### Feature 3.2: Edit Menu Item

**User Story:** As a store owner, I want to update my menu item details so that information stays current.

**Acceptance Criteria:**

- [ ] Same form as Create, pre-filled with existing values
- [ ] All fields editable except `id` and `created_at`
- [ ] Changing status from 'draft' to 'published' sets `published_at = NOW()`
- [ ] Changing status from 'published' to 'draft' does NOT clear `published_at`
- [ ] Save button triggers validation and updates database
- [ ] Success: stay on edit page with toast "Đã cập nhật món"
- [ ] Unsaved changes warning if user navigates away

**Technical Requirements:**

```typescript
// API Endpoint
PATCH /api/tenants/:tenantId/menu-items/:itemId

{
  "base_price": 80000,  // Price updated from 75k to 80k
  "is_featured": true
}

// Response (full item object)
{
  "id": "uuid",
  "name_vi": "Phở Bò Tái",
  "base_price": 80000,
  "is_featured": true,
  "updated_at": "2025-10-21T11:15:00Z"
}
```

**Business Rules:**

- Only fields included in PATCH body are updated (partial update)
- Changing SKU triggers uniqueness check within tenant
- If `track_inventory` changes from true→false, clear stock_quantity and low_stock_threshold

---

### Feature 3.3: Delete Menu Item (Soft Delete)

**User Story:** As a store owner, I want to remove outdated items without losing historical data.

**Acceptance Criteria:**

- [ ] Delete button shows confirmation modal: "Xóa món này? Không thể hoàn tác."
- [ ] Confirm → sets `deleted_at = NOW()` (soft delete)
- [ ] Item disappears from admin menu list and customer storefront
- [ ] Item still exists in database with `deleted_at` timestamp
- [ ] Related images, variants, add-ons also soft-deleted (cascade)
- [ ] Future feature: "Trash" view to restore deleted items within 30 days

**Technical Requirements:**

```typescript
// API Endpoint
DELETE /api/tenants/:tenantId/menu-items/:itemId

// Response
{
  "success": true,
  "message": "Đã xóa món thành công"
}
```

**Database Operations:**

```sql
UPDATE menu_items SET deleted_at = NOW() WHERE id = ? AND tenant_id = ?;
UPDATE item_images SET deleted_at = NOW() WHERE menu_item_id = ?;
UPDATE item_variants SET deleted_at = NOW() WHERE menu_item_id = ?;
UPDATE item_add_ons SET deleted_at = NOW() WHERE menu_item_id = ?;
```

---

### Feature 3.4: Reorder Menu Items (Drag & Drop)

**User Story:** As a store owner, I want to arrange my menu items in a specific order so that popular dishes appear first.

**Acceptance Criteria:**

- [ ] Menu list shows drag handles (≡ icon) on each item
- [ ] Owner can drag items up/down to reorder within category
- [ ] Dragging updates `display_order` immediately (optimistic UI)
- [ ] Order persists after page refresh
- [ ] Works on mobile (touch drag) and desktop (mouse drag)
- [ ] Visual feedback during drag (item elevates, drop zones highlight)

**Technical Requirements:**

```typescript
// API Endpoint
POST /api/tenants/:tenantId/menu-items/reorder

{
  "category_id": "uuid",
  "item_ids": ["uuid-3", "uuid-1", "uuid-2"]  // New order
}

// Backend updates display_order:
// uuid-3 → display_order = 0
// uuid-1 → display_order = 10
// uuid-2 → display_order = 20
```

**Implementation Notes:**

- Use gaps of 10 between items for easier future insertions
- If owner drags item to different category, update both `category_id` and `display_order`
- Debounce API calls (wait 500ms after drag ends before saving)

---

### Feature 3.5: Manage Categories

**User Story:** As a store owner, I want to organize my menu into categories so that customers can browse easily.

**Acceptance Criteria:**

**Initial State (After Registration):**

- [ ] Owner sees pre-created categories based on business type (from Feature 1.3)
- [ ] Sample items (if created) are already assigned to categories
- [ ] All defaults are fully editable

**Create New Category:**

- [ ] Simple form: Name (vi), Name (en), Description (vi/en)
- [ ] Categories display in menu list as collapsible sections
- [ ] New category gets `display_order = MAX(display_order) + 10`

**Edit/Reorder Categories:**

- [ ] Categories can be reordered same as menu items (drag & drop)
- [ ] Can rename category without affecting items
- [ ] Can add/edit English translation anytime

**Delete Category:**

- [ ] Delete button shows prompt: "Danh mục này có X món. Chuyển các món sang đâu?"
- [ ] Options:
  - Select another category from dropdown
  - "Chưa phân loại" (Uncategorized - creates if doesn't exist)
- [ ] After reassignment, category is deleted
- [ ] If category is empty (0 items), delete immediately without prompt

**Technical Requirements:**

```typescript
// API Endpoint - Create Category
POST /api/tenants/:tenantId/categories

{
  "name_vi": "Phở",
  "name_en": "Pho",
  "description_vi": "Các loại phở đặc sản"
}

// Response
{
  "id": "uuid",
  "name_vi": "Phở",
  "display_order": 0,
  "item_count": 0,
  "is_default": false
}

// Delete with reassignment
DELETE /api/tenants/:tenantId/categories/:categoryId
{
  "move_items_to_category_id": "uuid-other-category"
}
```

**Edge Cases:**

- Deleting last category → show warning "Cần ít nhất 1 danh mục"
- Default categories can be deleted (no special protection)
- Uncategorized items automatically go to first category or "Chưa phân loại"

---

## Storefront Publishing

**Strategic Note:** Storefront is now **deprioritized (5% effort)** in MVP. It's a data collection endpoint, not the core value proposition. Focus on getting basic publishing working quickly to enable data collection for AI.

### Feature 6.1: Template Selection

**User Story:** As a store owner, I want to choose a design template so that my website looks professional.

**Acceptance Criteria:**

- [ ] Gallery view showing 3 pre-built templates:
  - **Classic Restaurant:** Red/gold accents, grid layout, hero image
  - **Modern Café:** Minimalist, pastel colors, single-column layout
  - **Street Food:** Vibrant colors, large food photos, casual vibe
- [ ] Each template shows preview thumbnail and name
- [ ] Click template → preview in modal with sample menu data
- [ ] "Chọn mẫu này" button applies template to tenant's site
- [ ] Template can be changed later (Settings > Website > Template)

**Technical Requirements:**

```typescript
// API Endpoint
GET /api/templates

[
  {
    "id": "classic-restaurant",
    "name_vi": "Nhà Hàng Truyền Thống",
    "name_en": "Classic Restaurant",
    "thumbnail_url": "https://cdn.../preview-classic.jpg",
    "preview_url": "https://demo.ourplatform.com/classic"
  },
  // ... 2 more templates
]

// Apply template
POST /api/tenants/:tenantId/sites
{
  "template_id": "classic-restaurant"
}
```

**Database Operations:**

- Insert/update `sites` table with `template_id`
- Set default colors based on template (stored in `primary_color`, `secondary_color`)

---

### Feature 4.2: Customize Branding

**User Story:** As a store owner, I want to add my logo and colors so that the website matches my brand.

**Acceptance Criteria:**

- [ ] Upload logo: max 5MB, formats JPG/PNG/SVG, recommended 500x500px
- [ ] Logo preview shows immediately after upload
- [ ] Color pickers for:
  - **Màu chính (Primary):** Used for headers, buttons (default from template)
  - **Màu phụ (Secondary):** Used for accents, links (default from template)
- [ ] Color preview updates site preview in real-time
- [ ] Reset button restores template defaults
- [ ] Save changes → updates `sites` table, invalidates CDN cache

**Technical Requirements:**

```typescript
// API Endpoint
PATCH /api/tenants/:tenantId/sites/:siteId

{
  "logo_url": "https://cdn.../tenant-uuid/logo.png",
  "primary_color": "#d32f2f",
  "secondary_color": "#fbc02d"
}
```

**Business Rules:**

- Logo upload triggers image optimization (resize to 500x500, compress)
- Colors must be valid hex codes (#RRGGBB)
- Free tier: 1 logo only. Growth+ tier: logo + favicon

---

### Feature 4.3: Publish Website

**User Story:** As a store owner, I want to publish my website so that customers can access it online.

**Acceptance Criteria:**

- [ ] "Xuất bản" button on Website settings page
- [ ] Pre-publish validation checklist:
  - ✓ At least 1 published menu item
  - ✓ Site title set (defaults to business name)
  - ✓ Template selected
- [ ] Click "Xuất bản" → shows confirmation: "Website của bạn sẽ trực tuyến tại: `{slug}.ourplatform.com`"
- [ ] Confirm → sets `sites.status = 'published'`, `published_at = NOW()`
- [ ] Success screen shows:
  - ✓ "Website đã được xuất bản!"
  - Link to live site (opens in new tab)
  - QR code for menu (auto-generated)
  - Share buttons (Facebook, Zalo)
- [ ] Unpublish button available after publishing (sets status='draft')

**Technical Requirements:**

```typescript
// API Endpoint
POST /api/tenants/:tenantId/sites/:siteId/publish

// Response
{
  "status": "published",
  "published_at": "2025-10-21T12:00:00Z",
  "site_url": "https://pho-ha-noi-24.ourplatform.com",
  "qr_code_url": "https://cdn.../qr-codes/tenant-uuid.png"
}
```

**Background Jobs:**

1. Generate QR code pointing to site URL
2. Upload QR code to CDN
3. Trigger CDN cache invalidation (if site was previously published)
4. Send notification email: "Website của bạn đã trực tuyến!"

---

### Feature 4.4: Subdomain Routing

**User Story:** As a customer, I want to visit `{slug}.ourplatform.com` and see the store's menu.

**Acceptance Criteria:**

- [ ] Subdomain format: `{tenant.slug}.ourplatform.com`
- [ ] DNS wildcard configured: `*.ourplatform.com` → platform server
- [ ] Server extracts subdomain, queries `sites` table by subdomain
- [ ] If site not found or status='draft' → show 404 page
- [ ] If site status='published' → render storefront with tenant's data
- [ ] Storefront displays:
  - Site title and logo
  - Menu items grouped by category
  - Prices formatted in VND (₫75,000 not 75000)
  - Item images (if uploaded)
  - Template styling (colors, fonts)
  - **SKU NOT displayed** (internal use only)
- [ ] Mobile-responsive (works on 375px screens)
- [ ] Fast load time: <2s Time to Interactive on 4G

**Technical Requirements:**

```typescript
// Server-side routing (Next.js middleware)
export function middleware(req: NextRequest) {
  const hostname = req.headers.get('host');
  const subdomain = hostname?.split('.')[0];
  
  if (subdomain && subdomain !== 'www' && subdomain !== 'admin') {
    // Fetch site by subdomain
    const site = await db.sites.findOne({ subdomain });
    
    if (site?.status === 'published') {
      return rewrite(`/storefront?tenantId=${site.tenant_id}`);
    } else {
      return rewrite('/404');
    }
  }
}

// Storefront data fetching
GET /api/public/storefronts/:subdomain

{
  "site": {
    "title_vi": "Phở Hà Nội 24",
    "logo_url": "...",
    "primary_color": "#d32f2f"
  },
  "menus": [
    {
      "id": "uuid",
      "name_vi": "Thực Đơn Chính",
      "categories": [
        {
          "id": "uuid",
          "name_vi": "Phở",
          "items": [
            {
              "id": "uuid",
              "name_vi": "Phở Bò Tái",
              "description_vi": "...",
              "base_price": 75000,
              "thumbnail_url": "...",
              "is_spicy": false
              // Note: SKU field excluded from public API response
            }
          ]
        }
      ]
    }
  ]
}
```

**Performance Optimizations:**

- Cache site + menu data in Redis (TTL 5 minutes)
- CDN caching for static assets (logo, images)
- Database query uses index: `idx_sites_subdomain`

---

## Image Upload & Management

### Feature 7.1: Upload Menu Item Image

**User Story:** As a store owner, I want to add photos of my dishes so that customers can see what they look like.

**Acceptance Criteria:**

- [ ] Upload button on menu item create/edit form
- [ ] Supports drag-and-drop or file picker
- [ ] Accepted formats: JPG, PNG, WebP
- [ ] Max file size: 5MB (enforced client and server-side)
- [ ] Upload progress indicator (percentage bar)
- [ ] After upload: show thumbnail preview with remove button
- [ ] Multiple images allowed (up to 5 per item for Free tier)
- [ ] First uploaded image becomes `thumbnail_url` automatically
- [ ] Can set different image as primary (updates `is_primary` flag)

**Technical Requirements:**

```typescript
// API Endpoint
POST /api/tenants/:tenantId/menu-items/:itemId/images
Content-Type: multipart/form-data

{
  "image": File,
  "alt_text_vi": "Phở bò tái ngon"
}

// Response
{
  "id": "uuid",
  "original_url": "https://cdn.../originals/image-uuid.jpg",
  "thumbnail_url": "https://cdn.../thumbnails/image-uuid.jpg",
  "medium_url": "https://cdn.../medium/image-uuid.jpg",
  "is_primary": true,
  "display_order": 0
}
```

**Background Processing:**

1. Upload original to S3-compatible storage
2. Generate thumbnails:
   - Thumbnail: 300x300px (for admin lists)
   - Medium: 800x800px (for storefront)
   - Large: 1200x1200px (for lightbox)
3. Optimize images (compress, strip EXIF, convert to WebP)
4. Store URLs in `item_images` table

**Edge Cases:**

- Image upload fails → show retry button
- File too large → "Kích thước tối đa 5MB"
- Invalid format → "Chỉ chấp nhận JPG, PNG, WebP"
- Free tier limit (5 images) → "Nâng cấp để thêm ảnh"

---

## Subscription Management

### Feature 8.1: View Current Plan

**User Story:** As a store owner, I want to see my subscription details so that I know my limits and benefits.

**Acceptance Criteria:**

- [ ] Subscription page shows current plan card with:
  - Plan name (Gói Miễn Phí / Gói Phát Triển / Gói Chuyên Nghiệp)
  - **Updated pricing:** (₫0 / ₫500,000/tháng / ₫1,500,000/tháng)
  - Current usage vs limits:
    - **NEW: Locations:** 1/1 (Free), 2/3 (Growth), 5/10 (Pro)
    - Menu items: 18/20 (Free), unlimited (Growth+)
    - AI recommendations: ✗ (Free), Basic (Growth), Advanced (Pro)
    - Storage: 45MB/100MB (Free), 1GB (Growth), 10GB (Pro)
  - Features included:
    - ✓ Single location (Free)
    - ✗ Multi-location (Growth+)
    - ✗ AI recommendations (Growth+)
    - ✗ Cross-location analytics (Growth+)
    - ✗ Business consultant calls (Pro)
  - Renewal date or "Không giới hạn" for Free tier
- [ ] "Nâng cấp gói" button (for Free tier users) - emphasize multi-location + AI value
- [ ] "Quản lý gói" button (for paid tier users)

**Technical Requirements:**

```typescript
// API Endpoint
GET /api/tenants/:tenantId/subscriptions/current

{
  "subscription": {
    "plan": {
      "name": "starter",
      "display_name_vi": "Gói Miễn Phí",
      "price_vnd": 0,
      "max_menu_items": 20,
      "max_images_per_item": 5,
      "max_storage_mb": 100,
      "custom_domain_enabled": false,
      "remove_branding": false
    },
    "status": "active",
    "current_period_end": null
  },
  "usage": {
    "menu_items": 18,
    "total_images": 54,
    "storage_mb": 45.2
  }
}
```

---

### Feature 6.2: Upgrade Plan (Placeholder)

**User Story:** As a store owner, I want to upgrade to a paid plan so that I can add more menu items.

**Acceptance Criteria:**

- [ ] Pricing comparison table shows all 3 tiers side-by-side
- [ ] Highlight differences: Free (20 items), Growth (unlimited), Pro (unlimited + analytics)
- [ ] "Chọn gói" button for each tier
- [ ] For MVP: clicking button shows message: "Tính năng thanh toán đang phát triển. Vui lòng liên hệ `support@ourplatform.com` để nâng cấp."
- [ ] Future: integrate MoMo/ZaloPay payment flow

**Technical Requirements:**

```typescript
// Placeholder endpoint (returns not implemented)
POST /api/tenants/:tenantId/subscriptions/upgrade

{
  "plan_id": "uuid-growth-plan",
  "billing_interval": "monthly"
}

// Response
{
  "error": "FEATURE_NOT_IMPLEMENTED",
  "message": "Tính năng thanh toán đang phát triển",
  "contact_email": "support@ourplatform.com"
}
```

---

## Non-Functional Requirements

### Performance

- [ ] Admin dashboard pages load in <2s on 4G
- [ ] Storefront pages load in <2s on 4G
- [ ] Image uploads complete in <5s for 2MB files
- [ ] API response time p95 <500ms

### Accessibility

- [ ] All forms keyboard-navigable (tab order logical)
- [ ] Touch targets minimum 44x44px on mobile
- [ ] Color contrast ratio ≥4.5:1 for text
- [ ] Alt text for all images

### Browser Support

- [ ] Chrome/Edge (latest 2 versions)
- [ ] Safari iOS (latest 2 versions)
- [ ] Chrome Android (latest 2 versions)
- [ ] No IE11 support required

### Localization

- [ ] All UI text in Vietnamese by default
- [ ] Currency formatted as ₫75,000 (not 75000 VND)
- [ ] Dates formatted as dd/mm/yyyy (not mm/dd/yyyy)
- [ ] Phone numbers formatted as +84 90 123 4567

---

## Analytics Dashboard (MVP Priority)

**Strategic Note:** This is now the **core product focus (80% effort)**. Menu/storefront are data collection tools (15%). See [product-strategy-refined.md](./product-strategy-refined.md) and [analytics-ai-strategy.md](./analytics-ai-strategy.md) for full strategy.

### Feature 5.1: Multi-Location Performance Comparison

**User Story:** As a chain owner, I want to compare my locations' performance so that I can identify what works where and replicate success.

**Acceptance Criteria:**

**Cross-Location Dashboard ("All Locations" view):**

- [ ] Table comparing all locations side-by-side:
  - Location name
  - Today's revenue (with % change from yesterday)
  - Today's orders
  - Average order value
  - Top selling item
  - Status indicator (🟢 performing well, 🟡 below average, 🔴 needs attention)
- [ ] Sort by any column (revenue, orders, AOV)
- [ ] Click location row → drill down to location-specific analytics
- [ ] "Export CSV" button for reporting

**Location Performance Heatmap:**

- [ ] Visual map showing location performance by color intensity
- [ ] Hover shows quick stats
- [ ] Click to drill down

**AI Insights for Chains:**

- [ ] "Location A sells 2x more [item] than Location B - consider local marketing"
- [ ] "Your District 1 stores have 30% higher AOV - focus premium items there"
- [ ] "Location C inventory waste is 2x chain average - investigate"

**Technical Requirements:**

```typescript
// API Endpoint - Chain-Wide Comparison
GET /api/tenants/:tenantId/analytics/locations-comparison?date=today

// Response
{
  "date": "2025-10-21",
  "locations": [
    {
      "location_id": "uuid",
      "location_name": "Highlands Coffee - Nguyễn Huệ",
      "revenue_today": 2500000,
      "revenue_yesterday": 2100000,
      "change_pct": 19.0,
      "orders_today": 87,
      "avg_order_value": 28736,
      "top_item": "Cà phê sữa đá",
      "status": "good" // good | average | needs_attention
    },
    {
      "location_id": "uuid-2",
      "location_name": "Highlands Coffee - Lê Lợi",
      "revenue_today": 1800000,
      "revenue_yesterday": 1950000,
      "change_pct": -7.7,
      "orders_today": 65,
      "avg_order_value": 27692,
      "top_item": "Bạc xỉu",
      "status": "needs_attention"
    }
  ],
  "chain_totals": {
    "total_revenue": 4300000,
    "total_orders": 152,
    "avg_order_value": 28289
  },
  "ai_insights": [
    {
      "type": "performance_gap",
      "priority": "high",
      "message": "Địa điểm Nguyễn Huệ bán được nhiều cà phê sữa đá hơn 2 lần so với Lê Lợi. Xem xét chiến lược marketing tại Lê Lợi.",
      "actionable": true,
      "actions": ["View details", "Create promotion"]
    }
  ]
}
```

---

### Feature 5.2: Location-Specific Daily Business Overview

**User Story:** As a store owner/manager, I want to see today's performance for my location so that I can understand what sold well and plan for tomorrow.

**User Story:** As a store owner, I want to see today's performance at end of day so that I can understand what sold well and plan for tomorrow.

**Acceptance Criteria (Per-Location View):**

**Context:** When viewing a specific location (not "All Locations" chain view)

**Summary Cards (Today's Metrics):**

- [ ] Display total revenue today (VND) with comparison to yesterday (% change, ↑/↓ indicator)
- [ ] Display total orders today with comparison to yesterday
- [ ] Display average order value with comparison to yesterday
- [ ] **NEW: Location rank** (if chain): "#2 of 5 locations" with percentile
- [ ] All Vietnamese number formatting (₫1,250,000 not 1250000)

**Top Performing Items:**

- [ ] Show top 5 best-selling items by quantity sold
- [ ] Display: item name, quantity, total revenue
- [ ] Visual ranking indicators (🥇🥈🥉 for top 3)

**Items Needing Attention:**

- [ ] Section: "Món ế ẩm" (Slow-moving items)
  - Show items with 0 sales today
  - Show items with < 2 sales in last 3 days
- [ ] Section: "Tồn kho thấp" (Low stock items)
  - Show items where stock < threshold
  - Display current stock quantity

**Revenue Chart (7 Days):**

- [ ] Line chart showing daily revenue for last 7 days
- [ ] Highlight current day with different color
- [ ] Show average line for comparison
- [ ] Mobile-responsive design

**Date Range Filter:**

- [ ] Quick filters: Hôm nay (default), Hôm qua, 7 ngày, 30 ngày
- [ ] All metrics update when filter changes

**Technical Requirements:**

```typescript
// API Endpoint
GET /api/tenants/:tenantId/analytics/dashboard?date=today

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
    }
  },
  "top_items": [
    {
      "menu_item_id": "uuid",
      "name_vi": "Cà phê sữa đá",
      "quantity_sold": 18,
      "revenue": 450000,
      "rank": 1
    }
    // ... top 5
  ],
  "slow_moving_items": [
    {
      "menu_item_id": "uuid",
      "name_vi": "Trà đào cam sả",
      "days_since_last_sale": 3
    }
  ],
  "low_stock_items": [
    {
      "menu_item_id": "uuid",
      "name_vi": "Phở bò",
      "current_stock": 5,
      "threshold": 10
    }
  ],
  "revenue_chart": {
    "labels": ["15/10", "16/10", "17/10", "18/10", "19/10", "20/10", "21/10"],
    "data": [950000, 1100000, 850000, 1300000, 1050000, 1087000, 1250000]
  }
}
```

**Data Source (MVP - Manual Entry):**

For MVP without POS integration:

- [ ] Owner manually logs sales at end of day
- [ ] Simple form: "Hôm nay bán được bao nhiêu?" (How much did you sell today?)
- [ ] Optional: Log individual item quantities sold
- [ ] System tracks storefront views automatically (frontend tracking)

**Future Enhancement:**

- Real-time data from POS integration
- Automated order tracking from delivery partners
- Hourly breakdown charts
- AI recommendations
- See [analytics-ai-strategy.md](./analytics-ai-strategy.md) for full roadmap

**Performance Requirements:**

- [ ] Dashboard loads in <1s (use pre-aggregated daily metrics)
- [ ] Charts render smoothly on mobile
- [ ] Offline support: show last cached data with timestamp

---

## Out of Scope for MVP

The following features are explicitly **NOT** included in MVP:

- ❌ Zalo Mini App integration
- ❌ Payment gateway implementation (MoMo, ZaloPay, VNPay)
- ❌ Custom domain support
- ❌ Online ordering / shopping cart
- ❌ Customer reviews
- ❌ Advanced analytics (AI recommendations, demand forecasting) - See roadmap in [analytics-ai-strategy.md](./analytics-ai-strategy.md)
- ❌ Email marketing
- ❌ Multi-language storefront switching
- ❌ Reservation system
- ❌ Delivery partner integrations (GrabFood, ShopeeFood)

---

## Definition of Done

A feature is considered complete when:

- ✅ Code implemented and merged to `main` branch
- ✅ Unit tests written and passing (>80% coverage for business logic)
- ✅ Integration tests passing for API endpoints
- ✅ Manual testing completed on staging environment
- ✅ Acceptance criteria verified by product owner
- ✅ Vietnamese translations reviewed by native speaker
- ✅ Performance benchmarks met (load time <2s)
- ✅ Documentation updated (API docs, user guide)

---

**Document Owner:** Product Manager
**Next Review:** Before Sprint 1 kickoff  
**Feedback:** Open an issue with label `mvp-acceptance-criteria`
