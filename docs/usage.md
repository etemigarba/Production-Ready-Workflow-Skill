# Usage Guide

## Quick Start

### Basic Invocation

```bash
# In Claude Code terminal:
/production-ready-workflow
```

Or describe your goal naturally:

> "Make this app production-ready — remove all mock data, wire the UI to real APIs, and verify it works end-to-end."

### Typical Workflow

```
1. Navigate to your project root
2. Invoke the skill
3. Answer Phase 0 clarifying questions
4. Review and approve the Phase 1 plan
5. Watch surgical implementation (Phase 3)
6. Review self-review findings (Phase 4)
7. Run live verification (Phase 5)
8. Receive readiness report (Phase 6)
```

## Phase-by-Phase Usage

### Phase 0 — Understand Before You Plan

**What you'll do:**
- Confirm task understanding in 2–4 sentences
- Provide codebase access (`src/`, `project_documents/`)
- Answer clarifying questions about:
  - Missing API endpoints (`/api/config`, `/api/lookup/*`, `/settings/defaults`)
  - Auth/RBAC model
  - Environment (local DB vs cloud)
  - Test/validation expectations
  - In-scope routes/sub-pages

**Your input needed:**
- "Yes, `/api/config` exists" or "No, build it"
- "Roles: admin, editor, viewer"
- "Local PostgreSQL in Docker"
- "Use Playwright for E2E"
- "Scope: dashboard, settings, user management"

### Phase 1 — Plan & De-Risk

**What you'll see:**
- Major → minor → micro task lists (Frontend/Logic/Backend)
- Risk review: items ordered most→least risky with mitigations
- Completion-tracking checklist
- Subagent assignment (if parallel available)

**Your decision:**
- Approve plan → "Proceed"
- Request changes → "Modify Phase 1: [specific changes]"
- Defer risky items → "Feature-flag items 3, 7, 12"

### Phase 2 — Constraints (Non-Negotiable)

**Automatic enforcement — no input needed:**
- Data Source Inversion applied
- Dead UX retired
- E2E data integrity enforced
- NFRs implemented

### Phase 3 — Surgical Implementation

**What happens:**
- Scoped refactors with annotations per file
- Checklist updates in real-time
- Orphan cleanup (mock factories, unused fixtures)

**Monitor progress:**
- Watch task completion in checklist
- Review diffs as they're applied

### Phase 4 — Senior Self-Review

**What you'll receive:**
- Findings ordered critical → least critical
- Each finding: location, issue, fix
- Fixes applied in priority order

### Phase 5 — Verification & Live Simulation

**Automatic execution:**
- Verification script runs (greps for forbidden patterns)
- Backend + frontend started
- E2E simulation via browser automation or HTTP flows
- Real server responses captured and validated

**If failures occur:**
- Exact failure points identified
- Fixes applied
- Re-run until green

### Phase 6 — Readiness Report

**Deliverable:**
```
Production-Readiness: 94%
├── Wired Endpoints: 47/50
├── Constraints Satisfied: 23/24
├── Tests Passing: 156/158

Remaining Work:
├── src/components/Chart.tsx:15 — mock data array (fix: connect to /api/analytics)
├── src/hooks/useSettings.ts:8 — hardcoded default (fix: fetch /settings/defaults)

Checklist: ✓ 142/142
Risk Mitigations: ✓ 12/12
Verification: ✓ All green
```

## Advanced Usage

### Scoped Execution

Run only specific phases:

```bash
# Phase 0-2 only (analysis & planning)
/production-ready-workflow --phases 0-2

# Phase 3-4 only (implementation & review)
/production-ready-workflow --phases 3-4

# Phase 5 only (verification on existing changes)
/production-ready-workflow --phase 5
```

### Dry Run Mode

```bash
/production-ready-workflow --dry-run
# Shows what would be done without making changes
```

### Verbose Output

```bash
/production-ready-workflow --verbose
# Detailed logging for debugging
```

### Target Specific Paths

