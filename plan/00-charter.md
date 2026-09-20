# 00 — Charter

Status: draft for review. Owner: jan. Peer reviewer: agent.
This file is current-state prose, not a decision log. Decisions live in `20-decisions/`.
Facts about the host harness live in `10-dsh-capability-map.md` and are re-verified, not remembered.

## Problem

Pauline is a Gymnasium teacher in Schleswig-Holstein (subjects: Deutsch, Musik).
Her current tool is the ChatGPT web UI plus copy-paste, used two ways: as a **document factory** (Arbeitsblätter, Aufgaben, Klassenarbeiten, Erwartungshorizonte) and as a **rubber duck** (discussing literature, Aufgabenstellungen, Anforderungen).

What that workflow cannot do, in order of cost to her:

1. **No grounding.** Output ignores her own prior materials, so every result is generic and she rewrites it anyway. In discussion mode it cannot answer "what do I already have on Balladen?" at all.
2. **No idiom.** Output does not follow Schleswig-Holstein *Fachanforderungen*: the *Operatoren* vocabulary, the *Anforderungsbereiche* I/II/III split, the published Erwartungshorizont structure (expected Leistungen + Bewertungskriterien + percentage weighting per task).
3. **No provenance.** She puts her name on an Erwartungshorizont. A hallucinated Lehrplan reference or an invented Operatoren definition is a professional risk for her, not for the model.
4. **No corpus.** Her materials exist on disk in large quantity, with many near-duplicates, searchable only by filename and `Strg+F`.
5. **No document output.** ChatGPT returns a chat bubble; she needs a printable `.docx` in her house format, with images, that she can iterate on round after round.

The product is therefore **a customized chat partner over her corpus** — discuss, dig, produce, revise (`20-decisions/0007`) — **not** a search engine and not a one-shot generator. Retrieval exists to make both conversation and generation grounded, idiomatic, and checkable. Search-only would be a ten-times smaller project; `20-decisions/0002` records why it was rejected.

**Competitive landscape, honestly:** Schleswig-Holstein already gives teachers **AIS.chat** — a state-run, DSGVO-blessed, free chatbot (VIDIS/Schulportal login, Dienstvereinbarung, mandatory AI-competency training). ChatGPT remains the incumbent. Dewey's answer to "why not AIS.chat" must stay sharp: grounding in *her* corpus, *her* house format, *her* documents on *her* disk, artifacts she iterates on and owns as files. None of that is compliance — it is the product.

## User

One named user: Pauline. Private macOS, admin rights. Non-engineer.
Consequences that are requirements, not preferences:

