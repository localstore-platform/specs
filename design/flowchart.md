# Platform Plan Flowchart

Below is a set of focused Mermaid diagrams (split for readability). If you prefer a single diagram, we can keep a combined view — but splitting makes it easy to zoom and edit each area.

## Onboarding flow

```mermaid
flowchart TB
  Start([User visits platform]) --> OW["Onboarding Wizard<br/>(phone OTP primary)"]
  OW --> BT["Business type selection"]
  BT --> LOCQ{"Multiple locations?"}
  LOCQ -- Yes --> LOCSET["Location setup wizard<br/>(add locations)"]
  LOCQ -- No --> DEFAULTLOC["Create default location"]
  LOCSET --> MENUDEF["Starter menu & templates"]
  DEFAULTLOC --> MENUDEF
  MENUDEF --> DONE["Onboarding complete → Admin Dashboard"]
```

## Admin dashboard (analytics-first)

```mermaid
flowchart LR
  AD["Admin Dashboard (mobile-first)"]
  AD --> LS["Location selector (All / Specific)"]
  AD --> A1["AI Analytics (daily overview)"]
  AD --> A2["Cross-location comparison"]
  AD --> R["AI Recommendations (MVP)"]
  AD --> M["Menu & Inventory (location-aware)"]
  AD --> L["Locations management"]
  AD --> S["Settings & Subscription"]
```

## Integrations & Publishing (phased)

```mermaid
flowchart TB
  AUTH["Auth: Phone OTP primary<br/>Facebook OAuth optional"]
  POS["POS integrations (Phase 2)"]
  PAY["Payments (Phase 2): MoMo, ZaloPay, VNPay"]
  DEL["Delivery connectors (future): Grab/ShopeeFood/Gojek"]
  AUTH --> POS --> PAY --> DEL
  %% connect publishing into the same flow so it renders as one graph
  POS --> SF

  subgraph PUB[" "]
    direction TB
    PUBTITLE["Publishing (MVP deprioritized)"]
    SF["Storefront publishing (minimal)"]
    SUB["Subdomain issuance (slug).ourplatform.com"]
    QR["QR per location"]
    %% connect the title to SF with an invisible link so layout places the title above SF
    PUBTITLE --> SF
    SF --> SUB --> QR
    %% hide the title node border and the connector
    style PUBTITLE fill:transparent,stroke:transparent
    linkStyle 4 stroke:transparent,stroke-width:0
  end
```

## Infra, Non-functional & Testing

```mermaid
flowchart TB
  MT["Multi-tenant isolation (tenant_id scope)"]
  S3["S3-compatible storage\n(image optimization, limits)"]
  PERF["Performance: <2s TTI on 4G"]
  OBS["Observability & analytics hooks"]
  SEC["Security: JWT, RLS, rate limiting"]
  MT --> S3 --> PERF --> OBS --> SEC

  TEST["Testing & Launch: sprint 1-3, QA, low-bandwidth tests"]
  OBS --> TEST
  PERF --> TEST
```

## Legend & Notes

- This diagram maps the primary user flow (onboarding → dashboard → publish) and cross-cutting concerns (integrations, multi-tenancy, infra, testing).
- VN-specific items are emphasized: Zalo OAuth, MoMo/ZaloPay/VNPay, vi-VN & VND defaults, VN templates and sample menu items.
- Data isolation: all DB queries and storage keys must be scoped by `tenant_id`.
- Small, offline-friendly drafts are recommended for the admin dashboard.

## How to preview

- In VS Code: open `design/flowchart.md` and use a Mermaid preview extension (e.g., "Markdown Preview Mermaid Support") or the built-in Markdown preview if it supports Mermaid.

## Optional: export to PNG (local)

If you want a PNG, install mermaid-cli (requires Node):

```bash
# zsh
npm install -g @mermaid-js/mermaid-cli
mmdc -i design/flowchart.md -o design/flowchart.png
```

(If `mmdc` complains about reading a full markdown file, copy the Mermaid code block into a `.mmd` file first.)
