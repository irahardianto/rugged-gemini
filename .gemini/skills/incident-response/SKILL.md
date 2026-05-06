---
name: incident-response
description: >-
  Structured incident workflow: severity classification, triage, diagnosis,
  mitigation, postmortem, and prevention. Template-driven with blameless review.
---

# Incident Response

Structured framework for handling production incidents with blameless postmortems.

## When to Invoke
- Production incidents (P0-P3)
- Service degradation or outages
- Post-incident analysis and learning
- Improving incident response processes

## Severity Classification

| Severity | Impact | Response Time | Examples |
|---|---|---|---|
| **P0** | Complete outage, data loss risk | Immediate (< 15 min) | Service down, data corruption |
| **P1** | Major degradation, many users affected | < 30 min | Core feature broken, severe performance |
| **P2** | Partial degradation, some users | < 2 hours | Non-critical feature broken, slow queries |
| **P3** | Minor issue, workaround available | < 1 business day | UI glitch, minor performance |

## Incident Workflow

### 1. Detect & Alert
- Automated monitoring triggers alert
- User reports issue
- On-call engineer acknowledges

### 2. Triage
- Classify severity (P0-P3)
- Assess blast radius (users, services, data)
- Identify incident commander
- Open communication channel

### 3. Diagnose
- Form hypotheses (use `debugging-protocol` skill)
- Collect evidence (logs, traces, metrics)
- Identify root cause
- Document timeline

### 4. Mitigate
- Implement immediate fix (rollback, feature flag, hotfix)
- Verify mitigation effectiveness
- Communicate status to stakeholders
- Continue monitoring

### 5. Resolve
- Confirm service fully recovered
- Close incident
- Schedule postmortem (within 48 hours for P0-P2)

## Postmortem Template

```markdown
# Incident Postmortem: {title}
Date: {date}
Severity: P{0-3}
Duration: {start} → {resolved}
Author: {name}

## Summary
{1-2 sentence impact summary}

## Timeline
| Time | Event |
|---|---|
| HH:MM | {event description} |

## Root Cause
{description with evidence}

## Contributing Factors
- {factor with context}

## What Went Well
- {positive observation}

## What Could Be Improved
- {improvement area}

## Action Items
| Action | Owner | Due Date | Status |
|---|---|---|---|
| {specific action} | @{person} | YYYY-MM-DD | Open |
```

## Principles
- **Blameless** — focus on systems and processes, not individuals
- **Evidence-based** — every claim backed by data (logs, traces, metrics)
- **Action items are SMART** — specific, measurable, assigned, realistic, time-bound
- **Share learnings** — postmortems are public within the organization

## Related
- Debugging Protocol @.gemini/skills/debugging-protocol/SKILL.md
- Logging and Observability Principles @.gemini/skills/logging-and-observability-principles/SKILL.md
- Monitoring and Alerting Principles @.gemini/skills/monitoring-and-alerting-principles/SKILL.md
