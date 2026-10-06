# 10 — Curation first: the Archive on a wild drive

Status: agent draft, 2026-10-06. Deep dive on the overnight reframe: what
"categorize, clean up, and curate an index" concretely means when the corpus is
a desktop-mess, and the trust model that makes it safe for a non-engineer.
`01-product-plan.md` §5.1 states the claim; this doc is the substance.

## 1. The input reality

The corpus is not "her Deutsch folder." It is a **wild school drive**:
everything she ever saved from a decade-plus of desktop accumulation, with
duplicated subfolders in folders, the same sheet under three names, last
year's version next to this year's. Consequences that constrain the design:

1. **The mess is the input, not a precondition.** The draft's pipeline assumed a
   single corpus folder that was *already organized*. Here, ingestion must walk
   the mess as-is. Nothing in the pipeline may require folder semantics to be
   correct — a wrong folder name must degrade a classification hint to noise,
   never crash or poison the index.
2. **Folder structure is a weak, noisy signal.** In a messy drive, a folder
   named "Balladen" is useful; a folder named "Kopien" or "Neues Ordner (3)" is
   useless; a folder named "Klasse 7" containing a Klasse 8 sheet is actively
   misleading. So: **folder path is one inference input among several, with
   explicitly low weight, and content always wins over location.** Her
   corrections (overrides plane) beat everything, including content.
3. **Filenames are a strong, structured signal.** German teacher naming
   conventions are regular and information-dense:
   `Arbeitsblatt_Balladen_Kl7_final_v2_NEU.docx`, `Klassenarbeit Deutsch 2023
   EF.docx`, `Effi Briest Erwartungshorizont SH.docx`. A cheap deterministic
   pre-pass (no LLM) should parse these for: artifact type
   (Arbeitsblatt/Klassenarbeit/Erwartungshorizont/Aufgaben), Jahrgang markers
   (Kl7, EF, Q2, "Klasse 8"), year stamps, version markers (final, v2, NEU,
   Kopie), and proper nouns. This is free, explainable, and it is the *best
   evidence* for the LLM classifier, which sees filename + folder + content +
   the parsed hints and is asked to fill gaps — not to re-derive what the
   filename already said. (This also makes the inference *auditable*: "I think
   this is Klasse 7 because the filename says Kl7 and the text mentions
   *achtjährige*…")
4. **Unreadable files are a first-class category, not an error.** A decade of
   accumulation contains: scanned PDFs (photos of printed handouts), legacy
   formats (`.doc`, `.rtf`, `.pages`, `.numbers`, `.eps`), corrupt files,
   empty files, 200 MB exam-archiv zips. Each must land in the catalog with a
   status: `readable | partial | unreadable:<reason> | skipped:<reason>` — with
   German display text — **and be visible in the catalog screen**, because the
   moment she sees "37 files could not be read" *as an honest list she can
   ignore*, it becomes trust. The moment it is silent, it becomes "it missed
   stuff" and the trust engine breaks. (Principle 3: show its reasoning,
   including its limits.)
5. **Volume and duplicates are the scale story.** "Quite large" (draft Q4) plus
   "many near-duplicates" means: the first run is a long batch job over
   thousands of files with a meaningful fraction that are redundant. Design for
   it: incremental discovery by content hash + mtime, resumable/cancellable
   background job (`ctx.jobs`), per-file progress, and **classification caching
   by content hash** so a re-run of an unchanged file is free.

## 2. Job 1 — Understand every file

The pipeline is the draft's `0008` chain, kept, with the curation-specific
additions:

```
discover → extract → normalize → chunk → [filename pre-pass] → classify/tag/summarize → dedupe-mark → index
```

- **discover** walks the drive root (or roots, per A1). Records content hash,
  mtime, size, path. Incremental: only touched files re-run. *New:* records the
  folder-path hint per file for the classifier.
- **extract** — docx/odt/pdf/md/html in the thin slice; `.doc`/`.rtf`/scans
  deferred or excluded-with-reason pending Q4. Every failure is a typed code
  (`CORPUS_FILE_UNREADABLE` with a structured `reason`), never a crash, never
  silent.
