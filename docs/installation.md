# Installation Guide

## Prerequisites

Before installing the `production-ready-workflow` skill, ensure you have:

### Required
- **Claude Code** (latest version) with ECC/opencode configured
- **Node.js 18+** — for running verification scripts and build tools
- **Git** — for version control and cloning
- **A target codebase** — Full-stack web application (React/Vue/Svelte + Node.js/Python/Go backend)

### Recommended
- **Agent-browser skill** — Enables Phase 5 browser automation for live E2E simulation
- **Playwright** — If using scripted HTTP flows instead of browser automation
- **TypeScript 5+** — For schema validation (Zod/io-ts) integration
- **React Query / SWR / RTK Query** — For centralized server state management

### Optional
- **Storybook** — If your project uses it for component documentation
- **ESLint + Prettier** — For code quality enforcement
- **Knip / ts-prune** — For orphaned module detection in Phase 3

## Installation Methods

### Method 1: ECC/opencode Native (Recommended)

```bash
# 1. Clone the repository
git clone https://github.com/etemi/production-ready-workflow.git

# 2. Copy to your ECC skills directory
mkdir -p ~/.opencode/skills/production-ready-workflow
cp production-ready-workflow/SKILL.md production-ready-workflow/production-ready-workflow.skill ~/.opencode/skills/production-ready-workflow/

# 3. Register in opencode.json
cat >> ~/.opencode/opencode.json << 'EOF'
{
  "skills": {
    "production-ready-workflow": {
      "path": "~/.opencode/skills/production-ready-workflow"
    }
  }
}
EOF
```

### Method 2: Direct Skill Installation (Claude Code)

```bash
# In your project root, create .claude/skills directory
mkdir -p .claude/skills/production-ready-workflow

# Copy skill files
cp SKILL.md production-ready-workflow.skill .claude/skills/production-ready-workflow/

# The skill will auto-load on next Claude Code session
```

### Method 3: Git Submodule (For Teams)

```bash
# Add as submodule to your project
git submodule add https://github.com/etemi/production-ready-workflow.git .claude/skills/production-ready-workflow
git submodule update --init --recursive
```

## Verification

After installation, verify the skill loads correctly:

```bash
# In Claude Code, run:
/production-ready-workflow --help

# Or check skill listing
/skills list | grep production-ready-workflow
```

Expected output:
```
production-ready-workflow  ✓  End-to-end workflow for auditing and refactoring full-stack web apps into production-ready, zero-mock-data systems
```

## Configuration

### Environment Variables

Create `.env.local` in your project root:

```bash
# Required for Phase 0 bootstrap
CONFIG_ENDPOINT_URL=http://localhost:3000/api/config
INITIAL_FETCH_TIMEOUT_MS=5000

# Optional: API base URL
API_BASE_URL=http://localhost:3000

# Optional: Database (local dev)
DATABASE_URL=postgresql://user:pass@localhost:5432/myapp_dev
```

### Skill Configuration (Optional)

Create `.claude/skills/production-ready-workflow/config.json`:

```json
{
  "phase0": {
    "confidenceThreshold": 0.96,
    "maxClarifyingQuestions": 10
  },
  "phase1": {
    "enableParallelSubagents": true,
    "maxParallelTasks": 3
  },
  "phase2": {
    "strictMode": true,
    "allowedHardcodedConstants": [
      "CONFIG_ENDPOINT_URL",
      "INITIAL_FETCH_TIMEOUT_MS"
    ]
  },
  "phase5": {
    "useBrowserAutomation": true,
    "simulationTimeoutMs": 120000
  }
}
```

## Troubleshooting

### Skill Not Loading
- Verify `SKILL.md` and `production-ready-workflow.skill` are in the same directory
- Check `~/.opencode/opencode.json` has correct path
- Restart Claude Code session

### Permission Errors
```bash
chmod +x ~/.opencode/skills/production-ready-workflow/production-ready-workflow.skill
```

### Missing Dependencies
```bash
# Install recommended tooling
npm install -g knip ts-prune playwright @playwright/test
```

## Updating

```bash
cd ~/.opencode/skills/production-ready-workflow
git pull origin main
# Restart Claude Code to reload
```

## Uninstalling

```bash
# Remove from opencode.json
# Delete skill directory
rm -rf ~/.opencode/skills/production-ready-workflow
```