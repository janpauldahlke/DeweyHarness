# 50 — Battle plan

Written 2026-09-20, ~02:40, at jan's request after the round-2 review.
Sequencing and effort for one solo developer working with agents.
Effort figures are honest ranges including reading, debugging, and the rework that always happens; they are not optimistic-path numbers.

## The short answers

**Doable?** Yes. Nothing here is research. Extraction, FTS5, structured classification, `.docx` templates, and a Cordis bundle are all solved problems; the work is integration and pedagogy data. The one genuine unknown is Phase 1 — whether composing an out-of-tree dsh bundle is pleasant or a fight. That is why Phase 1 is a gate and not a task.

**Valuable?** Two different values, and they pull apart:
- *For Pauline:* real and immediate, conditional on grounding + house format actually working. Today she retypes everything from a chat bubble. If step 4-6 of the v1 trace works, that is hours of her week back.
- *For jan:* learning Cordis/dsh deeply, a possible product, and a demonstrable "local AI on own infrastructure" story. Longer horizon, and it wants features Pauline does not.
Keep them separate when trading scope. When they conflict, v1 serves Pauline.

**Can we stabilize all the tools?** Not "all" — and that is the point. Stability comes from *removal*, per `0009`. v1 ships **four** model-facing tools (`catalog_find`, `corpus_read`, `document_generate`, `present`) plus the ask tool. Four tools with sharp contracts are more reliable than twelve with clever ones. Every later tool is a decision record.

## Scope freeze

Freeze at the start of Phase 2. After that, new ideas go to the backlog below with a date — not into the build.

v1 is: **one artifact family** (Arbeitsblatt + Erwartungshorizont), **one subject** (Deutsch Sek I, SH), **one corpus folder**, **text only**, **no interaction memory**, **no Musik**, **no Hamburg**.

This is a rule for a specific known failure mode: a solo dev whose user is his partner has no natural "no". Every conversation adds a capability, and surface area — not difficulty — is what kills solo projects. The backlog exists so good ideas are honoured without being built.

## Phases

| # | Phase | Hours | Gate to pass before continuing |
|---|---|---|---|
| 0 | Recon: her corpus slice + her real prompts | 6-10 (mostly waiting on her) | 20-50 real files in hand; 20-30 real requests written down |
| 1 | **Harness spike (go/no-go)** | 6-12 | One trivial out-of-tree bundle + profile boots via `dsh web`; one hello tool registered and callable; OpenRouter route answering |
| 2 | Ingestion: extract → normalize → chunk → FTS5 + metadata store | 25-40 | Her real corpus indexes end-to-end; incremental re-run touches only changed files; every failure is a typed code |
| 3 | Catalog: classify/tag/summarize + dedupe clustering | 15-25 | Vocabulary-validated metadata on real files; duplicate clusters surfaced, nothing deleted |
| 4 | Retrieval tools: `catalog_find` + `corpus_read` with provenance | 10-16 | Correct source document in top-5 on ≥80% of the Phase-0 query fixture |
| 5 | `.docx` templates: house format + AI metadata | 15-30 | One real Arbeitsblatt and one real Erwartungshorizont reproduced; **round-trip edit of a Word-touched file without destroying her formatting** |
| 6 | Preset, persona, Deutsch-SH skills, Operatoren data | 10-20 | Pauline reads a generated sheet and says it sounds like her material, not like ChatGPT |
| 7 | Tests: retrieval fixture, honesty-on-empty, template snapshots | 10-20 | Fixture runs keyless in CI; empty-corpus case passes |
| 8 | Packaging for her Mac: pinned install, first run, output dir | 10-20 | She starts it, points at a folder, and gets a document **without jan in the room** |
| | **Total v1** | **~110-180** (midpoint ~145) | |

At 8-12 focused hours/week that is roughly 3-4 months. At 20 h/week, 6-8 weeks. The variance lives almost entirely in Phases 1, 2, and 5.

## Recommended: the thin slice first (~50-70 h)

Do not build Phases 5 and 8 first. Build the ugly version, put it in her hands, and find out whether grounding is worth anything before investing in templates and packaging.

| Cut | Thin slice | Deferred to |
|---|---|---|
| Formats | `.md` + `.docx` only | `.odt`, `.pdf`, OCR → Phase 2b |
| Dedupe | classify only, no clustering | Phase 3b |
| Output | Markdown file she pastes into Word | Phase 5 |
| Tools | `catalog_find`, `corpus_read`, `present` | Phase 5 |
| Skills | one: `arbeitsblatt-deutsch-sh` | Phase 6 |
| Tests | retrieval fixture + manual smoke checklist | Phase 7 |
| Run | **jan starts it, she uses the browser** | Phase 8 |

The last row does not contradict the charter's "terminal is not an interface": during validation jan is the operator and the browser is her surface. The charter constraint binds at Phase 8, when she must run it alone.

Thin-slice success test, and it is a real test: **she voluntarily brings it a task she would otherwise have given ChatGPT.** Not "it works" — she reaches for it. If that does not happen after two weeks of her real use, the grounding thesis is wrong and no amount of Phase 5 polish fixes it. That is the cheapest possible moment to learn the most expensive fact about this project.

## Deliverables

Two kinds; keep them distinct so "done" is never ambiguous.

