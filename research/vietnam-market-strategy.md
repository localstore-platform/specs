# Go-to-Market Strategy: Vietnam

> **For complete product strategy and target market details, see [../planning/product-strategy-refined.md](../planning/product-strategy-refined.md)**
>
> **For pricing details, see [../planning/monetization-strategy.md](../planning/monetization-strategy.md)**

---

## Positioning

We are an **AI-powered business intelligence platform** for Vietnamese restaurant operators (chains + skilled operators), not a menu website builder.

**Target:** ~110,000 potential customers (50k chains + 60k skilled operators), growing 15-20% annually

## Competitive Advantages

1. **Data Moat:** Multi-tenant learning across chains and single locations
2. **Vietnam-Specific Patterns:** Tết holidays, weather impacts, local preferences
3. **Deep Integrations:** MoMo, ZaloPay, GrabFood, Zalo create switching costs
4. **AI Improvement Over Time:** 6+ months of data = highly personalized insights

## Mobile-First Design

- Dashboard optimized for low-cost Android phones over 4G
- Location switcher for chains to jump between stores
- Push notifications for critical alerts (low stock, unusual sales drops)
- Offline support with cached reports

## Go-to-Market Tactics

### Sales Strategy: Target Chains Directly

1. **Identify chains:** Scrape Google Maps for businesses with 2+ locations
2. **Cold outreach:** "We analyzed your 3 locations - Location A underperforming by 20%. Want free audit?"
3. **Free 1-month trial:** Prove AI value with real data
4. **Land & expand:** Start with 2-3 locations, expand as they grow

**Channels:**

- LinkedIn outreach to F&B owners
- Facebook Groups for restaurant owners ("Cộng đồng chủ quán cà phê")
- Partnership with POS vendors (we integrate, they refer)
- Content marketing: Blog on "Quản lý chuỗi quán ăn thế nào cho hiệu quả"

### Why This Beats Menu Builders

**Old positioning (menu builder):**

- Compete with 100+ tools on features
- Commoditized, easy to copy
- Price-sensitive, low switching costs

**New positioning (AI intelligence):**

- Data moat: multi-tenant learning
- High switching costs: lose personalized AI
- Premium pricing justified by ROI
- Network effects: more tenants = better models

## Success Metrics

**Year 1 Target:**

- 50 chains paying (avg 3 locations each = 150 locations total)
- ₫500k average revenue per chain
- Proven impact: 10-15% revenue increase for customers
- NPS >50, <5% monthly churn

**Value Proof Points:**

- "Our AI helped reduce food waste by 30%" (testimonial)
- "Location manager time saved: 2 hours/day" (case study)
- "Revenue increase after 3 months: 18%" (data)

## Technical Requirements for Vietnam Market

1. **Mobile-first:** Flawless on Android 4G (<2s load time)
2. **Vietnamese-first:** All UI, formatting (₫1,250,000), dates (dd/mm/yyyy)
3. **Deep integrations:** MoMo, GrabFood, weather APIs (for AI context)
4. **Multi-tenant database:** Shared DB with tenant_id + location_id isolation
5. **AI training pipeline:** Cloud ML (Google AI or AWS SageMaker)
6. **Real-time analytics:** Pre-aggregated metrics, <100ms dashboard load

## Risks & Mitigation

### Risk: Cold Start Problem

New chains have no data.

- **Mitigation:** Rule-based recommendations day 1, multi-tenant baseline, 3-month onboarding support

### Risk: Bad AI Advice Damages Trust

- **Mitigation:** Show confidence levels, explain reasoning, A/B test recommendations, insurance fund

### Risk: Competitors Copy Features

- **Mitigation:** Data moat (years of multi-tenant data), speed to market, deep integrations

## Next Steps

1. **Week 1-2:** Interview 10 chain owners, validate ₫500k-1.5M/month willingness to pay
2. **Week 3-4:** Build multi-location database schema, basic AI recommendation engine
3. **Week 5-8:** MVP with manual data entry → prove dashboard value
4. **Week 9-12:** POS integration → automate data collection
5. **Month 4+:** Launch, iterate based on customer feedback, expand to Thailand/Indonesia
