# Phase 1: Plan & De-Risk

## Purpose

Transform understanding into an executable, de-risked plan with completion tracking and parallel execution strategy.

## Planning Methodology

### Three-Pillar Decomposition

Every plan covers all three pillars simultaneously:

```
┌─────────────────────────────────────────────────────────────────┐
│                        PLAN STRUCTURE                            │
├──────────────────────┬──────────────────────┬───────────────────┤
│      FRONTEND        │       LOGIC          │      BACKEND      │
├──────────────────────┼──────────────────────┼───────────────────┤
│ Major: Design System │ Major: Data Structs  │ Major: API Layer  │
│ Minor: Components    │ Minor: Algorithms    │ Minor: Data Access│
│ Micro: Tokens        │ Micro: Cache Mgmt    │ Micro: Validation │
└──────────────────────┴──────────────────────┴───────────────────┘
```

### Task Granularity

| Level | Scope | Example | Est. Hours |
|-------|-------|---------|------------|
| **Major** | Architectural pillar | "Replace all mock data with live API bindings" | 4-8 |
| **Minor** | Feature area | "Wire dashboard charts to `/api/analytics`" | 1-3 |
| **Micro** | Single file/func | "Replace `STATUS_OPTIONS` const with `useStatusLookup` hook" | 0.25-1 |

## Plan Structure

### Output: `phase1-plan.json`

```json
{
  "pillars": {
    "frontend": {
      "major": [
        {
          "id": "FE-1",
          "title": "Design System Integration",
          "description": "Apply Glassmorphism tokens, WCAG 2.2 AA compliance",
          "minors": ["FE-1.1", "FE-1.2", "FE-1.3"],
          "estimateHours": 6
        }
      ],
      "minors": {
        "FE-1.1": { "title": "Color/spacing tokens", "files": ["src/styles/tokens.ts"], "estimate": 1 },
        "FE-1.2": { "title": "Glassmorphism components", "files": ["src/components/ui/Card.tsx"], "estimate": 2 },
        "FE-1.3": { "title": "Responsive breakpoints", "files": ["src/styles/breakpoints.ts"], "estimate": 1 }
      },
      "micros": { ... }
    },
    "logic": { ... },
    "backend": { ... }
  },
  "risks": [...],
  "checklist": [...],
  "schedule": { ... }
}
```

## Risk Review

### Risk Scoring

```
Risk Score = Impact × Probability × Detection Difficulty

Impact: 1-5 (1=cosmetic, 5=data loss/security)
Probability: 1-5 (1=rare, 5=certain)
Detection: 1-5 (1=obvious in review, 5=production only)
```

### Standard Risk Catalog

| Risk | Score | Mitigation |
|------|-------|------------|
| Missing `/api/config` endpoint | 5×4×3=60 | Feature-flag config fallback; build endpoint first |
| Breaking existing auth | 5×3×4=60 | Incremental cutover; contract tests on auth endpoints |
| N+1 queries in new endpoints | 4×4×3=48 | Query analysis in Phase 4; load test in Phase 5 |
| Cache invalidation bugs | 4×3×4=48 | Explicit invalidation keys; integration tests |
| RBAC bypass on new routes | 5×2×5=50 | Fail-closed default; explicit allowlist per route |
| Bundle size regression | 3×3×2=18 | Bundle analyzer in CI; size budgets |
| TypeScript errors from new types | 2×4×2=16 | Strict mode; type-check in verification script |

### Risk Output Format

```json
{
  "risks": [
    {
      "id": "R-1",
      "title": "Missing /api/config endpoint",
      "pillar": "backend",
      "score": 60,
      "impact": 5,
      "probability": 4,
      "detection": 3,
      "mitigation": "Build endpoint in Phase 3 before frontend wiring; feature-flag with localStorage fallback",
      "owner": "backend-subagent",
      "status": "mitigated"
    }
  ]
}
```

## Completion Checklist

### Derived from Tasks

Every major → minor → micro becomes a checklist item:

```markdown
## Completion Checklist

### Frontend
- [ ] FE-1.1 Color/spacing tokens defined
- [ ] FE-1.2 Glassmorphism Card component
- [ ] FE-1.3 Responsive breakpoints
- [ ] FE-2.1 Dashboard layout components
- [ ] FE-2.2 Chart components wired to API
- [ ] FE-3.1 Settings page live bindings
- [ ] FE-3.2 User management CRUD wired

### Logic
- [ ] LO-1.1 User session cache (LRU, 5min TTL)
- [ ] LO-1.2 Analytics data transformation
- [ ] LO-2.1 Optimistic update helpers

### Backend
- [ ] BE-1.1 GET /api/config endpoint
- [ ] BE-1.2 GET /api/lookup/* endpoints
- [ ] BE-1.3 GET /settings/defaults endpoint
- [ ] BE-2.1 POST /api/analytics endpoint
- [ ] BE-2.2 RBAC middleware on all routes
- [ ] BE-3.1 Database migrations for config table
```

### Checklist Tracking

- Updated in real-time during Phase 3
- Visible in Phase 4 review
- Final status in Phase 6 report

## Subagent Scheduling

### Parallelization Rules

1. **Never parallelize** tasks touching same files
2. **Group by pillar** — Frontend/Logic/Backend can run parallel
3. **Within pillar** — Independent minors can run parallel
4. **Dependencies** — Explicit ordering where required

### Schedule Example

```
Time 0-2h:    Phase 0-1 (single agent)
Time 2-4h:    BE-1.1, BE-1.2, BE-1.3  (parallel backend subagents)
Time 4-6h:    FE-1.1, FE-1.2, FE-1.3  (parallel frontend subagents)
Time 6-8h:    FE-2.1, FE-2.2, LO-1.1  (depends on BE-1.*)
Time 8-10h:   BE-2.1, BE-2.2          (depends on FE-2.*)
Time 10-12h:  FE-3.1, FE-3.2, LO-1.2  (depends on BE-2.*)
Time 12-14h:  Integration, cleanup
Time 14-15h:  Phase 4 Review
Time 15-16h:  Phase 5 Verification
Time 16-16.5h: Phase 6 Report
```

### Subagent Communication

```json
{
  "subagents": [
    {
      "id": "backend-1",
      "pillar": "backend",
      "tasks": ["BE-1.1", "BE-1.2", "BE-1.3"],
      "dependencies": [],
      "artifacts": ["openapi.yaml", "prisma/migrations"]
    },
    {
      "id": "frontend-1",
      "pillar": "frontend",
      "tasks": ["FE-1.1", "FE-1.2", "FE-1.3"],
      "dependencies": [],
      "artifacts": ["tokens.ts", "components/ui/"]
    }
  ]
}
```

## Plan Approval

### Human Gate

Before Phase 2, present:

1. **Plan Summary** — 3 pillars, total estimates, risk count
2. **Risk Heatmap** — Visual risk matrix
3. **Checklist Preview** — All items
4. **Schedule** — Timeline with parallelization

### Approval Options

- **Approve** → Proceed to Phase 2
- **Modify** → Specific changes requested
- **Defer** → Move risky items to later phase with feature flags
- **Split** → Create separate skill invocation for subset

## Next Phase

→ [Phase 2: Mandatory Constraints](Phase-2-Constraints.md)