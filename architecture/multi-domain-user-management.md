# Multi-Domain User Management Architecture

**Created:** October 22, 2025  
**Status:** Implementation Ready  
**Feature:** Tenant subdomain isolation + Role-based access + Scalable onboarding

## Overview

This document defines the multi-domain architecture for user management, enabling:

- **Tenant isolation** by subdomain (pho-bo-hanoi.lsp.com vs banh-mi-saigon.lsp.com)
- **Role-based access** (owners on main platform, staff on tenant subdomains)
- **Scalable onboarding** (phone invites for small businesses, permanent codes for chains)

---

## Domain Architecture

### 1. Main Platform (app.localstoreplatform.com)

**Access:** Owners only

**Purpose:** Strategic management across all locations

**Features:**

- Cross-location analytics dashboard
- AI recommendations (strategic insights)
- Staff management (invite, remove, permissions)
- Subscription & billing
- Integration settings (MoMo, ZaloPay, POS)
- Multi-location configuration

**Authentication Flow:**

```typescript
// Owner logs in at app.localstoreplatform.com
POST /api/v1/auth/otp/verify
{
  "phone": "0901234567",
  "otp": "123456",
  "subdomain": null  // or "app"
}

// Response
{
  "accessToken": "eyJ...",
  "user": {
    "id": "user-123",
    "role": "owner",
    "canAccessMainPlatform": true,
    "canAccessSubdomain": "pho-bo-hanoi",
    "tenant": {
      "id": "tenant-123",
      "subdomain": "pho-bo-hanoi",
      "name": "Phở Bò Hà Nội",
      "mainPlatformUrl": "https://app.localstoreplatform.com",
      "storeOperationsUrl": "https://pho-bo-hanoi.localstoreplatform.com"
    }
  }
}
```

---

### 2. Tenant Subdomains ({tenant-slug}.localstoreplatform.com)

**Access:** Owner + Managers + Staff (tenant-specific)

**Purpose:** Daily operations for specific business

**URL Pattern:** `{tenant-slug}.localstoreplatform.com`

**Examples:**

- `pho-bo-hanoi.localstoreplatform.com` → Phở Bò Hà Nội
- `banh-mi-saigon.localstoreplatform.com` → Bánh Mì Sài Gòn
- `cafe-sua-da.localstoreplatform.com` → Cafe Sữa Đá

**Features (Role-dependent):**

- Location-specific dashboard (today's metrics)
- Sales entry (primary staff function)
- Menu management (manager can edit, staff can toggle availability)
- Inventory updates
- Shift clock in/out (future)
- Limited analytics (current location only)

**Authentication Flow:**

```typescript
// Staff logs in at pho-bo-hanoi.localstoreplatform.com
// Subdomain auto-detected from hostname

POST /api/v1/auth/otp/verify
{
  "phone": "0909876543",
  "otp": "123456",
  "subdomain": "pho-bo-hanoi"  // Extracted from req.hostname
}

// Backend validates user belongs to this tenant
{
  "accessToken": "eyJ...",
  "user": {
    "id": "user-456",
    "role": "staff",
    "canAccessMainPlatform": false,
    "canAccessSubdomain": "pho-bo-hanoi",
    "tenant": {
      "subdomain": "pho-bo-hanoi",
      "storeOperationsUrl": "https://pho-bo-hanoi.localstoreplatform.com"
    },
    "primaryLocation": {
      "id": "location-789",
      "name": "Chi nhánh Hai Bà Trưng"
    }
  }
}

// Staff CANNOT access:
// - app.localstoreplatform.com (403 Forbidden)
// - banh-mi-saigon.localstoreplatform.com (403 Forbidden)
```

---

### 3. Public Storefronts (Phase 2)

**Access:** Customers (public)

**URL Pattern:** `{tenant-slug}.lsp.menu` or custom domain

**Purpose:** Customer-facing menu, online ordering

---

## Database Schema

### Tenants Table (Updated)

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  subdomain VARCHAR(63) UNIQUE NOT NULL,  -- URL-safe slug
  phone VARCHAR(20) NOT NULL,
  email VARCHAR(255),
  status VARCHAR(50) DEFAULT 'active',
  subscription_tier VARCHAR(50) DEFAULT 'free',
  trial_ends_at TIMESTAMP,
  
  -- Onboarding configuration (PRO plan)
  staff_code VARCHAR(20),          -- "STAFF2025"
  manager_code VARCHAR(20),        -- "MGR2025"
  auto_approve_staff BOOLEAN DEFAULT false,
  auto_approve_managers BOOLEAN DEFAULT false,
  max_users INT DEFAULT 5,         -- Based on plan
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,
  
  CONSTRAINT subdomain_format CHECK (subdomain ~ '^[a-z0-9]+(-[a-z0-9]+)*$'),
  CONSTRAINT subdomain_length CHECK (length(subdomain) BETWEEN 3 AND 63)
);

