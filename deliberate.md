---
description: "Orchestrated decision-making with discovery, brainstorm, weigh, and probe phases using independent agents. Use for decisions where you don't have full context — crosses team/system boundaries, unknown external behavior, or decisions where prior art matters."
argument-hint: <problem statement or decision question>
allowed-tools: ["Agent", "Read", "Grep", "Glob", "WebSearch", "WebFetch", "Bash(open *)", "AskUserQuestion"]
---

# Deliberate

Orchestrates discover -> brainstorm -> weigh -> probe using independent agents. Discovery agents search docs, the web, and the codebase to surface context you didn't provide. Probe runs isolated — no deliberation context.

## When to Use

- Decisions crossing team or system boundaries
- Architectural decisions with unknown external behavior
- Decisions where prior art, ADRs, or existing patterns matter
- Decisions where no single person has full context

## When NOT to Use

- Internal decisions with well-understood scope — use `/decide`
- Single-service bug fixes
- Decisions where you already have comprehensive context

## Input

Takes a problem statement as argument. If none provided, ask for one.

## Phase Orchestration

Read `~/.claude/deliberate/finding-format.md` before processing discovery results.

### Phase 1: Discovery (2 agents in parallel)

Spawn 2 agents IN PARALLEL (single message, multiple Agent calls):

**Docs & Web Discovery Agent:**
```
You are a discovery agent. Find documented decisions, prior art, and
external constraints related to this problem:

"{problem_statement}"

Search strategy:
1. WebSearch for existing approaches, patterns, and prior art
2. WebFetch relevant pages or docs for detail
3. Search local project docs — look for ADRs, READMEs, CLAUDE.md files,
   any markdown files documenting decisions
4. For each finding, note whether it's a decision record, draft, or general reference
5. Flag anything older than 6 months as potentially stale

Return format (under 800 tokens) using the finding schema from
~/.claude/deliberate/finding-format.md:

## Docs & Web Discovery Findings
{findings}

## Coverage
- Sources searched: {list}
- Gaps: {what wasn't found}

Max 5 findings. Prioritize decision records over general references.
```

**Codebase Discovery Agent (subagent_type: Explore):**
```
You are a codebase discovery agent. Find related implementations,
patterns, and prior art in the codebase for this problem:

"{problem_statement}"

Search strategy:
1. Find the primary code paths related to this problem
2. Search for related implementations or patterns
3. Check git log for recent changes to relevant files (last 3 months)
4. Read CLAUDE.md files and READMEs for architectural context
5. For each finding, verify the code exists on the current branch

Return format (under 800 tokens) using the finding schema from
~/.claude/deliberate/finding-format.md:

## Codebase Discovery Findings
{findings}

## Coverage
- Directories searched: {list}
- Gaps: {areas not searched}

Max 7 findings. Confirmed-in-code findings are highest value.
```

Wait for both. Collect findings. If an agent fails, note the gap and proceed.

### Discovery Checkpoint (Human Review)

Present findings:

```
## Discovery Findings

### From Docs & Web
{findings or "No results found"}

### From Codebase
{findings or "No results found"}

### Coverage Gaps
{What wasn't searched and why}

### Contradictions
{Any conflicting findings, with both sides shown}
```

Then ask:

```
questions: [
  {
    question: "Review the discovery findings. Proceed to brainstorm?",
    header: "Findings",
    multiSelect: false,
    options: [
      { label: "Proceed", description: "Findings look correct, continue" },
      { label: "Some findings are stale", description: "I'll flag which to ignore" },
      { label: "Missing context", description: "I'll add context discovery missed" }
    ]
  }
]
```

If the user flags stale findings, remove them. If the user adds context, include it as `confirmed-by-human`.

### Phase 2: Brainstorm

Spawn 1 brainstorm agent with the problem statement and all discovery findings.

