# System Architecture

This document outlines the technical architecture for the Menu Site SaaS platform.

## High-Level Diagram

```text
┌─────────────────────────────────────────────────────────────┐
│                         End Users                           │
│              (Store Customers - Website Visitors)           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    CDN / Edge Network                       │
│              (Static Assets, Cached Content)                │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  Load Balancer / API Gateway                │
└──────────────────────┬──────────────────────────────────────┘
                       │
       ┌───────────────┴────────────────┐
       ▼                                ▼
┌──────────────────┐          ┌──────────────────┐
│  Store Dashboard │          │  Customer-Facing │
│   (Admin Panel)  │          │     Websites     │
│   React/Next.js  │          │     Next.js      │
└────────┬─────────┘          └────────┬─────────┘
         │                              │
         └──────────────┬───────────────┘
                        ▼
         ┌──────────────────────────┐
         │     API Gateway          │
         │  (Rate Limiting, Auth)   │
         └──────────┬───────────────┘
                    │
         ┌──────────┴────────────┐
         ▼                       ▼
┌─────────────────┐    ┌──────────────────┐
│  Auth Service   │    │   Core API       │
│  (JWT, OAuth)   │    │  (Business Logic)│
└─────────────────┘    └────────┬─────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
         ┌──────────────┐ ┌─────────┐ ┌─────────┐
         │  PostgreSQL  │ │  Redis  │ │   S3    │
         │   Database   │ │  Cache  │ │ Storage │
         └──────────────┘ └─────────┘ └─────────┘
```

## Technology Stack

- **Frontend:** Next.js (React) with Tailwind CSS
- **Backend:** Node.js (NestJS) or Go (Fiber)
- **Database:** PostgreSQL
- **Cache:** Redis
- **Storage:** AWS S3 or compatible
- **Deployment:** Docker, Vercel/Netlify for Frontend