-- Index for subdomain lookup (most common query)
CREATE INDEX idx_tenants_subdomain ON tenants(subdomain) WHERE deleted_at IS NULL;

-- Prevent reserved subdomains
CREATE FUNCTION check_reserved_subdomain() RETURNS TRIGGER AS $$
BEGIN
  IF NEW.subdomain IN ('app', 'www', 'api', 'admin', 'dashboard', 'mail', 'help', 'support') THEN
    RAISE EXCEPTION 'Subdomain % is reserved', NEW.subdomain;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validate_subdomain 
  BEFORE INSERT OR UPDATE ON tenants
  FOR EACH ROW EXECUTE FUNCTION check_reserved_subdomain();
```

### Users Table (Updated)

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  subdomain VARCHAR(63) NOT NULL,  -- Cached for quick lookup
  phone VARCHAR(20) NOT NULL,
  email VARCHAR(255),
  full_name VARCHAR(255),
  role VARCHAR(20) NOT NULL,  -- 'owner', 'manager', 'staff'
  
  -- Multi-domain access control
  can_access_main_platform BOOLEAN DEFAULT false,  -- true for owners
  primary_location_id UUID REFERENCES locations(id),
  
  -- Permissions (JSONB for flexibility)
  permissions JSONB DEFAULT '{
    "canViewFinancials": false,
    "canEditMenu": false,
    "canManageStaff": false,
    "canApproveDiscounts": false,
    "canEditPrices": false
  }'::jsonb,
  
  -- Invitation tracking
  invited_by UUID REFERENCES users(id),
  invited_at TIMESTAMP,
  invite_method VARCHAR(50),  -- 'phone_invite' or 'permanent_code'
  
  is_active BOOLEAN DEFAULT true,
  last_login_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,
  
  UNIQUE(tenant_id, phone),
  CONSTRAINT valid_role CHECK (role IN ('owner', 'manager', 'staff'))
);

CREATE INDEX idx_users_subdomain ON users(subdomain) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_tenant_role ON users(tenant_id, role) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_phone ON users(phone) WHERE deleted_at IS NULL;
```

### User Access Locations (Many-to-Many)

```sql
CREATE TABLE user_access_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  location_id UUID NOT NULL REFERENCES locations(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT NOW(),
  
  UNIQUE(user_id, location_id)
);

CREATE INDEX idx_user_access_locations_user ON user_access_locations(user_id);
CREATE INDEX idx_user_access_locations_location ON user_access_locations(location_id);
```

### User Invitations Table

