# Reviewer Prompts

The peer-review round is the single highest-leverage step in the council protocol. It is what separates the council from "ask Claude five times." Every reviewer sees all five anonymized advisor responses and answers the same three questions independently.

Five reviewers, dispatched in parallel, in one assistant message.

---

## Reviewer prompt template

Substitute `{FRAMED_QUESTION}` and the five `{RESPONSE_X}` blocks before dispatching.

```
You are a reviewer on an LLM Council. Five advisors independently answered a
user's question. You now read all five and answer three questions about the
set as a whole.

The framed question the advisors were given:
---
{FRAMED_QUESTION}
---

The five anonymized responses:

**Response A:**
{RESPONSE_A}

**Response B:**
{RESPONSE_B}

**Response C:**
{RESPONSE_C}

**Response D:**
{RESPONSE_D}

**Response E:**
{RESPONSE_E}

Answer these three questions. Reference responses by letter. Be specific.

1. Which response is the strongest, and why? Pick exactly one letter.
2. Which response has the biggest blind spot, and what is it missing? Pick
 exactly one letter. The blind spot must be something the response should
 have addressed but didn't, not something it chose not to cover.
3. What did ALL FIVE responses miss that the council should consider? This is
 the highest-value question. The gap between five strong perspectives often
 reveals what nobody thought to mention.

Keep the full response under 200 words. Be direct. Do not hedge. Do not
summarize the responses, the chairman already has them.
```

---

## Why three questions, in this order

1. **Strongest + why**, forces each reviewer to make a call, not average. Five independent "strongest" picks cluster meaningfully.
2. **Biggest blind spot**, forces each reviewer to look for failure mode, not success. Balances question 1.
3. **What all five missed**, the reason the council works at all. Individual advisors are chosen for distinct angles but share blind spots (recency bias, the user's framing, shared training data). Only the meta-view catches the collective miss.

---

## What NOT to do

- Do not give reviewers the advisor → letter mapping. Their ranking should be on merit, not on which thinking style they like.
- Do not shuffle the letters per reviewer, all five reviewers see the SAME A to E mapping, so their "pick one letter" answers are comparable.
- Do not let a reviewer refuse to pick ("all are strong"). If that happens, re-spawn that reviewer with `prompt` amended: append "You must pick one letter for questions 1 and 2, even if the margin is thin. That is the job." Do not re-spawn more than once.
- Do not shorten the reviewer response budget below 200 words, question 3 needs space to do real work.
