# 20 — Phases and gates

Status: agent draft, 2026-10-06. Sequencing for one solo developer working with
agents, restructured around the curation-first ordering (`01-product-plan.md`
§2). Replaces the draft battle plan's phase table; keeps its hour ranges,
honesty rules, test strategy, and backlog discipline. Every gate is **evidence,
not effort** — a phase is done when its gate holds, no matter what the calendar
says.

## 0. How to read this

- **A phase is a hypothesis test.** Each gate answers one question and can
  *flip* a decision record. The flip table in §4 is the map of where the plan
  is allowed to change its mind.
- **The thin slice is the MVP.** It runs Phase 0 → 1 → 2 → 3 → 4 → 5 → 6 at
  reduced cuts (marked below), then hands her something she can hate for two
  weeks. Full v1 fills the cuts (Phase 7–8 polish, the catalog screen v2, the
  suggested-structure report) only after the thin slice has survived real use.
- **Hours are honest ranges** (reading, debugging, rework included), as in the
  draft. The variance lives in P1 (bundle friction), P2 (extraction on a real
  mess), P5 (round-trip), and — new — **the catalog screen UI** (see §2 P3).
- **Scope freeze** at the end of the thin slice: after that, new ideas go to
  the backlog with a date and a trigger, not into the build.

## 1. The phase table

| # | Phase | Cuts in thin slice | Hours | Gate to pass |
|---|---|---|---|---|
| 0 | **Recon**: her drive, her prompts, her exemplars | — (whole phase is thin) | 6-10 (mostly waiting on her) | A real corpus slice (20-50 files) + one small first-run folder (50-200 files) in hand; 20-30 real requests written down; one real Arbeitsblatt + one real Erwartungshorizont in her house format; backup situation answered; OpenRouter key working |
| 1 | **Harness spike (go/no-go)** | — | 6-12 | One trivial out-of-tree bundle + profile boots via `dsh web`; one hello tool registered and callable; OpenRouter route answering. **If composing is a fight: stop, reopen `0001`.** |
| 2 | **Archive — ingest**: discover → extract → normalize → chunk → FTS5 + metadata rows | formats: md/docx/odt/pdf + honest exclusions; filename pre-pass included | 25-40 | Her real corpus cut indexes end-to-end unattended; incremental re-run touches only changed files; every failure a typed code; corpus mounted read-only and audit-confirmed; unreadable files listed with reasons |
| 3 | **Archive — catalog**: classify/tag/summarize + dedupe-mark + overrides + catalog surface | dedupe: **exact-hash only** (near-dup deferred); catalog surface: **agent-rendered** (table via preview, corrections via chat); health view: Markdown report | 25-45 | Vocab-validated metadata on real files, every field with confidence + evidence; exact-dup clusters surfaced, nothing deleted; **one-click** corrections land in the overrides plane and survive a forced full rebuild; a non-engineer can navigate the surface without jan explaining (jan role-plays her until she can) |
| 4 | **Retrieval tools**: `catalog_find` + `corpus_read` with provenance | — | 10-16 | Correct source document in top-5 on ≥ 80% of her real "what do I have on X?" fixture; every citation resolves to a real file; honest-on-empty passes |
| 5 | **`.docx` templates**: house format + AI metadata | one artifact family (Arbeitsblatt + its Erwartungshorizont) | 15-30 | One real Arbeitsblatt and one real Erwartungshorizont reproduced in her house format; **round-trip edit of a Word-touched file without destroying her formatting**; AI-assistance metadata present (`0010`) |
| 6 | **Partner**: preset, persona, Deutsch-SH skills, Operatoren data, **exemplar retrieval** | one skill: `arbeitsblatt-deutsch-sh` | 10-20 | Pauline reads a generated sheet and says it sounds like her material, not ChatGPT; the generation cites 2-3 of her own prior artifacts (exemplars, `notes/01`) and her source files (provenance, `0002`) |
| 7 | **Tests**: retrieval fixture, honesty-on-empty, template snapshots | retrieval fixture + manual smoke checklist only | 10-20 | Fixture runs keyless in CI; empty-corpus case passes; smoke checklist exists (~12 steps) and is run by jan |
| 8 | **Packaging for her Mac**: pinned install, first run, output dir | — | 10-20 | She starts it, points at the drive, and gets a document **without jan in the room** |
| | **Total v1** | | **~115-200** (midpoint ~155) | |

The thin slice total (P0–P6 at cuts) is roughly **70-110 h**; full v1 adds
P7–P8 plus the catalog-surface v2 and the suggested-structure report
(`10-curation-first.md` §4) — the map deliverable that turns curation into
*cleanup*.