```sql
CREATE TABLE user_invitations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  subdomain VARCHAR(63) NOT NULL,
  location_id UUID REFERENCES locations(id),  -- null for multi-location
  invited_by UUID REFERENCES users(id),       -- null for permanent codes
  
  phone VARCHAR(20) NOT NULL,
  role VARCHAR(20) NOT NULL,
  
  -- Invitation details
  invite_code VARCHAR(50) UNIQUE NOT NULL,
  invite_method VARCHAR(50) NOT NULL,  -- 'phone_invite' or 'permanent_code'
  
  -- Approval workflow
  status VARCHAR(50) DEFAULT 'pending',  -- pending, approved, rejected, expired, accepted
  approved_by UUID REFERENCES users(id),
  approved_at TIMESTAMP,
  
  expires_at TIMESTAMP,  -- null for permanent codes
  accepted_at TIMESTAMP,
  accepted_by UUID REFERENCES users(id),
  
  created_at TIMESTAMP DEFAULT NOW(),
  
  CONSTRAINT valid_invite_role CHECK (role IN ('manager', 'staff')),
  CONSTRAINT valid_status CHECK (status IN ('pending', 'approved', 'rejected', 'expired', 'accepted'))
);

CREATE INDEX idx_invitations_tenant ON user_invitations(tenant_id);
CREATE INDEX idx_invitations_code ON user_invitations(invite_code);
CREATE INDEX idx_invitations_status ON user_invitations(status);

-- Auto-expire invitations
CREATE FUNCTION expire_invitations() RETURNS TRIGGER AS $$
BEGIN
  UPDATE user_invitations 
  SET status = 'expired' 
  WHERE status = 'pending' 
    AND expires_at IS NOT NULL 
    AND expires_at < NOW();
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Run daily
CREATE OR REPLACE FUNCTION schedule_expire_invitations()
RETURNS void AS $$
BEGIN
  PERFORM expire_invitations();
END;
$$ LANGUAGE plpgsql;
```

---

## Subdomain Middleware (NestJS)

### Extract Subdomain from Hostname

```typescript
// src/common/middleware/subdomain.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class SubdomainMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const host = req.get('host') || '';
    const subdomain = this.extractSubdomain(host);
    
    // Attach to request for downstream use
    req['subdomain'] = subdomain;
    req['isMainPlatform'] = subdomain === 'app' || subdomain === null;
    req['isTenantSubdomain'] = subdomain !== null && subdomain !== 'app';
    
    next();
  }
  
  private extractSubdomain(host: string): string | null {
    // Examples:
    // pho-bo-hanoi.localstoreplatform.com → "pho-bo-hanoi"
    // app.localstoreplatform.com → "app"
    // localhost:3000 → null (dev mode)
    
    const parts = host.split('.');
    
    // Development: localhost or IP
    if (parts.length < 2 || host.includes('localhost') || /^\d+\.\d+\.\d+\.\d+/.test(host)) {
      // Check for subdomain in header (for local testing)
      return null;
    }
    
    // Extract first part as subdomain
    const subdomain = parts[0];
    
    // Validate format (lowercase alphanumeric + hyphens)
    if (!/^[a-z0-9]+(-[a-z0-9]+)*$/.test(subdomain)) {
      return null;
    }
    
    return subdomain;
  }
}
```

### Apply Middleware Globally

```typescript
// src/app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { SubdomainMiddleware } from './common/middleware/subdomain.middleware';

@Module({
  imports: [
    // ... other imports
  ],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(SubdomainMiddleware)
      .forRoutes('*');  // Apply to all routes
  }
}
```

---

## Authentication with Subdomain Validation

### Updated Auth Service

