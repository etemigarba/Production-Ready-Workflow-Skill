# Phase 2: Mandatory Architectural Constraints

## Purpose

Enforce four non-negotiable architectural constraints that define production readiness. These are **hard rules** — not guidelines, not configurable, not optional.

## The Four Constraints

### 2.1 Data Source Inversion

**Principle:** No hardcoded defaults, enums, or tuning constants in frontend code.

#### What Must Change

| Category | Before (Forbidden) | After (Required) |
|----------|-------------------|------------------|
| Defaults | `const DEFAULT_SORT = 'desc'` | `fetch('/settings/defaults').then(r => r.sort)` |
| Enums | `const STATUSES = ['pending','active']` | `fetch('/api/lookup/statuses')` |
| Filters | `const FILTERS = {category: [...]}` | `fetch('/api/lookup/filters')` |
| Pagination | `const PAGE_SIZE = 20` | `response.pageInfo.pageSize` or `/api/config` |
| Tuning | `const DEBOUNCE_MS = 300` | `/api/config` → `config.debounceMs` |

#### Permitted Exceptions (Bootstrap-Only)

```typescript
// ONLY these may be hardcoded — they must exist before ANY network call
const CONFIG_ENDPOINT_URL = '/api/config';      // The config endpoint itself
const INITIAL_FETCH_TIMEOUT_MS = 5000;          // Timeout for first fetch
const API_BASE_URL = '/api';                    // Base path (env-configurable)
```

#### Implementation Pattern

```typescript
// src/lib/config.ts — Single source of truth
export async function getConfig(): Promise<AppConfig> {
  const response = await fetch(CONFIG_ENDPOINT_URL, {
    signal: AbortSignal.timeout(INITIAL_FETCH_TIMEOUT_MS)
  });
  if (!response.ok) {
    throw new ConfigFetchError(response.status);
  }
  return response.json();
}

// src/hooks/useConfig.ts — React integration
export function useConfig() {
  return useQuery({
    queryKey: ['config'],
    queryFn: getConfig,
    staleTime: Infinity, // Config rarely changes
    retry: 3,
    retryDelay: 1000
  });
}

// Usage anywhere
function MyComponent() {
  const { data: config } = useConfig();
  const pageSize = config?.pageSize ?? 20; // TypeScript knows it exists
  // ...
}
```

### 2.2 Retire Dead UX & Placeholder Logic

**Principle:** Zero mock UI, zero no-op handlers, zero prop-drilled mock objects.

#### Forbidden Patterns (Auto-Detected)

```typescript
// ❌ Mock data arrays
const mockUsers = [{ id: 1, name: 'John' }, ...];
const chartData = [10, 20, 30, 40];

// ❌ Lorem ipsum
<div>{'Lorem ipsum dolor sit amet...'}</div>

// ❌ No-op handlers
<button onClick={() => {}}>Save</button>
<button onClick={() => console.log('TODO')}>Delete</button>

// ❌ Disabled/dead controls
<select disabled><option>Coming soon</option></select>
<input placeholder="Not implemented" readOnly />

// ❌ Static dropdowns/enums
<select><option>Option 1</option><option>Option 2</option></select>

// ❌ Hardcoded pagination
const PAGE_SIZE = 25;

// ❌ Hardcoded navigation
<nav><a href="/dashboard">Dashboard</a><a href="/settings">Settings</a></nav>
```

#### Required Replacements

```typescript
// ✅ Live data with proper states
function UserList() {
  const { data: users, isLoading, error, isEmpty } = useUsers();
  
  if (isLoading) return <UserListSkeleton />;
  if (error) return <ErrorDisplay error={error} onRetry={refetch} />;
  if (isEmpty) return <EmptyState message="No users found" action={<CreateUserButton />} />;
  
  return (
    <Table>
      {users.map(u => <UserRow key={u.id} user={u} />)}
    </Table>
  );
}

// ✅ Real mutation with cache invalidation
function DeleteButton({ userId }: { userId: string }) {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: () => api.deleteUser(userId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['user', userId] });
    }
  });
  
  return <Button onClick={() => mutation.mutate()} disabled={mutation.isPending}>
    {mutation.isPending ? 'Deleting...' : 'Delete'}
  </Button>;
}

// ✅ Dynamic enums from lookup
function StatusSelect({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  const { data: statuses } = useStatusLookup(); // Fetches /api/lookup/statuses
  
  return (
    <select value={value} onChange={e => onChange(e.target.value)}>
      {statuses?.map(s => <option key={s.id} value={s.id}>{s.label}</option>)}
    </select>
  );
}
```

