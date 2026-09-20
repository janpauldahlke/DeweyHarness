# 0003 — Lexical retrieval first; embeddings, if ever, private to our plugin

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

The instinct on "RAG system" is vectors. Three observations push back:

1. **Query shape.** Her queries are proper-noun and metadata heavy: *Balladen*, *Effi Briest*, *Klasse 7*, *Klassenarbeit*, a textbook chapter, a Textsorte. Lexical matching plus facets does this better than dense retrieval, and "do I already have a sheet on this?" is a near-duplicate identity question, not a semantic-similarity one.
2. **Dependency cost.** dsh has no embeddings seam in `ctx.llm` (verified). Adding vectors means either an upstream core change — which `0001` forbids for v1 — or an embedding provider dependency inside our plugin. On the current model-routing question (`30-open-questions` Q1) that provider may be a cloud API, which makes retrieval a network dependency in the hot path of every generation.
3. **Precedent that already works in-tree.** `packages/session-query/session-query-sqlite/src/schema.ts` builds `fts5` virtual tables and uses `bm25()` and `highlight()`, including snippet positioning across diacritics and punctuation. German umlauts and ß are already a solved problem there. The technique and the driver (`node:sqlite`) are proven in this exact codebase.

## Decision

v1 retrieval is **SQLite FTS5 + BM25 over a metadata-faceted document index**, in our own database file, exposed as a `ctx.tools` tool.

If dense retrieval is added later, it is a **second retriever behind the same service seam**, and the embedding provider stays a private dependency of our plugin — never a core `ctx.llm` change. Results from both retrievers merge behind one tool contract, so the model-facing surface does not change when retrieval does.

## Alternatives considered

- **Vectors first (Qdrant, as in eris).** Rejected: adds a running service on Pauline's machine, an embedding provider, and a network dependency, to solve a problem her query distribution does not primarily have. eris needed it for fuzzy self-memory recall; this corpus needs precise topical identity.
- **Add an embeddings seam to dsh core and upstream it.** Attractive long-term — it is a real gap. Rejected for v1 per `0001`. Revisit when we have measured that lexical retrieval fails on a real query class.
- **External MCP retrieval server** (Memorix, Engram, etc.). dsh documents the wiring, and it costs us almost nothing. Rejected as the primary path: these are agent-memory systems (entity/relation/observation graphs), not document corpora, and the one dsh documents as semantic-capable still ships substring matching by default. We would end up owning the interesting part anyway, with an extra process to supervise on a non-engineer's laptop.

## Consequences

- We own the ingestion pipeline entirely: extraction, chunking, normalization, index build, incremental reindex, per-file failure reporting in German. That is the bulk of v1 code.
- German needs explicit handling: compound nouns (*Unterrichtsmaterialiensammlung*), umlaut folding, ß/ss. FTS5 tokenizer choice is a real decision, not a default; evaluate `unicode61` with diacritic folding against a custom tokenizer, and measure on her actual queries.
- Chunk identity must be stable across reindexes because provenance (`0002`) cites it. Content-hash the chunk, not its ordinal position.
- The index must be rebuildable from source files without her input. That is `0004`.
- Retrieval quality is measurable offline against a fixture, which makes the acceptance bar in `00-charter.md` checkable in CI rather than arguable at the dinner table.
