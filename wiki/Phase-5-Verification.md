# Phase 5: Verification & Live Simulation

## Purpose

Empirically validate the implementation through automated checks, live stack execution, and end-to-end simulation. **Do not theorize — execute.**

## Verification Pillars

### 1. Verification Script (Static Analysis)

Automated detection of forbidden patterns and validation of required structures.

```bash
#!/bin/bash
# verification-script.sh
# Run at start of Phase 5

set -e

echo "🔍 Phase 5 Verification Script"
echo "================================"

FAIL=0

# --- Forbidden Patterns ---

echo "1. Checking for hardcoded mock data..."
# Hardcoded arrays in components
if grep -rn "const.*=\s*\[.*\]" src/components/ src/pages/ --include="*.tsx" | \
   grep -vE "test|stories|\.d\.ts" | \
   grep -v "useQuery\|useMutation\|useState\|useRef"; then
  echo "   ❌ FAIL: Hardcoded arrays found in components"
  FAIL=1
else
  echo "   ✓ PASS: No hardcoded arrays in components"
fi

# Hardcoded objects
if grep -rn "const.*=\s*{.*}" src/components/ src/pages/ --include="*.tsx" | \
   grep -vE "test|stories|\.d\.ts|useState\|useRef\|useMemo\|useCallback" | \
   grep -v "theme\|config\|styles"; then
  echo "   ❌ FAIL: Hardcoded objects found in components"
  FAIL=1
else
  echo "   ✓ PASS: No hardcoded objects in components"
fi

# No-op handlers
if grep -rn "onClick={() => {}}" src/ --include="*.tsx" | grep -vE "test|stories"; then
  echo "   ❌ FAIL: No-op onClick handlers found"
  FAIL=1
else
  echo "   ✓ PASS: No no-op handlers"
fi

# Lorem ipsum
if grep -ri "lorem ipsum" src/ --include="*.tsx" | grep -vE "test|stories"; then
  echo "   ❌ FAIL: Lorem ipsum placeholder text found"
  FAIL=1
else
  echo "   ✓ PASS: No lorem ipsum"
fi

# Inline magic numbers (excluding config bootstrap)
if grep -rnE "\b[0-9]{2,}\b" src/components/ src/hooks/ --include="*.tsx" | \
   grep -vE "test|stories|line|column|port|timeout|size|limit|page" | \
   grep -v "INITIAL_FETCH_TIMEOUT_MS\|CONFIG_ENDPOINT_URL"; then
  echo "   ⚠️  WARN: Potential magic numbers (review manually)"
fi

# --- Required Structures ---

echo "2. Checking required endpoints..."
ENDPOINTS=(
  "http://localhost:3000/api/config"
  "http://localhost:3000/api/lookup/statuses"
  "http://localhost:3000/api/lookup/categories"
  "http://localhost:3000/api/lookup/priorities"
  "http://localhost:3000/settings/defaults"
)

for endpoint in "${ENDPOINTS[@]}"; do
  if curl -sf "$endpoint" > /dev/null; then
    echo "   ✓ $endpoint"
  else
    echo "   ❌ FAIL: $endpoint not responding"
    FAIL=1
  fi
done

# --- Schema Validation ---

echo "3. Checking schema validation on API hooks..."
HOOK_FILES=$(find src/hooks -name "*.ts" -not -name "*.test.ts" -not -name "*.d.ts")
VALIDATION_COUNT=0

for file in $HOOK_FILES; do
  if grep -q "zod\|schema\.parse\|\.parse(" "$file"; then
    VALIDATION_COUNT=$((VALIDATION_COUNT + 1))
  else
    echo "   ⚠️  WARN: No schema validation in $file"
  fi
done

echo "   Hooks with validation: $VALIDATION_COUNT / $(echo $HOOK_FILES | wc -w)"

# --- Four States Check ---

echo "4. Checking four-state rendering..."
# This requires AST analysis — simplified grep check
FOUR_STATE_COMPONENTS=$(grep -rl "isLoading\|isError\|isEmpty" src/components/ --include="*.tsx" | wc -l)
TOTAL_COMPONENTS=$(find src/components -name "*.tsx" -not -name "*.test.tsx" -not -name "*.stories.tsx" | wc -l)

echo "   Components with state checks: $FOUR_STATE_COMPONENTS / $TOTAL_COMPONENTS"

if [ "$FOUR_STATE_COMPONENTS" -lt "$((TOTAL_COMPONENTS * 80 / 100))" ]; then
  echo "   ⚠️  WARN: Less than 80% components handle loading/error/empty"
fi

# --- Test Suite ---

echo "4. Running test suite..."
if npm test -- --passWithNoTests 2>&1 | tail -10; then
  echo "   ✓ PASS: All tests pass"
else
  echo "   ❌ FAIL: Tests failing"
  FAIL=1
fi

# --- Type Check ---

echo "5. Type checking..."
if npm run typecheck 2>&1 | grep -q "error"; then
  echo "   ❌ FAIL: TypeScript errors"
  FAIL=1
else
  echo "   ✓ PASS: No TypeScript errors"
fi

# --- Lint ---

echo "6. Linting..."
if npm run lint 2>&1 | grep -q "error"; then
  echo "   ❌ FAIL: Lint errors"
  FAIL=1
else
  echo "   ✓ PASS: No lint errors"
fi

# --- Summary ---

echo ""
echo "================================"
if [ $FAIL -eq 0 ]; then
  echo "✅ VERIFICATION SCRIPT: PASSED"
  exit 0
else
  echo "❌ VERIFICATION SCRIPT: FAILED"
  exit 1
fi
```

