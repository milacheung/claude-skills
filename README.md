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