### 2.3 End-to-End Data Integrity & Sub-Page Wiring

**Principle:** Every page, sub-page, modal, drawer fetches its own data independently.

#### Data Fetching Rules

| Pattern | Required | Forbidden |
|---------|----------|-----------|
| Main page | `useQuery(['dashboard'])` | Prop-drilling `dashboardData` |
| Sub-page | `useQuery(['report', reportId])` | Receiving `report` prop from parent |
| Modal | `useQuery(['user', userId])` | `user` prop from parent list |
| Drawer | `useQuery(['settings', section])` | `settings` from context |

#### Optimistic Updates

```typescript
// Pattern: Optimistic update with server rollback
function useToggleStatus(entityId: string, currentStatus: string) {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (newStatus: string) => api.updateStatus(entityId, newStatus),
    onMutate: async (newStatus) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['entity', entityId] });
      
      // Snapshot previous value
      const previous = queryClient.getQueryData(['entity', entityId]);
      
      // Optimistically update
      queryClient.setQueryData(['entity', entityId], (old) => ({
        ...old,
        status: newStatus
      }));
      
      return { previous };
    },
    onError: (err, newStatus, context) => {
      // Rollback to snapshot
      queryClient.setQueryData(['entity', entityId], context?.previous);
      toast.error('Failed to update status');
    },
    onSettled: () => {
      // Always refetch after mutation
      queryClient.invalidateQueries({ queryKey: ['entity', entityId] });
    }
  });
}
```

#### Contract Validation

```typescript
// src/schemas/api.ts — Every API response validated
import { z } from 'zod';

export const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(100),
  role: z.enum(['admin', 'editor', 'viewer']),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime()
});

export const UserListResponseSchema = z.object({
  data: z.array(UserSchema),
  pageInfo: z.object({
    page: z.number().int().positive(),
    pageSize: z.number().int().positive(),
    total: z.number().int().nonnegative(),
    hasNext: z.boolean()
  })
});

// In query hook
export function useUsers(page: number) {
  return useQuery({
    queryKey: ['users', page],
    queryFn: async () => {
      const response = await fetch(`/api/users?page=${page}`);
      const json = await response.json();
      return UserListResponseSchema.parse(json); // Throws on invalid
    }
  });
}
```

#### RBAC Fail-Closed

```typescript
// src/lib/auth/rbac.ts
type Role = 'admin' | 'editor' | 'viewer';
type Permission = 'users:read' | 'users:write' | 'settings:read' | 'settings:write';

const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  admin: ['users:read', 'users:write', 'settings:read', 'settings:write'],
  editor: ['users:read', 'users:write', 'settings:read'],
  viewer: ['users:read', 'settings:read']
};

export function hasPermission(role: Role | undefined, permission: Permission): boolean {
  if (!role) return false; // Fail-closed: unknown role = no permissions
  return ROLE_PERMISSIONS[role]?.includes(permission) ?? false;
}

// Route guard
export function RequirePermission({ permission, children }: { permission: Permission; children: React.ReactNode }) {
  const { data: session } = useSession();
  
  if (!hasPermission(session?.user?.role, permission)) {
    return <AccessDenied requiredPermission={permission} />;
  }
  
  return <>{children}</>;
}

// Usage
<RequirePermission permission="users:write">
  <CreateUserButton />
</RequirePermission>
```

### 2.4 Non-Functional Requirements

#### Unified Error Handling

