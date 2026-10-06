# 01 — Product plan: Dewey

Status: **agent draft for review, 2026-10-06 (overnight).** Not a decision log.
Builds on jan's draft in `plan/` (charter + 10 decision records, all `proposed`);
flips its *emphasis* where the overnight reframe demands it, keeps everything that
still holds. Every flip is argued, not performed. Companion docs:
`10-curation-first.md` (the reframe, in depth), `20-phases-and-gates.md`
(sequencing), `30-risks-and-bets.md` (what can sink it), `40-open-questions.md`
(what only jan/Pauline can answer), `50-first-build.md` (the smallest first thing),
`notes/` (the three ideas carrying the most weight).

## 1. The product in one paragraph

Dewey is a **personal teaching partner for one teacher**. It takes her chaotic
collection of teaching files — the "wild school drive" — and turns it into a
**curated catalog she can trust** (the *Archive*): every file understood,
summarized, classified with its reasoning shown, duplicates surfaced, nothing
touched without her say-so. Standing on that Archive, it **talks to her in her
idiom** — discussing literature and *Anforderungen*, and **producing her
documents** (Arbeitsblätter, Erwartungshorizonte, Klassenarbeiten) as editable
`.docx` files in her house format, citing her own files by name. The product is
not the retrieval, and not the generation. The product is **her trust in it, and
her time back**.

## 2. What changed since the draft — and why that matters

The draft in `plan/` assumed a *given* corpus: "Pauline points Dewey at her
Deutsch folder." Two facts broke that assumption:

1. **The corpus is a wild drive.** It is the desktop-mess: everything she ever
   saved, duplicated subfolders inside folders, three names for the same sheet.
   (jan, 2026-10-06.)
2. **The long-run goal is the curation itself.** "Categorization, clean up the
   drive/workspace, and curate an index with metadata about all docs, which will
   also make them findable later" — called **the prerequisite before the real
   work** (jan, 2026-10-06).

The consequence is not a rewrite. The draft's good decisions all survive
(`0002` provenance, `0004` derived index + overrides, `0008` catalog/dedupe,
`0009` no shell, `0010` AI marking, `0001` out-of-tree bundle, `0005` subject as
configuration). What changes is the **center of gravity**:

- The draft **led with the Partner** (grounded chat + generation) and treated
  ingest/catalog as the plumbing that gets it there (Phases 2–3, means to an end).
- This plan **leads with the Archive** (curation + trust) because the reframe
  says it is the prerequisite, and because — as argued below — it is also where
  the value, the trust, and the name live.

> **The central claim of this plan:** Dewey's core value is turning her chaotic
> drive into a curated, trusted, findable knowledge base. The grounded
> partner is the *second* value, only as good as the Archive beneath it. Build
> and ship in that order.

This is tested, not asserted: `30-risks-and-bets.md` §5 argues the strongest case
*against* the ordering (a search-only product is the fallback the draft itself
names) and says under which evidence the ordering must flip back.

## 3. The user

**Pauline.** Gymnasium teacher in Schleswig-Holstein. Deutsch and Musik.
Non-engineer. Private macOS, admin rights. Busy in the ways teachers are busy:
term rhythm, Klassenarbeit cycles, printing, the drive that has accreted for
decades. She is *one* user.

Honesty rule: **I have not met her and have not seen the drive.** This plan
writes for the *description* jan gave (desktop messi, non-engineer, Deutsch+Musik
Gymnasium, uses ChatGPT as factory and rubber duck) and every assumption about
her is flagged as an assumption. She is the user of record; the description is
my proxy. Where the two might disagree, the description loses and the question
goes to `40-open-questions.md`.

**jan** is builder, operator, and maintainer. His needs (testability, debuggability,
the dsh learning path) never override hers in a conflict — the draft's rule, kept.
His second interest — a demonstrable "local AI on own infrastructure" story — is
real but it wants features she does not; `20-phases-and-gates.md` keeps them
separate when trading scope.

