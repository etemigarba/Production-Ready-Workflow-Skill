# Phase 4: Senior Engineer Self-Review

## Purpose

Conduct a hostile, thorough code review of all changes before verification. The agent reviews its own diff as a senior engineer would — identifying every error, inconsistency, and risk.

## Review Methodology

### Review Persona

Adopt the mindset of a **hostile senior reviewer**:
- Assume every change has a bug
- Look for what *could* go wrong, not what *looks* right
- Prioritize production impact over code aesthetics
- Question every abstraction, every optimization, every "clever" solution

### Review Dimensions (Ordered by Criticality)

| Priority | Dimension | Focus |
|----------|-----------|-------|
| **1. Correctness** | Logic errors, race conditions, boundary cases | Does it actually work? |
| **2. Security** | Input validation, auth bypass, injection, secrets | Can it be exploited? |
| **3. Performance** | N+1 queries, memory leaks, bundle size, blocking | Will it scale? |
| **4. Maintainability** | Coupling, duplication, naming, SOLID | Can others maintain it? |
| **5. Observability** | Logs, metrics, traces, error context | Can we debug production issues? |

## Review Process

### Step 1: Diff Analysis

```bash
# Get full diff
git diff HEAD~${COMMITS_SINCE_PHASE3_START} --stat

# Per-file review
git diff HEAD~${COMMITS_SINCE_PHASE3_START} src/hooks/useStatusLookup.ts
git diff HEAD~${COMMITS_SINCE_PHASE3_START} src/components/StatusFilter.tsx
# ... repeat for every changed file
```

### Step 2: Systematic Review Checklist

#### Correctness (Priority 1)

```markdown
## Correctness Findings

### CRITICAL: Race condition in useStatusLookup
**File:** src/hooks/useStatusLookup.ts:15
**Issue:** No request deduplication — rapid mounts cause multiple fetches
**Fix:** Add `queryKey: ['lookup', 'statuses']` with `staleTime` (already done) ✓
**Remaining:** Add `refetchOnMount: false` for lookup data

### HIGH: Optimistic update rollback uses stale snapshot
**File:** src/hooks/useToggleStatus.ts:28
**Issue:** `context.previous` captured after `cancelQueries` but before `setQueryData`
**Fix:** Move snapshot before cancelQueries

### MEDIUM: Empty state triggers on null vs undefined inconsistently
**Files:** src/components/UserList.tsx, src/components/Chart.tsx
**Issue:** UserList checks `!data`, Chart checks `data?.length === 0`
**Fix:** Standardize on `isEmpty` utility

### LOW: TypeScript `any` in error handler
**File:** src/lib/errors/unified.ts:45
**Issue:** `body: unknown` cast to `any` for property access
**Fix:** Use proper type guards
```

#### Security (Priority 2)

```markdown
## Security Findings

### CRITICAL: RBAC bypass on /api/admin/* routes
**File:** src/app/api/admin/route.ts
**Issue:** Middleware checks `session.user.role === 'admin'` but session can be spoofed
**Fix:** Validate JWT signature server-side; use middleware that verifies token

### HIGH: SQL injection risk in dynamic query
**File:** src/lib/db/queries.ts:67
**Issue:** `WHERE ${column} = ${value}` string interpolation
**Fix:** Use parameterized queries / query builder

### MEDIUM: Secrets in build output
**File:** next.config.js:12
**Issue:** `process.env.DATABASE_URL` used in client bundle via `NEXT_PUBLIC_` prefix
**Fix:** Move to server-only env; use `serverRuntimeConfig`

### LOW: Missing rate limiting on mutation endpoints
**File:** src/app/api/users/route.ts
**Issue:** POST /api/users has no rate limit
**Fix:** Add rate limiter middleware
```

#### Performance (Priority 3)

```markdown
## Performance Findings

### HIGH: N+1 queries in dashboard load
**File:** src/app/api/dashboard/route.ts
**Issue:** Fetches user, then loops to fetch each user's posts
**Fix:** Single query with JOIN or batch loader

### MEDIUM: Bundle size increased 15%
**Files:** src/components/ui/*.tsx (new Glassmorphism components)
**Issue:** Heavy use of `framer-motion` in base components
**Fix:** Lazy-load animations; use CSS-only glassmorphism

### LOW: Unnecessary re-renders in UserList
**File:** src/components/UserList.tsx:34
**Issue:** `useUsers` returns new array reference each render
**Fix:** Memoize with `useMemo` or stable query key
```

#### Maintainability (Priority 4)

```markdown
## Maintainability Findings

### MEDIUM: God component: Dashboard.tsx (450 lines)
**File:** src/pages/Dashboard.tsx
**Issue:** Handles layout, data fetching, charts, stats, activity
**Fix:** Split into DashboardLayout, DashboardCharts, DashboardStats, DashboardActivity

### LOW: Inconsistent naming: useUsers vs useUserList vs getUsers
**Files:** Multiple hooks
**Issue:** No consistent convention
**Fix:** Adopt `use<Resource><Action>` convention (useUsers, useUser, useCreateUser)

### LOW: Magic string 'admin' in 12 files
**Files:** Multiple
**Issue:** Role strings scattered
**Fix:** Centralize in `src/lib/auth/roles.ts`
```

