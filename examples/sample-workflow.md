# Sample Workflow: E-Commerce Admin Dashboard

This example demonstrates a complete invocation of the `production-ready-workflow` skill on a realistic e-commerce admin dashboard codebase.

## Project Context

**Codebase:** `acme-ecommerce-admin`  
**Stack:** React 18 + TypeScript, Express + TypeScript, PostgreSQL + Prisma  
**State:** React Query v5 + Zustand  
**Schema:** Zod v3  
**Scope:** Dashboard, Orders, Products, Customers, Settings, Reports

## Invocation

```bash
cd acme-ecommerce-admin
/production-ready-workflow
```

## Phase 0 Interaction

### Agent: Task Understanding

> **My Understanding:**
> You want to transform the Acme E-Commerce Admin Dashboard into a production-ready system by eliminating all mock data, wiring live API bindings, enforcing SOLID architecture, and verifying via live end-to-end simulation. The target is a React/Express app with PostgreSQL. In scope: Dashboard, Orders, Products, Customers, Settings, Reports pages and all sub-pages/modals. Out of scope: Public storefront, payment integration (handled by separate service). Success = zero mock data, all endpoints wired, 95%+ readiness score.

> **Confidence: 85%** — Need clarification on endpoints and auth.

### User: Clarifications

> **Q1:** Does `/api/config` exist? Returns `{ pageSize, debounceMs, retryCount, featureFlags }`?
> **A1:** No — needs to be built. Feature flags: `newCheckout`, `advancedReports`, `customerInsights`.
>
> **Q2:** Does `/api/lookup/*` exist? Needed for: order statuses, product categories, customer tiers, countries?
> **A2:** Partially — `/api/lookup/order-statuses` and `/api/lookup/product-categories` exist. Need `/api/lookup/customer-tiers` and `/api/lookup/countries`.
>
> **Q3:** Does `/settings/defaults` exist? Returns `{ theme, view, sortOrder, timezone }`?
> **A3:** No — needs to be built.
>
> **Q4:** Auth system? Roles?
> **A4:** NextAuth v5 with JWT. Roles: `admin`, `manager`, `support`, `viewer`. Permissions: RBAC with explicit allowlists.
>
> **Q5:** Local DB?
> **A5:** Docker PostgreSQL (docker-compose.yml exists). Prisma migrations current.
>
> **Q6:** Test framework?
> **A6:** Vitest for unit, Playwright for E2E. Coverage target: 80%.
>
> **Q7:** Routes in scope?
> **A7:** `/`, `/dashboard`, `/orders`, `/orders/:id`, `/products`, `/products/:id`, `/customers`, `/customers/:id`, `/settings`, `/reports`, `/reports/:type`. All modals/drawers in scope.

### Agent: Updated Understanding

> **My Understanding (Updated):**
> Transform Acme E-Commerce Admin Dashboard (React/Express/PostgreSQL) to production-ready. Build missing endpoints: `/api/config`, `/settings/defaults`, `/api/lookup/customer-tiers`, `/api/lookup/countries`. Use NextAuth v5 with roles admin/manager/support/viewer. Local Docker PostgreSQL. Vitest + Playwright. Scope: 10 routes + all sub-pages. Need to build 4 endpoints, wire 50+ components.

> **Confidence: 97%** — Gate passed.

## Phase 1 Plan Summary

### Frontend Pillars (Est: 18h)
- **FE-1:** Design System (Glassmorphism, WCAG 2.2 AA) — 4h
- **FE-2:** Dashboard Components (Charts, Stats, Activity) — 5h
- **FE-3:** CRUD Pages (Orders, Products, Customers) — 6h
- **FE-4:** Settings & Reports — 3h

### Logic Pillars (Est: 8h)
- **LO-1:** Caching & Optimization (LRU, batch loading) — 3h
- **LO-2:** Optimistic Updates (Orders, Products, Customers) — 3h
- **LO-3:** Derived Data (Computed, not stored) — 2h

### Backend Pillars (Est: 16h)
- **BE-1:** Core Endpoints (Config, Lookups, Settings) — 4h
- **BE-2:** CRUD APIs (Orders, Products, Customers) — 6h
- **BE-3:** Reports & Export APIs — 4h
- **BE-4:** Auth/RBAC Hardening — 2h

### Top Risks
1. **R-1 (Score 60):** Missing `/api/config` — Build first, feature-flag
2. **R-2 (Score 50):** RBAC bypass — Server-side validation
3. **R-3 (Score 48):** N+1 in Reports — Batch loader pattern

### Checklist: 142 items

