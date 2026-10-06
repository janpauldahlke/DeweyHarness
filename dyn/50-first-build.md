# 50 — First build: the smallest thing her hands can test

Status: agent draft, 2026-10-06. The concrete *smallest thing to build first*
so the thesis gets tested by Pauline's hands, not by arguments. This is the
thin slice of `20-phases-and-gates.md` rendered as a build list: what gets
written, in what order, and when to stop. No code is written in these docs —
this is the spec for the code.

## 1. The shape of the first build

**One folder, one catalog, one conversation, one worksheet family.**

Pauline points Dewey at her "school drive" (or its Deutsch cut — N2). Dewey
reads it overnight-while-she-watches, shows her a catalog of her own files
(summaries, metadata, confidence, the duplicates, the honest gap list), lets
her correct a few rows, answers "what do I have on X?", and — when she asks —
produces an Arbeitsblatt + its Erwartungshorizont as a `.docx` in her house
format, citing her own files, revised across a couple of rounds, with the
edit diff captured. German agent, English chrome, one pinned dsh install,
jan starts it, she uses the browser.

That is the whole first build. Everything else is the backlog with dates
(`20` §7).

## 2. The build list (what exists at the end)

| # | Component | One-line spec | Phase |
|---|---|---|---|
| 1 | **Bundle + profile** | Out-of-tree bundle set + profile against pinned dsh `0.1.6-alpha.2`; boots via `dsh web`; hello tool callable (P1 gate) | P1 |
| 2 | **Ingestion pipeline** | discover → extract (md/docx/odt/pdf) → normalize → chunk (structural) → FTS5 + rows; incremental by hash+mtime; resumable background job with live per-file feed; every failure a typed code + German text; corpus mounted read-only, audit-confirmed | P2 |
| 3 | **Filename pre-pass** | Deterministic parser for German naming conventions (artifact type, Jahrgang, year, version markers, proper nouns) → structured hints per file, used as evidence, never as sole authority | P2 |
| 4 | **Classification** | LLM pass (cheap route) over content + filename + folder hints → Fach, Land, Jahrgang, Thema, Textsorte, Kompetenzbereich, Operatoren, 1–3 sentence summary; vocab-validated; per-field confidence + evidence line; cached by content hash | P3 |
| 5 | **Dedupe (exact)** | Content-hash clusters; representative suggested (newest mtime / dominant folder); clusters surfaced; nothing deleted or merged | P3 |
| 6 | **Two-plane store** | Derived store (rebuildable) + overrides store (her corrections, indexer write-free); full rebuild preserves all overrides (tested); effective metadata = override ?? inference | P3 |
| 7 | **Catalog surface v1 (agent-rendered)** | The agent renders the catalog as a filterable table (Markdown/HTML in the preview): rows with summary, key metadata, confidence, evidence; low-confidence rows phrased as questions; duplicate-cluster view; health view (unreadables/skips/awaiting-decisions); corrections happen in chat ("mark this one as Klasse 9") | P3 |
| 8 | **Retrieval tools** | `catalog_find` (facets + BM25 over summaries/titles; honest on empty) and `corpus_read` (chunks with resolvable provenance); both logged (model-visible ⟺ logged) | P4 |
| 9 | **`.docx` templates** | One family: Arbeitsblatt + Erwartungshorizont, her house format (Kopfzeile Name/Klasse/Datum, Punktespalten, merged cells, the *sinnvolle Abweichungen* clause); deterministic in-process rendering; AI-assistance metadata + Quellenangabe (`0010`, `0002`) | P5 |
| 10 | **Versioned output** | Generated artifacts land in a separate output dir, new version file per round (non-destructive); edit-diff capture (generated vs. her final) stored per artifact | P5 |
| 11 | **Partner preset** | Persona (Deutsch, Gymnasium, SH, precise-warm-colleague register — N7), system-prompt sections (Operatoren data, house-format rules, the "cite her files / say you don't know" contract), one skill `arbeitsblatt-deutsch-sh`, **exemplar retrieval** (2–3 of her own prior artifacts of the target type injected as few-shot style exemplars, `notes/01`) | P6 |
| 12 | **Smoke checklist** | ~12 numbered steps (start → ingest → correct → ask → produce → revise → verify) runnable by jan in ~10 minutes | P7 |

