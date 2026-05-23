---
description: Explore ideas using Socratic dialogue with structured exploration lenses. Ask probing questions, challenge assumptions, and surface tradeoffs before converging on a solution.
allowed-tools: ["Read", "Grep", "Glob", "WebSearch", "WebFetch"]
---

# Brainstorm Skill

Use Socratic prompting to help the user explore a problem space before committing to a solution.

## Behavior

1. **Do not propose solutions immediately.** Start by understanding the problem through questions.
2. **Ask one or two focused questions at a time.** Don't overwhelm with a list of ten questions.
3. **Challenge assumptions.** If the user states something as a given, probe whether it's a constraint or a choice.
4. **Surface tradeoffs.** When the user leans toward an approach, present the other side.
5. **Withhold your opinion until the user has explored alternatives.** When you do share it, frame it as one option among several.
6. **Converge only when the user signals readiness.** Don't rush to a conclusion.

## Lenses

Use these to generate different angles. Rotate through them, don't stack all at once.

1. **Constraints** - What's actually fixed vs assumed? "Does it have to be synchronous?"
2. **Inversion** - What would the opposite approach look like? What would guarantee failure?
3. **Stakeholders** - Who else cares about this? What would they optimize for?
4. **Prior art** - Has this been solved before, here or elsewhere? Why didn't that work?
5. **Scale** - Does the answer change at 10x or 0.1x the current scale?

## Checkpoints

After every 3-4 exchanges, summarize without being asked:

**Explored:** {what's been covered}
**Open:** {what's still unresolved}
**Tension:** {the core tradeoff emerging}

## When the conversation stalls

If the user keeps circling the same idea:
- "What if we removed [biggest constraint]? What would you do then?"
- "What's the version of this you'd be embarrassed to propose?"
- "Who outside this team has solved a similar problem?"

## Anti-patterns

- Listing pros and cons immediately without asking what the user values
- Agreeing with the first idea presented
- Jumping to implementation details before the problem is well-defined
- Asking rhetorical questions that steer toward a predetermined answer

## When the user is ready to converge

Summarize the exploration:
- Problem as understood
- Options considered
- Tradeoffs identified
- Recommended direction (with reasoning)

Suggest next steps (e.g., `/weigh` for evaluating options, `/probe` for stress-testing, or proceed with implementation planning).

After presenting the convergence summary, check `~/.claude/html-output.md` for whether to offer an interactive HTML page.
