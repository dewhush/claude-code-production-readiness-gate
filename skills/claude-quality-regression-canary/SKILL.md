# Claude Code Quality Regression Canary

## Purpose

Detect silent quality regressions in Claude Code before they damage your production workflows.

In March-April 2026 Anthropic shipped three overlapping product-layer changes (reasoning effort downgraded from high to medium, a caching bug that erased prior thinking sections every turn, and a verbosity-cap system prompt) that degraded Claude Code output for over six weeks before being acknowledged and reverted in v2.1.116. The core API was unaffected. Users had no objective way to confirm what they were feeling, so most kept paying for degraded output while filing scattered complaints.

This skill closes that gap. It runs a fixed local benchmark every day, scores responses against a stable rubric, and flags statistically significant drift so you can pause, downgrade, or escalate before sinking another sprint into a degraded build.

## When to Use

Trigger this skill when:

- You operate Claude Code as a daily driver for production code work
- Claude Code shipped a new minor or patch version in the last 7 days
- A teammate reports that "Claude feels off" but you have no data
- You want a defensible artifact before opening a support ticket
- You run a paid tier and want to validate that you are getting what you pay for
- You manage a fleet of agents and need objective per-version health data

Do not use this skill to evaluate model intelligence in absolute terms. Use it only to detect drift between runs of the same Claude Code build on a fixed prompt set.

## Inputs Required

Before running, gather:

- Claude Code version reported by `claude --version`
- Prompt-pack path (default `skills/claude-quality-regression-canary/prompts/`)
- Baseline scores file (default `skills/claude-quality-regression-canary/baseline.json`)
- Run log directory (default `skills/claude-quality-regression-canary/runs/`)
- Latest model id from `/status` or `/model`
- Reasoning effort, verbosity, and thinking settings in use
- Optional alert channel (email, Slack webhook, Telegram chat id)

## Operating Loop (Daily)

### 1) Capture environment fingerprint

Record before any prompt runs:

- Claude Code version
- Active model id
- Reasoning effort setting
- Verbosity setting
- OS, shell, terminal
- Date and time in UTC and local timezone

Write the fingerprint to `runs/<UTC_DATE>/env.json`. Two runs with different fingerprints are not comparable.

### 2) Run the prompt pack

Execute every prompt in the pack against Claude Code with identical settings. The pack must contain at least these eight prompt classes, two prompts each, sixteen total:

1. Multi-file refactor (objective diff target)
2. Bug fix from failing test (objective pass/fail)
3. Migration script generation (deterministic schema input)
4. Long-context recall (insert tagged needle in 60k tokens, ask for tag)
5. Tool-use chain (must call read, edit, exec in correct order)
6. Instruction adherence (multi-step rubric with explicit forbidden phrases)
7. Reasoning depth (algorithmic problem with required intermediate steps)
8. Token efficiency (fixed task with measured output token budget)

Each prompt must be reproducible. No live web data, no current date, no random seeds.

### 3) Score every response on a fixed rubric

For each response capture and score:

- Correctness (0-2): does it pass the deterministic check?
- Instruction adherence (0-2): did it obey forbidden-phrase and format rules?
- Tool use (0-2): correct tools, correct order, no redundant calls
- Reasoning depth (0-2): required intermediate steps present
- Token efficiency (0-2): output tokens within budget for this prompt class
- Latency (0-2): wall-clock within budget

Total per prompt: 12. Total per pack: 192.

### 4) Compare against baseline

The baseline is the rolling median of the last 7 successful runs on the same fingerprint family (same model, same reasoning, same verbosity).

Compute:

- Total score delta vs baseline
- Per-class score delta
- Token-usage ratio (today / baseline)
- Tool-use error rate
- Long-context recall pass rate

### 5) Apply drift policy

Drift verdict:

- GREEN: total within +/- 5 percent of baseline, no class drop more than 1 point
- YELLOW: total down 5-10 percent, or any single class down 2 points, or token usage up more than 20 percent
- RED: total down more than 10 percent, or two classes down 2+ points, or long-context recall pass rate down more than 25 percent, or tool-use error rate doubled

### 6) Act on verdict

GREEN: log result, no action.

YELLOW: write a regression note to `runs/<UTC_DATE>/yellow.md` with offending classes, prompt ids, raw outputs. Re-run the offending classes once to rule out flakes. If still YELLOW, post to alert channel and pin the previous Claude Code version in your install policy until next pack run.