Deliberately absent: catalog screen v2 (custom node), near-duplicate
clustering, suggested-structure report, OCR, images, Musik, web research,
locale pack, preferences doc beyond a stub. (The preferences doc gets a *stub*
— an editable Markdown file in the output dir that the agent may read — so Q8's
pattern is live at zero cost; it is not a feature yet.)

## 3. Build order and why

1. **P0 first (it's the project):** the drive slice, the week of prompts, the
   two exemplar documents, the backup answer, the OpenRouter key. *Nothing
   downstream is a fact until this lands.*
2. **P1 spike** (the go/no-go gate) — before any pipeline code, because a
   bundle that fights the loader changes everything downstream.
3. **P2 ingest, filename pre-pass first.** The pre-pass has zero dependency
   risk, produces immediate visible evidence on her real files, and feeds
   everything after it. Then extraction on the actual file mix (the census
   from P0 says which extractors matter).
4. **P3 catalog** — classification, dedupe, two-plane store, the agent-rendered
   surface. This is where the first "wow" becomes real (the duplicate report).
5. **P4 retrieval tools** — thin now that the catalog exists; the fixture
   (her real "what do I have on X?" questions) is written during P0, not
   invented here.
6. **P5 templates** — against the real exemplars from P0; the round-trip test
   runs on a file *she* has touched in Word.
7. **P6 partner** — the preset + skill + exemplar retrieval; then the
   two-week window opens.
8. **P7 checklist** as each piece lands (the checklist grows with the build,
   not after it).

The order is dependency-honest: each step's output is the next step's input
(exemplars → templates; catalog → retrieval; retrieval → partner). The only
parallelizable pair is P2's extractor work and P5's template-experiment
(same dependency set: the real files).

## 4. The route probe (before routes are "settled")

A half-day experiment that de-risks Q1 *and* B5 together:

- Take P0's real Arbeitsblatt request + her exemplar.
- Run it through **(a)** the local Qwen3.8-27B Q4 (the agent's own family —
  the live question jan posed: does the German idiom hold on a 27B?) and
  **(b)** the cloud route (Q1's Qwen3.8-max-class candidate).
- Judge both against the exemplar on four axes: house-format fit, Operatoren
  usage, *her* voice, and factual/idiom errors.
- Record the result as a decision input: generation route, classification
  route, and (if (a) surprises) the ceiling of the local model.

The probe costs a morning and answers "which weights do we trust with her
words" before the P6 gate depends on the answer.

## 5. Definition of done — the first build

jan ticks these off, in order:

- [ ] P0: slice + prompts + exemplars + backup answer + key, all in this repo's
      out-of-tree notes location (not the corpus, not the drive)
- [ ] P1 gate: hello bundle boots, hello tool callable, OpenRouter answers
- [ ] Her drive cut ingests unattended; the feed is visible; the gap list is
      honest; corpus audit shows zero writes
- [ ] Every catalog row has summary + metadata + confidence + evidence;
      corrections via chat survive a forced full rebuild (tested)
- [ ] Duplicate clusters surfaced; the "1,240 → ~700" number is real
- [ ] ≥ 80% top-5 on her "what do I have on X?" fixture; empty case says so
- [ ] One Arbeitsblatt + one Erwartungshorizont reproduced in her house format
      from the exemplars; round-trip on her Word-touched file intact
- [ ] The route probe is recorded (local 27B vs. cloud, four axes)
- [ ] Smoke checklist exists and jan has run it once end-to-end
- [ ] **Handoff:** she starts it, points at the drive, corrects one row, asks
      one "do I have" question, and gets one worksheet — with jan in the room
      only as observer

Then the two-week window (`20` §3) starts, and the plan's next input is her
behavior, not anyone's argument.

## 6. What the first build is NOT (so scope stays out)

- Not a search product. No search UI she browses; the catalog is the surface,
  the questions are the interface.
- Not a file organizer. No moves, renames, deletes — the map, not the surgery.
- Not a general assistant. No web, no images, no Musik, no second subject.
- Not a platform. One user, one machine, one pinned version.
- Not "the app." It is the *smallest thing that tests the thesis* — the
  product definition is allowed to redraw itself around whatever her two weeks
  show (that is the point; the plan is downstream of use).
