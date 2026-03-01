# Claude Code Skills

A curated collection of analytical thinking skills for [Claude Code](https://claude.com/claude-code).

## Skills

### brainstorm
Explore ideas using Socratic dialogue. Asks probing questions, challenges assumptions, and surfaces tradeoffs before converging to a solution. Use when you need to think through a problem before committing to a direction.

**Usage:** `/brainstorm`

### the-fool
Structured critical reasoning with 5 modes: Socratic questioning, dialectic synthesis, pre-mortem analysis, red teaming, and evidence audit. Use when you need to stress-test a plan, decision, or architecture.

**Usage:** `/the-fool`

**Author:** [Jeffallan](https://github.com/Jeffallan) | License: MIT

## Installation

Copy the skill directories into your Claude Code config:

```bash
# brainstorm (goes in commands/)
cp brainstorm/SKILL.md ~/.claude/commands/brainstorm.md

# the-fool (goes in skills/)
cp -r the-fool ~/.claude/skills/the-fool
```

## Notes

These skills are **thinking tools** — they don't write or edit code. They work for technical and non-technical decisions alike: business strategy, product planning, architecture choices, career decisions, etc.
