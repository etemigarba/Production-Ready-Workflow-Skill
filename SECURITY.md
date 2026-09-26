# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security vulnerability
in the `production-ready-workflow` skill, please report it responsibly.

### How to Report

**Do not create a public GitHub issue** for security vulnerabilities.

Instead, please email us directly at:
**etemi@etherealmultimedia.tech**

Include the following information:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Any suggested fix (if you have one)

### Response Timeline

- **Acknowledgment**: Within 48 hours
- **Initial Assessment**: Within 5 business days
- **Fix Development**: Within 30 days for critical issues
- **Public Disclosure**: After fix is released (coordinated with reporter)

## Security Considerations for Users

### Skill Execution Environment

The `production-ready-workflow` skill operates within your local development environment:
- No network calls except to your local backend during Phase 5 verification
- No code execution — skill orchestrates file operations only
- All operations on local filesystem
- Secrets never logged — environment variables redacted in output

### Best Practices

1. **Review the skill code** before running — `SKILL.md` and `production-ready-workflow.skill` are readable
2. **Run in isolated environment** — Use feature branches, not main
3. **Verify changes** — The skill creates incremental commits for review
4. **Git-based rollback** — Every phase commits for safety

### Known Considerations

| Area | Consideration | Mitigation |
|------|---------------|------------|
| File System Access | Skill reads/writes project files | Review diffs before committing |
| Git Operations | Skill creates commits/branches | Automatic, but review before push |
| Local Backend | Phase 5 starts your backend | Ensure no production config used |
| Browser Automation | Phase 5 may use agent-browser | Runs against localhost only |

## Vulnerability Disclosure

We follow responsible disclosure. When a fix is ready:
1. Patch released with security fix
2. Advisory published in GitHub Security Advisories
3. Credit given to reporter (unless anonymity requested)

Thank you for helping keep this skill secure!