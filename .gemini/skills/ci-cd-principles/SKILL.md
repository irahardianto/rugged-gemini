---
name: ci-cd-principles
description: >-
  CI/CD pipeline patterns: stages, Dockerfile, GitHub Actions, artifact management,
  environment promotion, rollback. Layered by deployment complexity.
user-invocable: false
---

## CI/CD Principles

> Applies when writing CI/CD manifests (Dockerfile, docker-compose, GH Actions, GitLab CI).
> Layered by complexity — apply only relevant levels.

### Complexity Levels

| Level | When | Additions |
|---|---|---|
| 0 — All | Always | Lint, test, security scan, secrets |
| 1 — Containerized | Docker image = artifact | Multi-stage build, image scan, SBOM |
| 2 — Orchestrated | K8s or managed platform | Deploy strategies, GitOps |

Level 2: load @.gemini/skills/ci-cd-gitops-kubernetes/SKILL.md

### Level 0 — Pipeline Stages (in order)

1. Lint → 2. Build → 3. Unit Test → 4. Integration Test → 5. Security Scan → 6. Deploy

Rules: fail fast (cheapest first). Deterministic. Under 15 min. Never skip failures. Build once, deploy many.

### Level 0 — Deploy Targets

```bash
# Docker Compose
docker compose up --build

# Cloud Run
gcloud run deploy myapp \
  --image gcr.io/project/myapp:$GIT_SHA \
  --region us-central1

# Vercel
vercel deploy --prod
```

K8s: use GitOps — @.gemini/skills/ci-cd-gitops-kubernetes/SKILL.md

### Level 0 — GitHub Actions

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
          cache: true
      - run: gofumpt -l -e -d .
      - run: go vet ./...
      - run: staticcheck ./...

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
          cache: true
      - run: go test -race -cover ./...

  security:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Scan for secrets
        uses: trufflesecurity/trufflehog@v3
      - name: Audit dependencies
        run: go run golang.org/x/vuln/cmd/govulncheck@latest ./...
```

Rules: pin action versions (`@v4`). `needs:` for ordering. Cache deps. `go-version-file` over hardcoded. Secrets via `${{ secrets.NAME }}`.

### Level 1 — Dockerfile

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /bin/api ./cmd/api

FROM gcr.io/distroless/static-debian12
COPY --from=builder /bin/api /bin/api
EXPOSE 8080
CMD ["/bin/api"]
```

Rules: multi-stage. Pin versions (never `:latest`). Copy deps first (layer cache). Minimal runtime. Never copy `.env`/secrets/`.git`.

### Level 1 — Docker Compose

```yaml
services:
  backend:
    build:
      context: ./apps/backend
    ports:
      - "8080:8080"
    env_file: .env
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Rules: health checks for deps. `service_healthy` condition. Pin versions. Volumes for persistence. No hardcoded creds.

### Level 1 — Image Scan + SBOM

Cosign keyless signing (OIDC, no key management):

```yaml
  build:
    needs: security
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - name: Install Cosign
        uses: sigstore/cosign-installer@v3

      - name: Build and push image
        id: build
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Scan container image
        run: |
          trivy image \
            --severity HIGH,CRITICAL \
            --exit-code 1 \
            ghcr.io/${{ github.repository }}:${{ github.sha }}

      - name: Generate SBOM
        run: |
          syft ghcr.io/${{ github.repository }}:${{ github.sha }} \
            -o cyclonedx-json > sbom.json

      - name: Attest SBOM to image (keyless)
        run: |
          cosign attest \
            --predicate sbom.json \
            --type cyclonedx \
            ghcr.io/${{ github.repository }}:${{ github.sha }}
```

Verify: `cosign verify-attestation --type cyclonedx ghcr.io/org/app@sha256:<digest>`

ORAS for arbitrary artifacts beyond Cosign attestation model. Non-containerized apps: `npm audit`/`yarn audit` instead.

### Environment Promotion

`dev → staging → production`. Same artifacts promote. Config via env vars, not build flags. Never deploy direct to prod without staging.

### Feature Flags

Do NOT implement unless PRD/arch requires. See @.gemini/skills/feature-flags-principles/SKILL.md.

### Checklist

**Always:**
- [ ] Stages in order (lint→build→test→security→deploy)
- [ ] All versions pinned
- [ ] Dependency caching
- [ ] No secrets in config
- [ ] Secret scanning in CI
- [ ] Health checks for deps
- [ ] Pipeline < 15 min

**Container (L1):**
- [ ] Multi-stage Docker
- [ ] Image scanned (HIGH/CRITICAL)
- [ ] SBOM attested (Cosign keyless)

**K8s (L2):**
- [ ] Deploy strategy defined
- [ ] GitOps (no `kubectl apply` in prod)
- [ ] Secrets from external store
- See @.gemini/skills/ci-cd-gitops-kubernetes/SKILL.md

### Related
- Code Completion Mandate @.gemini/rules/code-completion-mandate.md
- Security Mandate @.gemini/rules/security-mandate.md
- Security Principles @.gemini/rules/security-principles.md
- Git Workflow @.gemini/skills/git-workflow/SKILL.md
- Project Structure @.gemini/rules/project-structure.md
- Testing Strategy @.gemini/rules/testing-strategy.md
- GitOps + K8s @.gemini/skills/ci-cd-gitops-kubernetes/SKILL.md
- Feature Flags @.gemini/skills/feature-flags-principles/SKILL.md