```
You are the brainstorm agent for /deliberate. Explore the solution space.

## Problem
{problem_statement}

## Discovery Findings (treat as context, not absolute constraints)
{consolidated_findings}

Findings marked "unconfirmed-doc-only" are hypotheses, not hard constraints.
Findings marked "confirmed-in-code" or "confirmed-in-prod" are reliable.
Findings marked "[STALE]" — treat with skepticism.

## Instructions

1. Explore at least 3 genuinely different approaches
2. For each approach, identify tradeoffs
3. Challenge assumptions — what's actually fixed vs assumed?
4. Converge on a recommended direction with reasoning

## Critical: External Assertions

At the end, list ANY assertions you made about systems or behavior you
didn't verify from the findings. These will be fact-checked.

Return (under 2000 tokens):

## Options Explored
### Option 1: {name}
{2-3 sentences + tradeoffs}

### Option 2: {name}
{2-3 sentences + tradeoffs}

### Option 3: {name}
{2-3 sentences + tradeoffs}

## Core Tension
{The fundamental tradeoff}

## Recommended Direction
{Which option and why}

## Assertions About External Systems
{List assertions, or: "No external assertions made."}
```

### Phase 3: Fact-Check (Conditional)

If brainstorm made external assertions, spawn a fact-check agent per assertion (max 3):

```
You are a fact-check agent. Verify this assertion:

## Assertion
"{assertion_text}"

## Instructions
1. Search for evidence in code, docs, or web
2. Report what you found with verification status

Return (under 400 tokens):

## Fact-Check Result
- **Assertion:** {what was claimed}
- **Verdict:** confirmed | refuted | inconclusive
- **Evidence:** {what you found}
- **Source:** {where}
- **Verification:** {confirmed-in-code | confirmed-in-prod | unconfirmed-doc-only}
- **Link:** {URL or file path}
```

### Phase 4: Weigh

Spawn 1 weigh agent with problem, brainstorm output, and all findings including fact-checks.

```
You are the weigh agent for /deliberate. Evaluate and rank strategies.

## Problem
{problem_statement}

## Brainstorm Output
{brainstorm_output}

## Discovery Findings (with verification status)
{consolidated_findings_including_fact_checks}

## Instructions

1. Restate the problem in one sentence
2. Evaluate each strategy against:
   - Blast radius, backward compatibility, operational complexity,
     data integrity risk, implementation effort, testability
3. Mark any strategy that depends on an unverified claim as [UNVERIFIED]
4. Prune the weakest strategy
5. Recommend the winner with actionable next steps

Do NOT recommend a strategy with an [UNVERIFIED] dependency.

**Rating:** 3 stars = ideal | 2 stars = acceptable | 1 star = concern

Return (under 2000 tokens):

## Problem
{one sentence}

## Strategies
{For each: name, ratings, brief rationale}

## Pruned
**{name}** — {why}

## Recommendation
**{name}** — {why it wins, next steps}

## Unverified Dependencies
{Claims the recommendation depends on that aren't confirmed}
```

### Phase 5: Probe (3 agents in parallel)

Spawn 3 probe agents IN PARALLEL with ONLY the problem + recommendation. Do NOT include brainstorm or discovery findings — they attack cold.

Use the same three probe prompts from `/decide` (correctness, operations, blast radius).

Wait for all three. Consolidate by severity. Overall verdict: "needs rethinking" if any probe says so.

### Phase 6: Human Gate

```
## Recommendation
{From weigh}

## Failure Modes
{Merged from all 3 probes, sorted by severity then likelihood}

## Unverified Findings
{Discovery items with "unconfirmed-doc-only" that influenced recommendation}

## Contradictions
{Same-tier conflicts that couldn't be auto-resolved}

## Questions for You
{Specific items where human context would change the recommendation}

## Pre-mortem Scenarios
### Correctness
### Operations
### Blast Radius

## Verdict
{Overall verdict}

## Coverage
{What was and wasn't searched}
```

After presenting, check `~/.claude/html-output.md` for whether to offer an interactive HTML page.

Suggest next steps based on verdict:
- **Safe to proceed:** move to implementation
- **Needs changes:** address issues, re-run `/probe` on revised plan
- **Needs rethinking:** use `/brainstorm` to explore fresh
