---
name: dependency-management-principles
description: >-
  Dependency management: version pinning, vulnerability scanning,
  dependency hygiene. go.mod, package.json, Cargo.toml, pubspec.yaml.
user-invocable: false
---

## Dependency Management

### Version Pinning
Production: exact versions (1.2.3, not ^1.2.0). Prevents supply chain attacks, unexpected breakage. Reproducible builds.

Lock files: `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `Cargo.lock`, `go.sum`, `pubspec.lock`, `poetry.lock`.

### Minimize
Every dep = liability (security, build time, maintenance). Before adding: "50 lines to implement?" "Critical?" "Actively maintained?" "Latest stable?"

### Organize Imports
1. Standard library → 2. External deps → 3. Internal modules. Alphabetical within groups. Remove unused.

### Checklist
- [ ] Production deps pinned to exact versions
- [ ] Lock file committed
- [ ] Each dep justified (not <50 lines to implement)
- [ ] All deps maintained + latest stable
- [ ] Imports organized (stdlib→external→internal)
- [ ] Unused imports removed
- [ ] CVE scan clean (run before every release)
- [ ] Licenses audited against project allowlist

### Vulnerability Scanning
Run per-language CVE scanner as part of CI. Fail on critical/high severity.
For the full per-language command reference and SBOM/license compliance procedures, load:
→ `@.gemini/skills/supply-chain-security/SKILL.md`

### Related
- Security Mandate GEMINI.md § Security Mandate
- Security Principles GEMINI.md § Security Principles
- Supply Chain Security @.gemini/skills/supply-chain-security/SKILL.md