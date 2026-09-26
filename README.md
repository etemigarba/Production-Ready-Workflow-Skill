# production-ready-workflow

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue.svg)](https://claude.ai/code)
[![ECC Compatible](https://img.shields.io/badge/ECC-Compatible-green.svg)](https://github.com/etemi/opencode)
[![Year](https://img.shields.io/badge/Year-2026-orange.svg)]()
[![Version](https://img.shields.io/badge/Version-1.0.0-purple.svg)]()

> **A 6-phase agentic skill that transforms full-stack web applications into zero-mock-data, production-grade systems.** Eradicates hardcoded values, wires live API bindings, enforces SOLID architecture, and validates via live end-to-end simulation.

## Overview

The `production-ready-workflow` skill provides a systematic, battle-tested methodology for auditing and refactoring any full-stack web application into a production-ready state. Designed by a Principal/Staff Software Architect with 15+ years of experience shipping high-traffic, enterprise-grade web applications.

### Key Capabilities

- **Zero Mock Data** — Eliminates every hardcoded array, lorem-ipsum placeholder, static dropdown, and magic number
- **Live API Wiring** — Replaces all mock bindings with real server-driven endpoints (`/api/config`, `/api/lookup/*`, `/settings/defaults`)
- **SOLID Architecture** — Enforces Single Responsibility, layered modularity, and reusable component patterns
- **Contract Validation** — Zod/schema validation on every API response before UI consumption
- **RBAC Fail-Closed** — Protected routes with explicit role allowlists; unknown roles denied by default
- **Four Resource States** — Explicit `loading`, `error`, `empty`, `ready` rendering for every async operation
- **Senior Self-Review** — Hostile code review phase catches race conditions, inefficiencies, and bug-prone patterns
- **Live Verification** — Real end-to-end simulation via browser automation or scripted HTTP flows

## Installation

### Prerequisites

- **Claude Code** (latest version) with ECC/opencode configured
- **Node.js 18+** — for running verification scripts and build tools
- **Git** — for version control and cloning
- **A target codebase** — Full-stack web application (React/Vue/Svelte + Node.js/Python/Go backend)

### Recommended

- **Agent-browser skill** — Enables Phase 5 browser automation for live E2E simulation
- **Playwright** — If using scripted HTTP flows instead of browser automation
- **TypeScript 5+** — For schema validation (Zod/io-ts) integration
- **React Query / SWR / RTK Query** — For centralized server state management

### Quick Install

```bash
# Clone this repository
git clone https://github.com/etemi/production-ready-workflow.git

# Or copy the skill files to your ECC skills directory
cp SKILL.md production-ready-workflow.skill ~/.opencode/skills/production-ready-workflow/
```

### ECC/opencode Registration

Add to your `.opencode/opencode.json`:

```json
{
  "skills": {
    "production-ready-workflow": {
      "path": "~/.opencode/skills/production-ready-workflow"
    }
  }
}
```

## Usage

### Invocation

```bash
# In Claude Code terminal:
/production-ready-workflow
```

Or invoke directly in Claude Code:

> "Make the app production-ready"  
> "Remove mock data and wire the UI to the live API"  
> "Audit production readiness of this codebase"  
> "Retire dead UX and hardcoded fallbacks"

### What Happens

The skill executes a **6-phase workflow**:

| Phase | Description | Output |
|-------|-------------|--------|
| **Phase 0** | Understand Before You Plan (≥96% confidence gate) | Task understanding confirmation, codebase inventory, assumptions |
| **Phase 1** | Plan & De-Risk | Major/minor/micro task lists, risk review with mitigations, completion checklist |
| **Phase 2** | Mandatory Architectural Constraints | Data Source Inversion, Dead UX Retirement, E2E Data Integrity, NFRs |
| **Phase 3** | Surgical Implementation | Scoped refactors with annotations, orphan cleanup |
| **Phase 4** | Senior-Engineer Self-Review | Critical→least critical findings, ordered fixes |
| **Phase 5** | Verification & Live Simulation | Verification script, real stack execution, E2E simulation, failure capture |
| **Phase 6** | Readiness Report | Production-readiness %, remaining work, checklist results |

## Architecture

### Three Pillars

```
┌─────────────────────────────────────────────────────────────┐
│                  production-ready-workflow                   │
├──────────────────────────┬────────────────────────┬──────────┤
│       Frontend           │        Logic           │  Backend │
├──────────────────────────┼────────────────────────┼──────────┤
│ • Design System          │ • Data Structures      │ • Secure  │
│ • Components             │ • Algorithms           │   API     │
│ • State Mgmt             │ • O(1)/O(n) prefs      │ • Data    │
│ • Responsive             │ • Cache Mgmt           │   Access  │
│ • CSR + Lazy             │ • Lifecycle Map        │ • Request │
└──────────────────────────┴────────────────────────┴──────────┘
```

### Non-Negotiable Constraints (Phase 2)

1. **Data Source Inversion** — All defaults, enums, pagination from server
2. **Dead UX Retirement** — No mock UI, no no-op buttons, no prop-drilled mock objects
3. **E2E Data Integrity** — Sub-pages fetch independently, optimistic updates, RBAC fail-closed
4. **NFR Compliance** — Unified error handling, skeletons, 4-state rendering, centralized server state

## Documentation

- [Installation Guide](docs/installation.md)
- [Usage Guide](docs/usage.md)
- [Architecture Overview](docs/architecture.md)
- [Contributing Guidelines](docs/contributing.md)
- [Wiki](wiki/Home.md) — Phase-by-phase deep dive

## Examples

See [examples/sample-workflow.md](examples/sample-workflow.md) for a complete invocation example.

## License

MIT License — Copyright (c) 2026 Prof. Etemi Joshua Garba

Free for adoption, editing, and refactoring. No explicit permission required.

## Author

**Prof. Etemi Joshua Garba**  
Ethereal Multimedia Technology — R&D Software Development  
[GitHub](https://github.com/etemi) • [LinkedIn](https://linkedin.com/in/etemijoshua)

## Related Skills

- [debug-and-fix-bugs](../debug-and-fix-bugs) — Surgical bug hunting companion
- [frontend-ui-ux-designer](../frontend-ui-ux-designer) — UI/UX design workflow
- [systematic-implementation](../systematic-implementation) — Full SDLC orchestration
- [loop-engineer](../loop-engineer) — Autonomous agent loop framework

---

*Part of the [Agentic Engineering Skills](https://github.com/etemigarba/Agentic-Engineering-Skills) collection.*