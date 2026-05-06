---
name: feature-flags-principles
description: >-
  Feature flag patterns ONLY when PRD/arch requires. Flag evaluation,
  lifecycle, testing, infrastructure. Do NOT load speculatively.
user-invocable: false
---

## Feature Flags Principles

> **CAUTION:** Do NOT implement unless PRD/arch explicitly requires: gradual rollout, A/B testing, kill switches, or tier gating. Flags add real complexity (evaluation infra, lifecycle, test permutations). No spec → deploy directly.

### When to Use

| Use Case | Type | Example |
|---|---|---|
| Gradual rollout | Release | `new-checkout: 0→5→25→100%` |
| Kill switch | Ops | `use-legacy-payment` |
| A/B test | Experiment | `button-color-test` |
| Tier gating | Permission | `pro-tier-analytics` |

### Infrastructure

> Flag backend MUST be specified in tech arch doc. Do NOT choose independently — ask user.

| Approach | When | Notes |
|---|---|---|
| Managed SaaS | Teams >5, multi-env | LaunchDarkly, Flagsmith, Unleash |
| Self-hosted | Full control | Unleash/Flagsmith OSS |
| Firebase Remote Config | Mobile + Firebase | Firebase SDK |
| Static (YAML/env) | Solo, simple on/off | Startup load, no runtime targeting |

### Evaluation Rules

- Server-side for security-sensitive (never trust client)
- Evaluate at request boundary — never inside pure business logic
- Default disabled (fail closed if service unreachable)
- Wrap flag checks once (service method/middleware, not scattered)

```
// ✅ Flag at boundary
func (s *Service) CreateOrder(ctx context.Context, req Request) (Response, error) {
    useNewPricing := s.flags.IsEnabled(ctx, "new-pricing-engine", req.UserID)
    if useNewPricing {
        return s.handleWithNewPricing(ctx, req)
    }
    return s.handleWithLegacyPricing(ctx, req)
}

// ❌ Flag inside business logic
func calculateDiscount(items []Item) float64 {
    if flags.IsEnabled("new-discount-rules") { ... }   // NO
}
```

### Lifecycle

Flags are temporary. Debt accumulates fast.
- Every flag has an owner
- Release flags: max 90 days
- 100% rollout → create removal ticket immediately
- Ops flags can be permanent (documented)
- Experiment flags removed when concluded
- Review flags in sprint planning

### Testing

- Unit: test each branch independently (on + off)
- Integration: default to production-expected state
- Do NOT enumerate all flag combinations
- Use test-only override (`flags.Override(ctx, "flag", true)`)

### Checklist
- [ ] Infra in tech arch doc
- [ ] Backend provisioned + accessible
- [ ] Owner + expiry per flag
- [ ] Server-side for security paths
- [ ] Default: disabled
- [ ] Tests cover both paths
- [ ] Removal ticket at 100%

### Related
- CI/CD @.gemini/skills/ci-cd-principles/SKILL.md
- Architectural Patterns GEMINI.md § Architectural Patterns
- Core Design GEMINI.md § Core Design Principles (YAGNI)
- Security Mandate GEMINI.md § Security Mandate
