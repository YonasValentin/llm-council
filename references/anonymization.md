# Anonymization, deterministic shuffle

Between Step 3 (advisors respond) and Step 5 (reviewers peer-review), the five advisor responses must be relabeled A to E in a randomized but reproducible order, and persona language must be stripped so reviewers can't infer which advisor wrote which response.

This file specifies the exact algorithm. Do not improvise, inconsistent anonymization across sessions makes transcripts incomparable, and leaked persona markers undermine the peer-review round's independence.

---

## The permutation algorithm

The five advisors in canonical order are:

```
1. Contrarian
2. FirstPrinciples
3. Expansionist
4. Outsider
5. Executor
```

There are 5! = 120 ways to map these to the letters A, B, C, D, E. The shuffle picks one permutation deterministically from the session timestamp.

### Steps

1. Get the Unix epoch seconds from the timestamp captured at the start of Step 7 (the filename timestamp). Run:

 ```
 date +%s
 ```

 Call that value `T`.

2. Compute `index = T mod 120`.

3. Convert `index` to the permutation at that rank, using factorial-base decomposition (Lehmer code). The algorithm:

 ```
 advisors = ["Contrarian", "FirstPrinciples", "Expansionist", "Outsider", "Executor"]
 permutation = []
 n = index
 for k from 4 down to 0:
 factorial_k = k!
 pick = n // factorial_k
 n = n % factorial_k
 permutation.append(advisors.pop(pick))
 ```

 `permutation` now has five advisor names in a specific order.

4. Assign letters in order: `permutation[0] → A`, `permutation[1] → B`, …, `permutation[4] → E`.

### Why factorial-base

It's the only mapping where every one of the 120 permutations is reachable with exactly one `index` value. Other schemes (e.g. sorting by hash) bias toward certain orders. The Lehmer code guarantees uniform sampling.

### Worked example

Suppose `T = 1745232000` (a Sunday in April 2025).

```
index = 1745232000 mod 120 = 0

k=4: factorial_4 = 24. pick = 0 // 24 = 0. n = 0 % 24 = 0.
 advisors.pop(0) = "Contrarian". → A
k=3: factorial_3 = 6. pick = 0 // 6 = 0. n = 0.
 advisors.pop(0) = "FirstPrinciples". → B
k=2: factorial_2 = 2. pick = 0 // 2 = 0. n = 0.
 advisors.pop(0) = "Expansionist". → C
k=1: factorial_1 = 1. pick = 0 // 1 = 0. n = 0.
 advisors.pop(0) = "Outsider". → D
k=0: factorial_0 = 1. pick = 0 // 1 = 0. n = 0.
 advisors.pop(0) = "Executor". → E
```

Mapping for this session: `Contrarian=A, FirstPrinciples=B, Expansionist=C, Outsider=D, Executor=E`.

Different `T`, different mapping. The peer reviewers receive A to E in that order but have no way to reverse-engineer which advisor is which.

---

## Persona-stripping regex list

Before labeling, strip these patterns from each response (case-insensitive, anchored to the start of the response or the start of any paragraph):

```
^As (the |a )?Contrarian[:]?\s*
^As (the |a )?First[\s-]?Principles[\s-]?Thinker[:]?\s*
^As (the |a )?Expansionist[:]?\s*
^As (the |a )?Outsider[:]?\s*
^As (the |a )?Executor[:]?\s*
^From (the |my |an? )?Contrarian['\u2019]?s?\s+(perspective|angle|view|lens|stance)[:]?\s*
^From (the |my |an? )?First[\s-]?Principles\s+(perspective|angle|view|lens|stance)[:]?\s*
^From (the |my |an? )?Expansionist\s+(perspective|angle|view|lens|stance)[:]?\s*
^From (the |my |an? )?Outsider\s+(perspective|angle|view|lens|stance)[:]?\s*
^From (the |my |an? )?Executor\s+(perspective|angle|view|lens|stance)[:]?\s*
^Looking at this as (the |a )?(Contrarian|First[\s-]?Principles[\s-]?Thinker|Expansionist|Outsider|Executor)[:]?\s*
\bI['\u2019]m the (Contrarian|First[\s-]?Principles[\s-]?Thinker|Expansionist|Outsider|Executor)\b[.]?\s*
\bmy role as (the |a )?(Contrarian|First[\s-]?Principles[\s-]?Thinker|Expansionist|Outsider|Executor)\b[.]?\s*
\b(as|being) (the |a )?(Contrarian|First[\s-]?Principles[\s-]?Thinker|Expansionist|Outsider|Executor)\b
```

After stripping, also collapse any double spaces or leading punctuation that the strip created, so responses read naturally. If a response becomes too short or incoherent after stripping (rare, means the advisor opened 80% persona), re-spawn that one advisor with the wrapper and include this line appended: "Do not open with 'As the X' or any persona framing. Start directly with your point."

---

## What to write where

After anonymization, emit one markdown block for the reviewer prompt:

```
**Response A:**
[stripped text of advisor assigned to A]

**Response B:**
[stripped text of advisor assigned to B]

...through Response E.
```

Hold the mapping (Contrarian→A, etc.) in the main session's scratch only. It MUST appear in the final `council-transcript.md` (under `## Anonymization Map`) so users can audit later. It MUST NOT appear in `council-report.html`, the HTML labels sections by advisor name directly, so no map is needed there.

---

## Edge case: an advisor fails to respond

If one of the five `Agent` calls in Step 3 returns empty or an error, re-spawn that one advisor before moving on to Step 4. Do not attempt anonymization with four responses, the peer-review prompt assumes exactly five labeled responses.
