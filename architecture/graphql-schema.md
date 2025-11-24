# GraphQL Schema Definition

**Created:** October 22, 2025  
**Status:** Implementation Ready  
**Description:** Complete GraphQL schema for Local Store Platform API

## Overview

This document defines the complete GraphQL schema including:

- **Queries**: Read operations
- **Mutations**: Write operations  
- **Subscriptions**: Real-time updates via WebSocket
- **Types**: Object models with field definitions
- **Scalars**: Custom scalar types
- **Directives**: Authentication and authorization

---

## Custom Scalars

```graphql
scalar DateTime
scalar JSON
scalar Upload
```

---

## Enums

```graphql
enum UserRole {
  OWNER
  MANAGER
  STAFF
}

enum RecommendationType {
  PRICING
  INVENTORY
  PROMOTION
  CROSS_SELLING
  ONBOARDING
}

enum RecommendationPriority {
  HIGH
  MEDIUM
  LOW
}

enum RecommendationStatus {
  PENDING
  APPROVED
  DISMISSED
}

enum TransactionStatus {
  PENDING
  COMPLETED
  CANCELLED
  REFUNDED
}

enum PaymentMethod {
  CASH
  MOMO
  ZALOPAY
  VNPAY
  BANK_TRANSFER
}

enum SubscriptionTier {
  FREE
  BASIC
  PRO
  ENTERPRISE
}

enum LocationStatus {
  ACTIVE
  INACTIVE
  SUSPENDED
}
```

---

## Core Types

### User & Authentication

```graphql
type User {
  id: ID!
  phone: String!
  email: String
  fullName: String
  role: UserRole!
  isActive: Boolean!
  tenant: Tenant!
  
  # Multi-domain access control
  canAccessMainPlatform: Boolean!  # true for owners only
  canAccessSubdomain: String!      # tenant subdomain
  primaryLocation: Location
  canAccessLocations: [Location!]! # for managers with multi-location access
  
  # Invitation tracking
  invitedBy: User
  invitedAt: DateTime
  inviteMethod: String  # "phone_invite" or "permanent_code"
  
  createdAt: DateTime!
  updatedAt: DateTime!
}

type AuthPayload {
  accessToken: String!
  refreshToken: String!
  user: User!
  expiresIn: Int!
}

type OtpRequestPayload {
  message: String!
  expiresIn: Int!
}
```

### Tenant & Location

```graphql
type Tenant {
  id: ID!
  name: String!
  subdomain: String!  # URL-safe slug (e.g., "pho-bo-hanoi")
  phone: String!
  email: String
  status: String!
  subscriptionTier: SubscriptionTier!
  trialEndsAt: DateTime
  
  # Multi-domain URLs
  mainPlatformUrl: String!     # "https://app.localstoreplatform.com"
  storeOperationsUrl: String!  # "https://pho-bo-hanoi.localstoreplatform.com"
  publicMenuUrl: String        # "https://pho-bo-hanoi.lsp.menu" (Phase 2)
  
  # Onboarding configuration (PRO plan)
  onboardingConfig: OnboardingConfig
  
  locations: [Location!]!
  users: [User!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type OnboardingConfig {
  staffCode: String            # Permanent code: "STAFF2025"
  managerCode: String          # Permanent code: "MGR2025"
  autoApproveStaff: Boolean!
  autoApproveManagers: Boolean!
  maxUsers: Int!
  currentUsers: Int!
  joinUrl: String!  # "https://pho-bo-hanoi.localstoreplatform.com/join"
  qrCodeUrl: String # Generated QR code for posters
}

type Location {
  id: ID!
  name: String!
  address: String
  phone: String
  status: LocationStatus!
  tenant: Tenant!
  
  # Metrics (resolved via DataLoader)
  todayMetrics: DashboardMetrics
  
  # Relations
  menuItems: [MenuItem!]!
  transactions: [Transaction!]!
  recommendations: [Recommendation!]!
  
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Dashboard Metrics

```graphql
type DashboardMetrics {
  # Today's metrics
  todayRevenue: Float!
  todayOrders: Int!
  todayCustomers: Int!
  averageOrderValue: Float!
  
  # Comparisons
  revenueChange: Float!  # % change from yesterday
  ordersChange: Float!
  customersChange: Float!
  
  # Time data
  date: DateTime!
  hourlyTrends: [HourlyTrend!]!
  topItems: [TopItem!]!
}