- Terminal is not an interface. Browser UI or desktop app only.
- First run points at one folder. No chunk sizes, no embedding dimensions, no vector DB ports.
- **Agent language is always German** — conversation, documents, questions. Non-negotiable; it is the product.
- **UI chrome may be English in v1** (jan's call; dsh ships `en`). A German language-pack plugin is desired, not blocking. Independent of agent language.
- **Errors are typed codes + i18n, never hardcoded prose.** Every failure is a stable enum (`CORPUS_FILE_UNREADABLE`, `MODEL_ROUTE_MISSING_CREDENTIAL`, …) carrying structured data; display text resolves through an i18n layer (German strings for v1). This is also dsh's own convention (`StorageError` codes, `MISSING_CREDENTIAL`, `SANDBOX_UNAVAILABLE`), so it is the upstream-idiomatic choice, not just clean-code taste. Never a stack trace in her face; never string-matched error handling in our code.
- The installed version is pinned and does not change under her mid-term.

Secondary user: jan, as builder and maintainer. His needs (testability, debuggability) never override hers in a conflict.

## Scope boundary

In scope for v1: **Deutsch, Sekundarstufe, Schleswig-Holstein, one corpus folder, one machine.**

| In | Out | Why |
|---|---|---|
| Ingest her own teaching materials, LLM-classified into a catalog | Ingesting anything by hidden default | Classification, dedupe marking, and exclusion are surfaced per file; nothing is silently dropped or deleted (`20-decisions/0008`) |
| Lexical retrieval (FTS5/BM25) + metadata facets, exposed as a dig tool | Dense/vector retrieval | v1 queries are proper-noun heavy; embeddings add a provider dependency for no measured gain yet (`20-decisions/0003`) |
| Discuss: literature, Anforderungen, lesson design — sessions with no artifact | — | First-class mode, not a failure to produce (`20-decisions/0007`) |
| Corpus stays on her disk (local folders or Schulcloud sync) | Cloud storage of the corpus as a service | We read a folder; where it syncs from is her existing setup, not our feature |
| Send chunks to a model endpoint only along the configured route | Hidden egress | The one real data-flow question is *which endpoint sees corpus text* — that is model routing (Q1), an engineering config, not a compliance project |
| Generate Arbeitsblatt, Aufgaben, Klassenarbeit, Erwartungshorizont, tables | Generate Noten, Gutachten, Elternbriefe | Different artifact class, different risk |
| Iterate on artifacts across rounds with live preview; images in and out | One-shot delivery | The loop is the product (`20-decisions/0007`) |
| `.docx` output via deterministic in-process templates | `.pdf` layout, `.pptx`/`.xlsx` (later), print-ready Satz, model-authored scripts | Output must be editable, house-format-correct by construction, and shell-free (`20-decisions/0009`) |
| AI-assistance metadata in every artifact; optional draft footer | Visible watermarks | Verified against AI Act Art. 50 + IQSH guidance (`20-decisions/0010`) |
| Deutsch | Musik | Music needs symbolic metadata over PDF/MusicXML, not text chunking — a second pipeline |
| Schleswig-Holstein | Hamburg, other Länder | Curriculum and Operatoren are state-owned; generalizing now means inventing an abstraction for one sample |
| Safety by capability removal: no shell, no code execution, corpus read-only | Sandboxed shell + approval dialogs | She cannot meaningfully approve `python3 /tmp/gen.py`; absent capability beats contained capability (`20-decisions/0009`) |

The subject/Länder dimension is **configuration, not code**: a subject is a folder plus a skills pack plus a metadata schema.
Proving that requires exactly two configurations (Deutsch-SH, then Deutsch-HH or Musik-SH) and no more.
Building the abstraction before the second instance exists is speculation; `20-decisions/0005` records the constraint instead.

## Success criterion (the v1 trace)

One end-to-end trace. If it works, there is a product. If any step fails, the failure is the next task.

```
1. Pauline points Dewey at her Deutsch folder (~N files: docx, pdf, md, odt).
2. Index builds unattended (background job, resumable); she sees per-file status
   and machine-readable failure codes with German display text for what failed.
3. She types: "Arbeitsblatt zu Balladen, Klasse 7, mit Erwartungshorizont, in meinem Stil."
4. Dewey digs via catalog -> reads 3-5 of HER OWN sheets, names each one,
   and states what it took from each.
5. Dewey produces a .docx in her house format: Aufgaben with SH Operatoren, an
   Erwartungshorizont structured as Erwartete Leistungen + Bewertungskriterien,
   Anforderungsbereiche marked, Quellenangabe naming her source files,
   AI-assistance metadata set (0010). The file appears as a deliverable and
   renders in the sidebar preview.
6. She reads it in the preview, says "Aufgabe 3 ist zu schwer, und die Tabelle
   braucht eine Spalte für Punkte" -> round two produces a NEW version file,
   preview updates, previous version untouched.
7. She opens the approved version in Word, edits it, prints it. Nothing is re-typed.
```

Steps 4 and 6 are the load-bearing ones. Grounded retrieval is the differentiator over ChatGPT (step 4); the iteration loop over a live artifact is the differentiator over a one-shot generator (step 6). The product is the loop, per `20-decisions/0007`.

Acceptance bar, measurable:

- **Retrieval:** on a fixed query set drawn from her real requests, the correct source document is in the top-5 for ≥ 80% of queries. Measured by a recorded fixture, not by opinion.
- **Grounding:** every factual claim about her corpus carries a file reference that resolves. Zero unresolved references in a passing run.
- **Idiom:** generated Aufgaben use only Operatoren from the SH list; the Erwartungshorizont contains the three Anforderungsbereiche explicitly. Checked against the Fachanforderungen Anhang IV.1 vocabulary.
- **Latency:** first usable response < 60 s on her machine, cloud model.
- **Failure honesty:** when the corpus has nothing relevant, Dewey says so and does not fabricate. This is a test case, not a hope.

## Non-goals

- Not a replacement for her judgment. Output is a draft she signs.
- Not multi-user, not multi-tenant, not a school deployment.
- Not local-first by architecture (unlike eris). Model routing is a configuration axis; see `30-open-questions` Q1.
- Not a fork of deepseek-harness. Out-of-tree bundle plus pinned dependency; see `20-decisions/0001`.
- Not a port of eris memory tiering *for the corpus*. The corpus is a document catalog (`20-decisions/0008`). A growing interaction-memory layer (her preferences, recurring decisions, "what we discussed about Effi Briest") is a *different* thing and is deferred, not rejected — see `30-open-questions` Q8.
- Not a general RAG framework. One corpus shape, one user, one machine.
- Not a compliance product. SH already ships AIS.chat for that. If a school needs state-blessed compliance, it uses AIS.chat; Dewey competes on grounding and artifacts, and says so.

## Constraints

| Constraint | Source | Effect on design |
|---|---|---|
| dsh is alpha, breaking changes guaranteed | dsh `README.md` | Pin exact version; `40-upstream-contract.md` lists every seam used so an upgrade is a checklist |
| dsh has no retrieval, no embeddings, no ingestion | verified, see `10-dsh-capability-map.md` | That is the project. Everything else is composition |
| dsh client locales are `zh`/`en`; `LocaleId` is open to language-pack plugins | `packages/client/locale` | German UI = optional language-pack plugin (Q2), not a v1 blocker; agent output language is independent and always German |
| `dsh-skill-office` needs deployment-supplied interpreters; its script path is model-authored code | dsh skill-office README + `20-decisions/0009` | v1 uses deterministic in-process `.docx` templates instead; skill-office is the escalation path |
| AI Act Art. 50 applies since 2026-08-02 (provider marking duty; deployer text duty is narrow) | verified, C(2026) 5054 guidelines | Metadata marking + optional draft footer, no watermark; `20-decisions/0010` |
| Personal data may sit in her corpus | jan; files stay local or in Schulcloud | Treated as an engineering data-flow question (which endpoint sees text = Q1), not a compliance project. jan is the decision-maker here and has heard the concern once; it is recorded, not relitigated |
| macOS, admin rights, two external GPUs available | jan | Local model inference is technically possible; whether it is *wise* is Q1 |
| No pandoc / LibreOffice on the current dev box | verified | Document pipeline dependencies must be declared and installed, not assumed |

## Naming

`deweyHarness` — after John Dewey (1859-1952): learning as inquiry, experience, and the connection of school material to lived practice, rather than transmission of fixed content.
"Dewey" also collides happily with library classification, which suits an archiver.
The name is not an acronym. Product name for Pauline is TBD and should be German and untechnical; the repo name is not a UI string.

## Open items this charter does not settle

See `30-open-questions.md`. The blocking one is Q1 (model routing), because it fixes the composed routes and the quality/cost bar. Q4 (a real corpus slice) and Q5 (her real queries) are not blocking design but are blocking *implementation* — both are cheap and should happen first.
