# Claude Code Weakness Radar

## Purpose

Use this skill to continuously track Claude Code ecosystem updates and convert newly observed production weaknesses into concrete skills.

This is a daily operating loop for:
- update intelligence (docs/changelog/community signals)
- weakness detection and prioritization
- skill creation/improvement with measurable impact

## When to Use

Use this skill when:
- Running a daily/weekly improvement routine for coding-agent reliability
- You need to keep skill packs aligned with latest Claude Code behavior
- You want a repeatable pipeline from "new issue found" to "new skill shipped"

Run daily at fixed time (for example 07:00 local) for best continuity.

## Inputs Required

Before running, gather:
- Last radar report date
- Existing skill inventory
- Current top production incidents/pain points
- Relevant Claude Code update sources (official docs/changelog/release notes/community reports)

## Operating Loop (Daily)

1. Update Scan
- Check latest Claude Code updates and notable behavior changes
- Extract only high-signal changes that affect production workflows
- Ignore cosmetic or low-impact noise

2. Weakness Detection
- List current weaknesses observed in production usage
- Classify by category:
  - reliability
  - deployment safety
  - debugging gaps
  - observability
  - workflow/coordination
  - security/safety friction

3. Priority Scoring
- Score each weakness on:
  - Impact (1-5)
  - Frequency (1-5)
  - Detectability gap (1-5)
  - Fixability via skill (1-5)
- Priority score = Impact + Frequency + Detectability gap + Fixability

4. Select Top Weakness
- Pick highest score item
- Write one-paragraph root-cause statement
- Define what success looks like if fixed

5. Skill Action
- Decide one action:
  - Create new skill
  - Improve existing skill
  - Merge overlapping skills
- Produce or update SKILL.md with:
  - purpose
  - trigger conditions
  - required inputs
  - step-by-step execution protocol
  - anti-patterns
  - output format
  - verification checklist

6. Validation
- Run at least one simulated scenario through the skill
- Confirm output is actionable, unambiguous, and production-safe
- Record gaps and patch immediately

7. Publish
- Commit changes with clear scope in message
- Push to GitHub
- Log daily summary entry

## Hard Rules

1. No update scan -> no skill action
2. No priority scoring -> no "top weakness" claim
3. No validation -> no publish
4. No vague skills (must include executable checklist and outputs)
5. No duplicate skills without explicit merge rationale

## Source Quality Policy

Prioritize sources in this order:
1. Official Claude Code docs/changelog/release notes
2. Verified maintainer announcements
3. Reproducible community reports
4. General social chatter (lowest trust)

If source confidence is low, mark finding as "tentative" and do not base primary skill work on it.

## Output Format

When running this skill, always produce:

1) Update Digest
- Date
- New high-signal changes
- Potential production impact

2) Weakness Radar Table
- Weakness
- Category
- Impact
- Frequency
- Detectability gap
- Fixability
- Priority score

3) Selected Focus
- Top weakness
- Root cause
- Why now

4) Skill Change
- New skill or updated skill path
- What changed
- Why it reduces risk

5) Validation Evidence
- Scenario tested
- Result
- Remaining gaps

6) Publish Summary
- Commit hash
- Repo URL
- Next-day follow-up item

## Weekly Consolidation (Every 7 Days)

Once per week:
- Review last 7 daily radar outputs
- Merge repetitive weaknesses into broader patterns
- Retire outdated skills or mark deprecated
- Promote proven skills into a "core pack"

## Metrics to Track

Track these weekly:
- New weaknesses detected
- Skills created
- Skills improved
- Mean time from weakness detection to published skill
- Repeated incident classes before vs after skill adoption

## Anti-Patterns to Reject

Reject these behaviors:
- "We saw one tweet, let's rewrite everything"
- "Let's create skill first, justify later"
- "No validation needed; checklist looks fine"
- "Keep piling skills without cleanup"
- "Treat all updates as urgent"

## Suggested Naming Convention

Use predictable names:
- `skills/claude-<domain>-<purpose>/SKILL.md`

Examples:
- `skills/claude-deploy-rollback-guard/SKILL.md`
- `skills/claude-debug-signal-triage/SKILL.md`
- `skills/claude-context-handoff-protocol/SKILL.md`

## Example Daily Run

- Update scan found: change in tool behavior affecting deploy-step reliability
- Weakness scores:
  - deploy verification drift: 16
  - incident handoff ambiguity: 13
  - test artifact traceability: 12
- Selected focus: deploy verification drift
- Action: created `skills/claude-deploy-verification-lock/SKILL.md`
- Validation: simulated rollout with rollback trigger checks, passed
- Publish: committed and pushed with next action to expand smoke-test templates
