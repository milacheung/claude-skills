# Claude Code Thinking Skills

A set of Claude Code skills for structured decision-making. Use them independently or as a workflow: **brainstorm** → **weigh** → **probe**.

These are **thinking tools** — they don't write or edit code. They work for technical and non-technical decisions alike: architecture choices, API design, migration strategies, product planning, or any problem with tradeoffs.

## The Skills

| Skill | Purpose | When to use |
|-------|---------|-------------|
| `/brainstorm` | Explore a problem space through Socratic dialogue | Before committing to an approach |
| `/weigh` | Generate and evaluate multiple strategies with tradeoff analysis | When choosing between options |
| `/probe` | Adversarial stress-testing of plans and decisions | Before implementing — find failure modes first |

## The Workflow

```
/brainstorm  →  Diverge. Explore the problem, challenge assumptions, surface options.
     ↓
/weigh       →  Evaluate. Score strategies against criteria, prune the weakest, recommend.
     ↓
/probe       →  Stress-test. Attack assumptions, run pre-mortems, find what breaks.
     ↓
Build        →  Implement with confidence. The plan survived scrutiny.
```

You don't always need all three. Use what fits:
- Quick decision? `/weigh` alone.
- Complex problem, unclear requirements? Start with `/brainstorm`.
- Plan looks good but feels risky? Jump to `/probe`.

## How is this different from extended thinking / plan mode?

Claude Code already has built-in thinking capabilities. These skills fill a different gap:

| | When | Who's thinking | What happens |
|---|---|---|---|
| **Extended thinking** | Any time | Claude alone (internal) | Claude reasons harder before responding. Deeper single-turn output, but still one perspective. |
| **Plan mode** | You know what to build | Claude alone | Claude explores the codebase and proposes implementation steps. Assumes the *what* is decided. |
| **`/brainstorm`** | You don't know what to build yet | You + Claude in dialogue | Claude asks questions, challenges assumptions, surfaces options you hadn't considered. |
| **`/weigh`** | You have options but haven't chosen | Claude evaluating your criteria | Generates 3+ strategies, scores each on tradeoffs, prunes the weakest, recommends a winner. |
| **`/probe`** | You've chosen but haven't committed | Claude as adversary | Attacks your plan with pre-mortems, red-teaming, and falsification. Finds failure modes before production does. |

The key difference: **these skills are collaborative and structured.** Extended thinking is Claude thinking harder alone. Plan mode is Claude planning alone. `/brainstorm` forces a *conversation* — Claude can't just generate an answer, it has to ask questions and let you steer. That surfaces things Claude wouldn't find on its own, because you bring domain knowledge that isn't in the codebase.

They also compose naturally with Claude's built-in features:
- Use `/brainstorm` to decide what to build → then enter **plan mode** to figure out how
- Use `/weigh` to pick a strategy → then use **extended thinking** for the complex implementation details
- Use `/probe` to stress-test a design → then **build** with confidence

## Installation

Copy the skill directories into your Claude Code skills directory:

```bash
# Global (available in all projects)
cp -r brainstorm ~/.claude/skills/
cp -r weigh ~/.claude/skills/
cp -r probe ~/.claude/skills/

# Or project-level (available in one project)
cp -r brainstorm .claude/skills/
cp -r weigh .claude/skills/
cp -r probe .claude/skills/
```

## Customization

These skills are designed to be forked and adapted:

- **`/probe`** — Add domain-specific focus areas. A trading team might add "market hours sensitivity" and "order integrity." A payments team might add "idempotency" and "currency precision." The generic version covers the universals (data integrity, race conditions, partial failure).
- **`/weigh`** — Adjust evaluation criteria for your domain. The defaults (blast radius, backward compatibility, operational complexity) work for most backend systems. Swap or extend as needed.
- **`/brainstorm`** — Add or swap exploration lenses. The included five are general-purpose, but you might add "Regulatory" for fintech or "Accessibility" for frontend work.
