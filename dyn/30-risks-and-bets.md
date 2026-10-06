# 30 — Risks and bets

Status: agent draft, 2026-10-06. The honest risk inventory, the load-bearing
bets, the failure stories, and the explicit diff against the draft (what this
plan changes, adds, and removes, with reasons). Written to be argued with, not
nodded at.

## 1. Load-bearing bets — what the whole thing stands on

A bet is load-bearing when the project fails *because it was wrong*, not
because execution slipped.

| # | Bet | If it's wrong | Checked at |
|---|---|---|---|
| B1 | **Curation-first is right**: the Archive is standalone value *and* the substrate *and* the trust engine | The product is a chat partner with expensive plumbing; ordering looks over-cautious | `20` flip table H-archive; two-week window |
| B2 | **Lexical retrieval is enough** for her query class (proper-noun heavy, facet-friendly) (`0003`) | Retrieval feels broken even with good numbers | P4 gate; compound-word probe |
| B3 | **dsh's seams are sufficient** for everything we need, out-of-tree, at the pinned alpha (`0001`) | We're blocked on a core-side change or a vanished seam | P1 gate; upstream contract re-verify |
| B4 | **Read-only corpus + two planes + rebuildability** is sufficient trust architecture for a non-engineer whose files are her professional record | She discovers a modification she didn't authorize (or fears she did) and stops using it | J3 acceptance bar; audit check |
| B5 | **Spec data + her exemplars** get idiom close enough that a generated artifact can pass "sounds like her" | The idiom bar is unreachable at the available model quality; the product is SH-correct but not *hers* | P6 gate (her verdict); Shrink proxy |
| B6 | **The schedule is real**: a thin slice in ~2 months of office-worker evenings, then two weeks of her use, then a decision | Stalls at the thin slice with her waiting; honeymoon expires | `20` hour ranges; P8 |
| B7 | **One artifact family** (Arbeitsblatt + its Erwartungshorizont) is enough to test the produce loop | The family she actually uses most isn't covered and the thesis is tested on a sample that misses her real work | P0 (her prompts say which family she lives in) |
| B8 | **Cloud generation is acceptable** for quality, with routing open (Q1); the data-flow question is engineering, not a compliance project | The model-route design must change early; or a school-context story demands local-only generation | Q1; two-week window |

## 2. Real risks, ranked

Damage × probability, from the top. Each with the mitigation already in the
plan (so a reader can check the mitigation is real, not decorative).

### R1 — Adoption (the project's real risk)
She doesn't use it — or uses it for two weeks, politely, then stops.
Sub-risks in order of likelihood:
- **(a) first-run friction:** the ingest is too slow, the results look wrong,
  the drive is messier than modeled → she never gives it a second look.
  *Mitigation:* Opening B (visible progress, her own files summarized,
  duplicates as the first wow); thin-slice corpus cut small enough to finish
  while she watches; honest-exclusion list (the 37 unreadables, named);
  P0 recon before any pipeline decision.
- **(b) ChatGPT is "good enough":** the grounding/idiom gap is smaller than
  the draft assumes, so the switch cost (new tool, new habits) beats the gain.
  *Mitigation:* this is exactly what the two-week window measures (Reach
  proxy); the thin slice is the cheapest possible test; if it fails, the
  archive half is still worth something (failsafe value).
- **(c) the blessed default:** AIS.chat is free, DSGVO-blessed, and her
  service provides it. *Mitigation:* don't compete on compliance (Principle 9);
  compete on grounding/idiom/artifacts — things AIS.chat structurally cannot do
  (it does not know *her* files).
- **(d) she never corrects the catalog:** inferences rot into confident noise;
  the Partner inherits it. *Mitigation:* low-confidence rows are *phrased as
  questions*, not facts; corrections are one click; the weekly health report
  re-asks small questions; the overrides plane makes her corrections durable,
  which makes correcting worth it.
*Honest note:* this risk cannot be engineered away, only shortened. The plan's
answer is: **make the first value visible in the first hour, and make the
thesis falsifiable in two weeks.**

