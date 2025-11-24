# Sprint 0.5: Foundation & Menu Demo

**Team:** Full-stack Developer (Solo or Small Team)  
**Goal:** Deliver working menu website with mock/real data showing Vietnamese restaurant offerings

**Status:** 🟢 In Progress (30% complete based on IMPLEMENTATION_PROGRESS.md)

**Note:** This sprint is flexible on timing. Focus on completing all stories sequentially, testing thoroughly at each step.

---

## Overview

This sprint establishes the foundational architecture and delivers a tangible demo:

- ✅ Three core repositories initialized (api, menu, contracts)
- ✅ Docker Compose development environment ready
- 🔴 Menu website displaying Vietnamese restaurant menu
- 🔴 Basic REST API serving menu data
- 🔴 Database schema for menu/categories deployed

**Strategy:** Demo-first approach - show something working before building authentication/mobile app.

---

## Success Criteria

### Functional Requirements

- [ ] Customer can view restaurant menu on mobile browser
- [ ] Menu displays categories (Đồ uống, Đồ ăn, etc.)
- [ ] Each item shows name, price in VND (75.000₫), description
- [ ] Menu loads in <2 seconds on 4G network
- [ ] Menu works on subdomain: `demo.lsp.menu` (or localhost for now)

### Technical Requirements

- [ ] Next.js menu website deployed (Vercel or localhost)
- [ ] NestJS API serving menu endpoints
- [ ] PostgreSQL database with menu_items and categories tables
- [ ] Docker Compose running all services (Postgres, Redis, API)
- [ ] TypeScript contracts package defining shared types

### Demo Requirements

- [ ] At least one Vietnamese restaurant menu (phở shop, cà phê, etc.)
- [ ] 3+ categories with 5+ items each
- [ ] All content in Vietnamese
- [ ] Mobile-responsive design tested on iPhone/Android
- [ ] QR code generated pointing to menu URL (optional nice-to-have)

---

## Part 1: Menu Website (Priority 1)

### Story 1.1: Menu Display Page

