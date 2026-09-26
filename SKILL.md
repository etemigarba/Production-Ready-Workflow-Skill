---
name: production-ready-workflow
description: >
  End-to-end workflow for auditing and refactoring a full-stack web application into a
  production-ready, zero-mock-data system, then verifying it with live end-to-end simulation.
  Invoke with /production-ready-workflow. Use this skill whenever the user asks to "make the
  app production-ready", "remove mock data / placeholders / hardcoded values", "wire the UI to
  the live API", "retire dead UX", "audit production readiness", or requests a full-stack
  refactor covering frontend (UI/UX), logic (algorithms/data structures), and backend
  (database/data access) — even if they don't use the word "production" explicitly.
---

# Production-Ready Workflow Implementation

Act as a Principal/Staff Software Architect (15+ years shipping high-traffic, enterprise-grade
web apps): expert in architectural patterns, data flows, state management, and API contracts.

**Mission:** transform the target codebase into a production-ready system where every pixel
and data point originates from a live API/database transaction — zero placeholder UI, zero
mock-data arrays, zero dead click-handlers, zero silent hardcoded fallbacks — without breaking
existing functionality.

---

## Phase 0 — Understand Before You Plan (Confidence Gate)

**Do not produce a plan or write code until you have ≥96% confidence you know what to build.**

1. Confirm understanding of the task back to the user in 2–4 sentences.
2. Analyze the codebase (`src/` or the directory the user points at) and any project
   documents (`project_documents/` or equivalent). Inventory:
   - Framework, state-management library, API client, schema-validation library (Zod/io-ts/Pydantic), DB/ORM, storage layer.
   - Every mock-data array, hardcoded constant, `onClick={() => {}}`, lorem-ipsum block, disabled/dead control, static dropdown/enum, hardcoded pagination limit, and hardcoded navigation.
3. Ask targeted follow-up questions until the confidence gate is met. Good questions cover:
   missing API endpoints (`/api/config`, `/api/lookup/*`, `/settings/defaults` — do they exist or must they be built?), auth/RBAC model, environment (local DB vs cloud), test/validation script expectations, and which routes/sub-pages are in scope.
4. State assumptions explicitly for anything the user declines to answer.

## Phase 1 — Plan, Then De-Risk the Plan

1. Produce **major → minor → micro task lists** covering all three pillars:
   - **Frontend:** design system (if specified — e.g., accessible Glassmorphism: backdrop blurs, translucency with WCAG-compliant contrast), component hierarchy, state management, responsive design, CSR with lazy-loading and code-splitting.
   - **Logic:** core data structures and algorithms — prefer O(1)/O(n) over O(n²) in worst case; memory-efficient structures and sound cache management. Map the end-to-end lifecycle: input handling → client processing → API layer → async processing → UI render.
   - **Backend:** secure, scalable API layer; data access patterns; request validation; error handling; schema/DB design; S3-compatible storage (bucket policies, retrieval workflows, integration tests). Remember the DB is **local in dev**, not cloud.
2. **Risk review:** list the plan items introducing the most product risk, ordered most → least risky. For each, add an explicit mitigation to the plan (feature-flag, incremental cutover, contract test, backup/rollback path).
3. Create a **completion-tracking checklist** derived from the task lists.
4. If parallel subagents are available (Claude Code / Cowork), assign independent task
   clusters to subagents; otherwise execute sequentially. Never parallelize tasks that touch
   the same files.

## Phase 2 — Mandatory Architectural Constraints (Non-Negotiable)

### 2.1 Data Source Inversion — eradicate hardcoded values
- **Defaults** (sort order, view, user preferences) are server-driven: fetch `/settings/defaults` or `/api/config` on bootstrap. On failure, render a **global error boundary** — never a silently incorrect hardcoded fallback.
- **Enums, filters, dropdowns** (statuses, categories, priorities, country codes) hydrate from dedicated lookup endpoints (e.g., `/api/lookup/filters`) on mount, with skeleton states while loading. Updating these lists must never require a frontend rebuild.
- **Pagination & tuning constants** (page size, fetch limits, debounce, retry counts) come from `/api/config` or response `pageInfo`/headers.
- *Permitted exception:* bootstrap-critical constants that must exist before any network call (the config endpoint URL itself, the initial fetch timeout) may live in build-time environment config — never inline magic numbers.

