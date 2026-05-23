---
description: Generate and evaluate multiple solution strategies. Produces ranked options with tradeoff analysis for technical decisions.
allowed-tools: ["Read", "Grep", "Glob", "WebSearch", "WebFetch"]
---

# Weigh Skill

Generate multiple parallel strategies for a problem, evaluate each against criteria, prune weak options, and recommend the strongest path forward.

## Process

1. **Clarify the problem.** Restate the problem in one sentence to confirm understanding.
2. **Generate 3+ strategies.** Each strategy should be a genuinely different approach, not variations of the same idea.
3. **Verify feasibility.** Check each strategy for unverified technical claims (see Verification Gate below).
4. **Evaluate each strategy** against the criteria below.
5. **Prune the weakest.** Explain why it was eliminated.
6. **Recommend.** Provide the winning strategy with actionable next steps.

## Verification Gate

Before evaluating strategies, check each one for **unverified technical claims**:

- Does this strategy depend on a specific library feature, API, or config option?
- Has that feature been confirmed to exist in the version we're using?
- Is the source a public API/doc, or an internal implementation detail?

If a strategy depends on an unverified external capability (library feature, framework config, third-party API), **verify it exists** using WebSearch, WebFetch, or code inspection before proceeding. Mark any unconfirmed dependency as **[UNVERIFIED]** in the strategy description.

Do NOT recommend an [UNVERIFIED] strategy as the winner. Either confirm the capability first or eliminate the strategy.

## Evaluation Criteria

Evaluate each strategy on:

| Criterion | Description |
|-----------|-------------|
| Blast radius | How much existing code/behavior is affected |
| Backward compatibility | Can this be rolled out without breaking consumers |
| Operational complexity | Deployment, monitoring, rollback difficulty |
| Data integrity risk | Risk of data loss, corruption, or inconsistency |
| Implementation effort | Scope of code changes required |
| Testability | How easily can this be verified end-to-end |

Adjust criteria based on the problem. Not all criteria apply to every decision.

## Output Format

Use star ratings normalized so more stars = better outcome for that criterion. For example, low blast radius = good = 3 stars.

Rating key: ⭐⭐⭐ = ideal | ⭐⭐ = acceptable | ⭐ = concern

Present each strategy's evaluation as a table:

```
## Problem
{One sentence}

**Rating:** ⭐⭐⭐ = ideal | ⭐⭐ = acceptable | ⭐ = concern

## Strategies

### Strategy 1: {Name}
{Description - 2-3 sentences}

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Blast radius | ⭐⭐⭐ | {why} |
| Backward compatibility | ⭐⭐⭐ | {why} |
| Operational complexity | ⭐⭐ | {why} |
| Data integrity risk | ⭐⭐⭐ | {why} |
| Implementation effort | ⭐⭐ | {why} |
| Testability | ⭐⭐⭐ | {why} |

### Strategy 2: {Name}
...

### Strategy 3: {Name}
...

## Pruned
**{Strategy name}** - {Why it was eliminated}

## Recommendation
**{Strategy name}** - {Why it wins, with actionable next steps}
```

## Example Usage

User: `/weigh` How should we handle payment webhook failures when the third-party provider is unavailable?

**Expected output:**

- Strategy 1: Retry with exponential backoff inline
- Strategy 2: Dead-letter queue with async retry worker
- Strategy 3: Graceful skip with alert and manual re-trigger

Each evaluated on blast radius (does retry block the pipeline?), data integrity (can payments be duplicated?), operational complexity (does this need a new worker?), etc. Weakest pruned with reasoning. Winner recommended with implementation steps.

## Anti-patterns

- Generating strategies that are trivially similar (e.g., three retry strategies with different intervals)
- Evaluating on criteria that don't matter for the specific problem
- Recommending without explaining why alternatives were rejected
- Skipping the prune step - every evaluation should eliminate at least one option

## Transition

After completing analysis, suggest `/probe` to stress-test the recommended strategy before committing. For low-risk changes, proceed directly to implementation.

After presenting the output, check `~/.claude/html-output.md` for whether to offer an interactive HTML page.