```typescript
// src/modules/auth/auth.service.ts
import { Injectable, UnauthorizedException, ForbiddenException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from '../../database/entities/user.entity';
import { Tenant } from '../../database/entities/tenant.entity';

@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
    @InjectRepository(Tenant)
    private tenantsRepository: Repository<Tenant>,
    // ... other dependencies
  ) {}
  
  async verifyOtp(
    phone: string,
    otp: string,
    subdomain: string | null,
  ): Promise<AuthPayload> {
    // Verify OTP (same as before)
    const storedOtp = await this.cacheManager.get<string>(`otp:${phone}`);
    if (!storedOtp || storedOtp !== otp) {
      throw new UnauthorizedException('Mã OTP không chính xác.');
    }
    
    // Find user by phone
    const user = await this.usersRepository.findOne({
      where: { phone },
      relations: ['tenant', 'primaryLocation'],
    });
    
    if (!user) {
      throw new UnauthorizedException('Số điện thoại chưa được đăng ký.');
    }
    
    // SUBDOMAIN VALIDATION
    if (subdomain === 'app' || subdomain === null) {
      // Main platform access
      if (!user.canAccessMainPlatform) {
        throw new ForbiddenException(
          'Bạn không có quyền truy cập trang quản lý chính. ' +
          `Vui lòng đăng nhập tại ${user.subdomain}.localstoreplatform.com`
        );
      }
    } else {
      // Tenant subdomain access
      if (user.subdomain !== subdomain) {
        throw new ForbiddenException(
          `Bạn không có quyền truy cập cửa hàng này. ` +
          `Vui lòng đăng nhập tại ${user.subdomain}.localstoreplatform.com`
        );
      }
    }
    
    // Update last login
    await this.usersRepository.update(user.id, {
      lastLoginAt: new Date(),
    });
    
    // Generate JWT
    const payload = {
      sub: user.id,
      phone: user.phone,
      tenantId: user.tenant.id,
      subdomain: user.subdomain,
      role: user.role,
      canAccessMainPlatform: user.canAccessMainPlatform,
    };
    
    const accessToken = this.jwtService.sign(payload);
    const refreshToken = this.jwtService.sign(payload, {
      expiresIn: this.configService.get('JWT_REFRESH_EXPIRATION'),
    });
    
    return { accessToken, refreshToken, user };
  }
}
```

### Subdomain Guard

```typescript
// src/common/guards/subdomain.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { Reflector } from '@nestjs/core';

@Injectable()
export class SubdomainGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const ctx = GqlExecutionContext.create(context);
    const { req, subdomain, isMainPlatform } = ctx.getContext();
    const user = req.user;
    
    if (!user) {
      return true; // Let AuthGuard handle this
    }
    
    // Main platform (app.localstoreplatform.com)
    if (isMainPlatform) {
      if (!user.canAccessMainPlatform) {
        throw new ForbiddenException(
          'Chỉ chủ quán mới có thể truy cập trang quản lý chính.'
        );
      }
      return true;
    }
    
    // Tenant subdomain
    if (subdomain) {
      if (user.subdomain !== subdomain) {
        throw new ForbiddenException(
          'Bạn không có quyền truy cập cửa hàng này.'
        );
      }
      return true;
    }
    
    return true;
  }
}
```

### Apply Guards Globally

```typescript
// src/app.module.ts
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './common/guards/jwt-auth.guard';
import { SubdomainGuard } from './common/guards/subdomain.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,
    },
    {
      provide: APP_GUARD,
      useClass: SubdomainGuard,
    },
  ],
})
export class AppModule {}
```

---

## User Invitation System

### Two Methods

#### Method 1: Phone Invitation (Small Businesses)

**Flow:**

1. Owner creates invitation at `app.localstoreplatform.com`
2. System sends SMS with link
3. Staff clicks link → Auto-redirected to tenant subdomain
4. Staff verifies phone with OTP
5. Account created immediately (no approval)

**Implementation:**

```typescript
// src/modules/users/users.service.ts
async inviteUser(
  ownerId: string,
  input: InviteUserInput,
): Promise<UserInvitation> {
  const owner = await this.usersRepository.findOne({
    where: { id: ownerId },
    relations: ['tenant'],
  });
  
  // Generate invitation code
  const inviteCode = this.generateInviteCode();
  
  // Create invitation record
  const invitation = this.invitationsRepository.create({
    tenantId: owner.tenant.id,
    subdomain: owner.tenant.subdomain,
    locationId: input.locationId,
    invitedBy: owner,
    phone: input.phone,
    role: input.role,
    inviteCode,
    inviteMethod: 'phone_invite',
    status: 'pending',
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24h
  });
  
  await this.invitationsRepository.save(invitation);
  
  // Generate invitation link
  const inviteLink = `https://${owner.tenant.subdomain}.localstoreplatform.com/join/${inviteCode}`;
  
  // Send SMS
  await this.smsService.send(input.phone, 
    `Chủ quán ${owner.tenant.name} mời bạn làm ${this.getRoleLabel(input.role)}. ` +
    `Nhấn link: ${inviteLink} (Hết hạn: 24h)`
  );
  
  return invitation;
}

