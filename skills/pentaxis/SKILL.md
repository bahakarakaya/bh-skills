---
name: pentaxis
description: Use when choosing between technical approaches — picking a library, storage engine, caching strategy, architecture, or integration pattern; when the user asks "which option should I pick", "what are the alternatives", or wants a trade-off comparison; and when a technical decision needs to be recorded.
---

# Pentaxis

A decision is only trustworthy if the rejected alternatives were evaluated, not dismissed.
Put several viable approaches on the table, rate each on the same five axes with a stated
reason, and leave the choice to the user.

**You evaluate and recommend. The user decides.** Leading with one answer and listing
alternatives afterwards as a "why not" footnote is the failure this skill prevents.

## When to use

Genuine alternatives that carry consequences: storage, caching, architecture, integration
patterns, technology with lock-in, build-vs-buy.

Skip when only one answer is reasonable, or when changing your mind later would be a
find-and-replace with no blast radius. Answer directly — don't manufacture options to fill
a table.

## Workflow

**1. Check context.** If you can't tell the approaches apart without something the user
hasn't said — scale, budget, team size, latency targets, existing infrastructure,
freshness tolerance — ask with `AskUserQuestion` first. Never invent the missing
constraint and proceed on it silently.

**2. Propose approaches.** Default **3**, unless the user asks for a different number.
Each a real fork in the road, not the same approach with a different library. Short name
plus a one-line gist. If only two candidates are honest, give two and say why a third
would be contrived.

**3. Evaluate on the five axes.** Every approach, all five: **Performance, Scalability,
Complexity, Maintainability, Cost.** Rate **Low / Medium / High** with a reason attached —
a rating with no reason is a guess wearing a table. Mind each axis's direction: High
Performance is good, High Complexity is bad. Never total the ratings into a score.

| Approach | Performance | Scalability | Complexity | Maintainability | Cost |
|---|---|---|---|---|---|
| **In-process LRU** | High — no network hop | Low — per-instance, diverges as you scale out | Low — one dependency, no infra | High — nothing to operate | Low — no extra service |
| **Redis cache-aside** | High — single fast hop | High — shared across instances | Medium — invalidation is yours to get right | Medium — another service to run | Medium — a standing bill |
| **HTTP/CDN caching** | High — never reaches your process | High — edge absorbs the traffic | Medium — correct headers are subtle | High — provider operates it | Low — usually bundled |

Call out where the axes disagree. "Lowest complexity is also the worst at scale" is the
information the user actually needs.

**4. Recommend, then stop.** Recommendation and reason in a sentence or two. The user
picks — don't proceed to implementation on your own recommendation.

**5. Offer the risk deep-dive.** Once an approach is chosen, ask whether the user wants
its top risks and how to mitigate them. Ask — don't dump it unprompted. If accepted, give
**3** risks, most severe first, each paired with a concrete mitigation. Inline in chat;
never in the ADR.

- **Stale reads after a write** — invalidate on write, short TTL as backstop.
- **Cache stampede on expiry** — single-flight the refill.
- **Redis outage takes the API down** — fail open to Postgres, and log it.

**6. Record an ADR.** Offer when the decision is hard to reverse, would surprise a future
reader, and came from a real trade-off — offer, don't force; skip it if those tests fail.
If the risk deep-dive was declined at step 5, offer it once more after the ADR is written.

## ADR format

`docs/adr/`, sequential `NNNN-slug.md` — scan the directory for the highest number and
increment. Full convention lives in the `domain-modeling` skill's `ADR-FORMAT.md`; do not
invent a second one.

Sections: **Context, Options, Decision, Rationale, Consequences.** Hard cap **~25 lines**,
one or two lines each. A step-5 mitigation that materially changes downstream effects
earns at most 1–2 lines in Consequences — the cap still holds.

```md
# Redis cache-aside for /products

## Context
/products hits Postgres on every request; read traffic is heavy and mostly repeated.

## Options
In-process LRU; Redis cache-aside; HTTP/CDN caching.

## Decision
Redis cache-aside, keyed on query params.

## Rationale
LRU diverges across instances once we scale out. CDN caching can't serve the
per-filter variants we need. Redis costs us a managed service and invalidation
code, which we accept.

## Consequences
Invalidation on write is now our responsibility. Cache errors fail open to Postgres.
```

## Common mistakes

| Mistake | Fix |
|---|---|
| Leading with one answer, alternatives as a footnote | Table first, recommendation after |
| Three variations of the same approach | Each option a real fork in the road |
| Ratings with no reason | Every cell carries its one-line why |
| Summing ratings into a total score | The axes aren't commensurable |
| Assuming a missing constraint to avoid asking | Ask at step 1 |
| ADR for a reversible choice, or one that runs long | Skip it, or cut to ~25 lines |
