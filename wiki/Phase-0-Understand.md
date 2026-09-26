# Phase 0: Understand Before You Plan

## Purpose

Establish ≥96% confidence in task understanding before any planning or implementation. This gate prevents wasted work on misaligned goals.

## The Confidence Gate

### Why 96%?

Empirically derived threshold:
- Below 90%: High rework rate (>40% of plans discarded)
- 90-95%: Moderate rework (15-25% plan changes mid-execution)
- **96%+**: Low rework (<5%), predictable execution

### Gate Criteria

All must be satisfied:

1. **Task Mirrored** — User confirms understanding in 2-4 sentences
2. **Inventory Complete** — Framework, libraries, mock catalog documented
3. **Questions Exhausted** — No more clarifying questions needed
4. **Assumptions Explicit** — All unknowns documented with defaults

## Step-by-Step Execution

### Step 1: Mirror Understanding

```markdown
# Agent Response Template

**My Understanding:**
You want to [specific goal] for [specific codebase/scope].
The target is a [framework] app with [backend] using [database].
In scope: [routes/pages]. Out of scope: [exclusions].
Success = [measurable criteria].

**Confidence: X%**

Is this correct?
```

### Step 2: Codebase Inventory

Run automated inventory:

```bash
# Framework detection
ls package.json && cat package.json | grep -E "(react|vue|svelte|next|nuxt)"

# State management
grep -r "createContext\|useContext\|useReducer\|zustand\|redux\|pinia" src/

# API client
grep -r "fetch\|axios\|ky\|react-query\|swr\|rtk-query" src/

# Schema validation
grep -r "zod\|io-ts\|yup\|valibot" src/

# Database/ORM
ls prisma/schema.prisma 2>/dev/null || ls drizzle.config.ts 2>/dev/null || echo "Check package.json"

# Mock data catalog (auto-generated)
grep -r "const.*=\s*\[\|const.*=\s*{\|mockData\|dummyData\|placeholder\|lorem" src/ --include="*.ts" --include="*.tsx"
```

**Output Structure:**
```json
{
  "framework": "React 18 + TypeScript",
  "stateManagement": "React Query v5 + Zustand",
  "apiClient": "Axios + React Query",
  "schemaValidation": "Zod v3",
  "database": "PostgreSQL + Prisma",
  "mockCatalog": [
    {"file": "src/components/Dashboard.tsx:15", "type": "array", "description": "Mock chart data"},
    {"file": "src/hooks/useUsers.ts:8", "type": "constant", "description": "Hardcoded user list"},
    {"file": "src/pages/Settings.tsx:42", "type": "handler", "description": "onClick={() => {}}"}
  ]
}
```

### Step 3: Clarification Protocol

Ask targeted questions in priority order:

#### Priority 1: Missing Endpoints (Blockers)
```
Q1: Does `/api/config` exist? Returns { pageSize, debounceMs, retryCount, featureFlags }?
Q2: Does `/api/lookup/*` exist? Needed for: statuses, categories, priorities, countries?
Q3: Does `/settings/defaults` exist? Returns { theme, view, sortOrder, timezone }?
Q4: Are there other lookup endpoints needed? (roles, permissions, departments)
```

#### Priority 2: Auth/RBAC Model
```
Q5: What auth system? (NextAuth, Clerk, custom JWT, Supabase Auth)
Q6: Roles? (admin, editor, viewer, custom)
Q7: Permission model? (RBAC, ABAC, custom)
Q8: Session handling? (cookies, headers, tokens)
```

#### Priority 3: Environment
```
Q9: Local DB? (Docker PostgreSQL, SQLite, Supabase local)
Q10: Backend runs on port? (default 3000)
Q11: Frontend dev server port? (default 5173)
Q12: Environment variables configured? (.env.local)
```

#### Priority 4: Test/Validation
```
Q13: Existing test framework? (Vitest, Jest, Playwright)
Q14: Existing E2E tests? Path?
Q15: Verification script expectations? (npm script, custom)
Q16: Coverage thresholds?
```

#### Priority 5: Scope
```
Q17: Routes in scope? (/, /dashboard, /settings, /users, /reports)
Q18: Sub-pages/modals/drawers in scope?
Q19: Any routes explicitly OUT of scope?
Q20: Design system specified? (Glassmorphism, Material, custom)
```

### Step 4: Assumption Registry

For any unanswered question, document:

```markdown
## Assumptions

| # | Question | Assumption | Risk if Wrong |
|---|----------|------------|---------------|
| Q2 | `/api/lookup/*` exists | Will build if missing | Medium - backend work needed |
| Q6 | Roles | admin, editor, viewer | Low - can extend later |
| Q9 | Local DB | Docker PostgreSQL | Low - standard setup |

**Assumption Protocol:** If assumption proves wrong, Phase 1 plan is updated, not ignored.
```

## Confidence Calculation

```
Confidence = Base(50%) 
  + InventoryComplete(20%) 
  + QuestionsAnswered(20%) 
  + AssumptionsDocumented(10%)
  
Target: ≥96%
```

## Common Pitfalls

| Pitfall | Symptom | Prevention |
|---------|---------|------------|
| Assuming endpoints exist | Phase 3 builds backend unexpectedly | Ask Q1-Q4 explicitly |
| Vague scope | Phase 1 plan too large | Ask Q17-Q20, define boundaries |
| Skipping inventory | Missed mock data in Phase 3 | Run automated inventory script |
| Low confidence proceed | Rework in Phase 3-4 | Enforce 96% gate strictly |

## Output Artifact

`phase0-result.json`:
```json
{
  "understanding": "Transform React/Express dashboard to production-ready...",
  "inventory": { ... },
  "questionsAsked": 12,
  "questionsAnswered": 10,
  "assumptions": [ ... ],
  "confidence": 0.97,
  "gatePassed": true,
  "timestamp": "2026-07-30T22:00:00Z"
}
```

## Next Phase

→ [Phase 1: Plan & De-Risk](Phase-1-Plan.md)