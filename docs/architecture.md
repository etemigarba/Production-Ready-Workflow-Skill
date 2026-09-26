# Architecture Overview

## System Context

The `production-ready-workflow` skill operates as an **agentic workflow orchestrator** within the Claude Code / ECC ecosystem. It transforms a target codebase through six deterministic phases, each building on the previous.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        production-ready-workflow                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Phase 0  │──│ Phase 1  │──│ Phase 2  │──│ Phase 3  │──│ Phase 4  │──... │
│  │Understand│  │ Plan     │  │Constrain │  │Implement │  │Review    │      │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
│                                                                                │
│  ┌──────────┐  ┌──────────┐                                                │
│  │ Phase 5  │──│ Phase 6  │                                                │
│  │ Verify   │  │ Report   │                                                │
│  └──────────┘  └──────────┘                                                │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Phase Architecture

### Phase 0: Confidence Gate (`ConfidenceGate`)

**Purpose:** Ensure ≥96% confidence before planning

**Components:**
- `TaskUnderstanding` — Mirror task back to user
- `CodebaseInventory` — Framework, libraries, mock data catalog
- `ClarificationEngine` — Targeted questions until gate met
- `AssumptionRegistry` — Explicit assumptions for unanswered items

**Output:** `Phase0Result { understanding, inventory, assumptions, confidence }`

### Phase 1: Planning & Risk (`Planner`)

**Purpose:** Produce executable, de-risked plan

**Components:**
- `TaskDecomposer` — Major → minor → micro across 3 pillars
- `RiskAnalyzer` — Product risk scoring with mitigations
- `ChecklistGenerator` — Completion tracking from tasks
- `SubagentScheduler` — Parallel task clustering (non-conflicting files)

**Output:** `Phase1Result { tasks, risks, checklist, schedule }`

### Phase 2: Constraints (`ConstraintEngine`)

**Purpose:** Enforce non-negotiable architectural rules

**Constraints (Hard-coded, not configurable):**

| Constraint | Implementation |
|------------|----------------|
| **Data Source Inversion** | `ConfigFetcher` bootstrap, `LookupHydrator` for enums, `PaginationResolver` |
| **Dead UX Retirement** | `MockDetector` → `LiveBinder` replacement |
| **E2E Data Integrity** | `IndependentFetcher` per sub-page, `OptimisticUpdater`, `RbacGuard` |
| **NFR Compliance** | `UnifiedErrorHandler`, `SkeletonRenderer`, `StateRenderer`, `ServerStateCentralizer` |

**Output:** `Phase2Result { appliedConstraints, violations }`

### Phase 3: Surgical Implementation (`Implementer`)

**Purpose:** Scoped, non-breaking refactors

**Components:**
- `ChangeLocator` — Exact module/function identification
- `IsolationWrapper` — Preserve untouched code
- `AnnotationLogger` — Per-file: removed, endpoint, fetch strategy
- `OrphanCleaner` — `grep`/`knip` verification before deletion

**Output:** `Phase3Result { changes, annotations, cleaned }`

### Phase 4: Senior Review (`Reviewer`)

**Purpose:** Hostile code review by the agent itself

**Review Dimensions (ordered):**
1. **Correctness** — Logic errors, race conditions, boundary cases
2. **Security** — Input validation, auth bypass, injection vectors
3. **Performance** — N+1 queries, memory leaks, bundle size
4. **Maintainability** — Coupling, duplication, naming, SOLID violations
5. **Observability** — Missing logs, metrics, traces

**Output:** `Phase4Result { findings[], fixesApplied[] }`

### Phase 5: Verification (`Verifier`)

**Purpose:** Empirical validation via live execution

**Components:**
- `VerificationScript` — Forbidden pattern detection, endpoint assertion, schema checks
- `StackRunner` — Backend (local DB) + Frontend startup
- `SimulationEngine` — Browser automation (agent-browser) or HTTP flows (Playwright/curl)
- `ResponseValidator` — Schema validation of captured responses

**Output:** `Phase5Result { scriptPass, stackRunning, simulationResults, failures }`

### Phase 6: Reporting (`Reporter`)

**Purpose:** Quantified readiness assessment