**The catalog screen is the biggest new UI surface in the project** — bigger
than the `.docx` layer, and dsh ships no document-catalog UI. Hence the staging:
thin slice = agent-rendered (the agent produces a filterable table as
Markdown/HTML in the preview, corrections happen in chat); full v1 = a custom
`ConversationNodeDefinition` with one-click override affordances (seam #18 in
the upstream contract). Staging is not a compromise: corrections via chat
satisfy the trust invariant (her corrections persist and are visible) — the
click surface is an efficiency gain for a surface she is already using.

## 2. What the thin slice is, precisely

**In:** P0 fully; P1 fully; P2 (md/docx/odt/pdf + honest exclusions, filename
pre-pass); P3 (classification with confidence+evidence, exact-hash dedupe,
agent-rendered catalog + chat corrections, overrides plane, full rebuild); P4;
P5 (one family); P6 (one skill, exemplar retrieval, provenance, honest-on-empty).
jan starts it; she uses the browser (draft rule, kept: the charter's
"terminal is not an interface" binds at P8).

**Deferred from the thin slice:** near-duplicate clustering (exact-hash is
enough to *show* the problem; Q4 tells us what real duplicates look like before
we build the machinery — `0008`); the suggested-structure report (full v1);
the catalog-surface v2 (full v1); OCR/scans (Q4-gated); images in/out (Q7);
Musik; `.pptx`/`.xlsx`; web research; interaction memory beyond the
preferences doc (Q8); German locale (Q2).

**The thin-slice success test** (draft's test, kept, extended to the Archive):
1. **Reach (Partner):** within two weeks of real use, she voluntarily brings a
   task she would otherwise have given ChatGPT. Not "it works" — she reaches.
2. **Stewardship (Archive):** during those two weeks she has corrected the
   catalog at least a few times *and left the corrections in*, or has asked an
   unprompted "what do I have on X?" that the catalog answered. Either one
   proves the Archive is load-bearing for her, not just for the Partner.
3. **Shrink (Partner):** the first generated artifacts' edit-diffs
   (`notes/02`) are measured so the trend can be tracked from artifact #1.

If (1) fails and (2) holds: the product is a *really good archive with a
discussion partner* — valuable, and the generate loop gets redesigned around
what the two weeks showed. If (2) fails and (1) holds: the Archive is plumbing
(we were wrong about its standalone value) — but the Partner still stands on
it, so the build order was right even if the value claim was wrong. If both
fail: the cheapest possible moment to learn the most expensive fact about this
project, exactly as the draft intended.

## 3. The first two weeks of real use (a spec, not a hope)

Between "thin slice in her hands" and "what do we do next," there is a two-week
instrumentation window. It must be *designed*, not improvised:

- jan keeps a **usage log** (her real sessions, what she asked, what she
  edited, what she ignored) — this is the Q5 fixture growing organically into
  the product's own telemetry.
- every generated artifact's **edit diff is captured** automatically (new
  feature, deliberately small: generated file vs. her final version, stored in
  the output store, `notes/02`).
- the catalog's **health report is regenerated weekly** (new unreadables,
  low-confidence rows, clusters awaiting her) so the Archive *asks* her small
  questions over time instead of dumping everything at once.
- end of window: jan + agent review the log against the three proxies
  (`01-product-plan.md` §6) and decide: extend, cut, or flip (see §4).

## 4. The flip table — where this plan is allowed to change its mind

