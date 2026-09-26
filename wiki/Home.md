# Production-Ready Workflow Wiki

Welcome to the comprehensive wiki for the **production-ready-workflow** skill. This wiki provides deep-dive documentation for each phase, advanced usage patterns, and architectural decisions.

## Table of Contents

### Phase Guides
- [Phase 0: Understand Before You Plan](Phase-0-Understand.md) — Confidence gate, codebase inventory, clarification protocol
- [Phase 1: Plan & De-Risk](Phase-1-Plan.md) — Task decomposition, risk analysis, checklist generation, subagent scheduling
- [Phase 2: Mandatory Constraints](Phase-2-Constraints.md) — Data Source Inversion, Dead UX Retirement, E2E Integrity, NFRs
- [Phase 3: Surgical Implementation](Phase-3-Implementation.md) — Scoped refactors, annotation logging, orphan cleanup
- [Phase 4: Senior Self-Review](Phase-4-Review.md) — Hostile review methodology, finding classification, fix prioritization
- [Phase 5: Verification & Simulation](Phase-5-Verification.md) — Verification scripts, stack execution, browser/HTTP simulation, response validation
- [Phase 6: Readiness Report](Phase-6-Report.md) — Scoring methodology, report structure, actionable outputs

### Advanced Topics
- [Architecture Deep Dive](../docs/architecture.md) — Three-pillar architecture, data flows, extension points
- [Configuration Reference](Configuration.md) — All config options, environment variables, CLI flags
- [Integration Patterns](Integration.md) — CI/CD, pre-commit hooks, library usage, custom rules
- [Troubleshooting](Troubleshooting.md) — Common issues, diagnostics, workarounds
- [Extending the Skill](Extending.md) — Custom verification rules, constraint exceptions, phase hooks

### Reference
- [Skill Definition](../SKILL.md) — Authoritative skill specification
- [Binary Skill File](../production-ready-workflow.skill) — ECC-compiled skill
- [Changelog](Changelog.md) — Version history
- [FAQ](FAQ.md) — Frequently asked questions

## Quick Links

| Need | Go To |
|------|-------|
| Install the skill | [Installation Guide](../docs/installation.md) |
| Run for the first time | [Usage Guide](../docs/usage.md#quick-start) |
| Understand the phases | [Phase 0](Phase-0-Understand.md) → [Phase 6](Phase-6-Report.md) |
| Configure for your project | [Configuration](Configuration.md) |
| Integrate with CI/CD | [Integration Patterns](Integration.md#cicd-integration) |
| Report a bug | [GitHub Issues](https://github.com/etemi/production-ready-workflow/issues) |
| Request a feature | [GitHub Discussions](https://github.com/etemi/production-ready-workflow/discussions) |

## Skill Philosophy

> **"Zero mock data, zero dead UX, zero silent fallbacks."**

This skill embodies the principle that production readiness is not a checklist — it's an architectural property. Every phase builds toward a system where:
- All data originates from live APIs
- Every UI element has real behavior
- Errors are explicit and actionable
- The system can be verified empirically, not theoretically

## Version

Current: **v1.0.0** (2026)

See [Changelog](Changelog.md) for version history.

## License

MIT License — Copyright (c) 2026 Prof. Etemi Joshua Garba

Free for adoption, editing, and refactoring.