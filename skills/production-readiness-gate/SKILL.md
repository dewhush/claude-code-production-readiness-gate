# Production Readiness Gate

## Purpose

Use this skill before any production deployment to reduce avoidable incidents. It enforces a strict go/no-go gate across code quality, infrastructure safety, data risk, rollback readiness, and post-deploy verification.

Primary weakness addressed in Claude Code production workflows: teams can ship working code without a structured readiness gate, causing outages from missing env vars, unsafe migrations, bad config, and no rollback path.

## When to Use

Use this skill when:
- Deploying to production (web, API, worker, infra)
- Running risky releases (schema migration, auth, billing, queues, caching)
- Performing hotfix deploys under time pressure
- Handing off deployment execution between people/agents

Do not skip this skill for "small" changes if they touch auth, payments, data, infra, or shared dependencies.

## Inputs Required

Collect these inputs before gate review:
- Service/repo name
- Target environment (prod/staging region)
- Release version/commit SHA
- Change summary (what changed + blast radius)
- Deployment method (manual, CI, blue/green, canary, rolling)
- Rollback method (exact command/process)
- Owner + on-call contact

## Hard Stop Rules (Automatic NO-GO)

If any item below is true, status is NO-GO:
1. Rollback path unknown or untested
2. Database migration is destructive without backup/safety plan
3. Required environment variables not verified
4. Health checks/smoke checks undefined
5. No monitoring/alert visibility for impacted service
6. Incident owner unavailable during deploy window

## Gate Checklist

### 1) Build & Test Integrity
- [ ] Clean install succeeds
- [ ] Build succeeds
- [ ] Unit/integration tests pass (or documented exceptions)
- [ ] Lint/type checks pass (or approved waiver)
- [ ] Artifact reproducible in CI

### 2) Config & Secrets Safety
- [ ] All required env vars present in target env
- [ ] Secret names/versions correct
- [ ] Feature flags defaults verified
- [ ] Config diff reviewed for prod impact

### 3) Data & Migration Safety
- [ ] Migration reviewed for lock/latency risk
- [ ] Backward compatibility confirmed (old app + new schema)
- [ ] Backup/snapshot taken (if needed)
- [ ] Data rollback method defined
- [ ] Migration dry run performed in staging/representative env

### 4) Runtime & Infrastructure Risk
- [ ] CPU/memory/connection pool impact estimated
- [ ] External dependencies status checked
- [ ] Rate limits and retry behavior reviewed
- [ ] Timeouts/circuit breakers validated
- [ ] Capacity/autoscaling readiness confirmed

### 5) Observability & Alerting
- [ ] Dashboards prepared (errors, latency, saturation, traffic)
- [ ] Alert thresholds appropriate for deploy window
- [ ] Log filters/queries prepared for new paths
- [ ] Error budget/SLO risk acknowledged

### 6) Rollout Strategy
- [ ] Deploy strategy chosen (canary/rolling/blue-green)
- [ ] Blast radius controls set (gradual traffic, flag gating)
- [ ] Pause/abort criteria defined
- [ ] Rollback command tested or drill-confirmed

### 7) Post-Deploy Verification
- [ ] Health endpoint checks
- [ ] Critical user journey smoke tests
- [ ] Background jobs/queues processing normally
- [ ] No severe regression in error/latency metrics
- [ ] Stakeholder confirmation message template ready

## Readiness Scoring

Score each section 0-2:
- 0 = not done / unknown
- 1 = partial / risky
- 2 = complete / verified

Sections:
1. Build/Test
2. Config/Secrets
3. Data/Migrations
4. Infra/Runtime
5. Observability
6. Rollout/Rollback
7. Post-Deploy Checks

Maximum score: 14

Decision policy:
- 13-14: GO
- 10-12: GO WITH GUARDS (explicit owner approval + tighter canary)
- <=9: NO-GO

Any Hard Stop Rule overrides score and forces NO-GO.

## Deployment Execution Template

Use this exact structure during execution:

1. Preflight summary
   - Service:
   - Version/SHA:
   - Strategy:
   - Rollback:
   - Owner/on-call:

2. Gate outcome
   - Score:
   - Hard stops: none / list
   - Decision: GO / GO WITH GUARDS / NO-GO

3. Live rollout log
   - T+0 deploy start
   - T+5 health checks
   - T+10 error/latency checks
   - T+15 critical journey verification
   - T+20 rollout continue/hold

4. Completion note
   - Final status
   - User impact
   - Follow-ups

## Rollback Playbook (Minimum)

Define before deploy:
- Trigger conditions (example: error rate >2x baseline for 5 min)
- Exact rollback command(s)
- Data rollback step (if schema/data touched)
- Who executes rollback
- Expected recovery time objective (RTO)

During rollback:
1. Announce rollback start
2. Execute rollback command
3. Validate health + core flows
4. Confirm metric normalization
5. Publish incident summary + next actions

## Hotfix Mode (Emergency)

Hotfix still needs gate, but compressed:
- Must satisfy all Hard Stop Rules
- Minimum checks required:
  - Build passes
  - Targeted tests pass
  - Env vars verified
  - Rollback command verified
  - Health + one critical journey smoke check

If any minimum check fails: NO-GO even in emergency.

## Output Format for Agents

When using this skill, produce:
1) Readiness table (section, score, evidence)
2) Decision (GO/GO WITH GUARDS/NO-GO)
3) Explicit blockers (if any)
4) Step-by-step deploy + rollback commands
5) Post-deploy verification evidence

## Anti-Patterns to Reject

Reject/flag these behaviors:
- "Deploy first, monitor later"
- "Rollback plan is re-deploy old code maybe"
- "No staging test but change is tiny"
- "Skipping smoke checks because CI is green"
- "No owner online; will check in morning"

## Example Quick Decision

- Build/Test: 2
- Config/Secrets: 2
- Data/Migrations: 1
- Infra/Runtime: 2
- Observability: 2
- Rollout/Rollback: 2
- Post-Deploy: 1

Score = 12/14 -> GO WITH GUARDS
Guardrails:
- 10% canary for 15 min
- Rollback if p95 latency +30% or error rate +100%