```bash
/production-ready-workflow --path ./src/features/dashboard
/production-ready-workflow --path ./apps/admin-panel
```

## Configuration Options

### CLI Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--phases <range>` | Phases to execute (e.g., `0-2`, `3`, `5-6`) | `0-6` |
| `--path <dir>` | Target codebase path | `.` |
| `--dry-run` | Show plan without executing | `false` |
| `--verbose` | Detailed logging | `false` |
| `--no-browser` | Disable browser automation | `false` |
| `--config <file>` | Custom config file | `.claude/skills/.../config.json` |

### Environment Variables

```bash
export PRODUCTION_READY_VERBOSE=1
export PRODUCTION_READY_NO_BROWSER=1
export PRODUCTION_READY_TIMEOUT_MS=180000
```

## Integration Patterns

### With CI/CD

```yaml
# .github/workflows/production-readiness.yml
name: Production Readiness Check
on: [pull_request]
jobs:
  readiness:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx claude-code /production-ready-workflow --phase 5 --no-browser
```

### With Pre-commit Hook

```bash
# .husky/pre-commit
npx claude-code /production-ready-workflow --phases 0-2 --dry-run
```

### As Library

```typescript
import { ProductionReadyWorkflow } from 'production-ready-workflow';

const workflow = new ProductionReadyWorkflow({
  projectRoot: process.cwd(),
  phases: [0, 1, 2, 3, 4, 5, 6],
  config: { strictMode: true }
});

const report = await workflow.execute();
console.log(report.readinessPercentage);
```

## Common Scenarios

### Scenario 1: New Feature Production-Ready

```bash
# After implementing a new feature branch
/production-ready-workflow --path ./src/features/new-feature
```

### Scenario 2: Legacy Codebase Migration

```bash
# Full codebase audit
/production-ready-workflow --verbose
# Expect Phase 0 to take longer — many clarifying questions
```

### Scenario 3: Pre-Release Gate

```bash
# In release pipeline
/production-ready-workflow --phase 5 --no-browser
# Fails CI if readiness < 90%
```

### Scenario 4: Refactor Verification

```bash
# After major refactor
/production-ready-workflow --phases 4-5
# Validates no regressions introduced
```

## Troubleshooting

### Phase 0 Stalls (Low Confidence)

**Symptom:** Repeated clarifying questions
**Fix:** Provide more context upfront:
- Share `project_documents/` if available
- Document known API endpoints
- List known mock data locations

### Phase 3 Breaks Existing Functionality

**Symptom:** Tests fail after refactor
**Fix:** The skill creates incremental commits. Rollback:
```bash
git revert HEAD~3..HEAD
# Or check specific commit
git show <commit-hash> --stat
```

### Phase 5 Simulation Fails

**Symptom:** Browser automation times out
**Fix:** 
- Increase timeout: `--config timeout:180000`
- Use HTTP flows instead: `--no-browser`
- Check backend is actually running on expected port

### Verification Script False Positives

**Symptom:** Flags legitimate code as mock
**Fix:** Update `.claude/skills/production-ready-workflow/config.json`:
```json
{
  "verification": {
    "ignorePatterns": [
      "test/fixtures/**",
      "**/*.stories.tsx",
      "**/*.test.ts"
    ]
  }
}
```

## Best Practices

1. **Run on feature branches** — Not main, to avoid disruption
2. **Commit before starting** — Clean baseline for diff comparison
3. **Answer Phase 0 thoroughly** — Reduces rework later
4. **Review Phase 1 plan** — Catch scope issues early
5. **Monitor Phase 3 diffs** — Ensure surgical, not sweeping changes
6. **Trust Phase 4 review** — It catches real issues
7. **Act on Phase 6 report** — Address remaining items before release

## Getting Help

- **Issues:** [GitHub Issues](https://github.com/etemi/production-ready-workflow/issues)
- **Discussions:** [GitHub Discussions](https://github.com/etemi/production-ready-workflow/discussions)
- **Skill docs:** `/production-ready-workflow --help`