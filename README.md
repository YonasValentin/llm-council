# llm-council

A Claude Code skill that stops you from trusting the first answer you get.

You ask Claude a question. You get one answer. It sounds smart. You move on. Two days later you realize the answer was shaped by how you asked, not by what was true. This skill fixes that. It spins up five advisors with genuinely different thinking styles, has them answer your question independently, makes them peer-review each other anonymously, and gives you a verdict with a single clear next step.

Runs entirely inside Claude Code as a native skill. No external services, no API keys, no extra installs.

## What this is

`llm-council` is a Claude Code skill. If you have not used Claude Code skills before, they are bundles of instructions plus prompt templates that sit in your `~/.claude/skills/` directory. Claude Code loads them on demand when your question matches the skill's triggers.

When you type a phrase like "council this" or "pressure-test this", the skill takes over:

1. It scans your workspace for context files (`CLAUDE.md`, `AGENTS.md`, memory folders) so the advisors have something real to chew on.
2. It frames your question as a neutral prompt.
3. It spawns five Claude Code sub-agents in parallel, one per advisor thinking style.
4. It anonymizes the five responses, then spawns five reviewers in parallel to peer-review the set.
5. It synthesizes everything into a five-section verdict.
6. It writes a clean HTML briefing and a full markdown transcript into `./council/<timestamp>/`, then opens the briefing in your browser.

Total time is around 2 to 4 minutes depending on how much context the advisors need to read.

## Why I built it

I kept noticing the same pattern. I would ask Claude something, get a confident answer, and then catch myself realizing the answer just matched my framing back at me. Ask the same question with the opposite framing and you get the opposite answer, equally confident. That is fine for writing an email. It is not fine for deciding whether to rewrite a backend, switch pricing models, or hire someone.

Having five advisors with genuinely different thinking styles answer independently, then peer-review each other without knowing who wrote what, cuts through that failure mode. The peer-review round is the part that surprised me the most. The single most valuable question is "what did all five miss?" Reviewers reading five strong takes side by side catch things no individual advisor saw, because the gap between five perspectives is where the collective blind spot lives.

## Install

You need Claude Code installed. If you do not have it yet, grab it from Anthropic. Then drop this skill into your skills directory.

```bash
cd ~/.claude/skills
git clone https://github.com/YonasValentin/llm-council.git
```

That is the whole install. Start a new Claude Code session and the skill is discoverable. Type `/llm-council` or just use a trigger phrase in any conversation.

If you prefer to keep your skills in a plugin folder, the standard `~/.claude/skills/llm-council/` layout works. Claude Code picks it up automatically on session start.

## How to use it

Trigger it with a sentence, not a command. The skill is tuned to catch natural phrasing, not specific flags.

Any of these will invoke it:

- `council this: should I rewrite our backend in Rust or invest that time in caching?`
- `pressure-test this pricing: $49/month vs $99/month with a free tier`
- `I am torn between hiring a senior engineer or two juniors. Convene the council.`
- `war-room this positioning: should we niche down to fintech or stay horizontal?`

The skill ignores trivial questions on purpose. "Should I use markdown or plaintext" is not a council question. You will get told that and given a direct answer instead.

Give the council as much context as you can in your prompt. If you have a `CLAUDE.md` or a `memory/` folder in your repo, the skill will read the relevant files before framing the question, so the advisors get grounded advice rather than generic takes.

## What you get back

Two files, in a fresh timestamped folder at `./council/<YYYY-MM-DD-HHMMSS>/`.

**`council-report.html`** is a self-contained briefing document. Warm paper background, serif body, inky blue accent. The chairman's verdict is at the top with five sections:

1. Where the council agrees.
2. Where the council clashes.
3. Blind spots the council caught.
4. The recommendation.
5. The one thing to do first.

Below that, each advisor's full response is in a collapsible section labeled by thinking style. Below that, each reviewer's critique is in another collapsible section. The whole thing is one HTML file. You can email it, commit it, print it.

**`council-transcript.md`** is the audit trail. Everything the council saw and produced, in plain markdown, plus the anonymization map showing which advisor was assigned which letter for the peer-review round. Useful if you want to diff two councils run on similar questions, or if you want to understand why the chairman sided with the minority.

## The five advisors

Each advisor is a distinct thinking style. They are not job titles. They are lenses designed to create three tensions in the room that cancel out each other's blind spots.

**The Contrarian** looks for what will fail. Assumes the idea has a fatal flaw and goes hunting. If the plan survives the Contrarian, it is probably not quietly broken.

**The First Principles Thinker** ignores your surface question and asks what you are really trying to solve. This is the advisor most likely to tell you that you are optimizing the wrong variable.

**The Expansionist** hunts for upside you are missing. What if this works better than you think? What adjacent opportunity are you leaving on the table because you are too anchored on the obvious path?

