# Note 02 — The edit diff as the free quality metric

Status: agent note, 2026-10-06. Deep dive: using her edits to generated
artifacts as the honest, ongoing, zero-cost quality signal — "how much did
she have to fix?" — and the small loop that turns those edits into standing
instructions.

## 1. The insight

`0007` says she will iterate: draft → read → change this → reorder → done.
That iteration was framed as a UX requirement (the revise loop). It is also
the **measurement instrument** — and it is the only measurement instrument
that measures what actually matters, because it measures *her* behavior, not
*our* proxy for it.

A generated artifact that needs a 40% rewrite has failed, no matter how clean
the citation cards look. A generated artifact she signs with two small edits
has succeeded, no matter what the retrieval fixture scores. The diff between
the generated file and her final version **is** the quality signal for the
idiom bar (R2 in `30`), and it is collected whether or not we remember to
collect it — the cost is zero if we keep the versions, which we already do
(`0007`: non-destructive, versioned copies).

## 2. What to capture

Per generated artifact, in the output store:

- **v0** — the file as generated (round 1).
- **v1 … vn** — each revision round (agent edits and her edits are both
  versioned; the versioning already exists).
- **vfinal** — the last version before she stops (or prints/saves).
- **The diff summary**: v0 → vfinal, computed deterministically (no model):
  - total: words/paragraphs added, removed, modified; normalized edit load
    (e.g., changed characters per 1,000 generated characters).
  - **by region**: task statements / instruction lines / content blocks /
    Erwartungshorizont table / Kopfzeile / images. (Region buckets come from
    the template structure — we generated it, so we know where each part is.
    This is why template-first generation (P5) makes the diff
    *interpretable*: a hand-rolled document has no stable regions.)
  - **by round**: how much moved in round 1 vs. round 2 vs. the last.
- **The exemplars used** (from `notes/01`) — so the diff can be compared
  against the exemplar condition.

**Deliberately not built for v1:** a diff UI, per-edit categorization,
anything that requires her attention. The diff report is a document jan reads
at the end of the two-week window (`20` §3). The instrumentation is storage
plus a report; the analysis is a human sitting down with it.

## 3. What it tells you (and the honest limits)

**It tells you:**
- **The trend (primary signal):** edit load per artifact type over time.
  Falling = the system is learning her; flat = steady-state quality (read the
  level); rising = something regressed (a route change? a template change? a
  dsh bump?). The Shrink proxy in `01` §6 is this trend.
- **Where it is weak (the regional view):** if the diff concentrates in the
  Bewertungskriterien, the validator floor is too loose there; if it
  concentrates in the opening instruction lines, the register/voice is off
  (the exemplar channel, `notes/01`); if it concentrates in the table
  structure, the template is fighting her Word file. The diff *localizes the
  weakness before anyone can articulate it* — which is exactly what a solo
  maintainer needs, because he does not have time to watch her edit every file.
- **The round shape:** large round-1 + small round-2 = "first draft is close,
  refinement is cheap" (good). Large final round = "she had to redo the end"
  (bad, or: she added context the agent lacked — see below).

**The honest limits:**
- **Her edits are not all errors.** Some are *information* the agent did not
  have ("this class is weaker than average — make Aufgabe 3 easier"), some are
  *taste* (she dislikes a phrasing the model is allowed to produce), some are
  *context* (a new textbook, a term change). The diff measures the *total
  gap*, not the model's error rate. Over-reading it as "the model is X%
  wrong" is the failure mode to avoid.
- **The calibration (optional, cheap):** at the end of the two-week window,
  one question per a few artifacts: "of your changes, which were things I got
  wrong, and which were things I couldn't know?" That single annotation
  splits the total gap into *quality gap* and *information gap* — and the
  information-gap answers are gold, because each one is a standing instruction
  the preferences doc can hold.
- **The baseline (one-off, optional):** to know Dewey is *better* than
  ChatGPT, not just *good*, one A/B during the window: 2–3 tasks she runs
  through both and edits both, diffing each. Her ChatGPT baseline is the
  incumbent bar; without it, Shrink tells you about Dewey, not about the
  comparison. Two or three tasks is enough for a direction; it is not a study.

## 4. The loop it closes (the real "memory," made local)

The edit diff is the input side of the cheapest useful version of interaction
memory — without any of eris's machinery:

```
she edits → diff summary (by region) → recurring edits become visible
        → jan (or the agent, with her OK) converts them to preference entries
        → preferences doc (Q8 pattern: plain Markdown, her plane, editable)
        → the skill's prompt section reads it: "always …, never …"
        → next artifact starts closer to her final version
        → the diff shrinks (Shrink)
```

This is *preference learning from her edits*, and it is local, cheap,
inspectable (she can read the preferences doc — it is a file she can edit),
and reversible (delete the line, the behavior goes). It satisfies the
long-running partner's "in meinem Stil" and "remember I prefer X" demands at
a fraction of the cost of semantic recall — and when (not if) recall
*measurably* fails, Q8's escalation trigger still applies, behind the same
seam.

## 5. Consequences for the build

- **P5 output store** gains two things: versioned artifact files (already
  required by `0007`) and the diff summary (new, deterministic, small).
- **The two-week window** (`20` §3) is where the diff report gets read — it
  is the window's central artifact, alongside the usage log.
- **No new user surface.** She sees nothing; the versions are already visible
  to her (the output folder is hers), and the diff is jan's instrument. The
  only user-facing consequence: her edits are *safe to make* precisely because
  versions are kept — the measurement depends on the safety property, which
  is nice: the metric is a *byproduct of doing the right thing*, not an
  added feature.
- **Privacy:** the diff lives in her output store, on her machine, about her
  own files. Nothing leaves. (If cloud generation is in the loop, the *files*
  have already been sent as generation inputs; the diff adds no new egress.)

## 6. One paragraph for the morning

She was going to edit the artifacts anyway — that is `0007`. If the versions
are kept, her edits come with a free, honest, ongoing quality report that
measures the one thing every other metric only guesses at: how much work the
machine left her. And the edits that repeat are her standing instructions
asking to be written down. That is the eris "runs long" instinct, kept
deliberately small: no vectors, no staged memory, one Markdown file and a diff
report. Build the loop, measure the shrink, and let the escalation trigger
stay dormant.