Assumptions table (each is a place this plan could be wrong):

| # | Assumption | If wrong | Where it bites |
|---|---|---|---|
| A1 | Her drive is one folder she can point at (a "school drive" she assembled), not scattered across the whole machine | We must handle N roots, and first-run UX gets harder | `50-first-build.md` |
| A2 | The corpus is mostly text-bearing docs (docx/odt/pdf/md), scans are a minority to be checked | OCR becomes v1-critical, not deferred | Q4 in `40-open-questions.md` |
| A3 | She will correct inferences *if* the cost is one click with evidence shown | Trust engine underperforms; the catalog becomes a black box | `notes/03-trust-calibration.md` |
| A4 | Her "house format" is stable enough to be captured as data + templates | Template layer becomes a chasing game | Q3 (draft), `20-phases-and-gates.md` |
| A5 | She reads German UI text comfortably but English chrome is fine for v1 | German locale pack moves forward | Q2 (draft) |
| A6 | Cloud generation is acceptable for quality; routing stays open (Q1) | Model-route design must change early | Q1 (draft) |

## 4. The problem, precisely

Four pains, in order of adoption risk (the reframe's reordering is the bold one):

- **P1 — The drive is a mess.** Duplicated subfolders in folders, near-duplicate
  sheets under three names, last year beside this year. She has organized it
  before and it rotted. She finds things by `Strg+F` and muscle memory.
  *New pain — this is the one she feels daily without any AI, and the one the
  reframe puts first.*
- **P2 — ChatGPT is ungrounded.** It has never seen her materials. Every result
  is generic, so she rewrites it anyway. In discussion it cannot answer
  "what do I already have on Balladen?"
- **P3 — ChatGPT has no SH idiom.** No *Operatoren* discipline, no
  *Anforderungsbereiche* I/II/III, no published Erwartungshorizont structure.
  A hallucinated Lehrplan reference in a document bearing her name is a
  professional risk for *her*, not for the model.
- **P4 — ChatGPT returns a chat bubble.** She needs a printable `.docx` in her
  house format, with images, that she can iterate on round after round.

The draft ordered P2–P4 with "no corpus" as the fourth item. P1 is promoted to
the top **not because it is bigger, but because it is felt continuously**, it is
the only pain that exists without any AI in the room, and it is the substrate
every other fix stands on. Grounding (P2/P3) is meaningless on a corpus the
system cannot find, classify, and trust.

## 5. Two halves, one product

### 5.1 The Archive — curation and trust

**What it is:** a persistent, derived, rebuildable catalog of her corpus. One
row per document: extracted text, content hash, summary (1–3 sentences),
inferred metadata (Fach, Land, Jahrgangsstufe, Thema, Textsorte,
Kompetenzbereich, Operatoren present) **each with a confidence and the evidence
for it**, duplicate cluster, ingest status. Above it, a second plane of her
corrections that no reindex may touch (`0004`, kept as-is). Around it, a
browse surface — the catalog screen — where she sees what Dewey thinks each
file is, what it dropped, what it thinks is a duplicate, and can override any
of it with one click.

**Why it is the core of the product, not the plumbing:**

1. **Standalone value.** A teacher whose drive is findable, summarized, and
   deduplicated is helped *before any AI conversation happens*. This is the
   failsafe value: if the Partner half never works, the project is not a total
   loss. (The draft's fallback scope — "search-only is the smaller project" —
   becomes a *product*, not a retreat.)
2. **It is the trust engine.** Trust in a new agent is not given by a demo; it is
   built in increments. The first increments are: *it understood my files; it
   showed me why; I corrected one thing and it kept the correction; it never
   touched an original.* The Partner inherits that trust. A generation partner
   arriving cold has to earn what the Archive has already earned.
3. **It sets the quality ceiling of the Partner.** Retrieval is only as good as
   the metadata it retrieves on. If 30% of catalog rows are confidently wrong,
   the Partner cites wrong files confidently — the worst failure mode the draft
   names (`0002`: "confidently wrong Erwartungshorizonte").
4. **The name lives here.** *Dewey* — the archivist, the classification — is a
   library, not a chat bubble. The product should mean what it is called.

**Three jobs, named, so they can be tested separately:**

| Job | The claim | How we know it is done |
|---|---|---|
| **J1 — Understand** | Every readable file gets text, a summary, and vocab-validated metadata with confidence | Ingest runs end-to-end on her real drive; per-file status visible; every failure a typed code with German display text |
| **J2 — Tell the truth** | "What do I have on X?" is answerable and honest; duplicates are surfaced as clusters; gaps are named | A fixed set of *her* real "do I have this?" questions, each answered correctly from the catalog alone; honest-on-empty as a test case |
| **J3 — Stay trustworthy** | Corrections survive reindex; the corpus is untouched; everything is rebuildable; nothing is silent | The two-plane invariant holds after a forced full rebuild; zero writes to the corpus (proven by read-only mount + audit); the catalog screen shows the evidence she clicked on to believe it |

Deep dive: `10-curation-first.md`.

### 5.2 The Partner — grounded discuss / produce / revise

The draft's `0007` is the description and it stays: **a persistent chat session
over her corpus with an artifact loop beside it; three modes, one agent, no
mode switching by the user.**

| Mode | Output | The bar |
|---|---|---|
| **Discuss** | conversation — literature, Anforderungen, lesson design, "is this Aufgabe too hard for Klasse 7" | Grounded when the corpus is relevant; *honestly silent when it is not*; retrieval cheap enough to call mid-thought |
| **Produce** | a file: Arbeitsblatt, Aufgaben, Klassenarbeit, Erwartungshorizont, table | House format by construction (templates, P5; stack question Q3); SH idiom from vendored data **plus her own prior artifacts as exemplars** (`notes/01`); provenance naming her source files; AI-assistance metadata (`0010`) |
| **Revise** | edits across rounds on a live artifact | Non-destructive (new version file, previous untouched); artifact addressable across turns; preview reloads |

What this plan adds to the draft's Partner is one dependency, stated plainly:

> **The Partner's quality is the Archive's quality plus the model's.**
> A perfect model on a half-wrong catalog still produces documents that cite
> the wrong files and sound plausible. The Partner phase must not start until
> the Archive's J1/J2 bars are met on *her* corpus, not on a test fixture.

And one idiom change, the single biggest lever on "it sounds like her, not
ChatGPT": **her own past artifacts are the primary style source** — the SH spec
tells the model what is *allowed*; her files tell it how *she writes*
(`notes/01-idiom-from-exemplars.md`).

### 5.3 Why the order is also an adoption order

First-run dynamics decide whether a non-engineer keeps a tool. Two openings:

- **Opening A (the draft's):** "Point me at your corpus" → a long, silent batch
  job → eventually, a chat window. The first hour is invisible work. A desktop
  messi watching a progress bar over her chaos has already decided how much she
  trusts this.
- **Opening B (this plan):** "Point me at your corpus" → the first thing she
  sees is **a catalog of her own files with plain-language summaries**, the
  duplicate clusters, the "this looks like Klasse 7 — correct?" nudge. The first
  hour is *visible value about her own stuff*, and correcting it is her first
  act of ownership. The Partner arrives as "oh — and it can write this for me,"
  not "oh — a chatbot."

Same pipeline, different surface order. Opening B costs little extra (the
catalog screen is J1/J2 output anyway, `notes/03`) and buys the trust the
Partner needs.

## 6. North star and how we'll know

**North star (unchanged from the draft's success trace):** *She uses it, and she
likes it.* A non-engineer teacher who reaches for it over ChatGPT and stays.

Proxies, measurable and honest:

| Proxy | Definition | Instrument |
|---|---|---|
| **Reach** (the Partner's) | She voluntarily brings a task she would otherwise have given ChatGPT | Count of organic requests outside any demo. First positive = one. Two weeks of real use with zero = thesis in trouble |
| **Shrink** (the Partner's) | Her edits to a generated artifact get *smaller* over time | Diff between generated file and her final version, per artifact type, tracked across artifacts (`notes/02`) |
| **Stewardship** (the Archive's) | She corrects the catalog *and leaves it corrected*; she asks "what do I have on X?" unprompted; duplicate reports get actioned | Override count + stale-override rate; unprompted "do I have" questions; clusters resolved |
| **Trust events** (both) | Moments she lets Dewey act, or tells someone about it | Qualitative log, kept by jan. One event outweighs a week of metrics |

**Deliberately not measured:** abstract retrieval precision (a means, not the
end), feature count, "hours saved" (unmeasurable; do not perform it). The
retrieval fixture still exists and still gates the Partner — as an *internal*
bar, not as the product's scoreboard.

## 7. Product principles — the spec's spine

Each principle exists to resolve a future dispute without re-litigating it.
Numbered so docs can cite them.

1. **Her files are sacred.** The corpus is read-only by default. No deletion, no
   modification, no moving of her files — in v1, at all. Curation in v1 means
   *understanding and surfacing*, not reorganizing. (Extends `0004`'s read-only
   invariant to the reframe's "clean up the drive" — the tension and its
   resolution are in `10-curation-first.md` §4.)
2. **The index is a servant, rebuildable at will.** Everything derived lives in
   a plane that can be deleted and rebuilt with no user input; her corrections
   live in a separate plane that no indexer may write. "Press rebuild" is
   always a valid repair. (`0004`, unchanged.)
3. **Show its reasoning.** Every inference carries confidence + evidence. Every
   citation resolves to a real file. "Ich weiß es nicht" is a first-class
   answer, not a failure. (`0002`, extended to the catalog surface.)
4. **Propose, never impose.** Duplicates are clusters she can view, not
   deletions. A suggested structure is a report she can act on, not a move. A
   generated file is a draft she signs. (Extends `0008`'s "never deletes".)
5. **Idiom is data + her exemplars, not model knowledge.** SH vocabulary is
   vendored data with source and date (`0005`); her own artifacts are the style
   source (`notes/01`). The validator checks against the data, and the prompt
   feeds the exemplars.
6. **The loop is the product.** Discuss is a first-class mode; a session with no
   artifact is a successful session; revision is non-destructive. (`0007`,
   unchanged.)
7. **Small beats clever.** Four model-facing tools with sharp contracts; every
   later tool is a decision record. (Draft battle plan, kept.)
8. **No capability without a date.** Every deferred idea sits in the backlog
   with a date and a trigger, not a build. (Draft battle plan, kept.)
9. **One user is a feature.** No multi-tenant, no school deployment, no
   compliance project. AIS.chat owns the compliance lane; Dewey owns grounding,
   idiom, and artifacts. (Draft, kept — and it is what makes the product
   small enough for one maintainer.)
10. **Fail in German, with a code.** Every failure is a stable typed enum with
    structured data; display text is German i18n. Never a stack trace, never a
    hardcoded English sentence in her face. (Draft charter, kept.)

## 8. MVP — the smallest thing that tests the thesis

The MVP is **one term's worth of her real life, not a feature list.**

**In (the whole of v1):**

- One corpus root: her "school drive" (or the Deutsch portion of it — A1/Q4
  decides the cut; see `40-open-questions.md`).
- **The Archive:** ingest (extract → normalize → chunk → classify → dedupe
  mark), the two-plane store, the catalog screen with confidence/evidence and
  one-click corrections, full rebuild, per-file status + typed failures.
- **The Partner, thin-sliced:** discuss mode grounded on the catalog;
  produce mode for **one artifact family** (Arbeitsblatt + its
  Erwartungshorizont), Deutsch Sek I, SH; `.docx` via deterministic templates;
  non-destructive revision; provenance cards; honest-on-empty.
- German agent, English chrome (v1). Cloud generation route + cheap classify
  route, both hot-swappable (Q1). Pinned dsh, out-of-tree bundle (`0001`).

**Out (v1), with the reason each is out:**

| Out | Why out (and the trigger to bring it in) |
|---|---|
| Musik | Second extraction pipeline, not a second skills pack (`0005`). Trigger: Deutsch-SH works and she asks |
| `.pptx` / `.xlsx` | After the `.docx` round-trip is proven (Q3). Trigger: a real request for one |
| File moving / restructuring the drive | V1 curation is *surfacing*, not *acting*. Trigger: she asks, twice, for Dewey to do the move — then as a consented, journaled, undoable capability with its own decision record |
| OCR / scanned PDFs | Until Q4 tells us how many scans exist. If >~10% of the corpus is scans, this flips early and becomes v1 |
| Vectors / dense retrieval | `0003`: only on measured lexical failure on a real query class |
| Interaction memory beyond the preferences doc | Q8: plain-Markdown preferences doc; escalate only on measured recall failure |
| German UI locale pack | Q2: English chrome is acceptable; agent language is German regardless |
| Web research | Draft battle plan: cheap toggle, but the trust-labelling condition must exist first |
| Multi-user / school deployment | Principle 9. AIS.chat owns that lane |

The **MVP success test** (both halves, both measurable):

1. **Archive:** her drive (or its Deutsch cut) is fully indexed; she has made at
   least a few corrections that survived a rebuild; she can answer "what do I
   have on X?" from the catalog on ≥ 80% of a fixed set of her real questions;
   duplicates are surfaced as clusters, nothing deleted.
2. **Partner:** she **voluntarily brings a task she would have given ChatGPT**,
   and the artifact she signs is closer to her final file than a ChatGPT output
   would have been (measured by the edit diff, `notes/02`).

If (1) holds and (2) does not after two weeks of real use: the grounding/idiom
thesis is in trouble and the product is a *really good archive* — which is
still worth something (Principle: failsafe value). If (1) does not hold: the
Partner is not buildable yet, and we fix the Archive. The ordering makes both
failures *informative* instead of entangled.

## 9. What "done" looks like (v1, end-state)

- A running Dewey on her Mac at a pinned version; starts, points at the drive,
  no jan in the room (draft battle plan Phase 8 bar, kept).
- Her corpus indexed, per-file status visible, corrections surviving rebuilds,
  the catalog screen in German.
- Generated `.docx` artifacts in her house format, versioned non-destructively
  in a separate output folder, carrying AI-assistance metadata (`0010`) and
  Quellenangabe naming her own source files (`0002`).
- A German-speaking partner that cites her files, says "I don't know" when the
  corpus is empty, and gets her edits smaller over time.
- jan's engineering deliverables intact: the bundle + profile, vendored SH
  pedagogy data with source and date, the retrieval fixture, the smoke
  checklist, the pinned upstream contract, the decision records.

## 10. Decision-ready summary — the five things jan can say yes/no to

1. **The ordering:** Archive first, Partner second — as build order *and* as
   value claim (the Archive is standalone worth). *(§2, §5.1)*
2. **V1 curation is surfacing, not moving:** no file moves, deletes, or
   renames by Dewey in v1; "clean up the drive" means *show her the cleanup,
   in an order she approves* — the map before the surgery. *(§5.1,
   `10-curation-first.md` §4)*
3. **The MVP is one term of her life:** her drive (Deutsch cut), the catalog
   screen, discuss + one artifact family, German agent / English chrome.
   Everything else is backlog-with-a-date. *(§8)*
4. **The idiom change:** her own prior artifacts become the primary style
   source (few-shot exemplars at generation time), with the SH spec as the
   validator — this is the single biggest lever on "sounds like her."
   *(§5.2, `notes/01`)*
5. **The scoreboard:** Reach + Shrink + Stewardship + trust events — not
   retrieval precision. The fixture stays as an internal gate only. *(§6)*

If all five are yes: `20-phases-and-gates.md` is the plan of record and
`50-first-build.md` is what gets built first.
