# 20 — Decisions

Append-only decision records. Mirrors the pattern dsh itself uses in `.agents/notes/implemented/`,
which is the only doc hygiene in this space that has demonstrably survived contact with agents.

## Rules

1. **One decision per file.** `NNNN-short-name.md`, numbered in decision order, never reused.
2. **Never edit a superseded decision.** Write a new record that supersedes it and flip only the `Status:` line of the old one to `superseded by NNNN`. The old text stays true about what was believed at the time — that is the point.
3. **Record the rejected options.** A decision without its alternatives is a slogan and cannot be revisited rationally.
4. **Date it.** Dates are how an agent knows which record is current when two disagree.
5. **Write it when the decision is made, not after.** A record reconstructed later documents the outcome, not the reasoning, and is worthless for the next person.
6. **A decision that has not been implemented is still a decision.** Do not delete records for plans that died; mark them `abandoned` with the reason.

## Status vocabulary

`proposed` · `accepted` · `implemented` · `superseded by NNNN` · `abandoned`

## Index

| # | Date | Decision | Status |
|---|---|---|---|
| [0001](0001-out-of-tree-bundle-not-a-fork.md) | 2026-09-20 | Out-of-tree bundle set against pinned dsh, not a fork | proposed |
| [0002](0002-grounded-generation-not-search.md) | 2026-09-20 | Product is grounded generation with provenance, not search | proposed |
| [0003](0003-lexical-first-embeddings-private.md) | 2026-09-20 | Lexical retrieval first; embeddings, if ever, private to our plugin | proposed |
| [0004](0004-index-derived-corrections-layered.md) | 2026-09-20 | Index is derived and rebuildable; user corrections live in a layer above it | proposed |
| [0005](0005-subject-as-configuration-not-code.md) | 2026-09-20 | Subject and Land are configuration, generalized only on the second instance | proposed |
| [0006](0006-adopted-and-rejected-dsh-gates.md) | 2026-09-20 | Which dsh engineering gates we adopt out-of-tree | proposed |
| [0007](0007-iterative-coauthoring-not-one-shot.md) | 2026-09-20 | Iterative co-authoring over a live preview; discuss is a first-class mode | proposed |
| [0008](0008-document-catalog-and-dedupe.md) | 2026-09-20 | LLM document catalog at ingest; two-step retrieval tool; dedupe never deletes | proposed |
| [0009](0009-capability-removal-no-shell.md) | 2026-09-20 | Her preset has no shell; templates over model-authored code | proposed |
| [0010](0010-ai-marking-metadata-not-watermark.md) | 2026-09-20 | AI-use marking: metadata + optional draft footer, no watermark | proposed |
