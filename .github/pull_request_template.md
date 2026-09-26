---
name: Pull Request Template
about: Template for PRs to production-ready-workflow
title: '[TYPE] '
labels: ''
assignees: ''
---

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Phase logic modification
- [ ] Verification rule addition
- [ ] Configuration enhancement
- [ ] Refactoring (no behavior change)
- [ ] Dependency update

## Description
Brief description of changes.

## Phase Impact
Which phase(s) does this affect?
- [ ] Phase 0: Understanding/Confidence Gate
- [ ] Phase 1: Planning/Risk/Checklist
- [ ] Phase 2: Constraints (Data Source Inversion, Dead UX, E2E Integrity, NFRs)
- [ ] Phase 3: Surgical Implementation
- [ ] Phase 4: Senior Self-Review
- [ ] Phase 5: Verification/Simulation
- [ ] Phase 6: Readiness Report
- [ ] Cross-cutting (CLI, config, installation)

## Changes Made
- File: `SKILL.md` — [what changed]
- File: `production-ready-workflow.skill` — [regenerated from SKILL.md]
- File: `docs/[file].md` — [what changed]
- File: `wiki/[file].md` — [what changed]
- File: `examples/sample-workflow.md` — [what changed]
- File: `.github/[file]` — [what changed]

## Testing Performed
- [ ] Skill loads in ECC/opencode
- [ ] Tested on sample project (describe stack)
- [ ] Phase 0-6 execution verified
- [ ] Verification script passes
- [ ] No regression in existing behavior
- [ ] Documentation updated and accurate

## Breaking Changes
- [ ] No breaking changes
- [ ] Breaking changes (describe):
  - Migration path for users:

## Checklist
- [ ] `SKILL.md` front-matter valid
- [ ] `production-ready-workflow.skill` compiled from SKILL.md
- [ ] All docs/ files updated
- [ ] All wiki/ files updated
- [ ] Example workflow reflects changes
- [ ] CHANGELOG.md updated (if applicable)
- [ ] Version bumped in package.json (if applicable)
- [ ] No hardcoded values in skill itself (dogfood check)
- [ ] License header present in new files

## Screenshots/Logs
If applicable, add screenshots of skill execution or relevant logs.

## Additional Notes
Any other information for reviewers.