**Hers (the product):** a running Dewey on her Mac at a pinned version; her corpus indexed with visible per-file status; generated `.docx` files in her house format, versioned non-destructively in an output folder, carrying AI-assistance metadata (`0010`); a German-speaking partner that cites her own files.

**His (the engineering):** the out-of-tree bundle packages + profile; the vendored SH pedagogy data with source and date; the retrieval fixture; the manual smoke checklist; the pinned upstream contract (`40-upstream-contract.md`); the decision records.

## Test strategy — three layers, honestly

**(a) Deterministic, no model, no key — ~80% of the test surface.** Extraction, normalization, chunking, FTS5 query building, metadata validation against the controlled vocabulary, dedupe clustering, template rendering, error-code mapping, provenance-reference resolution. Fast, keyless, and this is where correctness actually lives. If a bug would embarrass us in front of Pauline, it is almost certainly in this layer and almost certainly testable here.

**(b) Model-in-the-loop, recorded fixtures.** Classification quality, retrieval relevance on her real queries, honesty when the corpus has nothing. Non-deterministic, needs a key, so: assert on **properties, not text** — valid enum values, resolvable citations, top-5 membership, "said it did not know". Re-record deliberately, never silently.

**(c) Model↔harness interaction — the honest answer: do not automate it in v1.** Whether the model calls `catalog_find` when it should, whether `present` lands, whether the sidebar preview reloads, whether compaction dropped the artifact — dsh has snapshot-recorded e2e for exactly this, and it costs a key plus re-recording on every model or upstream change. For one user and one maintainer that is a bad trade.

Replace it with a **manual scripted smoke checklist**: ~12 numbered steps reproducing the v1 trace, run by jan with Pauline before every version she installs, plus after every dsh pin bump. Ten minutes, catches the real breakage, and honest about what agents cannot verify for us.

So: **test the contract, smoke the interaction.** Our tool schemas, prompt sections, provenance format, and error codes are deterministic and get real tests. The emergent behaviour of a model using them gets a checklist and a human.

The one invariant worth wiring as an executable check, because violating it fails silently: *model-visible ⟺ logged*. Retrieval results reaching a request must be reconstructable from the session log. dsh asserts this at runtime; our fixture should too.

## Web research — accepted, with one hard condition

Web dig is cheap scope creep: dsh already ships web search/fetch tools (`packages/web`), so it is a preset toggle, not a build. Fine, with a backlog entry, and one condition that must not be lost:

**Corpus sources and web sources are different trust classes and must never be interchangeable.** Web content is not her material and not vetted. So: every citation is labelled `corpus:` or `web:`; the idiom validator accepts only vendored SH Fachanforderungen data, never a web page; and a web-sourced claim may never silently satisfy a pedagogy requirement in an Erwartungshorizont. Otherwise the tool will eventually hand her a plausible Lehrplan reference it found on the internet — which is precisely the failure `0002` exists to prevent.

eris already does this pattern (allowlist, consent, session budget). Reuse the instinct, not the code.

## Backlog — good ideas, dated, not built

| Date | Idea | Why not now |
|---|---|---|
| 2026-09-20 | Web dig for research | Cheap toggle, but needs the trust-labelling condition above designed first |
| 2026-09-20 | Interaction memory / "semantic brain from eris" | Q8. Start with a plain-Markdown preferences doc; escalate only on measured recall failure |
| 2026-09-20 | Images in and out; OCR for scans | Q7. Blocked on Q4 (how many scans exist) and Q1 (image-capable routes). Needed for Musik |
| 2026-09-20 | Musik | Second extraction pipeline, not a second skills pack (`0005`) |
| 2026-09-20 | `.pptx` / `.xlsx` | After `.docx` round-trip is proven |
| 2026-09-20 | Reorder / restructure existing documents | Depends on the Phase-5 round-trip answer |
| 2026-09-20 | German language pack, upstreamed | Q2. English chrome is acceptable in v1; agent language is German regardless |
| 2026-09-20 | Hamburg / other Länder | The second instance that justifies extracting the `0005` abstraction |

## Cut list for `40-upstream-contract.md`

That file lists 24 seams. Touching 24 upstream surfaces is how an alpha dependency eats a solo schedule. **v1 needs about 8**: `ctx.tools`, `tools/pre-execute`, `ctx.systemPrompt.section()`, `dsh-agent-presets`, `ctx.skills` + `dsh-tool-skill`, `ctx.fs`, `dsh-llm-pi-ai`, bundle/profile manifests. The rest (locale pack, conversation nodes, sandbox, attachment, compaction, jobs) stay documented but unexercised until their phase arrives. Row 21-24 are Phase 5+ concerns.

Rule: **a seam we do not call is a seam that cannot break us.** Re-verify the 8 on every pin bump; the other 16 only when we start using them.

## What to do tomorrow, in order

1. Ask Pauline for the folder and for a week of her real prompts (Phase 0). Everything downstream is guessing until this lands.
2. Get OpenRouter key in, boot `dsh web` unmodified, have one conversation (Phase 1a, ~1 h). Confirms the host runs on the target machine before writing a line of ours.
3. Write the trivial bundle + profile + hello tool (Phase 1b). This is the gate. If it is a fight, stop and reopen `0001`.
4. Only then look at extraction libraries against her actual files (Phase 2).

Nothing else. Not the language pack, not the templates, not the memory design.