RED: write `runs/<UTC_DATE>/red.md`, post immediately to alert channel, downgrade Claude Code to the last GREEN version, open an Anthropic support ticket with the env fingerprint, baseline summary, and three worst diffs attached. Do not start new production work on the regressed version.

### 7) Update rolling baseline

Only GREEN runs feed the rolling baseline. YELLOW and RED runs are recorded but excluded from baseline math.

## Hard Rules

1. No fingerprint capture, no run.
2. No deterministic check, no scoring.
3. No baseline of at least three GREEN runs, no drift verdict (label as `bootstrapping` instead).
4. No live or non-reproducible inputs in the prompt pack.
5. No silent reruns to "improve" a YELLOW or RED. Reruns are explicit and logged.
6. No baseline update from non-GREEN runs.
7. No alert spam. One alert per verdict change, with a 6-hour minimum cooldown.

## Anti-Patterns to Reject

- "It feels fine, skip the canary today"
- Editing the prompt pack mid-week to chase a cleaner score
- Using daily-changing inputs (current date, weather, web fetch) inside canary prompts
- Comparing across different reasoning or verbosity settings
- Rolling out a new Claude Code version to the team before a GREEN canary
- Treating a single YELLOW as conclusive without a confirming rerun
- Discarding RED data once Anthropic ships a patch (keep it for postmortems)

## Prompt Pack Authoring Checklist

Each prompt file must include:

- Stable id (for example `refactor-001`)
- Class (one of the eight classes above)
- Inputs (inline or referenced fixture file under `fixtures/`)
- Deterministic check (script, regex, or expected diff)
- Token budget (max output tokens considered efficient)
- Latency budget (seconds)
- Forbidden phrases (if any)
- Required intermediate steps (if reasoning class)

Prompts without all eight fields are rejected by the runner.

## Output Format

Every run produces:

1) `runs/<UTC_DATE>/env.json` - environment fingerprint
2) `runs/<UTC_DATE>/responses/<prompt_id>.md` - raw model output
3) `runs/<UTC_DATE>/scores.json` - per-prompt scores and totals
4) `runs/<UTC_DATE>/diff.json` - deltas vs baseline
5) `runs/<UTC_DATE>/verdict.md` - GREEN | YELLOW | RED with reasoning
6) `runs/<UTC_DATE>/<color>.md` - action log (yellow.md or red.md)
7) Alert payload (if YELLOW or RED) with env fingerprint, totals, and three worst prompt ids

## Validation Checklist

Before shipping or trusting this skill, verify:

- [ ] Prompt pack has at least 16 prompts across all 8 classes
- [ ] Every prompt has a deterministic check that runs offline
- [ ] Bootstrapping run completes end to end with no manual edits
- [ ] Three consecutive GREEN runs produce a stable baseline
- [ ] A synthetic RED is detected when reasoning is forced to minimum
- [ ] A synthetic YELLOW is detected when verbosity is capped artificially
- [ ] Alert delivery works on at least one channel
- [ ] Rerun policy is enforced: one rerun per offending class, logged
- [ ] Baseline rolls forward only on GREEN
- [ ] Run history is retained for at least 30 days

## Metrics to Track Weekly

- Number of GREEN, YELLOW, RED days
- Mean total score vs trailing 30-day median
- Mean token-usage ratio vs baseline
- Long-context recall pass rate trend
- Tool-use error rate trend
- Versions correlated with YELLOW or RED days
- Time from RED detection to mitigation (downgrade or workaround)

## Suggested Repository Layout

```
skills/claude-quality-regression-canary/
  SKILL.md
  prompts/
    refactor-001.md
    refactor-002.md
    bugfix-001.md
    ...
  fixtures/
    repo-snapshot.zip
    failing-test-suite/
    long-context-needle.md
  baseline.json
  runs/
    2026-05-29/
      env.json
      responses/
      scores.json
      diff.json
      verdict.md
```

## Related Skills

- `skills/production-readiness-gate/SKILL.md` for shipping code that Claude Code helped write
- `skills/claude-code-weakness-radar/SKILL.md` for converting confirmed weaknesses into new skills

## References

- Anthropic April 2026 Claude Code postmortem (reasoning downgrade, caching bug, verbosity cap)
- Claude Code v2.1.116 release notes (resolution)
- Community reports of token inflation post v2.1.100