### 2. Stack Runner (Live Execution)

Start the real application stack:

```bash
#!/bin/bash
# start-stack.sh

echo "🚀 Starting application stack..."

# 1. Database (Docker)
echo "  Starting PostgreSQL..."
docker compose up -d postgres
sleep 5 # Wait for ready

# 2. Run migrations
echo "  Running migrations..."
npm run db:migrate

# 3. Seed development data
echo "  Seeding data..."
npm run db:seed

# 4. Backend
echo "  Starting backend on port 3000..."
npm run dev:backend &
BACKEND_PID=$!

# 5. Frontend
echo "  Starting frontend on port 5173..."
npm run dev:frontend &
FRONTEND_PID=$!

# 6. Health checks
echo "  Waiting for services..."
sleep 10

for i in {1..30}; do
  if curl -sf http://localhost:3000/health > /dev/null && \
     curl -sf http://localhost:5173 > /dev/null; then
    echo "  ✅ Stack healthy"
    break
  fi
  sleep 2
done

# Save PIDs for cleanup
echo $BACKEND_PID > .backend.pid
echo $FRONTEND_PID > .frontend.pid

echo "Stack running. Backend PID: $BACKEND_PID, Frontend PID: $FRONTEND_PID"
```

### 3. Simulation Engine

#### Option A: Browser Automation (agent-browser skill)

```typescript
// simulation-browser.ts
import { Browser } from 'agent-browser';

const browser = new Browser();
const page = await browser.newPage();

const results = [];

async function simulate(userFlow: UserFlow) {
  console.log(`Simulating: ${userFlow.name}`);
  
  try {
    // Navigate
    await page.goto(userFlow.url);
    await page.waitForLoadState('networkidle');
    
    // Execute steps
    for (const step of userFlow.steps) {
      await executeStep(page, step);
      
      // Capture response
      const response = await captureLastResponse(page);
      results.push({
        flow: userFlow.name,
        step: step.name,
        url: response.url(),
        status: response.status(),
        body: await response.json().catch(() => null),
        timestamp: new Date().toISOString()
      });
    }
    
    console.log(`  ✅ ${userFlow.name} completed`);
  } catch (error) {
    console.log(`  ❌ ${userFlow.name} failed: ${error.message}`);
    results.push({
      flow: userFlow.name,
      error: error.message,
      timestamp: new Date().toISOString()
    });
  }
}

// User flows to simulate
const flows: UserFlow[] = [
  {
    name: "User Login",
    url: "http://localhost:5173/login",
    steps: [
      { action: "fill", selector: "[name=email]", value: "test@example.com" },
      { action: "fill", selector: "[name=password]", value: "password123" },
      { action: "click", selector: "button[type=submit]" },
      { action: "wait", condition: "url", value: "/dashboard" }
    ]
  },
  {
    name: "Dashboard Load",
    url: "http://localhost:5173/dashboard",
    steps: [
      { action: "wait", condition: "selector", value: "[data-testid=charts]" },
      { action: "wait", condition: "selector", value: "[data-testid=stats]" }
    ]
  },
  {
    name: "Create User",
    url: "http://localhost:5173/users/new",
    steps: [
      { action: "fill", selector: "[name=email]", value: "new@example.com" },
      { action: "fill", selector: "[name=name]", value: "New User" },
      { action: "select", selector: "[name=role]", value: "editor" },
      { action: "click", selector: "button[type=submit]" },
      { action: "wait", condition: "url", value: "/users" }
    ]
  },
  // ... more flows
];

for (const flow of flows) {
  await simulate(flow);
}

// Validate responses against schemas
for (const result of results) {
  if (result.body) {
    validateAgainstSchema(result);
  }
}

await browser.close();
```

#### Option B: HTTP Flows (Playwright/curl)