type HourlyTrend {
  hour: Int!
  revenue: Float!
  orders: Int!
}

type TopItem {
  itemId: ID!
  itemName: String!
  category: String!
  quantity: Int!
  revenue: Float!
  imageUrl: String
}

type CrossLocationComparison {
  locationId: ID!
  locationName: String!
  revenue: Float!
  orders: Int!
  averageOrderValue: Float!
  topSellingItem: String
  revenueRank: Int!
}

type RevenueTrend {
  date: DateTime!
  revenue: Float!
  orders: Int!
  averageOrderValue: Float!
}
```

### AI Recommendations

```graphql
type Recommendation {
  id: ID!
  type: RecommendationType!
  priority: RecommendationPriority!
  status: RecommendationStatus!
  
  title: String!
  description: String!
  impact: String!
  
  # Action details (JSON)
  action: JSON!
  
  # Relations
  location: Location!
  
  # Metadata
  generatedAt: DateTime!
  approvedAt: DateTime
  dismissedAt: DateTime
  approvedBy: User
  
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Menu & Items

```graphql
type Category {
  id: ID!
  name: String!
  description: String
  sortOrder: Int!
  tenant: Tenant!
  items: [MenuItem!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type MenuItem {
  id: ID!
  name: String!
  description: String
  price: Float!
  imageUrl: String
  isAvailable: Boolean!
  sortOrder: Int!
  
  category: Category!
  location: Location!
  
  # Analytics
  totalSold: Int!
  revenue: Float!
  
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Sales & Transactions

```graphql
type Transaction {
  id: ID!
  transactionDate: DateTime!
  totalAmount: Float!
  discountAmount: Float!
  finalAmount: Float!
  paymentMethod: PaymentMethod!
  status: TransactionStatus!
  notes: String
  
  location: Location!
  items: [TransactionItem!]!
  
  createdAt: DateTime!
  updatedAt: DateTime!
}

type TransactionItem {
  id: ID!
  quantity: Int!
  unitPrice: Float!
  subtotal: Float!
  
  transaction: Transaction!
  menuItem: MenuItem!
  
  createdAt: DateTime!
}
```

### Notifications

```graphql
type Notification {
  id: ID!
  type: String!
  title: String!
  body: String!
  data: JSON
  
  isRead: Boolean!
  readAt: DateTime
  
  user: User!
  
  createdAt: DateTime!
  updatedAt: DateTime!
}

type NotificationDevice {
  id: ID!
  fcmToken: String!
  platform: String!  # ios, android, web
  deviceName: String
  isActive: Boolean!
  
  user: User!
  
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

---

## Queries

```graphql
type Query {
  # ==========================================
  # Authentication & User
  # ==========================================
  me: User!
  
  # ==========================================
  # User Management
  # ==========================================
  
  """List staff members (owner/manager only)"""
  staff(
    locationId: ID
    role: UserRole
    isActive: Boolean
    limit: Int
    offset: Int
  ): [User!]!
  
  """List pending invitations"""
  pendingInvitations(
    locationId: ID
    status: InvitationStatus
    limit: Int
    offset: Int
  ): [UserInvitation!]!
  
  # ==========================================
  # Dashboard
  # ==========================================
  
  """Get today's metrics for a location"""
  locationMetrics(locationId: ID!): DashboardMetrics!
  
  """Compare metrics across all tenant locations"""
  crossLocationComparison(
    tenantId: ID!
    startDate: DateTime!
    endDate: DateTime!
  ): [CrossLocationComparison!]!
  
  """Get revenue trends over time"""
  revenueTrends(
    locationId: ID!
    startDate: DateTime!
    endDate: DateTime!
    groupBy: String  # hour, day, week, month
  ): [RevenueTrend!]!
  
  # ==========================================
  # AI Recommendations
  # ==========================================
  
  """List AI recommendations for a location"""
  recommendations(
    locationId: ID!
    status: RecommendationStatus
    type: RecommendationType
    priority: RecommendationPriority
    limit: Int
    offset: Int
  ): RecommendationConnection!
  
  """Get single recommendation details"""
  recommendation(id: ID!): Recommendation!
  
  # ==========================================
  # Locations
  # ==========================================
  
  """List all locations for authenticated user's tenant"""
  locations(
    status: LocationStatus
    limit: Int
    offset: Int
  ): LocationConnection!
  
  """Get single location details"""
  location(id: ID!): Location!
  
  # ==========================================
  # Menu & Items
  # ==========================================
  
  """List menu categories"""
  categories(locationId: ID!): [Category!]!
  
  """List menu items with filtering"""
  menuItems(
    locationId: ID!
    categoryId: ID
    isAvailable: Boolean
    search: String
    limit: Int
    offset: Int
  ): MenuItemConnection!
  
  """Get single menu item"""
  menuItem(id: ID!): MenuItem!
  
  # ==========================================
  # Sales & Transactions
  # ==========================================
  
  """List transactions"""
  transactions(
    locationId: ID!
    startDate: DateTime
    endDate: DateTime
    paymentMethod: PaymentMethod
    status: TransactionStatus
    limit: Int
    offset: Int
  ): TransactionConnection!
  
  """Get single transaction"""
  transaction(id: ID!): Transaction!
  
  # ==========================================
  # Notifications
  # ==========================================
  
  """List notifications for authenticated user"""
  notifications(
    isRead: Boolean
    limit: Int
    offset: Int
  ): NotificationConnection!
  
  """Get unread notification count"""
  unreadNotificationCount: Int!
}

# ==========================================
# Connection Types (Pagination)
# ==========================================

type RecommendationConnection {
  edges: [RecommendationEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type RecommendationEdge {
  node: Recommendation!
  cursor: String!
}

type LocationConnection {
  edges: [LocationEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type LocationEdge {
  node: Location!
  cursor: String!
}

type MenuItemConnection {
  edges: [MenuItemEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type MenuItemEdge {
  node: MenuItem!
  cursor: String!
}

type TransactionConnection {
  edges: [TransactionEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type TransactionEdge {
  node: Transaction!
  cursor: String!
}

type NotificationConnection {
  edges: [NotificationEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type NotificationEdge {
  node: Notification!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

---

## Mutations

```graphql
type Mutation {
  # ==========================================
  # Authentication
  # ==========================================
  
  """Request OTP for phone number"""
  requestOtp(phone: String!, subdomain: String): OtpRequestPayload!
  
  """Verify OTP and receive JWT tokens"""
  verifyOtp(phone: String!, otp: String!, subdomain: String): AuthPayload!
  
  """Refresh access token"""
  refreshToken(refreshToken: String!): AuthPayload!
  
  """Logout (invalidate refresh token)"""
  logout: Boolean!
  
  # ==========================================
  # AI Recommendations
  # ==========================================
  
  """Approve a recommendation"""
  approveRecommendation(id: ID!): Recommendation!
  
  """Dismiss a recommendation"""
  dismissRecommendation(id: ID!, reason: String): Recommendation!
  
  """Manually trigger recommendation generation"""
  generateRecommendations(locationId: ID!): [Recommendation!]!
  
  # ==========================================
  # Locations
  # ==========================================
  
  """Create new location"""
  createLocation(input: CreateLocationInput!): Location!
  
  """Update location"""
  updateLocation(id: ID!, input: UpdateLocationInput!): Location!
  
  """Delete location (soft delete)"""
  deleteLocation(id: ID!): Boolean!
  
  # ==========================================
  # Menu & Items
  # ==========================================
  
  """Create category"""
  createCategory(input: CreateCategoryInput!): Category!
  
  """Update category"""
  updateCategory(id: ID!, input: UpdateCategoryInput!): Category!
  
  """Delete category"""
  deleteCategory(id: ID!): Boolean!
  
  """Create menu item"""
  createMenuItem(input: CreateMenuItemInput!): MenuItem!
  
  """Update menu item"""
  updateMenuItem(id: ID!, input: UpdateMenuItemInput!): MenuItem!
  
  """Delete menu item"""
  deleteMenuItem(id: ID!): Boolean!
  
  """Upload menu item image"""
  uploadMenuItemImage(itemId: ID!, file: Upload!): MenuItem!
  
  # ==========================================
  # Sales & Transactions
  # ==========================================
  
  """Create transaction (manual entry)"""
  createTransaction(input: CreateTransactionInput!): Transaction!
  
  """Update transaction"""
  updateTransaction(id: ID!, input: UpdateTransactionInput!): Transaction!
  
  """Cancel transaction"""
  cancelTransaction(id: ID!, reason: String): Transaction!
  
  # ==========================================
  # User Management & Invitations
  # ==========================================
  
  """Send phone invitation (SMS with link)"""
  inviteUser(input: InviteUserInput!): UserInvitation!
  
  """Generate permanent onboarding code (PRO plan)"""
  generateOnboardingCodes(config: OnboardingConfigInput!): OnboardingConfig!
  
  """Regenerate code if compromised"""
  regenerateCode(role: UserRole!): String!
  
  """Accept invitation via code or link"""
  acceptInvitation(inviteCode: String!, phone: String!): AuthPayload!
  
  """Approve pending user (if auto-approve disabled)"""
  approveUser(userId: ID!): User!
  
  """Remove staff member"""
  removeUser(userId: ID!): Boolean!
  
  """Update user permissions"""
  updateUserPermissions(userId: ID!, input: PermissionsInput!): User!
  
  # ==========================================
  # Tenant Management
  # ==========================================
  
  """Check subdomain availability during signup"""
  checkSubdomainAvailability(subdomain: String!): SubdomainAvailability!
  
  """Update tenant subdomain (one-time during onboarding)"""
  updateSubdomain(subdomain: String!): Tenant!
  
  # ==========================================
  # Notifications
  # ==========================================
  
  """Register FCM device token"""
  registerDevice(input: RegisterDeviceInput!): NotificationDevice!
  
  """Mark notification as read"""
  markNotificationRead(id: ID!): Notification!
  
  """Mark all notifications as read"""
  markAllNotificationsRead: Boolean!
  
  """Delete notification device"""
  deleteDevice(id: ID!): Boolean!
}

# ==========================================
# Input Types
# ==========================================

input CreateLocationInput {
  name: String!
  address: String
  phone: String
}

input UpdateLocationInput {
  name: String
  address: String
  phone: String
  status: LocationStatus
}

input CreateCategoryInput {
  name: String!
  description: String
  sortOrder: Int
}

input UpdateCategoryInput {
  name: String
  description: String
  sortOrder: Int
}

input CreateMenuItemInput {
  name: String!
  description: String
  price: Float!
  categoryId: ID!
  locationId: ID!
  isAvailable: Boolean
  sortOrder: Int
}

input UpdateMenuItemInput {
  name: String
  description: String
  price: Float
  categoryId: ID
  isAvailable: Boolean
  sortOrder: Int
}

input CreateTransactionInput {
  locationId: ID!
  transactionDate: DateTime!
  paymentMethod: PaymentMethod!
  items: [TransactionItemInput!]!
  discountAmount: Float
  notes: String
}

input TransactionItemInput {
  menuItemId: ID!
  quantity: Int!
  unitPrice: Float!
}

input UpdateTransactionInput {
  transactionDate: DateTime
  paymentMethod: PaymentMethod
  discountAmount: Float
  notes: String
}

input RegisterDeviceInput {
  fcmToken: String!
  platform: String!
  deviceName: String
}

# ==========================================
# User Management Input Types
# ==========================================

input InviteUserInput {
  phone: String!
  role: UserRole!
  locationId: ID         # null for multi-location managers
  method: InvitationMethod  # PHONE_INVITE (default) or PERMANENT_CODE
}

enum InvitationMethod {
  PHONE_INVITE      # Send SMS with link (small businesses)
  PERMANENT_CODE    # Use generated code (enterprise/chains)
}

input OnboardingConfigInput {
  autoApproveStaff: Boolean
  autoApproveManagers: Boolean
  customStaffCode: String    # Optional, auto-generate if null
  customManagerCode: String
}

input PermissionsInput {
  canAccessLocations: [ID!]
  canViewFinancials: Boolean
  canEditMenu: Boolean
  canManageStaff: Boolean
}

type SubdomainAvailability {
  available: Boolean!
  subdomain: String!
  suggestion: String  # Alternative if taken (e.g., "pho-bo-hanoi-2")
}

type UserInvitation {
  id: ID!
  tenant: Tenant!
  location: Location
  invitedBy: User
  phone: String!
  role: UserRole!
  
  # Multi-domain support
  subdomain: String!
  inviteCode: String!
  inviteLink: String!  # Full URL with code
  inviteMethod: InvitationMethod!
  
  # Approval workflow
  status: InvitationStatus!
  approvedBy: User
  approvedAt: DateTime
  
  expiresAt: DateTime  # null for permanent codes
  acceptedAt: DateTime
  createdAt: DateTime!
}

enum InvitationStatus {
  PENDING
  APPROVED
  REJECTED
  EXPIRED
  ACCEPTED
}
```

---

## Subscriptions

```graphql
type Subscription {
  # ==========================================
  # Dashboard Real-Time Updates
  # ==========================================
  
  """Subscribe to dashboard metrics updates"""
  dashboardUpdates(locationId: ID!): DashboardMetrics!
  
  """Subscribe to new transactions"""
  newTransaction(locationId: ID!): Transaction!
  
  # ==========================================
  # Notifications
  # ==========================================
  
  """Subscribe to new notifications"""
  newNotification: Notification!
  
  # ==========================================
  # AI Recommendations
  # ==========================================
  
  """Subscribe to new recommendations"""
  newRecommendation(locationId: ID!): Recommendation!
}
```

---

## Directives

```graphql
"""Require authentication (valid JWT)"""
directive @auth on FIELD_DEFINITION | OBJECT

"""Require specific user role"""
directive @hasRole(roles: [UserRole!]!) on FIELD_DEFINITION

"""Rate limit"""
directive @rateLimit(limit: Int!, window: Int!) on FIELD_DEFINITION
```

---

## Example Queries

### 1. Dashboard Home (Mobile)

```graphql
query DashboardHome($locationId: ID!) {
  location(id: $locationId) {
    id
    name
    todayMetrics {
      todayRevenue
      todayOrders
      todayCustomers
      averageOrderValue
      revenueChange
      ordersChange
      hourlyTrends {
        hour
        revenue
      }
      topItems {
        itemName
        quantity
        revenue
      }
    }
  }
  
  recommendations(locationId: $locationId, limit: 3, status: PENDING) {
    edges {
      node {
        id
        type
        priority
        title
        description
        impact
      }
    }
  }
}
```

### 2. Cross-Location Comparison

```graphql
query CrossLocationDashboard($tenantId: ID!, $startDate: DateTime!, $endDate: DateTime!) {
  crossLocationComparison(
    tenantId: $tenantId
    startDate: $startDate
    endDate: $endDate
  ) {
    locationId
    locationName
    revenue
    orders
    averageOrderValue
    topSellingItem
    revenueRank
  }
}
```

### 3. AI Recommendations List

```graphql
query RecommendationsList($locationId: ID!, $status: RecommendationStatus) {
  recommendations(locationId: $locationId, status: $status, limit: 20) {
    edges {
      node {
        id
        type
        priority
        status
        title
        description
        impact
        action
        generatedAt
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}
```

### 4. Menu Items with Performance

```graphql
query MenuPerformance($locationId: ID!) {
  menuItems(locationId: $locationId, limit: 100) {
    edges {
      node {
        id
        name
        price
        imageUrl
        isAvailable
        category {
          name
        }
        totalSold
        revenue
      }
    }
  }
}
```

---

## Example Mutations

### 1. Authentication Flow

```graphql
# Step 1: Request OTP
mutation RequestOTP {
  requestOtp(phone: "0901234567") {
    message
    expiresIn
  }
}

# Step 2: Verify OTP
mutation VerifyOTP {
  verifyOtp(phone: "0901234567", otp: "123456") {
    accessToken
    refreshToken
    expiresIn
    user {
      id
      phone
      role
      tenant {
        name
        subscriptionTier
      }
    }
  }
}

# Step 3: Refresh Token
mutation RefreshAccessToken($refreshToken: String!) {
  refreshToken(refreshToken: $refreshToken) {
    accessToken
    expiresIn
  }
}
```

### 2. Approve Recommendation

```graphql
mutation ApproveRec($id: ID!) {
  approveRecommendation(id: $id) {
    id
    status
    approvedAt
  }
}
```

### 3. Create Transaction (Manual Entry)

```graphql
mutation CreateSale($input: CreateTransactionInput!) {
  createTransaction(input: $input) {
    id
    transactionDate
    totalAmount
    finalAmount
    items {
      id
      menuItem {
        name
      }
      quantity
      unitPrice
      subtotal
    }
  }
}

# Variables
{
  "input": {
    "locationId": "location-123",
    "transactionDate": "2025-10-22T14:30:00Z",
    "paymentMethod": "CASH",
    "items": [
      {
        "menuItemId": "item-1",
        "quantity": 2,
        "unitPrice": 35000
      },
      {
        "menuItemId": "item-2",
        "quantity": 1,
        "unitPrice": 50000
      }
    ],
    "discountAmount": 10000
  }
}
```

### 4. Register FCM Device

```graphql
mutation RegisterFCM($input: RegisterDeviceInput!) {
  registerDevice(input: $input) {
    id
    fcmToken
    platform
    isActive
  }
}

# Variables
{
  "input": {
    "fcmToken": "fYh7x...FCM_TOKEN...zK9p",
    "platform": "ios",
    "deviceName": "iPhone 15 Pro"
  }
}
```

---

## Example Subscriptions

### 1. Live Dashboard Updates

```graphql
subscription LiveDashboard($locationId: ID!) {
  dashboardUpdates(locationId: $locationId) {
    todayRevenue
    todayOrders
    todayCustomers
    averageOrderValue
  }
}
```

### 2. New Transaction Notifications

```graphql
subscription NewSales($locationId: ID!) {
  newTransaction(locationId: $locationId) {
    id
    transactionDate
    finalAmount
    paymentMethod
    items {
      menuItem {
        name
      }
      quantity
      subtotal
    }
  }
}
```

### 3. Push Notifications

```graphql
subscription NotificationStream {
  newNotification {
    id
    type
    title
    body
    data
    createdAt
  }
}
```

---

## Error Handling

All GraphQL errors follow this structure:

```json
{
  "errors": [
    {
      "message": "Mã OTP không chính xác.",
      "extensions": {
        "code": "UNAUTHENTICATED",
        "timestamp": "2025-10-22T10:30:00Z"
      },
      "path": ["verifyOtp"]
    }
  ],
  "data": null
}
```

### Error Codes

- `UNAUTHENTICATED`: Invalid credentials or expired token
- `FORBIDDEN`: Insufficient permissions
- `BAD_USER_INPUT`: Validation error
- `NOT_FOUND`: Resource not found
- `INTERNAL_SERVER_ERROR`: Server error
- `RATE_LIMIT_EXCEEDED`: Too many requests

---

## Caching Strategy

### Query-Level Caching

```typescript
// Example: Cache dashboard metrics for 60 seconds
@Query(() => DashboardMetrics)
@CacheControl({ maxAge: 60 })
async locationMetrics(@Args('locationId') locationId: string) {
  // ...
}
```

### DataLoader (N+1 Prevention)

```typescript
// Batch load today's metrics for multiple locations
const metricsLoader = new DataLoader(async (locationIds: string[]) => {
  const metrics = await this.dashboardService.getBatchMetrics(locationIds);
  return locationIds.map(id => metrics[id]);
});
```

### Redis Cache Keys

- `dashboard:{location_id}:{date}` → Dashboard metrics (60s TTL)
- `recommendations:{location_id}` → AI recommendations (300s TTL)
- `menu:{location_id}` → Menu items (3600s TTL)
- `user:{user_id}` → User profile (1800s TTL)

---

## Implementation Notes

1. **Authentication**: All queries/mutations (except `requestOtp`, `verifyOtp`) require JWT in `Authorization: Bearer <token>` header
2. **Multi-tenancy**: Automatically scoped by authenticated user's `tenantId` in GraphQL context
3. **Location Scoping**: Verify user has access to requested `locationId` in resolver guards
4. **Pagination**: Use cursor-based pagination (relay-style connections) for scalability
5. **Real-time**: WebSocket subscriptions auto-join rooms based on `locationId` or `userId`
6. **Rate Limiting**: Apply per-endpoint rate limits using `@rateLimit` directive
7. **Monitoring**: Log slow queries (>1s), track resolver performance with Apollo Tracing

**Ready for NestJS implementation!** 🚀