**Repository:** `menu` (<https://github.com/localstore-platform/menu>)

**As a** customer  
**I want to** view the restaurant menu on my phone  
**So that** I can see what food/drinks are available and their prices

**Tasks:**

- [ ] Create dynamic route `app/[tenant]/menu/page.tsx` in menu repo
- [ ] Design mobile-first layout with Tailwind CSS
- [ ] Implement category tabs/sections
- [ ] Create MenuItem component showing:
  - Vietnamese name
  - English name (optional)
  - Price with VND formatting (75.000₫)
  - Description
  - Image placeholder
  - Available/unavailable badge
- [ ] Add loading skeleton (shimmer effect)
- [ ] Add error state with retry button
- [ ] Test on mobile viewports (375px, 414px)

**Acceptance Criteria:**

- Menu page accessible at `/demo/menu`
- Items grouped by category (collapsible sections)
- Prices display with Vietnamese formatting (thousands separator + ₫)
- Responsive design works on iOS Safari and Chrome Android
- Page loads in <2s with images optimized
- Works with mock data initially (hardcoded JSON)

**Definition of Done:**

- [ ] Code reviewed and merged to main
- [ ] Deployed to Vercel (or accessible on localhost)
- [ ] Screenshot shared in progress tracker
- [ ] Markdown linting passes
- [ ] Mobile testing completed

---

### Story 1.2: Vietnamese Currency Formatter

**Repository:** `menu` (initially) → `contracts` (final shared location)

**As a** developer  
**I want to** reusable VND currency formatting  
**So that** prices display consistently across the application

**Tasks:**

- [ ] Create `lib/utils/currency.ts` in menu repo
- [ ] Implement `formatVND(amount: number): string` function
  - Returns "75.000₫" format
  - Handles thousands separator (.)
  - Appends ₫ symbol
  - Handles edge cases (0, negative, undefined)
- [ ] Add unit tests with Jest
- [ ] Export from contracts repo as shared utility

**Test Cases:**

```typescript
formatVND(75000) → "75.000₫"
formatVND(1500000) → "1.500.000₫"
formatVND(0) → "0₫"
formatVND(undefined) → "—"
```

**Acceptance Criteria:**

- Function handles all test cases correctly
- Unit tests have 100% coverage
- Used in MenuItem component for price display
- Exported from `@localstore/types` package (contracts repo)

---

## Part 2: Backend API (Priority 2)

### Story 2.1: NestJS Project Setup

**Repository:** `api` (<https://github.com/localstore-platform/api>)

**As a** backend developer  
**I want to** have a working NestJS API  
**So that** I can serve menu data to the frontend

**Tasks:**

- [ ] Initialize NestJS project in api repo

  ```bash
  cd api/
  npx @nestjs/cli new src --skip-git
  ```

- [ ] Install dependencies:
  - `@nestjs/typeorm`, `typeorm`, `pg`
  - `@nestjs/config` for environment variables
  - `class-validator`, `class-transformer`
- [ ] Configure TypeORM connection to PostgreSQL
- [ ] Create `src/modules/menu/` module structure
- [ ] Setup environment variables in `.env`
- [ ] Update `docker-compose.yml` to run NestJS API
- [ ] Create health check endpoint `GET /health`
- [ ] Test with `curl http://localhost:3000/health`

**Acceptance Criteria:**

- `docker-compose up` starts NestJS API successfully
- API accessible at `http://localhost:3000`
- Health check returns `{"status":"ok"}`
- Hot reload works when editing TypeScript files
- No TypeScript errors or warnings

---

### Story 2.2: Menu REST API Endpoints

**Repository:** `api` (<https://github.com/localstore-platform/api>)

**As a** frontend developer  
**I want to** fetch menu data from the API  
**So that** I can display real data instead of mock data

**Tasks:**

- [ ] Create TypeORM entities:
  - `Category` (id, name, name_vi, display_order, tenant_id)
  - `MenuItem` (id, name, name_vi, price, category_id, tenant_id, status)
- [ ] Generate migration for menu tables
- [ ] Create MenuModule in NestJS
- [ ] Implement MenuController with endpoints:
  - `GET /api/menu/:tenantId` - Get all menu items for tenant
  - `GET /api/menu/:tenantId/categories` - Get categories
  - `GET /api/menu/:tenantId/items/:categoryId` - Filter by category
- [ ] Add query parameter: `?status=available`
- [ ] Implement MenuService with business logic
- [ ] Add request/response DTOs with validation
- [ ] Write unit tests for MenuService
- [ ] Test with Postman/curl

**API Spec Reference:** `architecture/api-specification.md` lines 400-600

**Example Response:**

```json
{
  "categories": [
    {
      "id": "cat-1",
      "name": "Cà phê",
      "name_en": "Coffee",
      "items": [
        {
          "id": "item-1",
          "name": "Cà phê sữa đá",
          "name_en": "Iced milk coffee",
          "price": 25000,
          "description": "Cà phê Robusta pha phin truyền thống",
          "status": "available"
        }
      ]
    }
  ]
}
```

**Acceptance Criteria:**

- Endpoints return correct JSON structure
- All items for tenant filtered correctly (multi-tenancy safe)
- Response time <200ms for typical menu (50 items)
- Validation errors return 400 with Vietnamese messages
- Unit tests cover happy path + error cases

---

### Story 2.3: Database Migration & Seed Data

**Repository:** `api` (<https://github.com/localstore-platform/api>)

**As a** backend developer  
**I want to** have database tables with sample data  
**So that** I can test the API with realistic Vietnamese restaurant data

**Tasks:**

- [ ] Create migration `001_create_menu_tables.ts`

  ```sql
  CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    name VARCHAR(100) NOT NULL,
    name_vi VARCHAR(100) NOT NULL,
    display_order INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
  );

  CREATE TABLE menu_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    category_id UUID REFERENCES categories(id),
    name VARCHAR(200) NOT NULL,
    name_vi VARCHAR(200) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    description TEXT,
    status VARCHAR(20) DEFAULT 'available',
    created_at TIMESTAMP DEFAULT NOW()
  );

  CREATE INDEX idx_menu_items_tenant ON menu_items(tenant_id);
  CREATE INDEX idx_menu_items_category ON menu_items(category_id);
  ```

- [ ] Create seed script `src/database/seeds/menu-seed.ts`
- [ ] Add Vietnamese coffee shop data:
  - Categories: Cà phê, Trà, Sinh tố, Đồ ăn nhẹ
  - 20+ items with realistic prices (15.000₫ - 75.000₫)
- [ ] Run migration: `npm run migration:run`
- [ ] Run seed: `npm run seed`
- [ ] Verify data in PostgreSQL

**Database Spec Reference:** `architecture/database-schema.md` lines 200-350

**Sample Seed Data:**

```typescript
const categories = [
  { name: 'Coffee', name_vi: 'Cà phê', order: 1 },
  { name: 'Tea', name_vi: 'Trà', order: 2 },
  { name: 'Smoothies', name_vi: 'Sinh tố', order: 3 },
];

const items = [
  {
    name: 'Iced milk coffee',
    name_vi: 'Cà phê sữa đá',
    price: 25000,
    category: 'Coffee',
    description: 'Cà phê Robusta pha phin truyền thống với sữa đặc',
  },
  {
    name: 'Vietnamese drip coffee',
    name_vi: 'Cà phê phin',
    price: 30000,
    category: 'Coffee',
    description: 'Cà phê nguyên chất pha phin, đậm đà',
  },
  // ... more items
];
```

**Acceptance Criteria:**

- Migration creates tables without errors
- Seed script populates 4 categories and 20+ items
- All prices in valid VND amounts (multiples of 1000)
- Data visible in database: `SELECT * FROM menu_items;`
- API endpoints return seeded data

---

## Part 3: Integration & Contracts (Priority 3)

### Story 3.1: API Integration with Menu Website

**Repository:** `menu` (<https://github.com/localstore-platform/menu>)

**As a** frontend developer  
**I want to** fetch menu data from the real API  
**So that** the menu website displays actual database content

**Tasks:**

- [ ] Configure API base URL in menu repo `.env.local`

  ```
  NEXT_PUBLIC_API_URL=http://localhost:3000
  ```

- [ ] Create API client `lib/api/menu-client.ts`
- [ ] Implement `fetchMenu(tenantId: string)` function using `fetch()`
- [ ] Update menu page to use real API instead of mock data
- [ ] Add error handling with user-friendly Vietnamese messages
- [ ] Add loading state during API call
- [ ] Test full stack: Database → API → Website
- [ ] Fix any CORS issues (if frontend and backend on different ports)

**Acceptance Criteria:**

- Menu page successfully fetches from `http://localhost:3000/api/menu/demo`
- Items and categories display correctly
- Loading spinner shows during fetch
- Error message displays if API is down: "Không thể tải menu. Vui lòng thử lại."
- CORS headers configured in NestJS (for localhost development)

---

### Story 3.2: Shared TypeScript Types (Contracts)

**Repository:** `contracts` (<https://github.com/localstore-platform/contracts>)

**As a** developer  
**I want to** type-safe API contracts  
**So that** frontend and backend stay in sync

**Tasks:**

- [ ] Create `src/menu/` folder in contracts repo
- [ ] Define TypeScript interfaces:

  ```typescript
  // src/menu/types.ts
  export interface Category {
    id: string;
    name: string;
    name_vi: string;
    displayOrder: number;
    items?: MenuItem[];
  }

  export interface MenuItem {
    id: string;
    categoryId: string;
    name: string;
    name_vi: string;
    price: number;
    description?: string;
    imageUrl?: string;
    status: 'available' | 'unavailable';
  }

  export interface MenuResponse {
    categories: Category[];
  }
  ```

- [ ] Create `src/utils/currency.ts` with `formatVND()` function
- [ ] Export all from `src/index.ts`
- [ ] Setup `package.json` for npm publishing:

  ```json
  {
    "name": "@localstore/types",
    "version": "0.1.0",
    "main": "dist/index.js",
    "types": "dist/index.d.ts"
  }
  ```

- [ ] Build package: `npm run build`
- [ ] (Optional) Publish to npm or use via local path

**Acceptance Criteria:**

- Types exported from `@localstore/types`
- API repo imports and uses types for DTOs
- Menu repo imports and uses types for API responses
- Build generates `.d.ts` type declaration files
- No TypeScript errors in either repo when using contracts

---

## Part 4: Polish & Demo Prep (Priority 4)

### Story 4.1: Mobile Optimization & Testing

**Repository:** `menu` (<https://github.com/localstore-platform/menu>)

**As a** product owner  
**I want to** ensure the menu works perfectly on mobile  
**So that** we can demo to Vietnamese restaurant owners on their phones

**Tasks:**

- [ ] Test on real devices:
  - iPhone (iOS Safari)
  - Android (Chrome)
- [ ] Optimize images (compress, use WebP)
- [ ] Add lazy loading for menu items
- [ ] Test on slow 3G network (Chrome DevTools throttling)
- [ ] Ensure <2s TTI on 4G network
- [ ] Fix any layout issues on small screens (320px width)
- [ ] Add touch-friendly tap targets (min 44px)
- [ ] Test Vietnamese text wrapping and font rendering
- [ ] Add favicon and meta tags

**Acceptance Criteria:**

- Menu loads in <2s on 4G (tested with Chrome throttling)
- All text readable without zooming
- Buttons/cards have min 44px tap targets
- Scrolling is smooth (60fps)
- No horizontal scrolling on 375px viewport
- Vietnamese characters display correctly (no encoding issues)

---

### Story 4.2: Demo Deployment

**Repository:** `menu` (<https://github.com/localstore-platform/menu>) + `api` (optional)

**As a** stakeholder  
**I want to** share a live demo URL  
**So that** I can show the menu to potential customers

**Tasks:**

- [ ] Deploy menu website to Vercel:

  ```bash
  cd menu/
  vercel --prod
  ```

- [ ] Configure environment variables in Vercel dashboard
- [ ] (Optional) Deploy API to Railway/Render/Fly.io or keep on localhost
- [ ] Test deployed website on mobile devices
- [ ] Create QR code pointing to demo URL
- [ ] Document demo URL in README
- [ ] Take screenshots for documentation
- [ ] Record 30-second demo video

**Demo URL Examples:**

- Menu: `https://menu-demo.vercel.app/demo/menu`
- API: `http://localhost:3000` (or `https://api-demo.railway.app`)

**Acceptance Criteria:**

- Demo URL accessible from any device with internet
- QR code successfully opens menu on phone
- Demo survives page refresh (no errors)
- SSL certificate valid (HTTPS)
- Demo URL shared in project README and Slack

---

## Definition of Done (Sprint 0.5)

### Code Quality

- [ ] All TypeScript code passes `npm run lint`
- [ ] No `any` types in production code
- [ ] Code reviewed (self-review OK for solo developer)
- [ ] Unit tests written for utilities (formatVND, etc.)
- [ ] All markdown files pass linting (MD022, MD032, etc.)

### Functionality

- [ ] Menu website displays Vietnamese restaurant menu
- [ ] API serves menu data from PostgreSQL
- [ ] Database has realistic seed data
- [ ] Full stack integration works end-to-end
- [ ] Mobile-responsive and tested on real devices

### Documentation

- [ ] README updated with setup instructions
- [ ] API endpoints documented (at least in code comments)
- [ ] Screenshots added to documentation
- [ ] Demo URL shared with team
- [ ] IMPLEMENTATION_PROGRESS.md updated

### Demo Readiness

- [ ] Can show working menu on phone to stakeholder
- [ ] Demo script prepared (30-second pitch)
- [ ] Known issues documented (if any)
- [ ] Next steps clearly defined

---

## Risks & Mitigation

### Risk 1: API and frontend on different domains (CORS issues)

- **Impact:** Medium (blocks API calls)
- **Probability:** High (localhost different ports)
- **Mitigation:**
  - Configure CORS in NestJS early (Day 3)
  - Test API integration immediately
  - Use proxy in Next.js config if needed

### Risk 2: Vietnamese text encoding issues

- **Impact:** Low (display problems)
- **Probability:** Medium
- **Mitigation:**
  - Use UTF-8 encoding everywhere
  - Test with real Vietnamese text early
  - Use proper fonts (system fonts work fine)

### Risk 3: Demo deployment issues

- **Impact:** Medium (no shareable URL)
- **Probability:** Low (Vercel is straightforward)
- **Mitigation:**
  - Deploy early (Day 6)
  - Keep API on localhost acceptable for Sprint 0.5
  - Document deployment steps in README

---

## Success Metrics

### Development

- Sprint completion: 100% (all stories done)
- Bugs found: <5 critical issues
- Code review turnaround: <24 hours (if applicable)

### Technical

- Menu page load time: <2s on 4G
- API response time: <200ms
- Zero TypeScript errors
- Mobile layout works 375px-414px viewports

### Demo

- Can show working demo to 3+ people
- Positive feedback on mobile UX
- No crashes or errors during demo
- Vietnamese restaurant owners understand the value prop

---

## Next: Sprint 1 (Authentication & Core Features)

**After Sprint 0.5, proceed to:**

- [ ] User authentication (phone OTP)
- [ ] Owner dashboard (metrics, trends)
- [ ] Menu management (CRUD operations)
- [ ] Mobile app (Flutter or Progressive Web App)

---

## Getting Started (Sprint 0.5)

```bash
# Part 1: Menu Website (Stories 1.1-1.2)
cd menu/
git checkout -b feat/menu-display-page
# Implement menu display page + currency formatter

# Part 2: Backend API (Stories 2.1-2.3)
cd ../api/
git checkout -b feat/nestjs-menu-api
npx @nestjs/cli new src --skip-git
# Setup NestJS + TypeORM + Database

# Part 3: Integration (Stories 3.1-3.2)
# Connect frontend to backend
# Setup shared contracts
# Test full stack

# Part 4: Polish & Deploy (Stories 4.1-4.2)
cd ../menu/
vercel --prod
```

**Sequential Goals:**

1. **Part 1 (menu repo):** Working menu page with mock data + VND formatter
2. **Part 2 (api repo):** Working API with real database data
3. **Part 3 (menu + contracts repos):** Full integration with shared types
4. **Part 4 (menu repo):** Polish, test, and deploy

---

**Ready to start Sprint 0.5!** 🚀

**Next Action:** Pick first story (Menu Display Page) and start implementing.