### R2 — Idiom (the fuzziest acceptance bar)
"It sounds like her material, not ChatGPT" is the bar that compounds: idiom
data quality × exemplar quality × model quality × template fidelity, and the
only final judge is a human who knows her own style. *Mitigation:* exemplar
retrieval (B5, `notes/01`); the validator as floor (SH-correct) vs. her
verdict as ceiling (hers); the P6 gate is *her* reaction, recorded, not a
rubric; Shrink proxy trends the gap over time; Q1 keeps the generation route
escalatable. *Residual:* if the ceiling stays out of reach at cloud-model
quality, the product is "SH-correct and well-cited," which is real value but
not the stated bar — and that is a decision to make on evidence, at P6.

### R3 — Corpus reality (the reframe's cost)
The drive is bigger, messier, or more scanned than modeled: 10k+ files, 40%
scans, legacy formats, no backup, personal data mixed in. *Mitigation:* P0
recon is the project's first act (a real slice + the backup question + the
format/scan census); honest-exclusion as a first-class category (not a bug);
content-hash dedupe divides classification cost by the redundancy factor;
OCR flips into v1 *early* if scans dominate (the trigger is written, not
discovered mid-build); the corpus stays read-only so no mess amplifies.
*Residual:* if the drive is so heterogeneous that one classification schema
fails, the catalog degrades to "readable files with summaries + a big honest
gap list" — still useful (the failsafe), still honest (Principle 3).

