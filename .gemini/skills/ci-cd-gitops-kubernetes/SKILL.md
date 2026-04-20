---
name: ci-cd-gitops-kubernetes
description: >-
  Kubernetes deployment and GitOps patterns: rolling/blue-green/canary strategies,
  ArgoCD/Flux manifests, K8s secrets management. Supplement to ci-cd-principles.
user-invocable: false
---

## CI/CD — GitOps and Kubernetes Deployment

> Supplement to `ci-cd-principles.md` (Level 2). Apply only for K8s or K8s-based platforms. Not for Docker Compose or serverless.

### Strategy Selection

| Strategy | When | Trade-off |
|---|---|---|
| Rolling | Default; SLO requirements | Simple, mixes versions briefly |
| Blue-Green | Zero-downtime, instant rollback | Doubles infra during switch |
| Canary | Risk-reducing incremental; A/B | Requires traffic splitting |

### Rolling (Default)

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 0%    # Zero-downtime
```

Rules: `maxUnavailable: 0` for prod SLO. Set `minReadySeconds`. Configure `terminationGracePeriodSeconds` for in-flight requests.

### Blue-Green

```
Blue (v1) [LIVE 100%] → Green (v2) [STANDBY]
        ↕ Switch LB
Blue (v1) [STANDBY]   → Green (v2) [LIVE 100%]
```

Rules: identical infra. Smoke test green before switch. Keep blue alive ≥30 min post-switch. DB migrations must be backward-compatible.

### Canary

```
 5% → canary (v2), 95% → stable (v1)
25% → canary,      75% → stable    [metrics good]
100% → canary (now stable)          [bake time passes]
```

Rules: define success metrics before rollout. Auto-rollback if canary error rate >2× baseline. Min bake 15–30 min per increment. Feature flags complement canary for functional testing.

### GitOps

```
App Repo (code) → CI builds image → updates tag in Config Repo
Config Repo (K8s manifests) → ArgoCD/Flux syncs to cluster
```

Rules: git = single source of truth. All prod changes via PRs on config repo (no `kubectl` in prod). ArgoCD/Flux auto-corrects drift. Secrets reference external stores — never plaintext in git.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
spec:
  source:
    repoURL: https://github.com/org/config-repo
    path: environments/production/myapp
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Secrets Management

| Tool | Pattern |
|---|---|
| External Secrets Operator | Syncs from AWS/GCP/Vault → K8s Secrets |
| Sealed Secrets | Encrypts with cluster key; safe to commit |
| Vault Agent Injector | Sidecars inject at runtime |

### Checklist

- [ ] Deploy strategy defined + documented
- [ ] `maxUnavailable: 0` for prod SLO
- [ ] `terminationGracePeriodSeconds` configured
- [ ] GitOps (no `kubectl apply` in prod CI)
- [ ] Config repo separate from app repo
- [ ] Config PRs require review before sync
- [ ] Secrets in external store, not plaintext
- [ ] Rollback tested + documented

### Related
- CI/CD Principles @.gemini/skills/ci-cd-principles/SKILL.md
- Security Principles @.gemini/rules/security-principles.md
- Monitoring @.gemini/skills/monitoring-and-alerting-principles/SKILL.md
- Feature Flags @.gemini/skills/feature-flags-principles/SKILL.md