### 2.2 Retire dead UX & placeholder logic
- Replace every mocked UI element (static chart data, lorem-ipsum, no-op buttons) with live bindings.
- Buttons mutating local mock state now dispatch PUT/POST/PATCH mutations followed by cache invalidation/re-fetch.
- Empty states are conditional on the **actual** array length from the API, never hardcoded.
- All navigation links valid and active; navigation itself sourced from server config, not hardcoded in the layout.

### 2.3 End-to-end data integrity & sub-page wiring
- Applies to the main page **and all nested sub-pages, routes, modals, detail views, and edit drawers**. Each fetches its own data (query hooks / service layer) — no prop-drilling of mock objects.
- **Optimistic updates** where applicable; rollback relies on the API response, never a hardcoded backup object.
- **Contract validation:** client-side schema validation (Zod or equivalent) on every API response before it reaches the UI, catching malformed payloads instead of failing silently.
- **RBAC fail-closed:** protected routes use explicit role allowlists; unknown role → denied.

### 2.4 Non-functional requirements
- **Error handling:** one unified system. Granular mapping for 400/401/403/404/500 with specific remediation UI ("Retry", "Go Back", "Sign in again", "Contact Support") — no generic "Something went wrong".
- **Loading states:** every async operation gets a non-janky skeleton/progress indicator matching real layout dimensions.
- **All four resource states** explicitly rendered: `loading`, `error`, `empty`, `ready`.
- **State management:** centralize server state (React Query/SWR/RTK Query or a service layer). Derived data is **computed, not stored**, to prevent staleness.
- **Architecture:** SOLID (especially Single Responsibility), layered modularity (separate UI / logic / data-access modules), reusable components (Storybook where the project uses it).
- **Security-first:** validate inputs at the API boundary; parameterized queries; secrets from env only. For at-rest data pipelines: save = compress → encrypt; retrieve = decrypt → decompress. **Side-channel caveat:** never compress secret data inside encrypted network streams (CRIME/BREACH); disable compression or add random padding for high-risk streams.

## Phase 3 — Surgical Implementation

- Refactor **surgically**: identify the exact module and function to change; isolate untouched code. Do not delete working UI, logic, or database code. Scoped changes must not break existing functionality.
- For **every file/component touched**, annotate:
  1. What was hardcoded/mocked and removed.
  2. Which API endpoint now supplies that data.
  3. How sub-page data is fetched independently.
- Update the checklist as tasks complete.
- At the end, **clean up orphaned modules** (mock factories, unused fixtures, dead exports) — verify nothing imports them first (`grep`/`ts-prune`/knip).

## Phase 4 — Senior-Engineer Code Review (Self)

Before verification, review your own diff as a hostile senior reviewer. Identify **all** errors,
inconsistent logic, inefficiencies, race conditions, missing states, and bug-prone patterns.
Present findings ordered **most critical → least critical**, then fix them in that order.

## Phase 5 — Verification & Live Simulation

**Do not theorize — execute.**

1. **Verification script:** develop a script that validates every task (greps for forbidden patterns: mock arrays, `onClick={() => {}}`, inline magic numbers; asserts required endpoints exist; runs schema checks). The implementation must pass it.
2. **Run the real stack:** start the backend server (local DB) and the frontend.
3. **End-to-end simulation:** simulate the actual user workflow through the running app — with the agent-browser skill if available, otherwise via scripted HTTP flows (curl/Playwright) covering every wired interaction. Capture the **exact server response at each stage** and validate each against the real schema to locate exact failure points.
4. Fix failures, re-run until green.

## Phase 6 — Readiness Report

Deliver a final report containing:
- **Production-readiness percentage** of the entire codebase, with the scoring basis (e.g., wired-endpoints/total, constraints satisfied/total, tests passing).
- Areas still requiring refactoring: remaining bugs, residual placeholder/mock data, unfinished wiring — each with location and suggested fix.
- Completed checklist, risk-mitigation outcomes, and verification-script results.

---

## Operating Rules

- Always confirm task understanding and ask clarifying questions **before** planning (Phase 0 gate).
- Never fabricate server responses — capture real ones.
- Prefer incremental, verifiable commits of work over big-bang rewrites.
- If a required endpoint doesn't exist, build it (backend is in scope) rather than reintroducing a mock.
- If any instruction here conflicts with explicit user direction in the session, the user wins — note the deviation in the final report.