### R4 — dsh alpha churn
The pinned alpha is explicitly going to break. Seams change; the session
format bumps; the web UI we depend on for the preview loop reworks.
*Mitigation:* exact pin + upstream contract (24 seams listed, 8 touched);
P1 gate before any investment; the 8-seam rule ("a seam we do not call is a
seam that cannot break us"); upgrade procedure with `--dump-config` diffing;
her machine upgrades last and deliberately. *Residual:* a whole seam
disappearing with no replacement forces an `0001` revisit — recorded as a
named failure, not a surprise.

### R5 — Scope (the solo dev's "no")
"Every conversation adds a capability, and surface area — not difficulty — is
what kills solo projects" (draft, kept verbatim). The specific temptation here
is *the user is his partner*: she is available for feature requests at all
hours, and she is a good user, so her requests are good requests. *Mitigation:*
freeze at thin-slice end; backlog-with-a-date (principle 8); the five
decision-ready items in `01` §10 are the *only* things jan can say yes/no to;
every new idea gets a trigger, not a build. *Residual:* if the trigger
discipline lapses, the project swells to "a teaching platform" and dies of
surface area. The backlog table in `20` §7 is the tripwire.

### R6 — `.docx` round-trip (the honest hard part of the loop)
Generating her house format is template work (solvable); *editing a file she
has touched in Word without destroying her formatting* is the genuinely hard
case, because Word's real-world documents carry the entropy of years of
manual editing. *Mitigation:* non-destructive versioning (each round writes a
new versioned file — the dangerous in-place case is structurally avoided);
template-first generation keeps our files *clean* (known structure), so
round-trip on our own files is easy and round-trip on her files is the
exception path; P5's gate is precisely this round-trip, so the failure is
caught before P6, not in front of her. *Residual:* if her house format is
more idiosyncratic than "just another table" (the draft's hope), the template
layer becomes a long tail — the exemplar set (P0) is what tells us.

### R7 — Schedule / single-maintainer fragility
jan has a 9–5 office day. The plan is ~155 h midpoint plus the two-week
window plus rework. One illness, one term deadline (his *or* hers), and the
honeymoon expires before the thin slice lands. *Mitigation:* the thin slice is
small by construction; the duplicate report (the first wow) is reachable with
only P2+P3 — so *even a stalled project has delivered visible value* by the
time it stalls; the two-week window is scheduled *around* term rhythm, not
against it. *Residual:* this risk is not fully mitigable; the honest answer is
that the MVP must stay small no matter what the calendar says.

### R8 — Trust break (low probability, catastrophic)
Two shapes: (a) the corpus is modified in a way she didn't authorize —
*mitigated structurally*: read-only mount + audit + the invariant is a test,
not a promise; (b) a confidently wrong citation lands in a document she signs
— a hallucinated Operatoren definition or Lehrplan reference in a graded
Erwartungshorizont is *her* professional risk. *Mitigation:* provenance must
resolve (unresolvable = test failure, `0002`); idiom checked against vendored
data, never web; honest-on-empty as an explicit test case; AI-assistance
metadata names the assistance (`0010`) so her document is honest by
construction. *Residual:* model-level errors inside a correctly-cited chunk
are the model's problem, shared with every LLM she uses — the product's duty
is that the *system's* errors (wrong file, wrong idiom, silent absence) are
caught or confessed.

### R9 — Model route (Q1, carried)
Cost/availability of cloud generation; quality variance across route changes;
Q1a (do the company endpoint's terms permit her school material — *check
before wiring it even as a test route*, draft's note, kept); the local 27B's
classification quality on real messy files. *Mitigation:* two routes from day
one, roles split by task (generation → best available; classification →
cheapest adequate), per-request config makes the split reversible; the cheap
probe (one real Arbeitsblatt through the local 27B vs. cloud, `50-first-build.md`)
is run *before* the routes are treated as settled. *Residual:* a route change
is a settings change, but a route *quality* change is an idiom change — the
Shrink proxy is the early warning.

### R10 — Builder bias (named, not hidden)
The agent co-thinker is the same model family that would run inside the
bundle (Qwen-class, local Q4). It will be a strong co-thinker and a fine local
model, and it will also carry blind spots: about German register in teacher
documents, about jan's style, about what "sounds like her" actually means to
*her*. *Mitigation:* every "her" judgment is gated on her verdict or on P0
data, never on the agent's; the plan marks every assumption about Pauline as
an assumption (`01` §3); jan is the plan's peer reviewer and the flip table
(`20` §4) is where disagreement is handled — by evidence, not by whoever is
more confident at 3 a.m.

## 3. Failure stories — what "sunk" looks like

- **S1 (the one-hour graveyard):** "Point me at your files" → 3 hours of
  progress bar → 30% unreadable, 10% misclassified → she nods, says it's
  interesting, closes the window. *Prevented by:* Opening B, small first cut,
  honest gap list, the duplicate wow in the first ten minutes.
- **S2 (the textbook voice):** she asks for a Balladen sheet; it's
  SH-correct, well-cited, and reads like a schoolbook's appendix; she rewrites
  half of it. ChatGPT felt closer. *Prevented by:* exemplars as first-class
  (`notes/01`), P6 gate on *her* verdict, Shrink tracking from artifact #1.
- **S3 (the quiet corruption):** months in, she finds a folder reorganized by
  something she didn't ask for. *Prevented by:* the map-before-surgery rule —
  in v1 there is *nothing* to corrupt (read-only mount, audit, no move
  capability exists).
- **S4 (the citation that bites):** a generated Erwartungshorizont cites her
  2023 sheet for a 2025 text feature; a colleague notices. *Prevented by:*
  resolvable citations as a test invariant, staleness visible in the catalog
  (year markers, `10` §3), the AI-marking metadata making the document's
  provenance honest.
- **S5 (the alpha rot):** dsh 0.1.7 moves the seams we pin; the upgrade is a
  week of archaeology and her install is frozen at an alpha that no longer
  gets web-UI fixes. *Prevented by:* the 8-seam rule, the contract
  re-verification, her-machine-upgrades-last. *Not eliminated:* alpha
  dependencies rot; the mitigation is speed and narrowness.
- **S6 (the scope creep):** she asks for Musik; then PowerPoint; then "can it
  also check my notes"; the backlog dates stop being dates. *Prevented by:*
  freeze + triggers + the five decision-ready items. *Tripwire:* any two
  backlog items built without their trigger being met.
- **S7 (the schedule slip):** the thin slice takes 4 months; by the time it
  lands, the honeymoon is over and her term has moved on. *Prevented by:*
  keeping the thin slice small and the duplicate report reachable early;
  *not eliminable* — this is why the MVP is "one term of her life," not "the
  whole system."

## 4. The diff against the draft — changes, additions, removals

Explicit, because "what would you change" deserves an answer, not a rewrite.

### Changed

| # | Draft | This plan | Why |
|---|---|---|---|
| C1 | Leads with the Partner (grounded chat/generation); ingest+catalog is plumbing (Phases 2–3) | Leads with the Archive: curation is standalone value, the trust engine, and the quality ceiling of the Partner | The overnight reframe: the corpus is a wild drive and curation is "the prerequisite before the real work." `01` §2 |
| C2 | Thin slice = dig → produce loop, no catalog surface at all | Thin slice includes the catalog surface, *agent-rendered* (table in preview, corrections via chat) | A curation-first product with no curation surface ships the pipeline and not the product. Staged: custom node is full-v1 (`20` §1) |
| C3 | P0 = corpus slice + real prompts | P0 adds: one real Arbeitsblatt + one real Erwartungshorizont in her house format (the idiom exemplars), and the backup question | Exemplar retrieval (A1) needs her exemplars in hand before P6, not at P6; the read-only promise should know whether her files have a backup (`10` §4) |
| C4 | Phase 3 includes dedupe clustering | Thin slice: **exact-hash dedupe only**; near-duplicate clustering moves to the backlog with a trigger | Clustering design is guesswork until Q4 shows what her duplicates actually look like (`0008`'s own caution); exact-hash already delivers the "wow" (1,240 files → ~700 documents) |
| C5 | Idiom = vendored SH data + persona | Idiom = vendored SH data (floor, validated) **+ her own prior artifacts as few-shot exemplars** (ceiling, her voice) | The single biggest lever on "sounds like her, not ChatGPT"; answers the "in meinem Stil" question with retrieval, not machinery (`notes/01`) |
| C6 | Quality measured by retrieval fixture + her one-line verdict | Scoreboard = Reach + Shrink + Stewardship + trust events; fixture demoted to internal gate | The fixture measures a means; the north star is her behavior. Her edits are the free, honest quality signal (`notes/02`) |
| C7 | "Clean up the drive" not addressed (corpus assumed given) | Curation = the *map*, not the surgery: suggested-structure report in v1; consented journaled file moves only as a later capability with its own record | Resolves the read-only invariant vs. the cleanup goal without breaking either (`10` §4) |
| C8 | First run = pipeline output, implicitly | First-run experience is a *designed deliverable* (Opening B, `10` §5) | For a non-engineer, the first hour decides adoption; it should be engineered, not incidental |
| C9 | No instrumentation window after the thin slice | A designed two-week window: usage log, automatic edit-diff capture, weekly health reports, proxy review | The thin slice's success test *is* two weeks of real use; the window makes that window measurable instead of anecdotal |

### Added

| # | Addition | Where |
|---|---|---|
| A1 | Exemplar retrieval (her artifacts as few-shot style source) | `notes/01`, P6 |
| A2 | Edit-diff capture as the free quality metric | `notes/02`, P5 output store, two-week window |
| A3 | Catalog screen as trust engine (confidence+evidence, corrections as ownership, staged UI) | `notes/03`, `10` §3, P3 |
| A4 | Suggested-structure report (the "map") | `10` §4, backlog |
| A5 | Filename pre-pass (deterministic German-naming parser as classifier input + evidence) | `10` §1.3, P2 |
| A6 | Honest-exclusion + index-health surface (unreadables/skips/low-confidence/clusters-aweating, named and visible) | `10` §1.4, §3 Q-class D |
| A7 | Two-week instrumentation window | `20` §3 |
| A8 | Flip table (which evidence flips which hypothesis) | `20` §4 |
| A9 | Assumptions table (every claim about Pauline, flagged) | `01` §3 |
| A10 | Stewardship proxy (the Archive's adoption metric) | `01` §6 |

### Removed / deprioritized

| # | Item | To | Why |
|---|---|---|---|
| R1 | Near-duplicate clustering in the thin slice | Backlog, trigger: Q4 shows real duplicates | C4 |
| R2 | Catalog screen v2 (custom node, one-click overrides) | Backlog, trigger: stewardship events exist | Keep the thin slice's UI surface near-zero |
| R3 | "Reorder/restructure existing documents" (draft backlog) | Subsumed by "consented file moves" (single backlog entry, one trigger) | Two entries describing the same capability is how scope sneaks in |
| R4 | — | — | **Nothing from the draft's engineering decisions is removed.** `0001`–`0010` all survive, several strengthened. The draft was good; the reframe moves its emphasis, not its foundations. |

## 5. The one paragraph jan should argue with

If I have one over-reach, it is this: **I have made the Archive *the* product
and the Partner the second act, when jan's words may only support "curation is
the prerequisite" (order) rather than "curation is the core value" (claim).**
The flip table (H-archive) and the two-week window are exactly where that
distinction gets tested, and the plan is written so that if jan's words were
the weaker one, the correction costs nothing: the build order was right either
way. But if he wakes and says "no, the archive is plumbing, the partner is the
product — I only meant *do it first*," then `01` §5.1's three value arguments
(standalone value, trust engine, quality ceiling) must be re-argued or cut, and
the suggested-structure report and catalog v2 lose their justification and go to
the backlog. That is the decision this plan most needs him to make in the
morning: **prerequisite or core?** Everything else follows.