```bash
#!/bin/bash
# simulation-http.sh

echo "🌐 Running HTTP simulation flows..."

# Login
echo "  Flow: User Login"
TOKEN=$(curl -s -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}' | \
  jq -r '.token')

if [ "$TOKEN" = "null" ] || [ -z "$TOKEN" ]; then
  echo "  ❌ Login failed"
  exit 1
fi
echo "  ✓ Login successful"

# Dashboard data
echo "  Flow: Dashboard Load"
DASHBOARD=$(curl -s -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/dashboard)
echo "$DASHBOARD" | jq -e '.charts and .stats and .activity' > /dev/null || exit 1
echo "  ✓ Dashboard data valid"

# Create user
echo "  Flow: Create User"
CREATE_RESPONSE=$(curl -s -X POST http://localhost:3000/api/users \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email":"new@example.com","name":"New User","role":"editor"}')
USER_ID=$(echo "$CREATE_RESPONSE" | jq -r '.id')
echo "  ✓ User created: $USER_ID"

# Verify user appears in list
echo "  Flow: Verify User List"
USERS=$(curl -s -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/users)
echo "$USERS" | jq -e ".data[] | select(.id==\"$USER_ID\")" > /dev/null || exit 1
echo "  ✓ User appears in list"

# Update user
echo "  Flow: Update User"
curl -s -X PATCH "http://localhost:3000/api/users/$USER_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated Name"}' | jq -e '.name=="Updated Name"' > /dev/null || exit 1
echo "  ✓ User updated"

# Delete user
echo "  Flow: Delete User"
curl -s -X DELETE "http://localhost:3000/api/users/$USER_ID" \
  -H "Authorization: Bearer $TOKEN" | jq -e '.success==true' > /dev/null || exit 1
echo "  ✓ User deleted"

# Verify deletion
USERS=$(curl -s -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/users)
echo "$USERS" | jq -e ".data[] | select(.id==\"$USER_ID\")" > /dev/null && exit 1 || true
echo "  ✓ User no longer in list"

echo ""
echo "✅ ALL HTTP FLOWS PASSED"
```

### 4. Response Validation

```typescript
// validate-responses.ts
import { z } from 'zod';
import { readFileSync } from 'fs';

const results = JSON.parse(readFileSync('simulation-results.json', 'utf-8'));

const schemas = {
  '/api/config': ConfigSchema,
  '/api/lookup/statuses': LookupArraySchema(StatusSchema),
  '/api/lookup/categories': LookupArraySchema(CategorySchema),
  '/api/dashboard': DashboardResponseSchema,
  '/api/users': UserListResponseSchema,
  '/api/users/:id': UserSchema
};

let pass = 0, fail = 0;

for (const result of results) {
  if (!result.body) continue;
  
  const schema = schemas[result.url] || schemas[normalizeUrl(result.url)];
  
  if (schema) {
    const parseResult = schema.safeParse(result.body);
    if (parseResult.success) {
      console.log(`✅ ${result.url} — valid`);
      pass++;
    } else {
      console.log(`❌ ${result.url} — INVALID`);
      console.log(`   Errors: ${JSON.stringify(parseResult.error.errors, null, 2)}`);
      fail++;
    }
  } else {
    console.log(`⚠️  ${result.url} — no schema defined`);
  }
}

console.log(`\nSchema Validation: ${pass} passed, ${fail} failed`);
process.exit(fail > 0 ? 1 : 0);

function normalizeUrl(url: string): string {
  return url.replace(/\/users\/[^/]+$/, '/users/:id');
}
```

## Failure Handling

### When Simulation Fails

1. **Capture exact failure** — Response body, status, stack trace
2. **Locate in code** — Map to specific file/function
3. **Fix surgically** — Minimal change to address root cause
4. **Re-run verification** — Full Phase 5 again

### Common Failure Patterns

| Failure | Typical Cause | Fix |
|---------|---------------|-----|
| 404 on lookup endpoint | Route not registered | Add route in backend router |
| 500 on config endpoint | DB migration missing | Run migrations, check schema |
| Schema validation error | Response shape mismatch | Fix backend serializer or frontend schema |
| Loading state never resolves | Query key mismatch | Align queryKey between hook and invalidation |
| RBAC 403 on valid role | Permission string mismatch | Sync ROLE_PERMISSIONS with backend |

## Cleanup

```bash
#!/bin/bash
# cleanup.sh

echo "🧹 Cleaning up..."

# Stop frontend
if [ -f .frontend.pid ]; then
  kill $(cat .frontend.pid) 2>/dev/null || true
  rm .frontend.pid
fi

# Stop backend
if [ -f .backend.pid ]; then
  kill $(cat .backend.pid) 2>/dev/null || true
  rm .backend.pid
fi

# Stop database
docker compose down

echo "✅ Cleanup complete"
```

## Next Phase

→ [Phase 6: Readiness Report](Phase-6-Report.md)