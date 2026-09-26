# Phase 3: Surgical Implementation

## Purpose

Execute the plan through scoped, non-breaking refactors. Each change is isolated, annotated, and verified before moving to the next.

## Surgical Principles

### 1. Identify Exact Module/Function

```bash
# Before any change, locate precisely
grep -rn "STATUS_OPTIONS" src/ --include="*.tsx"
# Output: src/components/StatusFilter.tsx:12
#         src/hooks/useStatusFilter.ts:5

# Analyze usage
grep -B5 -A5 "STATUS_OPTIONS" src/components/StatusFilter.tsx
```

### 2. Isolate Untouched Code

```typescript
// BEFORE: Monolithic component
function StatusFilter({ value, onChange }) {
  const STATUSES = ['pending', 'active', 'done']; // HARDCODED
  return <select value={value} onChange={onChange}>
    {STATUSES.map(s => <option key={s} value={s}>{s}</option>)}
  </select>;
}

// AFTER: Separated concerns
// src/hooks/useStatusLookup.ts — NEW FILE
export function useStatusLookup() {
  return useQuery({
    queryKey: ['lookup', 'statuses'],
    queryFn: () => fetch('/api/lookup/statuses').then(r => r.json())
  });
}

// src/components/StatusFilter.tsx — REFACTORED
function StatusFilter({ value, onChange }) {
  const { data: statuses, isLoading } = useStatusLookup();
  
  if (isLoading) return <SelectSkeleton />;
  
  return <select value={value} onChange={onChange}>
    {statuses?.map(s => <option key={s.id} value={s.id}>{s.label}</option>)}
  </select>;
}
```

### 3. Annotation Protocol

Every modified file gets a header annotation:

```typescript
/**
 * REFACTOR: production-ready-workflow Phase 3
 * 
 * CHANGES:
 * 1. REMOVED: Hardcoded STATUS_OPTIONS array (line 12)
 * 2. ADDED: useStatusLookup hook fetching from /api/lookup/statuses
 * 3. ADDED: Loading skeleton state
 * 4. SUB-PAGE: StatusFilter now fetches independently (no prop drilling)
 * 
 * VERIFICATION:
 * - grep "STATUS_OPTIONS" returns no results in src/components/
 * - /api/lookup/statuses returns 200 with expected schema
 * - Component renders loading → ready states correctly
 */
```

### 4. Checklist Updates

Real-time checklist tracking:

```markdown
## Phase 3 Progress

### Frontend
- [x] FE-1.1 Color/spacing tokens defined
- [x] FE-1.2 Glassmorphism Card component
- [ ] FE-1.3 Responsive breakpoints (in progress)
- [x] FE-2.1 Dashboard layout components
- [x] FE-2.2 Chart components wired to API
  - [x] Replaced mockChartData with useAnalytics hook
  - [x] Added loading/error/empty states
  - [x] Contract validation with AnalyticsResponseSchema
- [ ] FE-3.1 Settings page live bindings
```

## Orphan Cleanup

### Detection

```bash
# Find potentially orphaned mock files
find src -name "*.mock.ts" -o -name "*.fixture.ts" -o -name "*Mock*.ts" | head -20

# Check imports
for file in $(find src -name "*.mock.ts"); do
  if ! grep -r "$(basename $file .ts)" src/ --include="*.ts" --include="*.tsx" | grep -v "$file"; then
    echo "ORPHAN: $file"
  fi
done

# Use knip for comprehensive analysis
npx knip --production
```

### Cleanup Process

```bash
# 1. Verify no imports
grep -r "mockData\|dummyData\|fixtureData" src/ --include="*.ts" --include="*.tsx"

# 2. Check test files separately (may legitimately use mocks)
grep -r "mockData\|dummyData" test/ --include="*.ts" --include="*.tsx"

# 3. Delete confirmed orphans
rm src/mocks/userMock.ts src/fixtures/chartData.ts

# 4. Verify build still passes
npm run build
npm run typecheck
```

## Implementation Patterns

### Pattern 1: Constant → Endpoint

```typescript
// BEFORE
const PAGE_SIZE = 20;

// AFTER
// src/hooks/usePagination.ts
export function usePagination() {
  const { data: config } = useConfig();
  return config?.pageSize ?? 20; // Fallback only for type safety
}

// Usage
function UserTable() {
  const pageSize = usePagination();
  const { data } = useUsers({ pageSize });
}
```

### Pattern 2: No-Op Handler → Real Mutation