private generateInviteCode(): string {
  // Generate UUID or short code
  return randomBytes(16).toString('hex').toUpperCase().slice(0, 12);
}

private getRoleLabel(role: string): string {
  return role === 'manager' ? 'Quản lý' : 'Nhân viên';
}
```

#### Method 2: Permanent Code (Enterprise Chains)

**Flow:**

1. Owner generates permanent codes at `app.localstoreplatform.com`
2. Codes displayed as QR + text (print posters)
3. Staff scans QR → Opens join page
4. Staff enters code + phone
5. Auto-approved or pending based on settings

**Implementation:**

```typescript
// src/modules/users/users.service.ts
async generateOnboardingCodes(
  tenantId: string,
  config: OnboardingConfigInput,
): Promise<OnboardingConfig> {
  const tenant = await this.tenantsRepository.findOne({
    where: { id: tenantId },
  });
  
  // Generate codes if not provided
  const staffCode = config.customStaffCode || this.generatePermanentCode('STAFF');
  const managerCode = config.customManagerCode || this.generatePermanentCode('MGR');
  
  // Update tenant
  await this.tenantsRepository.update(tenantId, {
    staffCode,
    managerCode,
    autoApproveStaff: config.autoApproveStaff ?? false,
    autoApproveManagers: config.autoApproveManagers ?? false,
  });
  
  // Generate QR codes
  const joinUrl = `https://${tenant.subdomain}.localstoreplatform.com/join`;
  const qrCodeUrl = await this.qrCodeService.generate(joinUrl);
  
  return {
    staffCode,
    managerCode,
    autoApproveStaff: config.autoApproveStaff ?? false,
    autoApproveManagers: config.autoApproveManagers ?? false,
    maxUsers: tenant.maxUsers,
    currentUsers: await this.countActiveUsers(tenantId),
    joinUrl,
    qrCodeUrl,
  };
}

private generatePermanentCode(prefix: string): string {
  const year = new Date().getFullYear();
  const random = randomBytes(3).toString('hex').toUpperCase();
  return `${prefix}${year}-${random}`;  // e.g., "STAFF2025-A3F"
}

async acceptInvitation(
  inviteCode: string,
  phone: string,
): Promise<AuthPayload> {
  // Find invitation
  const invitation = await this.invitationsRepository.findOne({
    where: { inviteCode },
    relations: ['tenant', 'location'],
  });
  
  if (!invitation) {
    throw new NotFoundException('Mã mời không hợp lệ.');
  }
  
  // Check expiration (skip for permanent codes)
  if (invitation.expiresAt && invitation.expiresAt < new Date()) {
    throw new BadRequestException('Mã mời đã hết hạn.');
  }
  
  // Check if user already exists
  let user = await this.usersRepository.findOne({
    where: { tenantId: invitation.tenantId, phone },
  });
  
  if (user) {
    throw new ConflictException('Số điện thoại này đã được đăng ký.');
  }
  
  // Determine approval status
  const tenant = await this.tenantsRepository.findOne({
    where: { id: invitation.tenantId },
  });
  
  const requiresApproval = invitation.role === 'manager' 
    ? !tenant.autoApproveManagers 
    : !tenant.autoApproveStaff;
  
  // Create user
  user = this.usersRepository.create({
    tenantId: invitation.tenantId,
    subdomain: invitation.subdomain,
    phone,
    role: invitation.role,
    canAccessMainPlatform: false,
    primaryLocationId: invitation.locationId,
    invitedBy: invitation.invitedBy,
    invitedAt: invitation.createdAt,
    inviteMethod: invitation.inviteMethod,
    isActive: !requiresApproval,  // Active if auto-approved
    permissions: this.getDefaultPermissions(invitation.role),
  });
  
  await this.usersRepository.save(user);
  
  // Update invitation
  await this.invitationsRepository.update(invitation.id, {
    status: requiresApproval ? 'pending' : 'accepted',
    acceptedAt: new Date(),
    acceptedBy: user,
  });
  
  if (requiresApproval) {
    // Notify owner for approval
    await this.notifyOwnerForApproval(invitation.tenantId, user);
    throw new PendingApprovalException(
      'Tài khoản của bạn đang chờ quản lý phê duyệt. ' +
      'Bạn sẽ nhận được SMS khi được kích hoạt.'
    );
  }
  
  // Auto-approved: Generate JWT and return
  const payload = {
    sub: user.id,
    phone: user.phone,
    tenantId: user.tenantId,
    subdomain: user.subdomain,
    role: user.role,
    canAccessMainPlatform: false,
  };
  
  const accessToken = this.jwtService.sign(payload);
  const refreshToken = this.jwtService.sign(payload, {
    expiresIn: '30d',
  });
  
  return { accessToken, refreshToken, user };
}