```typescript
// src/lib/errors/unified.ts
export class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly status: number,
    message: string,
    public readonly remediation?: RemediationAction
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export type RemediationAction = 
  | { type: 'retry'; endpoint: string }
  | { type: 'goBack' }
  | { type: 'signIn' }
  | { type: 'contactSupport'; ticketId: string }
  | { type: 'refresh' };

// Error mapping
export function mapHttpError(status: number, body: unknown): AppError {
  switch (status) {
    case 400: return new AppError('VALIDATION_ERROR', 400, 'Invalid input', { type: 'retry', endpoint: '' });
    case 401: return new AppError('UNAUTHORIZED', 401, 'Session expired', { type: 'signIn' });
    case 403: return new AppError('FORBIDDEN', 403, 'Insufficient permissions', { type: 'goBack' });
    case 404: return new AppError('NOT_FOUND', 404, 'Resource not found', { type: 'goBack' });
    case 500: return new AppError('SERVER_ERROR', 500, 'Internal error', { type: 'contactSupport', ticketId: generateTicketId() });
    default: return new AppError('UNKNOWN_ERROR', status, 'Something went wrong', { type: 'retry', endpoint: '' });
  }
}

// Error boundary
export function ErrorDisplay({ error, onRetry }: { error: AppError; onRetry?: () => void }) {
  return (
    <Alert severity="error">
      <AlertTitle>{error.message}</AlertTitle>
      {error.remediation && (
        <RemediationActions action={error.remediation} onRetry={onRetry} />
      )}
    </Alert>
  );
}
```

#### Four Resource States (Mandatory)

```typescript
// Every async component MUST render all four states
interface ResourceState<T> {
  status: 'loading' | 'error' | 'empty' | 'ready';
  data?: T;
  error?: AppError;
}

function useResourceState<T>(query: UseQueryResult<T>) {
  if (query.isLoading) return { status: 'loading' as const };
  if (query.isError) return { status: 'error' as const, error: query.error as AppError };
  if (!query.data || (Array.isArray(query.data) && query.data.length === 0)) {
    return { status: 'empty' as const };
  }
  return { status: 'ready' as const, data: query.data };
}

// Component template
function ResourceComponent<T>({ 
  state, 
  renderLoading, 
  renderError, 
  renderEmpty, 
  renderReady 
}: {
  state: ResourceState<T>;
  renderLoading: () => React.ReactNode;
  renderError: (error: AppError) => React.ReactNode;
  renderEmpty: () => React.ReactNode;
  renderReady: (data: T) => React.ReactNode;
}) {
  switch (state.status) {
    case 'loading': return renderLoading();
    case 'error': return renderError(state.error!);
    case 'empty': return renderEmpty();
    case 'ready': return renderReady(state.data!);
  }
}
```

#### Centralized Server State

```typescript
// src/lib/query/client.ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 min
      gcTime: 10 * 60 * 1000,   // 10 min
      retry: 3,
      retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30000),
      refetchOnWindowFocus: false,
      refetchOnReconnect: true
    },
    mutations: {
      retry: 1,
      onError: (error) => {
        // Global error logging
        logger.error('Mutation failed', { error });
      }
    }
  }
});

// Derived data = computed, not stored
export function useUserStats(userId: string) {
  const { data: user } = useUser(userId);
  const { data: posts } = useUserPosts(userId);
  const { data: comments } = useUserComments(userId);
  
  // COMPUTED, not stored
  return useMemo(() => ({
    postCount: posts?.length ?? 0,
    commentCount: comments?.length ?? 0,
    engagementRate: posts?.length ? comments?.length / posts.length : 0
  }), [posts, comments]);
}
```

## Constraint Enforcement

### Automated Checks (Verification Script)

```bash
#!/bin/bash
# verification-script.sh

echo "🔍 Checking Phase 2 Constraints..."

# 1. No hardcoded enums/constants in components
echo "  Checking for hardcoded arrays..."
grep -r "const.*=\s*\[.*\]" src/components/ --include="*.tsx" | grep -v "test\|stories" && FAIL=1

# 2. No no-op handlers
echo "  Checking for no-op handlers..."
grep -r "onClick={() => {}}" src/ --include="*.tsx" && FAIL=1

# 3. No lorem ipsum
echo "  Checking for lorem ipsum..."
grep -ri "lorem ipsum" src/ && FAIL=1

# 4. Required endpoints exist
echo "  Checking required endpoints..."
curl -sf http://localhost:3000/api/config > /dev/null || FAIL=1
curl -sf http://localhost:3000/api/lookup/statuses > /dev/null || FAIL=1
curl -sf http://localhost:3000/settings/defaults > /dev/null || FAIL=1

# 5. Schema validation on all API hooks
echo "  Checking schema validation..."
grep -r "zod\|schema\.parse\|\.parse(" src/hooks/ --include="*.ts" | wc -l

# 6. Four states rendered
echo "  Checking four-state components..."
# Custom AST check for loading/error/empty/ready

[ "$FAIL" = "1" ] && exit 1 || exit 0
```

## Next Phase

→ [Phase 3: Surgical Implementation](Phase-3-Implementation.md)