## Phase 2 Constraints Applied

- **Data Source Inversion:** All 47 hardcoded values → endpoints
- **Dead UX:** 23 mock arrays, 12 no-op handlers, 3 lorem ipsum blocks → live bindings
- **E2E Integrity:** 15 sub-pages → independent fetching
- **NFRs:** Unified errors, 4-state rendering, React Query centralization

## Phase 3 Implementation Highlights

### Key Transformations

| Before | After | Files Changed |
|--------|-------|---------------|
| `const ORDER_STATUSES = [...]` | `useOrderStatusLookup()` → `/api/lookup/order-statuses` | 8 files |
| `const PAGE_SIZE = 25` | `usePagination()` → `/api/config` | 12 files |
| `onClick={() => {}}` (12x) | Real mutations with cache invalidation | 12 files |
| `mockChartData` in Dashboard | `useDashboardAnalytics()` → `/api/analytics` | 5 files |
| Prop-drilled `orders` to OrderDetail | `useOrder(id)` independent fetch | 3 files |
| Static navigation | `useNavigation()` → `/api/navigation` | 2 files |

### Orphan Cleanup
Removed: `src/mocks/orders.ts`, `src/fixtures/products.ts`, `src/fixtures/customers.ts`, `src/utils/mockData.ts`

## Phase 4 Review Findings

| Severity | Count | Fixed |
|----------|-------|-------|
| CRITICAL | 1 | 1 |
| HIGH | 3 | 3 |
| MEDIUM | 4 | 2 |
| LOW | 7 | 0 |

**Critical Fixed:** N+1 query in `/api/reports` (batch loader added)  
**High Fixed:** RBAC middleware on all routes, rate limiting on mutations, secrets removed from client bundle

## Phase 5 Verification

### Verification Script
```
✅ Forbidden patterns: 0 found
✅ Required endpoints: 50/50 responding
✅ Schema validation: 50/50 hooks validated
✅ Four-state rendering: 94% components compliant
✅ Test suite: 168/170 passing (2 skipped - Reports module)
✅ TypeScript: 0 errors
✅ Lint: 0 errors
✅ Bundle size: 1.9MB (budget: 2.5MB)
```

### Live Simulation (agent-browser)

| User Flow | Status | Duration |
|-----------|--------|----------|
| Login → Dashboard | ✅ PASS | 2.1s |
| Create Order → Verify in list | ✅ PASS | 3.4s |
| Edit Product → Verify update | ✅ PASS | 2.8s |
| Bulk Customer Export | ✅ PASS | 4.2s |
| Settings Save → Persist | ✅ PASS | 1.9s |
| Report Generation (PDF) | ✅ PASS | 5.1s |
| Role Switch (admin→support) | ✅ PASS | 1.5s |
| Session Refresh | ✅ PASS | 0.8s |

**All 8 flows passed.**

## Phase 6 Report

```
Production Readiness: 97.3% (Grade A)

Wired Endpoints: 50/50 (100%)
Constraints Satisfied: 24/24 (100%)
Tests Passing: 168/170 (99%) — 2 skipped for unwired legacy import

Remaining Work: 0 blocking items
Tech Debt: 3 LOW items (hook naming, role centralization, legacy modal)

Verdict: PRODUCTION READY — Deploy with confidence
```

## Commands Used

```bash
# Full workflow
/production-ready-workflow

# Dry run first
/production-ready-workflow --dry-run

# Scoped to Orders module only
/production-ready-workflow --path ./src/features/orders

# Verification only (post-refactor)
/production-ready-workflow --phase 5 --no-browser

# With custom config
/production-ready-workflow --config ./custom-config.json
```

## Time Breakdown

| Phase | Duration |
|-------|----------|
| Phase 0 | 15 min |
| Phase 1 | 35 min |
| Phase 2 | Instant |
| Phase 3 | 2h 18 min |
| Phase 4 | 42 min |
| Phase 5 | 18 min |
| Phase 6 | Instant |
| **Total** | **3h 48 min** |

## Key Takeaways

1. **Invest in Phase 0** — Clear endpoint inventory saved 2h of rework
2. **Build missing endpoints first** — Backend work in Phase 3 unblocks frontend
3. **Parallel subagents** — Frontend/Backend/Logic pillars ran concurrently
4. **Phase 4 catches real bugs** — N+1 and RBAC issues found pre-verification
5. **Live simulation > unit tests** — Caught 3 integration issues unit tests missed

---

*This example represents a typical medium-complexity application. Your mileage may vary based on codebase size, mock data density, and endpoint completeness.*