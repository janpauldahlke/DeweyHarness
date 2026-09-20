# 0004 — Index is derived and rebuildable; user corrections live in a layer above it

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

Metadata is what makes retrieval precise for this corpus: Jahrgangsstufe, Thema, Textsorte, Kompetenzbereich, Operatoren, Land, Fach. `00-charter.md` requires it be **inferred**, because Pauline will not tag files — any design whose correctness depends on her maintaining metadata is a design that fails on day three.

But inferred metadata is sometimes wrong, and she must be able to correct it. The naive implementation stores corrections in the same table as inferences, and the next reindex silently destroys them. That is the failure this decision exists to prevent.

This is the same principle jan already holds in eris: "the vault is plain text you own; the semantic index is derived data that can always be rebuilt from it." Applied here, it also becomes the agent-hygiene mechanism: a derived artifact is safe for an agent to regenerate, and regenerating it is how you repair drift without archaeology.

## Decision

Two planes, never merged:

**Derived plane** — the index. Contents: extracted text, chunks, content hashes, FTS5 tables, inferred metadata with a confidence and the evidence for each inference. Property: *deletable at any time and fully reconstructible from the source corpus plus the overrides plane, with no user input.*

**Overrides plane** — corrections. Contents: her edits to inferred fields, plus explicit facts she states ("this is Klasse 9, not 7"). Property: *never rewritten by the indexer, survives a full reindex, and wins over any inference.*

Effective metadata = `override ?? inference`. The indexer writes only the derived plane and never touches overrides, including when an override's target file disappears (a dangling override is surfaced as a warning, not deleted — she may have moved the file, and silently dropping her input is the same class of bug).

Chunk identity is a **content hash**, not an ordinal position, so provenance references from `0002` survive re-chunking and reordering.

Source files are **read-only**. Dewey writes generated artifacts to a separate output location, never into the corpus. Rationale: her corpus is her professional record; an agent that can modify it can corrupt the one thing that is not rebuildable.

## Alternatives considered

- **Single table with an `isUserEdited` flag.** Simpler, and works until the first reindex overwrites a row because a code path forgot the flag. Rejected: the invariant is enforced by a convention instead of by structure, and conventions lose to agents working at 2 a.m.
- **Corrections written back into the documents** (frontmatter in `.md`, custom properties in `.docx`). Attractive — corrections travel with the file. Rejected for v1: mutating her source files contradicts the read-only invariant above, and `.docx`/`.pdf` custom properties are a fragile write path. Revisit for a Markdown-only corpus later.
- **No overrides; re-infer everything each time, better.** Rejected: model inference is not idempotent enough for a field she has already told us the answer to, and re-asking her is the exact interaction cost that makes tagging schemes die.

## Consequences

- Full rebuild is a supported, tested operation and the recommended repair for any "the index seems wrong" report. This is worth more than it looks: it converts an unbounded class of state-drift bugs into "press the button".
- Storage layout must keep the two planes separable — different files, or different tables with a hard write-boundary in code. Enforced by the indexer never holding a write handle to the overrides store.
- Dangling overrides need a surfaced warning path, which means a small "index health" view in the UI.
- Confidence and evidence per inferred field are stored, not discarded. Cost: schema complexity. Benefit: the UI can say *why* it thinks a file is Klasse 7, which is what makes her trust it enough to correct it instead of abandoning the tool.
- Content-hash chunk identity makes near-duplicate detection ("do I already have this sheet?") nearly free, which is one of her real queries.