**Metrics:**
```
Readiness % = (WiredEndpoints/TotalEndpoints × 0.4) +
              (ConstraintsSatisfied/TotalConstraints × 0.3) +
              (TestsPassing/TotalTests × 0.3)
```

**Report Structure:**
- Overall percentage with scoring basis
- Remaining work: location, issue, suggested fix
- Completed checklist
- Risk mitigation outcomes
- Verification script results

## Three-Pillar Technical Architecture

### Frontend Pillar

```
┌────────────────────────────────────────────────────────────┐
│                      Frontend                               │
├────────────────────────────────────────────────────────────┤
│  Design System          │  Component Hierarchy              │
│  ├─ Tokens (colors,     │  ├─ Atomic (Button, Input)        │
│  │  spacing, type)      │  ├─ Molecular (Form, Card)        │
│  ├─ Glassmorphism       │  ├─ Organism (Header, Sidebar)    │
│  │  (backdrop-blur,     │  └─ Template (Dashboard, Settings)│
│  │   translucency)      │                                   │
│  ├─ WCAG 2.2 AA         │  State Management                 │
│  └─ Responsive          │  ├─ Server State (React Query)    │
│                         │  ├─ Client State (Zustand/Redux)  │
│  CSR + Lazy Loading     │  └─ Derived (selectors, memo)     │
│  ├─ Code Splitting      │                                   │
│  ├─ Suspense Boundaries │  Accessibility                    │
│  └─ Preload Critical    │  ├─ ARIA, Focus Management        │
└─────────────────────────┴───────────────────────────────────┘
```

### Logic Pillar

```
┌────────────────────────────────────────────────────────────┐
│                        Logic                                 │
├────────────────────────────────────────────────────────────┤
│  Data Structures       │  Algorithms                        │
│  ├─ O(1) Lookups       │  ├─ Complexity Budget             │
│  ├─ LRU Cache          │  ├─ O(n) preferred over O(n²)     │
│  ├─ Immutable Trees    │  ├─ Memoization                   │
│  └─ Event Sourcing     │  └─ Batch Processing              │
│                         │                                   │
│  Lifecycle Map         │  Concurrency                      │
│  Input → Client → API  │  ├─ Optimistic Updates            │
│  → Async → Render      │  ├─ Cache Invalidation            │
│                        │  └─ Race Condition Guards         │
└────────────────────────┴───────────────────────────────────┘
```

### Backend Pillar

```
┌────────────────────────────────────────────────────────────┐
│                       Backend                                │
├────────────────────────────────────────────────────────────┤
│  API Layer              │  Data Access                       │
│  ├─ Contract-First      │  ├─ Repository Pattern            │
│  │  (OpenAPI/Zod)       │  ├─ Query Builder (Kysely/Prisma) │
│  ├─ Versioning          │  ├─ Migrations                    │
│  ├─ Rate Limiting       │  └─ Connection Pooling            │
│  ├─ Auth (OAuth/OIDC)   │                                   │
│  ├─ RBAC/ABAC           │  Storage (S3-compatible)          │
│  └─ Error Mapping       │  ├─ Bucket Policies               │
│                         │  ├─ Retrieval Workflows           │
│  Schema/DB              │  └─ Integration Tests             │
│  ├─ BCNF Normalization  │                                   │
│  ├─ Indexing Strategy   │  Security                         │
│  ├─ Partitioning        │  ├─ Parameterized Queries         │
│  └─ Caching (Redis)     │  ├─ Secrets from Env              │
│                         │  ├─ Encrypt at Rest               │
│                         │  └─ CRIME/BREACH Mitigation       │
└────────────────────────┴───────────────────────────────────┘
```

## Data Flow

### Phase 0 → 1: Inventory to Plan

```
Codebase Inventory
       │
       ▼
┌──────────────────┐
│ Mock Data Map    │──► Task: Replace with endpoint
│ ├─ Arrays        │
│ ├─ Constants     │
│ ├─ No-op handlers│
│ └─ Static enums  │
└──────────────────┘
       │
       ▼
┌──────────────────┐
│ Missing Endpoints│──► Task: Build endpoint (backend in scope)
│ ├─ /api/config   │
│ ├─ /api/lookup/* │
│ └─ /settings/def │
└──────────────────┘
```

