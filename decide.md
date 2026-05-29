---
description: "Orchestrated brainstorm -> weigh -> probe with agent isolation. Use for internal decisions where you have the context but want independent stress-testing."
argument-hint: <decision question or problem statement>
allowed-tools: ["Agent", "Read", "Grep", "Glob", "Bash(open *)", "AskUserQuestion"]
---

# Decide

Runs brainstorm -> weigh -> probe as isolated agents. No discovery phase — use this when you already have the context. The value is probe independence: probe receives only the problem and recommendation, not the deliberation that produced it.

For decisions where context is incomplete or crosses boundaries you don't control, use `/deliberate` instead (adds discovery + fact-checking).

## When to Use

- Internal decisions (single-service changes, refactors, design choices)
- Decisions where you already have the context
- Any time you'd manually chain `/brainstorm` -> `/weigh` -> `/probe` but want probe to attack cold

## When NOT to Use

- Decisions crossing team/system boundaries with unknowns — use `/deliberate`
- Simple decisions with an obvious answer — just do it
- Pure exploration with no decision needed — use `/brainstorm` alone

## Input

Takes a problem statement or decision question as argument. If no argument provided, ask:

```
questions: [
  {
    question: "What decision do you need to make?",
    header: "Problem",
    multiSelect: false,
    options: [
      { label: "Design choice", description: "How to structure or implement something" },
      { label: "Refactor strategy", description: "How to restructure existing code" },
      { label: "Tradeoff evaluation", description: "Choosing between known options" }
    ]
  }
]
```

## Phase Orchestration

### Phase 1: Brainstorm

Spawn 1 brainstorm agent with the problem statement. No discovery findings.

```
You are the brainstorm agent for the /decide skill. Explore the solution
space for this problem using Socratic dialogue principles.

## Problem
{problem_statement}

## Instructions

1. Explore at least 3 genuinely different approaches
2. For each approach, identify tradeoffs
3. Challenge assumptions — what's actually fixed vs assumed?
4. Consider: constraints, inversion, stakeholders, prior art, scale
5. Converge on a recommended direction with reasoning

Return format (under 2000 tokens):

## Options Explored
### Option 1: {name}
{2-3 sentences + tradeoffs}

### Option 2: {name}
{2-3 sentences + tradeoffs}

### Option 3: {name}
{2-3 sentences + tradeoffs}

## Core Tension
{The fundamental tradeoff the decision comes down to}

## Recommended Direction
{Which option and why}
```

Wait for return.

### Phase 2: Weigh

Spawn 1 weigh agent with the problem statement and brainstorm output.

```
You are the weigh agent for the /decide skill. Evaluate and rank the
strategies explored by brainstorm.

## Problem
{problem_statement}

## Brainstorm Output
{brainstorm_output}

## Instructions

1. Restate the problem in one sentence
2. Evaluate each strategy against:
   - Blast radius (how much existing behavior is affected)
   - Backward compatibility (can this roll out without breaking consumers)
   - Operational complexity (deployment, monitoring, rollback)
   - Data integrity risk (risk of loss, corruption, inconsistency)
   - Implementation effort (scope of changes)
   - Testability (how easily verified end-to-end)
3. Prune the weakest strategy with reasoning
4. Recommend the winner with actionable next steps

**Rating:** 3 stars = ideal | 2 stars = acceptable | 1 star = concern

Return format (under 2000 tokens):

## Problem
{one sentence}

## Strategies
{For each strategy: name, ratings table, brief rationale}

## Pruned
**{Strategy name}** — {why eliminated}

## Recommendation
**{Strategy name}** — {why it wins, actionable next steps}
```

Wait for return. Extract the recommendation.

### Phase 3: Probe (3 agents in parallel)

Spawn 3 probe agents IN PARALLEL (single message, multiple Agent calls), each with ONLY:
- The problem statement
- The recommendation from weigh (strategy name, description, rationale)