| Hypothesis | Tested at | Failure evidence | The flip |
|---|---|---|---|
| **H-bundle**: out-of-tree composition is pleasant | P1 | The hello tool fights the loader for >a day, or a needed seam doesn't exist | Reopen `0001`; fallback is MCP-server-only retrieval + hand-rolled UI host (recorded, then argued) |
| **H-lexical**: FTS5 + facets is enough retrieval | P4 | <80% top-5 on her fixture *after* query normalization and German-compound fixes | Embeddings behind the existing seam (`0003`). The ordering does not change — the Partner still stands on the Archive; the Archive gets a better engine |
| **H-archive**: curation has standalone value (she uses the catalog without the Partner) | P3 gate + two-week window | Zero stewardship events: no corrections kept, no unprompted questions, no cluster actions after a real invitation | The Archive's *value* claim downgrades to "plumbing the Partner needs" — its *build order* stays (the Partner still requires it); the suggested-structure report and catalog v2 may be dropped to save the schedule |
| **H-grounding**: grounding + house format beats ChatGPT for her tasks | P6 + two-week window | No organic ChatGPT tasks arrive, or her edits to Dewey artifacts are *larger* than her ChatGPT rewrites | Generate loop shrinks; discuss + archive becomes the product; `0002`'s fallback scope becomes the v1 scope by decision record |
| **H-idiom**: vendored spec data + her exemplars → "sounds like her" | P6 gate + Shrink proxy | Generated artifacts are SH-correct but not *hers* (her verdict, not a rubric's) | Strengthen the exemplar channel (more/better exemplars, house-format data), or escalate the generation route (Q1). The validator bar does not move — it is the spec's floor, not the product's ceiling |
| **H-adopt**: she keeps it (the north star) | P8 + ongoing | She stops using it after the honeymoon | Diagnose via the proxies (which of Reach/Shrink/Stewardship is flat); the curation-first ordering means the diagnosis is localizable to one half instead of entangled |

Note the discipline: **no gate's failure flips the ordering itself.** The
ordering is a *means* (build the substrate before the thing that stands on it);
every hypothesis above is a *value or quality* claim about a half. If H-archive
and H-grounding both fail, the product is smaller, not differently ordered.
The ordering would only flip back to Partner-first on evidence that the Archive
is *cheap* (a day to build) *and* *valueless* — which contradicts the observed
corpus reality (the wild drive) and is checked at P0, not P3.

## 5. The test strategy (kept from the draft, unchanged in substance)

- **(a) Deterministic, no model, no key** — ~80% of the test surface:
  extraction, normalization, chunking, filename pre-pass, FTS5 query building,
  vocabulary validation, dedupe clustering, template rendering, error-code
  mapping, provenance resolution, **override-survives-rebuild** (new — the
  two-plane invariant is exactly the class of bug that fails silently).
- **(b) Model-in-the-loop, recorded fixtures** — classification quality,
  retrieval relevance, honesty-on-empty. Assert on properties, not text;
  re-record deliberately.
- **(c) Model↔harness interaction** — not automated in v1. The manual scripted
  smoke checklist (~12 steps: start, point at folder, watch ingest, correct
  one row, ask "what do I have on X", produce one artifact, revise it once,
  verify provenance resolves, verify the diff was captured) is run by jan
  before every version she installs and after every dsh pin bump.
- The executable invariant check: **model-visible ⟺ logged** (kept).

## 6. Do first, in order (the updated "tomorrow list")

1. **Ask Pauline** for: the drive (or the Deutsch cut), a week of her real
   prompts, one real Arbeitsblatt + one real Erwartungshorizont in her house
   format (the idiom exemplars — `notes/01`), and one sentence on whether she
   has a backup of the drive. Everything downstream is guessing until this
   lands. *(P0 — this is the project, not a phase.)*
2. Get the OpenRouter key in; boot `dsh web` unmodified; have one conversation
   (confirms the host runs on the target machine).
3. Write the trivial bundle + profile + hello tool. The gate. If it is a
   fight, stop and reopen `0001`.
4. Only then: extraction libraries against her actual files (P2), starting
   with the filename pre-pass (free signal, zero dependency risk).

Nothing else. Not the catalog screen v2, not the templates, not the memory
design.

## 7. Backlog — good ideas, dated, not built (carried over, extended)

| Date | Idea | Trigger to build |
|---|---|---|
| 2026-10-06 | **Suggested-structure report** (the "map" of `10-curation-first.md` §4) | thin slice in her hands; she has used the catalog a week |
| 2026-10-06 | **Consented file moves** (journaled, undoable, batch-approved) | she asks, twice, for Dewey to do the move |
| 2026-10-06 | Catalog screen v2 (custom node, one-click overrides) | stewardship events exist and the chat-correction friction is visible |
| 2026-09-20 | Web dig for research | trust-labelling condition designed (draft condition kept) |
| 2026-09-20 | Interaction memory / eris semantic brain | measured recall failure on the preferences-doc pattern (Q8) |
| 2026-09-20 | Images in/out; OCR for scans | Q4 scan count + Q1 image-capable routes (Q7) |
| 2026-09-20 | Musik | Deutsch-SH works and she asks |
| 2026-09-20 | `.pptx` / `.xlsx` | `.docx` round-trip proven (Q3) |
| 2026-09-20 | Near-duplicate clustering (minhash/simhash) | Q4 shows what her real duplicates look like; exact-hash alone is visibly insufficient |
| 2026-09-20 | German language pack, upstreamed | she is visibly annoyed by English chrome (Q2) |
| 2026-09-20 | Hamburg / other Länder | the second instance that justifies `0005` extraction |