- **normalize** — NFC, control chars, language detection (German expected;
  mixed expected: a folder of English literature texts is normal for a Deutsch
  teacher).
- **chunk** — structural: headings, Aufgaben boundaries, table boundaries.
  Content-hash identity (`0004`), unchanged.
- **filename pre-pass (new, deterministic, no LLM)** — §1.3. Emits structured
  hints per file. Fails silently to "no hints" (it is a hint source, not a
  requirement).
- **classify/tag/summarize** — LLM pass over content + filename hints + folder
  hint. Output: Fach, Land, Jahrgangsstufe, Thema, Textsorte, Kompetenzbereich,
  Operatoren present, 1–3 sentence summary — **validated against the vendored
  SH vocabulary** (invalid values rejected, not coerced, per `0005`/`0008`),
  **each field with a confidence and a one-line evidence string** ("Kl7 in
  filename; text addresses *achtjährige*; no counter-evidence"). The evidence
  string is what the catalog screen shows. Cheap route does this pass (Q1:
  local 27B or company endpoint — classification is an easy task; the expensive
  route is spent on generation, never on cataloguing).
- **dedupe-mark** — see §3.
- **index** — FTS5 + facets + catalog rows, derived plane (`0004`).

**Cost reality, stated:** one (or batched) LLM call per *distinct* document.
On a messy drive with a 30% duplicate rate, dedupe-aware batching (classify the
cluster representative once, propagate to exact twins) roughly divides the
cost by the redundancy factor. On a few-thousand-file drive that is still a
real batch job — an evening of compute, or an hour of cloud tokens. That is
fine *only if it is a visible, resumable job* (§1, consequence 5) and *only if
its result is worth the wait* (§5, first-run experience). First-run scope cut:
if the whole drive is too big for a fast first impression, the thin slice ingests
**one subject folder or one term's worth first** and the rest follows — the
winning demo is fast, the whole-corpus catalog is the long tail
(`01-product-plan.md` §8; `50-first-build.md`).

## 3. Job 2 — Tell the truth about what is there

The Archive must answer four question classes, each of which is a real,
frequent teacher question:

**Q-class A: "What do I have on X?"** (Balladen, Effi Briest, a Textsorte, a
year.) Answered by catalog: metadata facets + BM25 over summaries + title.
This is `catalog_find`'s job and the honest-on-empty test case lives here.
Note the reframe's bonus: on a messy drive this answer is *better* than what
she can produce herself, because she cannot enumerate her own duplicates and
stale versions — "do I already have a sheet on this?" is a near-duplicate
identity question, which exactly-hash + near-duplicate clusters answer directly
(`0003`'s observation, now the Archive's core query).

**Q-class B: "Which is the current one?"** (version families.) The dedupe
mechanism must distinguish the taxonomy of teacher duplicates:

| Kind | Example | Detection | Action |
|---|---|---|---|
| **Exact duplicate** | `AB_Balladen.docx` and `AB_Balladen (1).docx`, byte-identical | content hash | cluster; pick a representative (newest mtime, or the one whose path matches the cluster's dominant folder); mark the rest |
| **Near-duplicate / version family** | `_final`, `_v2`, `_NEU`, "Kopie von", re-saved with a typo fix | normalized-text similarity (minhash/simhash, cheap, explainable) | cluster; **order the family** (by mtime, and by version markers in the name); flag "looks like 3 versions of the same sheet" |
| **Stale version** | 2019 Klassenarbeit next to the 2025 one | year markers in name + content dates | not a duplicate — a *sibling*. The catalog shows both, with the year visible, and the duplicate report may suggest "this is probably superseded" — **suggestion only** (Principle 4) |
| **Legitimate twins** | a sheet she actually uses in two classes | undetectable automatically | this is why *nothing* is deleted or merged; the cluster is surfaced, she disposes |

**Dedupe never deletes. Never merges. Only marks and surfaces.** ("12 Dateien
sind Duplikate von 4 Originalen — anzeigen?") The cluster carries a suggested
representative and a suggested order. If the measurement later says lexical
similarity misses real duplicates, embeddings arrive behind the same seam
(`0003`) — not before.

**Q-class C: "Is the index right?"** The catalog screen is the answer:
per-file row (name, path, summary, key metadata with confidence, evidence,
status), filters (Fach, Jahrgang, Textsorte, status, cluster), the duplicate
clusters as their own view, and the overrides she has made. One click
corrects a field; the correction lands in the overrides plane and survives a
rebuild (`0004`). Dangling overrides (file moved/removed) are surfaced as
warnings, not deleted — her input is never silently dropped (`0004`).

**Q-class D: "What is missing / broken?"** An index-health surface: unreadable
files, skipped files, low-confidence rows, clusters awaiting her decision,
dangling overrides. This is the "I dropped X" honesty made visible — the
Archive tells her what it *couldn't* do, which is the opposite of what a black
box does, and exactly what makes it trustworthy.

## 4. Job 3 — Stay trustworthy: the read-only invariant meets "clean up the drive"

This is the one real design tension the reframe creates, and it deserves
treatment instead of a footnote.

**The tension.** `0004` (kept) says: source files are read-only; Dewey writes
generated artifacts to a separate output location, never into the corpus; the
corpus is her professional record and an agent that can modify it can corrupt
the one thing that is not rebuildable. The reframe says: "clean up the
drive/workspace" is the long-run goal. An agent that cannot move, rename, or
delete **cannot clean up**. Both are true.

**The resolution: curation in v1 is the map, not the surgery.**

1. **V1 delivers the map.** The catalog (what every file is), the duplicate
   report (what is redundant, ordered by family), the health report (what is
   unreadable/stale/unclassified), and — the piece that makes it a *cleanup*
   tool rather than a *search* tool — a **suggested-structure report**: a
   proposed target organization of her drive ("these 340 files would become 12
   folders: Deutsch/Kl7/Lesen/Balladen, …"), rendered as a **readable document
   she can work through**, with each proposed group showing its members and the
   evidence for the grouping. She can execute it with Finder, over a weekend,
   with zero risk to her files. That is real cleanup value, delivered entirely
   within the read-only invariant.
2. **Why not let Dewey do the moving in v1?** (a) File moves are the single
   highest-trust-break action available: one wrong cluster merge or one
   mis-classified batch and her professional record is scrambled — and
   "undo" on 400 moved files is not the same as undo on one. (b) A messy drive
   implies *no reliable backup assumption*; we have not verified she has one
   (A2-adjacent, Q in `40-open-questions.md`). (c) The map-first order costs
   nothing the surgery would, and it produces the exact artifact (her approved
   structure) that a future move-capability needs to be safe. (d) It keeps the
   v1 surface area small (Principle 7) — the move capability wants its own
   journal, undo, and consent UX, which is a feature, not a flag.
3. **The move capability is designed-for, not designed.** The trigger is
   explicit: *she asks, twice, for Dewey to do the move.* When it arrives:
   every operation is journaled (an append-only, human-readable log of
   from→to), batch-consented (she approves a group, not a file), undoable
   (one action restores the whole batch), and the journal is itself a corpus
   artifact she can read. It arrives as its own decision record with its own
   threat model — exactly the `0009` discipline applied to files. Until then,
   the invariant holds untouched.

So the reframe's "clean up" is satisfied in v1 as: **understanding the mess
completely, naming it honestly, and handing her a plan she can execute.**
The actual surgery is the first candidate for the post-v1 consented-write
surface.

## 5. The first-run experience (Opening B, made concrete)

What she sees in the first ten minutes after pointing Dewey at the drive:

1. **The job starts and is visible.** "Ich lese deine Dateien…" with a running
   per-file count and a live feed of what it is understanding ("*Arbeitsblatt
   Balladen Kl7* — gelesen, Arbeitsblatt, Klasse 7"). A non-engineer's tolerance
   for waiting is proportional to visible progress; the feed *is* the trust
   build. (The thin slice keeps the first corpus cut small enough that this
   feed finishes while she watches — `50-first-build.md`.)
2. **The catalog appears.** Her files, each with a plain German summary, its
   metadata, and confidence. The nudge pattern: rows with low confidence are
   visually marked and phrased as a question — "*Klasse 7?* (Filename sagt
   Kl7, Text nennt 8. Klasse)" — with the correction one click away.
3. **The duplicates surface as the first "wow."** For a desktop messi, "your
   1,240 files are really ~700 documents, here are the 540 duplicates, here's
   which of each family is the newest" is the moment the tool proves it
   *understands her mess better than she does*. This is the Archive's headline
   feature in the demo, ahead of any AI conversation.
4. **The honest gaps.** "37 Dateien konnte ich nicht lesen (12 gescannte PDFs,
   9 alte .doc, …)" — listed, clickable, ignorable.

Then, and only then, the conversation: "Frag mich, was du bei mir findest" —
and the Partner (discuss mode) answers from the catalog. The order matters:
**curation is demonstrated by her files, partnership is demonstrated by her
questions.**

## 6. Data shape (enough to argue about, not code)

Three stores, keeping the `0004` plane separation physical:

- **Derived store** (rebuildable, the indexer's only writable target):
  per-file rows (path, hash, mtime, status, extraction result, folder hint,
  filename hints), chunk rows (content-hash id, text, structural position),
  FTS5 tables, classification rows (field → value + confidence + evidence),
  cluster rows (members, representative, family order, similarity basis),
  job log (per-run, per-file outcomes).
- **Overrides store** (her plane; the indexer never holds a write handle):
  per-field corrections (file → field → value + note + date), plus explicit
  facts ("this is Klasse 9, not 7"). Effective metadata = override ?? inference.
- **Output store** (Dewey's writable area, separate from the corpus entirely):
  generated artifacts, versioned non-destructively (`0007`), plus the
  suggested-structure reports and the edit-diff records (`notes/02`).

Storage medium: SQLite (WAL, `node:sqlite`) for the derived store is the
obvious choice — the draft already notes `dsh-storage-sqlite` as the right
medium for index metadata, and FTS5 is proven in-tree. The override and output
stores can share the same file with hard write-boundary code, or be separate
files; either satisfies the invariant. Open until Q4's real corpus exists
(`40-open-questions.md`).

## 7. Acceptance bars for the Archive (concrete)

- **J1 Understand:** a full ingest of the thin-slice corpus runs to completion
  unattended; ≥ 95% of files land in a terminal state (readable or
  honestly-unreadable-with-reason); every classified row is vocab-validated;
  every row carries per-field confidence + evidence; incremental re-run touches
  only changed files.
- **J2 Tell the truth:** on a fixed set of ≥ 15 of *her* real "what do I have
  on X?" questions, the correct document(s) are in the top-5 catalog results
  for ≥ 80%; the duplicate report's clusters are spot-checked by her and she
  finds the error rate acceptable (her judgment, recorded, not our rubric);
  an empty-corpus question gets an honest "nothing there" answer.
- **J3 Stay trustworthy:** a forced full rebuild preserves every override
  (verified by test, not by hope); zero corpus writes in a full ingest (the
  corpus is mounted read-only and an audit check confirms it); the catalog
  screen renders the evidence for every row it shows.

These are the Archive's *internal* bars. The *product* bar is §6 of
`01-product-plan.md` (Stewardship: she corrects, leaves it corrected, asks
unprompted questions).

## 8. What the Archive is NOT (anti-goals, so they stay out)

- **Not a search engine she browses for its own sake.** P1 is not "she wants
  search"; it is "she wants the mess to stop being a mess." The catalog exists
  to make the *corpus* usable (by her and by the Partner), not to be a product
  surface people live in. (If she starts living in it, that is the Stewardship
  metric working, and we take the compliment.)
- **Not a file manager.** No rename, move, delete in v1 (§4).
- **Not a metadata entry form.** She does not tag files. Inferences are free;
  corrections are one click. Any design whose correctness depends on her
  maintaining metadata fails on day three (`0004`'s rule, kept).
- **Not a sync/backup service.** Her files stay where she keeps them (local
  folder or Schulcloud); Dewey reads the folder, it does not own it.
- **Not general RAG.** One corpus shape, one user, one machine (charter
  non-goal, kept).
