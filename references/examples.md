# Worked Examples

Two councils, condensed. Load this file if a council is coming out thin (generic advice, advisors converging too quickly) and you want to recalibrate against what "good" looks like.

Advisor outputs are abbreviated to the key move each advisor made. A real council is 150 to 300 words per advisor.

---

## Example 1, Live workshop vs self-paced course

**User's raw question:** "Should I launch a self-paced Claude Code course for $297, or a live workshop at $97 with the recording included?"

**Framed question:**

> Decision: launch format for a Claude Code product for non-technical solopreneurs, self-paced course at $297 or live workshop at $97 with recording included.
> User context: creator has sold self-paced courses for 3 years; audience is non-technical; AI tooling is moving fast.
> Workspace context: prior audience data (CLAUDE.md) shows 35k newsletter readers, non-technical; past course completion rates ~5%.
> Stakes: 2 to 3 months of build time; wrong format either burns the runway or leaves revenue on the table.

**Contrarian:** Self-paced courses average 3 to 5% completion in this category. Ninety-five percent of $297 buyers will churn, leave bad reviews, request refunds. The "scalable" answer leaves a trail of disappointed customers.

**First Principles Thinker:** The variable the user hasn't named is **speed of change**. A course filmed in March is outdated by May in this space. Self-paced assumes the knowledge is stable; it isn't.

**Expansionist:** A live workshop that bundles the recording gives the buyer both formats in one purchase. One production effort, twice the perceived value. $97 is probably too low for that bundle.

**Outsider:** "Claude Code course" doesn't tell me what I'd actually get. A non-technical reader doesn't know what Claude Code is. The format decision is downstream of a positioning problem.

**Executor:** Workshop prep is 2 to 3 weeks. Course prep is 8 to 12. Ship the workshop, use the recording to build the course later if demand holds.

**Peer review, question 3 (what did all five miss?):** Three of five reviewers converged: "The Contrarian's completion-rate argument and the First Principles Thinker's speed argument reinforce each other in a way neither advisor saw. A live workshop solves both problems simultaneously, completion is enforced by showing up live, AND the content is always current because it's taught live. That combination is the actual moat."

**Chairman's verdict:**
- Agrees: workshop beats course on speed-to-ship, content freshness, and completion rate.
- Clashes: price. Contrarian says $97 is signal-of-low-value; Expansionist says bundle is under-priced. Resolvable by separating bundle tiers.
- Blind spots: the two strongest arguments reinforce each other, not chosen by any individual advisor.
- Recommendation: live workshop with recording bundled, $197. Don't build the self-paced course yet.
- One thing first: open registration for a dated workshop in the next newsletter send. Target 50 signups before writing any material.

---

## Example 2, Rewrite backend in Rust vs invest in caching

**User's raw question:** "We have a Node API hitting latency issues. Should we rewrite the hot path in Rust, or invest the same time in a proper caching layer?"

**Framed question:**

> Decision: reduce API latency, rewrite hot path in Rust (est. 6 weeks) or build a caching layer (est. 4 weeks).
> User context: 2-person backend team, p99 latency is ~800ms on the hot path, 80% of traffic hits the same 3 endpoints.
> Workspace context: AGENTS.md notes the team is Node-native, no existing Rust experience. Prometheus dashboard shows CPU is not saturated.
> Stakes: whichever path is chosen consumes the team for 4 to 6 weeks.

**Contrarian:** A Rust rewrite by a team with no Rust experience is a foot-gun. The 6-week estimate is optimistic by 2x. Meanwhile the latency problem is still there while you're learning lifetimes.

**First Principles Thinker:** Latency has two causes, compute and IO. CPU isn't saturated, so this is IO-bound. Rust doesn't help IO-bound workloads much. The framing ("rewrite the hot path") assumes the bottleneck is code, but the evidence points elsewhere.

**Expansionist:** A caching layer well-designed is infrastructure you'll use on every endpoint for years. The Rust rewrite is a one-endpoint win. The caching investment compounds; the rewrite doesn't.

**Outsider:** I don't see what the endpoints actually do. "Hot path" is opaque. If 80% of traffic hits 3 endpoints and the data is mostly read-heavy, even dumb caching would help a lot.

**Executor:** Ship Redis in front of those 3 endpoints in a week. Measure. If latency is still bad, then consider Rust. Do NOT start a rewrite before measuring the cache's effect.

**Peer review, question 3:** "All five missed: the user didn't say whether stale reads are acceptable. Caching is cheap if stale reads are fine. If every read must be strictly fresh, the cache strategy gets expensive fast (write-through, invalidation). The answer to the question 'cache or Rust' is downstream of that constraint."

**Chairman's verdict:**
- Agrees: latency is IO-bound, not CPU-bound. Rust doesn't fix IO-bound.
- Clashes: Contrarian's skepticism of Rust timeline vs Expansionist's framing of caching as compounding infrastructure. They agree on direction but disagree on rationale.
- Blind spots: whether stale reads are acceptable, the actual load-bearing question, missed by all five advisors.
- Recommendation: caching. Start with Redis in front of the 3 endpoints this week. Do not start a Rust rewrite.
- One thing first: ask the product owner today, "can any of these three endpoints tolerate a 10-second stale read?" Cache strategy depends on the answer.

---

## What these examples illustrate

1. The peer-review "what did all five miss?" question consistently outperforms any individual advisor. Design for it.
2. Good councils produce **actionable** verdicts. "It depends" is a failure state.
3. The Outsider's "I don't know what X is" catch is most valuable when the user is deep in their own framing. Always keep the Outsider.
4. The Executor compresses timelines and proposes cheap experiments. That's the antidote to advisor-induced overthinking.