**The Outsider** has zero context about you, your field, your history, your audience. Responds only to what is literally in front of them. This one is the secret weapon. Experts develop blind spots about what is obvious. The Outsider catches the things your customers would never understand but you think are self-evident.

**The Executor** only cares about what you do Monday morning. Compresses timelines. Proposes the smallest experiment that would actually test the idea. Kills plans that sound brilliant but have no first step.

Together they create three natural tensions: Contrarian against Expansionist on downside versus upside, First Principles against Executor on rethink versus ship, and the Outsider sitting in the middle keeping everyone honest.

## When to convene

Good council questions have real stakes and multiple viable options:

- "Should I launch a $97 workshop or a $497 self-paced course for my Claude Code audience?"
- "Which of these three positioning angles is strongest?"
- "Am I crazy to pivot from B2B to B2C six months in?"
- "Here is my landing page copy. What is weak?"

## When not to convene

- The question has one correct answer (a factual lookup).
- The task is creation, not decision (write a tweet, refactor this function).
- The tradeoff is trivial (markdown vs plaintext for a note).
- You have already decided and just want validation. The council will probably tell you something you do not want to hear. That is the point.

If you are unsure, the skill will ask you once whether this is a real tradeoff you want pressure-tested, or whether you just want a direct answer.

## How it works under the hood

The skill is plain Claude Code. No external dependencies. The protocol is rigid on purpose.

1. The workspace scan pulls in up to three relevant context files so the advisors can be specific rather than generic.
2. The framing step restates the question neutrally, stripped of your emotional lean, with the context baked in. All five advisors receive exactly the same framed question.
3. Round 1 dispatches five parallel `Agent` calls in a single message. Parallel dispatch is mandatory. Sequential calls would let earlier advisors bleed into later ones through shared context, which defeats the independence.
4. The anonymization step runs a deterministic shuffle based on the session timestamp. All five responses get relabeled A through E, with persona language stripped via a regex list so reviewers cannot reverse-engineer who wrote what.
5. Round 2 dispatches five parallel reviewer calls. Every reviewer sees the same five anonymized responses and answers the same three questions: strongest response, biggest blind spot, what did all five miss.
6. The chairman synthesis runs in the main Claude Code session, not a sub-agent. This matters because the main session holds the anonymization map and can de-anonymize for the final report. A sub-agent cannot.
7. The output step reads two templates, substitutes placeholders, writes both files, and opens the HTML in your default browser with `open`.

## File structure

```
llm-council/
├── SKILL.md                          Core workflow, 164 lines
├── README.md                         This file
├── LICENSE                           MIT
├── references/
│   ├── advisor-prompts.md            Five advisor identities + wrapper
│   ├── reviewer-prompts.md           Peer review prompt template
│   ├── chairman-prompts.md           Synthesis prompt + output contract
│   ├── anonymization.md              Deterministic shuffle algorithm
│   └── examples.md                   Two worked councils
└── assets/
    └── templates/
        ├── report.html               Self-contained HTML briefing
        └── transcript.md             Markdown audit trail
```

Everything is plain text. No build step. No compilation. Claude Code reads the files directly.

## Requirements

- Claude Code. Any recent version. The skill uses the `Agent`, `Read`, `Write`, `Glob`, and `Bash` tools that ship with Claude Code by default.
- macOS, Linux, or WSL. The skill runs `date` and `open` via Bash. On Linux, replace `open` with `xdg-open` in your shell config if needed, or just open the generated HTML file manually.
- Nothing else. No API keys, no Python packages, no npm installs.

## Search terms

For anyone finding this via search: this is a Claude Code skill, a Claude Code plugin, an LLM council, a multi-agent debate pattern, a peer-review decision-making framework, a Claude Code skill for decisions, a Claude Code extension for critical thinking, and a Claude Code tool for high-stakes product decisions. If you are looking for a way to pressure-test decisions inside Claude Code, this is it.

## Tips from running it on my own decisions

A few things I have learned from counciling my own work:

The richer the input, the sharper the output. A one-line question gets a one-layer answer. A paragraph with numbers, constraints, and stakes gets a briefing you can actually act on.

Council the decisions where being wrong is expensive. The framework is overkill for "which font should I use." Save it for product direction, pricing, hiring, pivots, major bets.

If a council comes out thin, look at the framing. Nine times out of ten the thin council is a thin framing. Feed in more context and re-run.

Read the peer-review section first. The chairman verdict is a good summary, but the single most valuable paragraph in any council is usually the "what did all five miss?" block.

Do not run the council when you have already decided. It will tell you uncomfortable things. That is the entire point. If you are not ready to hear it, skip it.

## License

MIT. See `LICENSE`.

## Author

Built by Yonas Valentin.

If you run the council on something interesting, I would love to hear about it. Issues and pull requests welcome on this repo.