private getDefaultPermissions(role: string): any {
  if (role === 'manager') {
    return {
      canViewFinancials: true,
      canEditMenu: true,
      canManageStaff: false,
      canApproveDiscounts: true,
      canEditPrices: true,
    };
  }
  
  // Staff
  return {
    canViewFinancials: false,
    canEditMenu: false,
    canManageStaff: false,
    canApproveDiscounts: false,
    canEditPrices: false,
  };
}
```

---

## Role Permissions Matrix

| Feature | Owner | Manager | Staff |
|---------|-------|---------|-------|
| **Platform Access** | | | |
| Main platform (app.lsp.com) | ✅ | ❌ | ❌ |
| Tenant subdomain | ✅ | ✅ | ✅ |
| Cross-location view | ✅ | ❌ | ❌ |
| **Dashboard** | | | |
| View metrics | ✅ All | ✅ Assigned | ✅ Assigned |
| View financials (revenue/costs) | ✅ | ✅ | ❌ |
| AI recommendations | ✅ Approve/Dismiss | ✅ View only | ❌ |
| **Menu Management** | | | |
| View menu | ✅ | ✅ | ✅ |
| Edit items | ✅ | ✅ | ❌ |
| Edit prices | ✅ | ✅ | ❌ |
| Toggle availability | ✅ | ✅ | ✅ |
| **Sales** | | | |
| Enter sales | ✅ | ✅ | ✅ |
| Edit past sales | ✅ | ✅ | ❌ |
| View sales history | ✅ | ✅ | ❌ |
| **User Management** | | | |
| Invite staff | ✅ | ❌ | ❌ |
| Remove staff | ✅ | ❌ | ❌ |
| Approve pending users | ✅ | ❌ | ❌ |
| View staff list | ✅ | ✅ View only | ❌ |

---

## Pricing Tiers

| Feature | FREE | BASIC | PRO | ENTERPRISE |
|---------|------|-------|-----|------------|
| **Users** | 5 | 20 | 100 | Unlimited |
| **Locations** | 1 | 3 | 10 | Unlimited |
| **Phone Invites** | ✅ | ✅ | ✅ | ✅ |
| **Permanent Codes** | ❌ | ❌ | ✅ | ✅ |
| **Auto-Approve** | ❌ | ❌ | ✅ | ✅ |
| **QR Posters** | ❌ | ❌ | ✅ | ✅ |
| **Custom Subdomain** | ❌ | ❌ | ❌ | ✅ |
| **SSO Integration** | ❌ | ❌ | ❌ | ✅ |
| **Onboarding Analytics** | ❌ | ❌ | Basic | Advanced |
| **Training Materials** | ❌ | ❌ | ✅ | ✅ |

---

## Implementation Phases

### Sprint 2 (All Plans) - November 26 - December 23

**Week 1**: Subdomain Infrastructure

- Subdomain middleware and routing
- Database schema updates (subdomain fields)
- Tenant subdomain creation on signup
- Subdomain availability check API

**Week 2**: Phone Invitation System

- Create invitation GraphQL mutations
- SMS sending integration
- Invitation acceptance flow
- Owner approval workflow (optional)

**Week 3**: Role-Based Access Control

- Subdomain guard implementation
- Permission enforcement in resolvers
- Staff management UI (owner dashboard)
- User list and pending invitations

**Week 4**: Mobile App Updates

- Multi-domain support in Flutter
- Subdomain detection and storage
- Role-specific UI rendering
- Domain switcher (for owners)

### Sprint 3 (PRO Plan Feature) - January 2026

**Week 1**: Permanent Codes

- Generate onboarding codes mutation
- QR code generation service
- Standardized join page UI
- Code validation and redemption

**Week 2**: Auto-Approval Settings

- Tenant onboarding config management
- Auto-approve vs manual approval logic
- Owner notification for pending approvals
- SMS notification on approval

**Week 3**: Analytics & Posters

- Onboarding analytics dashboard
- Downloadable QR posters (PDF)
- Training materials library
- User activity tracking

---

## Testing Scenarios

### Test 1: Owner Multi-Domain Access

```bash
# Owner logs in at main platform
curl -X POST https://app.localstoreplatform.com/api/v1/auth/otp/verify \
  -d '{"phone":"0901234567","otp":"123456","subdomain":"app"}'