#### Observability (Priority 5)

```markdown
## Observability Findings

### MEDIUM: No error context in production logs
**Files:** All mutation hooks
**Issue:** Errors logged without userId, entityId, correlationId
**Fix:** Add `onError` handler with context enrichment

### LOW: Missing performance marks
**Files:** All query hooks
**Issue:** No `performance.mark` for query timing
**Fix:** Add `performance.mark('query-start')` / `measure` in queryFn
```

## Finding Classification

### Severity Levels

| Level | Criteria | Response Time |
|-------|----------|---------------|
| **CRITICAL** | Data loss, security breach, production crash | Fix immediately (before Phase 5) |
| **HIGH** | Functional bug, significant perf degradation | Fix before Phase 5 |
| **MEDIUM** | Maintainability debt, observability gap | Fix in Phase 3 if time, else track |
| **LOW** | Style, convention, minor optimization | Track in tech debt backlog |

### Fix Ordering

1. All CRITICAL → HIGH (blocking)
2. MEDIUM (if time permits)
3. LOW (deferred to backlog)

## Review Output

### Structured Findings

```json
{
  "reviewId": "review-2026-07-30-001",
  "timestamp": "2026-07-30T23:45:00Z",
  "reviewer": "production-ready-workflow (self)",
  "findings": [
    {
      "id": "F-1",
      "severity": "CRITICAL",
      "dimension": "correctness",
      "file": "src/hooks/useToggleStatus.ts",
      "line": 28,
      "title": "Optimistic update rollback uses stale snapshot",
      "description": "Snapshot captured after cancelQueries, missing intermediate state",
      "fix": "Move snapshot before cancelQueries",
      "status": "fixed",
      "fixCommit": "a1b2c3d"
    },
    {
      "id": "F-2",
      "severity": "HIGH",
      "dimension": "security",
      "file": "src/app/api/admin/route.ts",
      "line": 15,
      "title": "RBAC bypass via session spoofing",
      "description": "Client-side role check without server validation",
      "fix": "Add server-side JWT verification middleware",
      "status": "fixed",
      "fixCommit": "e4f5g6h"
    }
  ],
  "summary": {
    "critical": 2,
    "high": 3,
    "medium": 5,
    "low": 8,
    "total": 18,
    "fixed": 5,
    "deferred": 13
  }
}
```

## Self-Review Automation

### Automated Checks (Run Before Manual Review)

```bash
#!/bin/bash
# pre-review-checks.sh

echo "🤖 Automated pre-review checks..."

# 1. TypeScript strict mode
npm run typecheck -- --noEmit 2>&1 | grep -E "error|warning" || echo "✓ Clean"

# 2. ESLint with strict rules
npm run lint -- --max-warnings=0 2>&1 | tail -20

# 3. Complexity analysis
npx complexity-report src/ --threshold cyclomatic:10 --threshold halstead:50

# 4. Bundle size check
npm run build && npx bundlesize

# 5. Dependency audit
npm audit --audit-level=high

# 6. Secret scan
npx detect-secrets scan src/

# 7. Test coverage
npm test -- --coverage --coverageThreshold='{"global":{"statements":80}}'
```

### Review Template

```markdown
# Phase 4 Self-Review Report

## Summary
- Files reviewed: 23
- Lines changed: +1,247 / -892
- Commits in scope: 12
- Review duration: 45 minutes

## Findings by Severity
| Severity | Count | Fixed | Deferred |
|----------|-------|-------|----------|
| CRITICAL | 2 | 2 | 0 |
| HIGH | 3 | 3 | 0 |
| MEDIUM | 5 | 2 | 3 |
| LOW | 8 | 0 | 8 |
| **TOTAL** | **18** | **7** | **11** |

## Critical/High Fixes Applied
1. ✅ F-1: Fixed optimistic update rollback race condition
2. ✅ F-2: Added server-side RBAC validation
3. ✅ F-3: Fixed N+1 query in dashboard API
4. ✅ F-4: Removed secrets from client bundle
5. ✅ F-5: Added rate limiting to mutation endpoints

## Deferred (Tech Debt)
- F-6: Split Dashboard.tsx (MEDIUM) — tracked as TD-124
- F-7: Standardize hook naming (LOW) — tracked as TD-125
- F-8: Centralize role strings (LOW) — tracked as TD-126
- ... 8 more

## Sign-off
**Review Status:** PASSED WITH CONDITIONS
**Conditions:** Deferred items tracked in backlog with owner and due date
**Next Phase:** Proceed to Phase 5 Verification
```

## Next Phase

→ [Phase 5: Verification & Simulation](Phase-5-Verification.md)