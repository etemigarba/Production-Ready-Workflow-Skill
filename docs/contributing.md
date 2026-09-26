# Contributing Guidelines

Thank you for contributing to `production-ready-workflow`! This skill is part of the Agentic Engineering Skills collection and follows a structured contribution process.

## Code of Conduct

This project adheres to the [Contributor Covenant](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). By participating, you agree to uphold this code.

## How to Contribute

### 1. Report Issues

- **Bug Reports** — Use the [Bug Report Template](.github/ISSUE_TEMPLATE/bug_report.md)
- **Feature Requests** — Use the [Feature Request Template](.github/ISSUE_TEMPLATE/feature_request.md)
- **Questions** — Use [GitHub Discussions](https://github.com/etemi/production-ready-workflow/discussions)

### 2. Submit Changes

#### Fork & Clone

```bash
git clone https://github.com/YOUR-USERNAME/production-ready-workflow.git
cd production-ready-workflow
```

#### Create Branch

```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/your-bug-fix
# or
git checkout -b docs/your-doc-update
```

#### Make Changes

Follow the skill's own principles:
- **No mock data** — Test with real fixtures
- **SOLID** — Single responsibility per file
- **Document decisions** — Update relevant docs
- **Verify locally** — Run the skill on a test codebase

#### Test Your Changes

```bash
# 1. Install in local ECC
mkdir -p ~/.opencode/skills/production-ready-workflow
cp SKILL.md production-ready-workflow.skill ~/.opencode/skills/production-ready-workflow/

# 2. Test on sample project
cd /tmp && mkdir test-project && cd test-project
# Create minimal React + Node project
npx create-react-app . --template typescript
# Add Express backend

# 3. Run skill
claude-code /production-ready-workflow --dry-run
# Verify phases execute without error
```

#### Commit & Push

```bash
git add .
git commit -m "feat: add custom verification rule for X

- Adds X rule to verification script
- Updates docs/architecture.md with extension point
- Tests pass on sample project"

git push origin feat/your-feature-name
```

#### Open Pull Request

Use the [PR Template](.github/pull_request_template.md) and ensure:
- [ ] All CI checks pass
- [ ] Documentation updated
- [ ] No breaking changes without major version bump
- [ ] Changelog entry added (if applicable)

## Development Setup

### Local Development

```bash
# Clone your fork
git clone https://github.com/YOUR-USERNAME/production-ready-workflow.git
cd production-ready-workflow

# No build step required — skill files are interpreted directly
# But you can validate markdown:
npm install -g markdownlint-cli
markdownlint **/*.md
```

### Testing Framework

The skill is tested via **dogfooding** — running it on real codebases:

```bash
# Test matrix
# 1. React + TypeScript + Express + PostgreSQL
# 2. Vue 3 + TypeScript + Fastify + MySQL
# 3. SvelteKit + TypeScript + Hono + SQLite
# 4. Next.js 14 + App Router + Prisma + PostgreSQL
```

### Validation Checklist

Before submitting PR, verify:

- [ ] `SKILL.md` follows front-matter format
- [ ] `production-ready-workflow.skill` binary is valid (if modified)
- [ ] All docs/ files updated for changes
- [ ] Wiki pages updated for phase changes
- [ ] Examples reflect new usage
- [ ] No hardcoded values in skill itself (dogfood check)

## Skill Development Guidelines

### Modifying the Skill

The skill has two files:

1. **SKILL.md** — Human-readable definition (Markdown with front-matter)
2. **production-ready-workflow.skill** — Binary skill file (ECC format)

**Always update both** when changing skill behavior.

### SKILL.md Structure

```markdown
---
name: production-ready-workflow
description: >
  One-line summary for skill registry
---

# Title

## Phase N — Name

### N.1 Subsection

- Rule 1
- Rule 2
```

### Binary Skill File

The `.skill` file is an ECC-compiled format. To regenerate:

```bash
# Requires ECC CLI
ecc compile SKILL.md -o production-ready-workflow.skill
```

**Do not edit binary directly** — always compile from SKILL.md.

## Documentation Standards

### Markdown Style

- Use ATX headings (`#`, `##`, `###`)
- Code blocks with language hints
- Relative links for internal references
- Tables for structured data
- Mermaid diagrams for architecture

### Document Types

| File | Purpose | Update When |
|------|---------|-------------|
| README.md | Project overview | Any user-facing change |
| docs/installation.md | Setup instructions | Prerequisites, install methods |
| docs/usage.md | How to use | CLI flags, workflows, scenarios |
| docs/architecture.md | Technical design | Phase changes, data flows |
| docs/contributing.md | Contributor guide | Process changes |
| wiki/*.md | Phase deep-dives | Phase logic changes |

## Versioning

This project uses **Semantic Versioning** (SemVer):

- **MAJOR** — Breaking changes to skill interface or phase behavior
- **MINOR** — New phases, constraints, or backward-compatible features
- **PATCH** — Bug fixes, documentation, internal refactors

Version tracked in:
- `package.json` (for npm)
- Git tags (`v1.0.0`, `v1.1.0`, etc.)
- CHANGELOG.md

## Release Process

1. Update CHANGELOG.md
2. Bump version in package.json
3. Create git tag: `git tag v1.2.0`
4. Push tag: `git push origin v1.2.0`
5. GitHub Action publishes to npm (if configured)
6. Update ECC skill registry

## Review Criteria

PRs are evaluated on:

| Criterion | Weight |
|-----------|--------|
| Correctness — Does it work as described? | 30% |
| Alignment — Follows skill principles? | 25% |
| Documentation — Clear, complete, accurate? | 20% |
| Testing — Verified on real codebase? | 15% |
| Style — Consistent with codebase? | 10% |

## Recognition

Contributors are recognized in:
- CHANGELOG.md
- GitHub Contributors graph
- Annual credits in README

## Questions?

Open a [Discussion](https://github.com/etemi/production-ready-workflow/discussions) or email: etemi@etherealmultimedia.tech

---

*Thank you for making production-ready workflows better for everyone!*