**Do NOT include brainstorm deliberation or weigh evaluation tables.**

This isolation is the core design principle — probes attack the recommendation cold without anchoring bias from the reasoning that produced it.

**Correctness probe:**
```
You are a correctness probe agent for the /decide skill. Break the
recommended approach by finding correctness failures.

You have NOT seen the brainstorm deliberation. Attack cold.

## Problem
{problem_statement}

## Recommendation
{recommendation — strategy name, description, rationale only}

## Focus: Correctness only

- Data integrity — can data be lost, corrupted, or become inconsistent?
- Edge cases — what inputs or states does this not handle?
- Validation gaps — what goes unchecked?
- Race conditions — concurrent access, timing dependencies
- Error handling — what happens on partial failure?

Return (under 1200 tokens):

## Correctness Probe

### Assumptions Under Attack
1. {Assumption} — {Why it might not hold}

### Failure Modes
| # | Failure | Likelihood | Severity | Mitigation |
|---|---------|------------|----------|------------|

### Pre-mortem
{Scenario: this ships, what goes wrong first?}

### Verdict
{Safe / needs changes / needs rethinking}
```

**Operations probe:**
```
You are an operations probe agent for the /decide skill. Break the
recommended approach by finding operational failures.

You have NOT seen the brainstorm deliberation. Attack cold.

## Problem
{problem_statement}

## Recommendation
{recommendation — strategy name, description, rationale only}

## Focus: Operations only

- Rollback safety — can this be reverted cleanly?
- Silent failures — can this fail without alerting anyone?
- Partial failure — what if a dependency is down?
- Monitoring — is the failure observable?
- Deployment — does this require coordination or downtime?

Return (under 1200 tokens):

## Operations Probe

### Assumptions Under Attack
1. {Assumption} — {Why it might not hold}

### Failure Modes
| # | Failure | Likelihood | Severity | Mitigation |
|---|---------|------------|----------|------------|

### Pre-mortem
{Scenario: deployed to production, what breaks first?}

### Verdict
{Safe / needs changes / needs rethinking}
```

**Blast radius probe:**
```
You are a blast radius probe agent for the /decide skill. Break the
recommended approach by finding cross-system and downstream failures.

You have NOT seen the brainstorm deliberation. Attack cold.

## Problem
{problem_statement}

## Recommendation
{recommendation — strategy name, description, rationale only}

## Focus: Blast radius only

- API/contract breaks — does this change an interface consumers depend on?
- Downstream consumers — who reads data this change produces?
- Backward compatibility — can this deploy independently?
- Shared state — does this change shared DB schemas, queues, or config?
- Why does the current behavior exist? (Chesterton's fence)

Return (under 1200 tokens):

## Blast Radius Probe

### Assumptions Under Attack
1. {Assumption} — {Why it might not hold}

### Failure Modes
| # | Failure | Likelihood | Severity | Mitigation |
|---|---------|------------|----------|------------|

### Pre-mortem
{Scenario: a downstream consumer breaks — what happened?}

### Verdict
{Safe / needs changes / needs rethinking}
```

Wait for all three. Consolidate:
- Merge all failure modes into one table, sorted by severity then likelihood
- Overall verdict: "needs rethinking" if any probe says so; "needs changes" if any says so; "safe to proceed" only if all three agree

### Output

```
## Recommendation
{From weigh: winning strategy with rationale}

## Failure Modes
{Merged from all 3 probes, sorted by severity then likelihood}

## Pre-mortem Scenarios
### Correctness
{From correctness probe}
### Operations
{From operations probe}
### Blast Radius
{From blast radius probe}

## Verdict
{Overall verdict}
```

After presenting, check `~/.claude/html-output.md` for whether to offer an interactive HTML page.

Suggest next steps:
- **Safe to proceed:** suggest moving to implementation
- **Needs changes:** address specific issues, then re-run `/probe` on revised plan
- **Needs rethinking:** suggest `/brainstorm` to explore fresh
