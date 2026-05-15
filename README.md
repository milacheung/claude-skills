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

They compose naturally with Claude's built-in features. Here's a real example — adding a caching layer to an API:

### Example: Without these skills

```
You:     "Add Redis caching to the /products endpoint"
Claude:  [enters plan mode, proposes implementation steps]
         → builds it
```

Claude plans and builds what you asked for. But did you need Redis? Did you consider the cache invalidation problem? Was caching even the right fix — maybe the query just needs an index?

### Example: With these skills

```
/brainstorm  "The /products endpoint is slow, what should we do?"

Claude:  "What's slow — the DB query, serialization, or network?"
You:     "The DB query, it's doing a full table scan"
Claude:  "Have you considered adding an index before adding a cache?"
You:     "Good point. But we also get hit with traffic spikes..."
Claude:  [checkpoint] Explored: DB bottleneck, index vs cache, traffic patterns
         Open: do we need both? what invalidation strategy?
         Tension: simplicity (index) vs resilience (cache)

         → You now have 3 options you didn't start with
```

```
/weigh  "Compare: add DB index only, add Redis cache, or do both"

Claude:  Strategy 1: DB index only
         | Blast radius    | ⭐⭐⭐ | Migration only, no new infra |
         | Op. complexity   | ⭐⭐⭐ | Nothing to operate            |
         | Implementation   | ⭐⭐⭐ | One migration file            |

         Strategy 2: Redis cache
         | Blast radius    | ⭐⭐  | New dependency across the stack |
         | Op. complexity   | ⭐    | Redis cluster, monitoring, failover |
         | Implementation   | ⭐⭐  | Cache layer + invalidation logic    |

         Strategy 3: Index + cache
         | ...

         Recommendation: Start with the index. If traffic spikes
         still cause issues after indexing, add caching as a second step.

         → Clear winner with reasoning, not a guess
```

```
/probe  "Stress-test the index-first approach"

Claude:  Assumption 1: "The full table scan is the bottleneck"
         → Have you confirmed with EXPLAIN ANALYZE? Could be N+1 queries.

         Failure mode: Index helps reads but you have heavy writes
         during batch imports — index maintenance could slow inserts.

         Pre-mortem: It's Monday morning. The index migration ran in prod.
         Write latency spiked 3x during the nightly product import...

         Verdict: Safe to proceed, but add write latency monitoring
         and test the migration against a prod-sized dataset first.

         → Found a failure mode you would have hit in production
```

```
[plan mode]  "Add the index to the products table"

Claude:  [explores the codebase, finds the migration directory,
          checks existing indexes, proposes the implementation]

         → Now Claude plans the HOW, after you've decided the WHAT
```

```
[extended thinking]  "Write the migration with the optimal index
                      for our query patterns"

Claude:  [thinks deeply about column order, partial indexes,
          covering indexes for the specific queries]

         → Claude thinks harder on the implementation details
```

Each tool did what it's good at. The skills handled the **what** and **why**. Plan mode and extended thinking handled the **how**.

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
