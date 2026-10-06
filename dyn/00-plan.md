# 00 — Working plan for the night (Dynamis)

Written by the agent, overnight 2026-10-05 → 06. This is my *working* plan — the
shape of the work before the long thinking. It is not the product plan; it is
how I'll produce one. jan asked me to plan *before* thinking too long, so this
comes first on purpose.

## The task

Produce a **product plan** for Dewey in this folder (`dyn/`), overnight, in the
gap before jan wakes. He is asleep; no one is watching. I have free reign, but
the constraints are:

- Everything lives in `dyn/` (and `dyn/notes/`). I do not touch `plan/`, the
  README, or anything else in the repo.
- No git. No commits, no branch changes.
- No code. This is a thinking deliverable, not an implementation.
- Six hours / the night. Quality over quantity. I'd rather ship four sharp
  docs than ten filler ones.

## The two inputs I'm working from

1. **jan's draft** in `plan/` — strong, honest, but a *draft*, not settled. It
   leads with the chat/generation partner.
2. **jan's overnight reframe** (this conversation): the corpus is a **wild
   desktop mess** — duplicated subfolders in folders, everything she ever
   saved. The long-run goal is to **categorize, clean up, and curate a metadata
   index** of all her docs, which "will also make them findable later." He calls
   this **"the prerequisite before the real work."**

That reframe is the most important thing I have, and it changes the product's
center of gravity. I build around it.

## The central thesis I'm testing

> **Dewey's core value is not the chat partner — it is turning her chaotic
> drive into a *curated, trusted, findable* knowledge base (the Archive). The
> grounded chat/generation partner is the *second* value, and it is only as good
> as the Archive beneath it. So build and ship in that order.**

This inverts the draft's emphasis (which led with chat/generation) and matches
his instinct that curation is the prerequisite. The Archive is also where the
name *Dewey* (classification, catalog, archivist) actually lives, and it's
high-value *standalone* — a teacher with a clean, findable drive instead of a
desktop mess is already helped, before any AI conversation happens. And it is
the **trust-building surface**: she watches it understand her files, corrects it,
and comes to trust it. That trust is what makes the generation partner usable.

Everything in the docs is checked against this thesis. If a doc argues against
it, I write down why.

## The north star (unchanged, it survives the reframe)

**She uses it, and she likes it.** Not "the retrieval is 82%." Not "the bundle
boots." Those are *means*. The end is a non-engineer teacher who reaches for it
over ChatGPT and stays. Two measurable proxies I keep in sight the whole time:

- **Reach:** she voluntarily brings it a task she'd otherwise give ChatGPT.
- **Shrink:** her edits to a generated artifact get *smaller* over time
  (the generated → her-final diff is the honest quality signal; see notes).

## The doc map (what I'll produce, in order)

| File | What it is | Status |
|---|---|---|
| `00-plan.md` | This file: my working plan, thesis, method. | done |
| `01-product-plan.md` | **The deliverable.** Vision, user, problem, the two halves (Archive + Partner), north star, MVP, what-in/what-out. | done |
| `10-curation-first.md` | Deep dive on the reframe: what "categorize + clean up + curate index" concretely means on a wild drive, and the trust model that makes it safe for a non-engineer. | done |
| `20-phases-and-gates.md` | Sequencing. Phases with honest gates; the thin slice; when each hypothesis gets tested and can flip. | done |
| `30-risks-and-bets.md` | The real risks, the load-bearing bets, what could sink it, and exactly what I'd change/add/remove vs the draft. | done |
| `40-open-questions.md` | The questions only jan / Pauline can answer, ranked, each with my recommendation and what I'd do if I had to guess. | done |
| `50-first-build.md` | The concrete *smallest thing to build first* so the thesis gets tested by her hands, not my arguments. | done |
| `notes/01-idiom-from-exemplars.md` | Deep dive: her own past artifacts as the idiom source (not just the SH spec). | done |
| `notes/02-edit-diff-signal.md` | Deep dive: using her edits as the free, ongoing quality metric. | done |
| `notes/03-trust-calibration.md` | Deep dive: the catalog/correction surface as the trust engine. | done |

The core seven (00–50) are the product plan. The `notes/` are the three ideas
that carry the most of the thesis and deserve room to breathe. All ten are
written.

## Morning report (what I actually did)

- **The spine:** curation-first. The Archive is standalone value *and* the
  substrate *and* the trust engine; the Partner stands on it. If that is too
  strong a reading of the overnight reframe, `30` §5 says exactly what to
  change and where, and the flip table (`20` §4) is where the evidence goes.
- **The three new ideas that do the heaviest lifting:** idiom from her own
  exemplars (`notes/01`), the edit diff as the free quality metric
  (`notes/02`), and the catalog as the trust engine (`notes/03`).
- **The decision-ready part:** five yes/no items in `01` §10 and one ranked
  question in `40` §1 (N1: is the Archive a *prerequisite* or a *core value*?).
  `50` §5 is the checklist for the first build.
- **Constraints held:** everything in `dyn/`, no git, no code, nothing outside
  the folder touched. The draft in `plan/` is cited, not copied.

## The method (how I'll think)

- **Build on the draft, don't restart.** The draft's good decisions (out-of-tree
  bundle, provenance, no-shell, honest-on-empty, decision-record hygiene) stay.
  I reframe the *emphasis* and fill the *adoption* gap, I don't discard.
- **Product lens, not engineering lens.** The draft is strong on engineering and
  under-specs adoption. I weight the product/adoption side, because that's where
  "she uses and likes it" lives.
- **Every claim earns its place.** No filler, no rehashing his plan. If a section
  doesn't change a decision, it doesn't exist.
- **Be honest about the unknowns.** Where I'm guessing (I've never met Pauline, I
  haven't seen the drive), I say so and mark it as an open question, not a fact.
- **Leave it decision-ready.** Each doc ends with a concrete "so we do X" — a
  thing jan can react to in the morning, yes/no, not an essay.
- **Plan is downstream of use.** I resist the urge to over-specify. The plan's
  job is to be the cheapest artifact that survives contact with her and lets him
  ship a version she can hate.

## What "done" looks like for me

By morning: the core seven docs exist, the thesis is argued *and* honestly
stress-tested, the reframe is the spine, the risks are named plainly, the open
questions are ranked with my best guesses, and there's one crisp "build this
first." Plus: I did not touch anything outside `dyn/`, ran no git, wrote no
code, and the vault status is honest about what's done vs pending.

## What I will NOT do

- Not defend the draft as settled — it's a draft; I'll flip whatever the
  reframe and the adoption lens argue for, and say so.
- Not pretend I know Pauline. I'll write for the *description* he gave (desktop
  messi, non-engineer, Deutsch+Musik, busy teacher) and flag every assumption.
- Not inflate scope. The plan should make the *first* thing small, not the whole
  thing big.
- Not perform. No hype, no "this will be amazing." Just the best honest plan I
  can produce in the night, ready for a critical reader.