### Phase 3: Surgical Change Pattern

```
BEFORE                          AFTER
────────────────────────────────────────────────────────────
const STATUS_OPTIONS = [         // DELETE: Hardcoded enum
  'pending', 'active', 'done'
]                                // REPLACE WITH:
                                 const { data: statuses } = 
                                   useQuery(['lookup/status'],
                                     () => fetch('/api/lookup/status'))
                                 
// Component                      // Component
<select>                          <select>
  {STATUS_OPTIONS.map(s =>       {statuses?.map(s => (
    <option>{s}</option>           <option key={s.id}>{s.label}</option>
  ))}                              ))}
</select>                         </select>
```

### Phase 5: Verification Flow

```
Verification Script
       │
       ├─► grep "mock\|hardcoded\|onClick={() => {}}" → FAIL if found
       │
       ├─► curl /api/config → 200 + valid schema
       │
       ├─► curl /api/lookup/* → 200 + array response
       │
       └─► npm test → all pass
              │
              ▼
      Stack Runner
       │
       ├─► docker-compose up -d postgres
       ├─► npm run dev:backend (port 3000)
       └─► npm run dev:frontend (port 5173)
              │
              ▼
      Simulation Engine
       │
       ├─► Browser: Navigate → Click → Assert
       └─► HTTP: POST → GET → Validate schema
              │
              ▼
      Response Validator
       │
       └─► Zod.parse(response) → PASS/FAIL
```

## Extension Points

### Custom Verification Rules

```typescript
// .claude/skills/production-ready-workflow/verification-rules.ts
export const customRules = [
  {
    name: 'no-direct-database-imports',
    pattern: /from ['"]@\/db['"]/,
    message: 'Use repository layer, not direct DB imports'
  },
  {
    name: 'api-version-in-path',
    pattern: /\/api\/v\d+\//,
    message: 'API version must be in path'
  }
];
```

### Custom Constraint Exceptions

```json
// config.json
{
  "phase2": {
    "allowedHardcodedConstants": [
      "CONFIG_ENDPOINT_URL",
      "INITIAL_FETCH_TIMEOUT_MS",
      "MAX_FILE_SIZE_MB"
    ],
    "allowedMockPaths": [
      "test/**",
      "**/*.stories.tsx",
      "**/*.test.ts"
    ]
  }
}
```

### Custom Phase Hooks

```typescript
// hooks.ts
export const hooks = {
  beforePhase3: async (context) => {
    // Create backup branch
    await git.checkout('-b', `backup-before-phase3-${Date.now()}`);
  },
  afterPhase5: async (context) => {
    // Notify team
    await slack.post(`Readiness: ${context.report.readiness}%`);
  }
};
```

## Performance Characteristics

| Phase | Typical Duration | Complexity |
|-------|------------------|------------|
| Phase 0 | 5-15 min | O(files) inventory |
| Phase 1 | 10-30 min | O(tasks) decomposition |
| Phase 2 | Instant | Rule application |
| Phase 3 | 30-120 min | O(changes) surgical edits |
| Phase 4 | 15-45 min | O(diff) review |
| Phase 5 | 5-20 min | O(endpoints) simulation |
| Phase 6 | Instant | Report generation |

**Total:** 1-4 hours for typical medium codebase (~50 components, ~30 endpoints)

## Security Considerations

- **No code execution** — Skill orchestrates, doesn't run arbitrary code
- **Local-only** — All operations on local filesystem
- **No network calls** — Except Phase 5 to local backend
- **Secrets never logged** — Env vars redacted in output
- **Git-based rollback** — Every phase commits for safety

## Compatibility

| Component | Minimum Version | Tested Version |
|-----------|-----------------|----------------|
| Claude Code | 1.0.0 | Latest |
| ECC/opencode | 0.5.0 | Latest |
| Node.js | 18.0.0 | 20.x, 22.x |
| React | 18.0.0 | 18.x, 19.x |
| TypeScript | 5.0.0 | 5.x |
| Zod | 3.0.0 | 3.x |