# Response: canAccessMainPlatform: true

# Owner can also access tenant subdomain
curl -X POST https://pho-bo-hanoi.localstoreplatform.com/api/v1/auth/otp/verify \
  -d '{"phone":"0901234567","otp":"123456","subdomain":"pho-bo-hanoi"}'

# Response: Success (same owner)
```

### Test 2: Staff Subdomain Isolation

```bash
# Staff tries to access main platform (should fail)
curl -X POST https://app.localstoreplatform.com/api/v1/auth/otp/verify \
  -d '{"phone":"0909876543","otp":"123456","subdomain":"app"}'

# Response: 403 Forbidden
# "Bạn không có quyền truy cập trang quản lý chính"

# Staff can only access assigned tenant subdomain
curl -X POST https://pho-bo-hanoi.localstoreplatform.com/api/v1/auth/otp/verify \
  -d '{"phone":"0909876543","otp":"123456","subdomain":"pho-bo-hanoi"}'

# Response: Success

# Staff tries to access different tenant (should fail)
curl -X POST https://banh-mi-saigon.localstoreplatform.com/api/v1/auth/otp/verify \
  -d '{"phone":"0909876543","otp":"123456","subdomain":"banh-mi-saigon"}'

# Response: 403 Forbidden
# "Bạn không có quyền truy cập cửa hàng này"
```

### Test 3: Invitation Flow

```graphql
# Owner invites staff (at app.localstoreplatform.com)
mutation InviteStaff {
  inviteUser(input: {
    phone: "0912345678"
    role: STAFF
    locationId: "location-123"
    method: PHONE_INVITE
  }) {
    id
    inviteCode
    inviteLink
    expiresAt
  }
}

# Response
{
  "inviteLink": "https://pho-bo-hanoi.localstoreplatform.com/join/ABC123XYZ",
  "expiresAt": "2025-10-23T10:00:00Z"
}

# Staff accepts invitation
mutation AcceptInvite {
  acceptInvitation(
    inviteCode: "ABC123XYZ"
    phone: "0912345678"
  ) {
    accessToken
    user {
      id
      role
      subdomain
      canAccessMainPlatform
    }
  }
}

# Response
{
  "user": {
    "role": "staff",
    "subdomain": "pho-bo-hanoi",
    "canAccessMainPlatform": false
  }
}
```

---

**Implementation ready for Sprint 2!** 🚀
