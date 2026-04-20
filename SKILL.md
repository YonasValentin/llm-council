---
name: llm-council
description: >-
  Run any question, idea, or decision through a council of 5 AI advisors who
  independently analyze it, peer-review each other anonymously, and synthesize
  a verdict. MANDATORY TRIGGERS (always invoke on these exact phrases)
  'council this', 'run the council', 'convene the council', 'war-room this',
  'pressure-test this', 'stress-test this', 'debate this'. STRONG TRIGGERS
  (invoke when the user presents a real tradeoff with stakes) 'should I X or
  Y', 'which option', 'what would you do', 'is this the right move', 'validate
  this', 'get multiple perspectives', 'I can''t decide', 'I''m torn between'.
  Do NOT invoke for simple yes/no questions, factual lookups, creative writing
  tasks, or casual 'should I' without a real tradeoff (e.g. 'should I use
  markdown' is not a council question). DO invoke when the user presents a
  genuine decision with stakes, multiple options, and enough context that
  multiple angles add value.
---

# LLM Council

One AI gives you one answer. That answer feels smart because it was shaped by how you asked. Ask the same question with different framing and you get a different answer, often opposite, equally confident.

The council breaks that loop. Five advisors with different thinking styles answer your question independently. They peer-review each other anonymously. A chairman synthesizes everything into a verdict with a clear recommendation and one concrete next step.

This skill runs the whole protocol inside a single Claude Code session.

## When NOT to convene

If the question fits one of these, just answer directly instead of convening:

- Factual lookup (one correct answer exists).
- Creation task (write, summarize, translate, refactor).
- Trivial choice (e.g. "should I use markdown or plaintext for this note").
- The user has already decided and wants validation. Warn them the council may dissent, then proceed only if they confirm.

If you are unsure, ask the user once: "Is this a real tradeoff you want pressure-tested, or do you want a direct answer?"

## The seven steps

This workflow is rigid. Execute in order. Do not skip, merge, or parallelize across steps.

### Step 1. Scan workspace for context (30 seconds max)

Before framing the question, surface any workspace context the advisors will need. Use Glob and Read. Do not spend more than 30 seconds on this step.

Look for:

- `CLAUDE.md`, `AGENTS.md`, `GEMINI.md` (user or project instructions).
- `memory/**/*.md` (user profile, past decisions, voice).
- Files the user referenced by name or @-mention.
- Recent `council/**/council-report.html` artifacts so you avoid re-counciling ground already covered.

Pick at most 2 to 3 files that would move advisors from generic to grounded. Skip entirely if nothing relevant is present. Never read the whole workspace indiscriminately.

### Step 2. Frame the question

Restate the user's raw question as a single neutral prompt with four parts:

1. Core decision, stripped of emotional lean.
2. User-provided context: facts, numbers, constraints.
3. Workspace context: the 2 to 3 facts from Step 1 that matter here.
4. Stakes: why being wrong is expensive.

Do not inject your own opinion. Do load enough context that advisors can be specific rather than generic.

If the question is too vague to frame ("council this: my business"), ask exactly ONE clarifying question and wait. Then frame.

Save the framed question. Every advisor and every reviewer receives this exact text.

### Step 3. Convene the council (5 advisors in parallel)

Load `references/advisor-prompts.md`. It contains five identity blocks (Contrarian, First Principles, Expansionist, Outsider, Executor) plus a shared wrapper.

Emit all five `Agent` tool calls in a SINGLE assistant message. This is mandatory. Sequential calls defeat the entire point, because later advisors would see earlier ones through context bleed.

For each advisor, the tool call is:

```
Agent(
  description: "<Advisor name>, council round 1",
  subagent_type: "general-purpose",
  prompt: <shared wrapper + advisor block + framed question + workspace snippets>
)
```

Each advisor produces 150 to 300 words, no preamble, leaning fully into their angle. If an advisor hedges, discard and re-spawn that one advisor. (Re-spawning is only allowed for hedging, not for style preference.)

Collect all five outputs before moving on.

### Step 4. Anonymize for peer review

Load `references/anonymization.md` and follow the deterministic shuffle algorithm exactly. Do NOT improvise your own scheme.

Outputs of Step 4:

- One markdown block with five sections labeled `Response A` through `Response E`, in shuffled order, with persona language stripped per the regex list.
- An internal mapping (Contrarian to C, FirstPrinciples to A, etc.) held in your scratchpad. This mapping NEVER goes to reviewers. It WILL go into the final transcript.

### Step 5. Peer review (5 reviewers in parallel)

Load `references/reviewer-prompts.md`. Again, emit all five `Agent` tool calls in a SINGLE assistant message.

```
Agent(
  description: "Reviewer <N>, council round 2",
  subagent_type: "general-purpose",
  prompt: <reviewer template + framed question + all five anonymized responses>
)
```

Every reviewer sees the same five anonymized responses and answers the same three questions:

1. Which response is strongest, and why? (pick one letter)
2. Which response has the biggest blind spot, and what is it missing? (pick one letter)
3. What did ALL FIVE miss?

Under 200 words each. Direct, no hedging. The peer-review round is the single highest-leverage step in the whole protocol. Question 3 consistently surfaces things no individual advisor saw.

### Step 6. Chairman synthesis (in this session, not a sub-agent)

Load `references/chairman-prompts.md`. Synthesize in the main session. Never delegate. The main session holds the anonymization map and can de-anonymize for the final report. A sub-agent cannot.

Produce the verdict with this exact five-section structure:

```
## Where the Council Agrees
## Where the Council Clashes
## Blind Spots the Council Caught
## The Recommendation
## The One Thing to Do First
```

The chairman may overrule the majority when minority reasoning is stronger. Say so explicitly when it happens. End with a single concrete next step. One thing, not a list.

### Step 7. Write artifacts and open the report

Create `./council/<timestamp>/` where `<timestamp>` is local time formatted `YYYY-MM-DD-HHMMSS`. Use `date "+%Y-%m-%d-%H%M%S"` via Bash to generate it.

Inside that folder, produce two files by filling templates with string substitution:

- `council-report.html`: Read `assets/templates/report.html`, replace every `{{HANDLEBAR}}` placeholder, Write the result.
- `council-transcript.md`: Read `assets/templates/transcript.md`, fill it the same way, Write the result. The transcript includes the anonymization map. The HTML does not.

Then run `open <path>/council-report.html` via Bash to show the briefing.

Finish with a short message to the user: the recommendation, the one thing to do first, and the path to the report. Nothing more. The user can click into the HTML for detail.

## Operating rules

These are non-negotiable. Violating them corrupts the protocol.

- All five advisor calls MUST be in the SAME assistant message. Parallel dispatch only.
- All five reviewer calls MUST be in the SAME assistant message. Parallel dispatch only.
- Reviewers MUST NOT see the advisor-to-letter mapping. Ever.
- Advisors and reviewers see the framed question, not the user's raw message.
- Chairman synthesis happens in the main session, not a sub-agent.
- Never edit files outside `./council/<timestamp>/`.
- The HTML report is a briefing document. The markdown transcript is the audit trail. Always produce both.

## Resources

- `references/advisor-prompts.md`: five advisor identity blocks and the shared prompt wrapper. Loaded in Step 3.
- `references/reviewer-prompts.md`: peer-review prompt template. Loaded in Step 5.
- `references/chairman-prompts.md`: chairman synthesis template. Loaded in Step 6.
- `references/anonymization.md`: deterministic shuffle algorithm and regex strip list. Loaded in Step 4.
- `references/examples.md`: two worked councils to compare against. Load if the user asks for an example, or if a council session looks thin and you want to recalibrate.
- `assets/templates/report.html`: HTML briefing template. Filled in Step 7.
- `assets/templates/transcript.md`: markdown transcript skeleton. Filled in Step 7.
