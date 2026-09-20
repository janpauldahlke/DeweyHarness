# 0008 — Document catalog: classify-summarize-dedupe at ingest, retrieve as a tool

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan's design, which is better than the one implied by `0003` alone: "we need like a document catalog, where the LLM classifies and tags and summarizes the documents, then we index and if she asks the LLM can use this as tool to dig." Plus: "Pauline's corpus is quite large btw and also a bit of initial filtering would help to remove the many duplicates she has, but this process only needs to be run initially and then later when she drops in a new document."

Three facts this rests on:

- Her corpus is large and contains many near-duplicates (same sheet saved under three names, last-year's version beside this-year's, `Arbeitsblatt_final_v2_NEU.docx`).
- The expensive LLM work (classify, tag, summarize) is a **batch** job at ingest, not a per-query cost.
- The agent needs to *dig* on demand — including mid-discussion (`0007`), where there may be no artifact at all.

This refines `0003`, it does not replace it: FTS5 remains the retrieval primitive; the catalog is what makes retrieval precise and cheap.

## Decision

Ingestion is a pipeline of stages, each with its own failure mode surfaced per file:

```
discover -> extract -> normalize -> chunk -> classify/tag/summarize -> dedupe -> index
```

| Stage | Does | Notes |
|---|---|---|
| discover | walk corpus folders, detect new/changed/removed files by content hash + mtime | incremental: only touched files re-run |
| extract | text from docx/odt/pdf/md/html | extraction stack still open (Q3/Q4); scanned PDFs need OCR or are excluded with a reason |
| normalize | strip control chars, unify unicode (NFC), record language | German compound handling lives in the tokenizer, not here |
| chunk | structural chunks: headings, Aufgaben boundaries, table boundaries | content-hash identity per `0004` |
| classify/tag/summarize | **LLM pass**: Fach, Land, Jahrgangsstufe, Thema, Textsorte, Kompetenzbereich, Operatoren present, plus a 1-3 sentence summary | structured output validated against the SH vocabulary from `0005`; invalid values are rejected, not coerced |
| dedupe | exact duplicates (content hash) + near-duplicates (minhash/simhash over normalized text, or embedding similarity if ever available) | never deletes anything — marks a duplicate *cluster* and picks a representative |
| index | write FTS5 + metadata facets + catalog rows | derived plane per `0004` |

**The catalog** is the resulting queryable surface: one row per document with its metadata, summary, chunk count, duplicate cluster, and ingest status. It is what the agent's `dig` tool reads first.

**Retrieval as a tool, in two steps, not one:**

1. `catalog_find` — search the catalog: metadata facets + BM25 over summaries + title. Returns document-level candidates with summaries. Cheap, no chunk text.
2. `corpus_read` — given a document id (or chunk ids), return the actual text chunks. Expensive in tokens, so it is a separate deliberate call.

Rationale for splitting: top-k chunk injection (`0003`'s naive form) is what jan rightly worries about — "idk if this context inject of top k findings helps or confuses an agent". Dumping 10 semi-relevant chunks into every request is how agents get derailed. Document-first retrieval lets the agent *reason about which documents matter* using summaries, then read only what it needs. This is progressive disclosure, the same pattern dsh uses for skills (catalog of names+descriptions first, bodies on invocation) and for ToolSearch.

**Duplicates are never deleted.** The corpus is read-only (`0004`). Dedupe marks clusters and surfaces them ("12 Dateien sind Duplikate von 4 Originalen — anzeigen?"). She decides. Silent deletion of her files is the single fastest way to lose her trust in the tool.

## Alternatives considered

- **Chunk-level top-k injection only** (classic RAG). Rejected: it is the pattern jan's instinct correctly distrusts. Fragments lose document context — a chunk from a Klassenarbeit without knowing it *is* a Klassenarbeit produces nonsense grounding. Kept as the second step (`corpus_read`), not the first.
- **One mega-tool that does everything.** Rejected: no progressive disclosure, no way for the agent to plan, and the model sees 10 chunks whether it wanted them or not.
- **Classify at query time instead of ingest.** Rejected: it makes every query pay the LLM classification cost, and it cannot dedupe without seeing everything at once.
- **Dedupe by deleting.** Rejected above; also, near-duplicate detection has false positives, and a false positive that deletes is unrecoverable.

## Consequences

- Ingest is a long batch job over a large corpus → `ctx.jobs` (background work) with per-file progress, cancellable, resumable. A first run over thousands of files must survive being interrupted. This is a real engineering requirement, not a nice-to-have.
- The classify pass has a **cost**: one LLM call per document (or per batch). On a large corpus with a cloud model that is real money and real time. Mitigations: batch documents per call, cache by content hash (a re-index of an unchanged file must not re-classify), and allow the local model to do classification while the cloud model does generation — classification is a much easier task than grounded German pedagogy, so this is where a 27B model is genuinely adequate. This feeds Q1.
- Structured-output validation is mandatory: the classifier must emit values from the controlled vocabulary (`0005`), and an invalid enum is a rejected row, not a coerced one. Fail loud, per dsh invariants.
- Near-duplicate detection quality determines whether the catalog is trustworthy. Start with exact-hash + normalized-text similarity (cheap, explainable); only reach for embeddings if measurement says lexical dedupe misses real duplicates. Consistent with `0003`.
- The catalog itself is derived data (`0004`): rebuildable, never authoritative over her files, and her corrections live in the overrides plane.
