# 0002 — Product is grounded generation with provenance, not search

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan's framing: "a RAG system for all her school documents, an archiver indexer, finder ... like a developer harness BUT for teachers".
That framing centers search.

The competing observation: Pauline can already find her files. Her stated pain is the ChatGPT copy-paste loop, whose defect is not that it cannot locate her Balladen sheet — it is that it has never seen it, does not write in Schleswig-Holstein idiom, and cannot be checked. An Erwartungshorizont carries her name; a fabricated Operatoren definition or Lehrplan reference is her professional risk.

## Decision

The product is **grounded generation with verifiable provenance**. Retrieval is a means, not the deliverable.

This makes three things hard requirements rather than features:

1. **Citation that resolves.** Every claim sourced from the corpus names a file, and the reference must resolve to a real chunk of a real file. An unresolvable reference is a test failure, not a UX nit.
2. **Idiom from vendored data, not from model knowledge.** SH *Fachanforderungen* Operatoren (Anhang IV.1), the three Anforderungsbereiche, and the published Erwartungshorizont structure ship as dated data files in the repo. The model is told to use them; the validator checks against them.
3. **Honest absence.** When the corpus has nothing relevant, the system says so instead of generating plausible filler. This is an explicit test case.

Note the source-level subtlety that must survive into the prompt and the validator: the Fachanforderungen state Operatoren are *not* strictly mapped to one Anforderungsbereich — "Alle Operatoren verlangen Leistungen im Anforderungsbereich II und zusätzlich in mindestens einem weiteren." A one-Operator-to-one-Bereich lookup table would be wrong by source.

## Alternatives considered

- **Search-first (archiver/finder).** Much smaller: ingest, index, expose a search tool, done. Genuinely tempting, and if the v1 trace shows generation is unreliable while retrieval is solid, this is the fallback scope. Rejected as the v1 goal because it competes with her existing `Strg+F` plus a filename convention, and it leaves the actual copy-paste pain untouched.
- **Generation-first, retrieval later.** Rejected: ungrounded German pedagogy output is exactly what she already has, and it is the version that produces confidently wrong Erwartungshorizonte — the highest-harm failure mode available to this product.

## Consequences

- Provenance touches every layer: chunk identity must be stable, retrieval results must be loggable (dsh invariant: *model-visible ⟺ logged*, so retrieval reaching a request needs a session event), and the UI must render a source card, not just prose.
- We need an evaluation fixture from day one — a recorded set of her real queries with known-correct source documents. Without it, "the retrieval is good" is unfalsifiable and the project will drift.
- We need a pedagogy validator, not just a generator: a generated Arbeitsblatt must be checkable against the Operatoren list and the Bereich structure. This is more code than a chat wrapper and it is the part that makes the output trustworthy.
- Raises the bar on model quality. A weak model cannot do grounded German pedagogy well, which feeds directly into `30-open-questions` Q1.
