# Claude Code Production Readiness Gate

> Ship to production with confidence, not vibes.

A practical, high-signal skill for Claude Code workflows that prevents avoidable production incidents by enforcing a **structured deploy gate** before release.

---

## What is this?

**Production Readiness Gate** is a deployment safety skill that evaluates whether a release is truly ready for production.

It helps answer:
- Is this deployment safe to roll out now?
- What must be fixed before release?
- If things break, do we know exactly how to recover?

This repo includes:
- `skills/production-readiness-gate/SKILL.md` – full skill playbook
- This README – installation, configuration, usage, and examples

---

## Why this exists (the production weakness it fixes)

A common failure in coding-agent + CI/CD workflows:

✅ Code compiles
✅ Tests pass
❌ Release still causes incidents

Root causes are usually outside pure code correctness:
- missing env vars/secrets in prod
- unsafe migrations
- no tested rollback command
- no deploy-time observability checks
- no explicit GO/NO-GO policy

This skill closes that gap.

---

## Core Outcomes

When you apply this skill, you get:

1. **Hard-stop safety rules** (automatic NO-GO conditions)
2. **7-domain readiness checklist**
3. **Quantitative score** (GO / GO WITH GUARDS / NO-GO)
4. **Deployment execution template**
5. **Rollback playbook**
6. **Hotfix mode guardrails**

---

## Quick Start (2 minutes)

### 1) Install skill files

Copy this folder into your Claude/OpenClaw workspace skills directory:

```text
skills/production-readiness-gate/SKILL.md
```

### 2) Call the skill before any production deploy

Ask your agent to run a readiness gate on the target release using the skill checklist and scoring.

### 3) Decide using policy

- **13–14**: GO
- **10–12**: GO WITH GUARDS
- **<=9**: NO-GO
- Any hard-stop violation: **NO-GO** regardless of score

---

## Installation (Step-by-step)

### Option A — Existing workspace

1. Open your workspace root.
2. Create folder (if missing):
   - `skills/production-readiness-gate/`
3. Place `SKILL.md` in that folder.
4. Commit to your repo.

### Option B — New project bootstrap

1. Create project folder.
2. Add standard workspace structure.
3. Add this skill under `skills/production-readiness-gate/`.
4. Document in your internal ops docs that all prod deploys require this gate.

---

## Configuration Guide

This skill works out-of-the-box, but you should configure release policy values for your environment.

### Recommended policy config (human/org level)

- **Deploy strategy default:** canary or rolling
- **Rollback trigger:** error rate >2x baseline for 5 min
- **Latency guardrail:** p95 >30% baseline increase
- **Owner requirement:** release owner + on-call must be online
- **Migration policy:** destructive changes require backup + explicit approval

### Suggested service-level metadata to capture

For each service, maintain:
- health endpoint URL
- critical smoke-test paths
- rollback command
- dashboards/alerts links
- migration owner

---

## How to Use (Interactive Flow)

Use this conversation pattern with your coding assistant:

### Prompt 1 — Start gate

> Run Production Readiness Gate for release `<service>` at commit `<sha>` to `<environment>`. Provide checklist evidence, section scores, and final decision.

### Prompt 2 — Tighten risk

> If decision is GO WITH GUARDS, generate specific rollout guardrails, abort conditions, and rollback steps.

### Prompt 3 — Execute with logging

> Produce a minute-by-minute rollout log template for T+0 to T+20 and include validation checks.

### Prompt 4 — Post-deploy summary

> Summarize deployment result, user impact, and follow-up tasks.

---

## What the Skill Evaluates (7 Domains)

1. Build/Test Integrity
2. Config/Secrets Safety
3. Data/Migration Safety
4. Runtime/Infra Risk
5. Observability & Alerting
6. Rollout & Rollback Readiness
7. Post-Deploy Verification

---

## Decision Framework

### Hard-stop rules (automatic NO-GO)

- rollback unknown/untested
- destructive migration without safety plan
- required env vars unverified
- smoke checks undefined
- no monitoring visibility
- no available owner during deploy

### Score policy

Each domain is scored 0–2:
- 0 = unknown/not done
- 1 = partial/risky
- 2 = verified/complete

Total max: 14

---

## Example Usage Scenario

Release: API auth service

- Build/Test: 2
- Config/Secrets: 2
- Data/Migrations: 1
- Runtime/Infra: 2
- Observability: 2
- Rollout/Rollback: 2
- Post-deploy: 1

**Total: 12 => GO WITH GUARDS**

Guards:
- 10% canary for 15 min
- pause if auth failure >1.5x baseline
- auto rollback if p95 latency >30% increase for 5 min

---

## Hotfix Mode

Even in emergency, these minimum checks are mandatory:
- build pass
- targeted tests pass
- env vars verified
- rollback command verified
- health + one critical user journey smoke check

If any fails: **NO-GO**

---

## Team Adoption Checklist

- [ ] Add this skill to your shared workspace
- [ ] Make readiness gate mandatory in release SOP
- [ ] Add required deploy metadata per service
- [ ] Train team on GO WITH GUARDS usage
- [ ] Run one rollback drill per critical service

---

## Repository Structure

```text
.
└── skills/
    └── production-readiness-gate/
        └── SKILL.md
```

---

## FAQ

### Is this only for large releases?
No. Small changes can still break production if config, migration, or runtime assumptions are wrong.

### Does this replace CI?
No. CI validates build correctness; this validates release readiness and operational safety.

### Can I tune scoring?
Yes. Keep hard-stop rules strict, but calibrate guardrails for your environment.

---

## Contributing

Improvements welcome:
- stronger rollback drills
- domain-specific check packs (payments, auth, queues)
- better smoke test templates

---

## License

Use internally or adapt for your org playbooks.