```typescript
// BEFORE
<button onClick={() => {}}>Save</button>

// AFTER
function SaveButton({ formData }) {
  const mutation = useMutation({
    mutationFn: (data) => api.saveForm(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['form'] });
      toast.success('Saved');
    }
  });
  
  return <Button onClick={() => mutation.mutate(formData)} disabled={mutation.isPending}>
    {mutation.isPending ? 'Saving...' : 'Save'}
  </Button>;
}
```

### Pattern 3: Prop-Drilled Mock → Independent Fetch

```typescript
// BEFORE: Parent fetches, passes to children
function Dashboard() {
  const mockData = getMockDashboardData(); // HARDCODED
  return (
    <Charts data={mockData.charts} />
    <Stats data={mockData.stats} />
    <RecentActivity data={mockData.activity} />
  );
}

// AFTER: Each component fetches independently
function Dashboard() {
  return (
    <DashboardLayout>
      <Charts />      {/* Fetches /api/analytics/charts */}
      <Stats />       {/* Fetches /api/analytics/stats */}
      <RecentActivity /> {/* Fetches /api/activity/recent */}
    </DashboardLayout>
  );
}

// src/components/Charts.tsx
export function Charts() {
  const { data, isLoading, error } = useAnalyticsCharts();
  // ... four states
}
```

### Pattern 4: Static Navigation → Server-Driven

```typescript
// BEFORE
const NAV_ITEMS = [
  { href: '/dashboard', label: 'Dashboard' },
  { href: '/settings', label: 'Settings' }
];

// AFTER
// src/hooks/useNavigation.ts
export function useNavigation() {
  return useQuery({
    queryKey: ['navigation'],
    queryFn: () => fetch('/api/navigation').then(r => r.json()),
    staleTime: Infinity
  });
}

// src/components/Navigation.tsx
export function Navigation() {
  const { data: navItems } = useNavigation();
  // ... render with loading state
}
```

## Verification During Implementation

### Per-Change Verification

```bash
# After each file change:
# 1. Type check
npm run typecheck

# 2. Lint
npm run lint

# 3. Unit tests for affected module
npm test -- src/components/StatusFilter.test.tsx

# 4. Build check
npm run build
```

### Incremental Commits

```bash
# Commit each logical change
git add src/hooks/useStatusLookup.ts src/components/StatusFilter.tsx
git commit -m "refactor: replace hardcoded statuses with /api/lookup/statuses

- Added useStatusLookup hook
- Updated StatusFilter to use live data
- Added loading skeleton state
- Closes FE-2.3"

git add src/hooks/usePagination.ts src/components/UserTable.tsx
git commit -m "refactor: replace hardcoded PAGE_SIZE with /api/config

- Added usePagination hook reading from config
- Updated UserTable to use dynamic page size
- Closes FE-2.1"
```

## Common Implementation Challenges

### Challenge: Circular Dependencies

```typescript
// PROBLEM: useConfig needs auth, auth needs config
// SOLUTION: Bootstrap config separately

// src/lib/bootstrap.ts
export async function bootstrapApp() {
  // 1. Fetch minimal config (no auth required)
  const config = await fetch('/api/config').then(r => r.json());
  
  // 2. Initialize query client with config
  queryClient.setQueryData(['config'], config);
  
  // 3. Initialize auth
  await authClient.init(config.auth);
  
  // 4. Now app can render
  return config;
}
```

### Challenge: Loading State Flash

```typescript
// PROBLEM: Config loads fast, but UI flashes skeleton
// SOLUTION: Stale-while-revalidate with persistent cache

export function useConfig() {
  return useQuery({
    queryKey: ['config'],
    queryFn: getConfig,
    staleTime: 24 * 60 * 60 * 1000, // 24 hours
    initialData: () => {
      // Read from localStorage for instant render
      const cached = localStorage.getItem('app-config');
      return cached ? JSON.parse(cached) : undefined;
    },
    // Update cache on success
    onSuccess: (data) => {
      localStorage.setItem('app-config', JSON.stringify(data));
    }
  });
}
```

### Challenge: Migration from Global State

```typescript
// PROBLEM: App uses Redux for everything
// SOLUTION: Gradual migration, coexist during transition

// Keep Redux for client-only state (UI, forms)
// Move server state to React Query
// Use sync middleware for cross-cutting concerns

// src/lib/query/sync.ts
export function syncReduxToQuery(store: ReduxStore) {
  return store.subscribe(() => {
    const state = store.getState();
    // Sync user session to query cache
    queryClient.setQueryData(['session'], state.auth.session);
  });
}
```

## Next Phase

→ [Phase 4: Senior Self-Review](Phase-4-Review.md)