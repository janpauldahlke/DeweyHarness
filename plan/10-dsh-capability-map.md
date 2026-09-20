# 10 — deepseek-harness capability map

Host harness: `/home/hagbard/dev/deepseek-harness`, version `0.1.6-alpha.2`, MIT.
Every row below was read from source or docs in that checkout on 2026-09-20, not recalled.
This file is expensive to reconstruct; keep it factual and update it when dsh is re-pinned.

## What dsh provides and we compose

| Need | Mechanism | Where | Notes |
|---|---|---|---|
| Per-session tool/prompt/skill set | `dsh-agent-presets` (`agent.cordis.yml`) | `packages/preset/agent-presets` | Only an empty session may switch presets. Treat each preset as trusted configuration — it grants the capabilities of the plugins it selects |
| System-prompt persona | `dsh-persona` row | `packages/preset/persona` | The "you teach Deutsch in SH" text lives here, not in a hand-written prompt file |
| Prompt sections | `ctx.systemPrompt.section()` with ordering + scope-local shadowing | `docs/subsystems/system-prompt.md` | How the Operatoren list and the house-format rules reach the model |
| Authoring recipes | `ctx.skills` + `dsh-skill-filesystem` | `packages/skill/*` | kebab-case, `<name>/SKILL.md` or `<name>.md`; ranks: project `.dsh/skills` (100) < `.agents/skills` (200) < custom (300) < user-dsh (400) < user-agents (500) < bundled (600). Nearest layer wins outright |
| Model-facing skill tool | `dsh-tool-skill` | `packages/skill/tool-skill` | Injects a catalog of names+descriptions only; bodies load on invocation. Catalog re-injected on digest change |
| Office authoring | `dsh-skill-office` (`office-docx`, `office-pptx`, `office-xlsx`) | `packages/skill/skill-office` | **Deployment supplies interpreters, authoring libraries, execution tools, file delivery tool.** Bundled Python env by default |
| Office → PDF | `ctx.officeToPdf` (LibreOffice via `@deepseek-ai/libreoffice-kit`) | `packages/document/office-to-pdf` | Preview/QA only. "Creates no model-facing tool or Session event." Not an ingestion path |
| Corpus folder as a project | `ctx.workspaceRegistry` | `packages/workspace/workspace` | Durable project dirs + sessions per dir. Invisible to models |
| Disk access + policy | `ctx.fs`, `dsh-fs-local`, `dsh-fs-sandbox`, `dsh-fs-observation-policy` | `packages/fs/*` | |
| Permission gates | `tools/pre-execute` waterfall returning typed `PreToolDecision` | `docs/cookbook/extension-cookbook.md` | Where ingestion-side classification and write-allowlists get enforced |
| Durable KV / SQLite | `dsh-storage`, `dsh-storage-sqlite` (`node:sqlite`, doc-per-row, WAL) | `packages/storage/*` | **Host-side only: contributes no prompt, tool, or schema.** Not a search backend, but the right medium for index metadata |
| Full-text search | FTS5, proven in-tree | `packages/session-query/session-query-sqlite/src/schema.ts:127` | `CREATE VIRTUAL TABLE ... USING fts5(...)`, `bm25()`, `highlight()`, snippet positioning across diacritics. Exists for session logs; the technique is reusable for a document index |
| Cloud or local models | `dsh-llm-pi-ai` | `packages/llm/llm-pi-ai` | Multi-provider: pi-ai catalogs, OpenAI-compatible gateways, self-hosted servers. Per-request credential resolution via the credentials seam; `apiKeyEnv` is a reference, never a literal. Routes add/override from `settings.yaml`, effective next request, no restart. Dormant with zero routes |
| DeepSeek-direct route | `dsh-llm-deepseek` | `packages/llm/llm-deepseek` | Can be mounted beside pi-ai; route names must not collide |
| Browser UI | `apps/web`, profile `web` | | `pnpm dsh web`; port 3080 |
| Desktop app | `apps/desktop` (Electron), Desktop Host | | Reserved `$DSH_HOME/profiles/desktop`, default port 19387. Separate lockfile from CLI |
| Out-of-tree distribution | bundle (`dsh.bundle` → `cordis.patch.yml`) + profile (`dsh.profile`) + `dsh plugin add` | `docs/user/develop/basic/publish.md` | **This is the supported extension path.** A bundle is authored and distributed; a profile is booted. Nothing is both |
| Layer order | bundles in profile order → profile patch → home patch → `--patch` overlay | `docs/architecture.md` | A patch replaces a row's whole config by id, or inserts rows |
| MCP servers | `dsh-mcp-client`, one plugin per server | `packages/mcp` | Tools appear as `mcp__<server>__<tool>` |
| Background work | `ctx.jobs`, `job_*` tools | `packages/jobs` | Where a long corpus rebuild belongs |
| Scheduled follow-ups | `ctx.schedule` | `packages/schedule` | Re-index on a timer, if ever needed |
| Compaction | `ctx.compaction` + `dsh-compaction-basic` | `packages/compaction` | Relevant once retrieval results are large |
| Sub-agent delegation | `ctx.subagents` providers + `dsh-tool-subagent` | `packages/subagent` | Candidate for an indexer agent separate from the authoring agent |
| Durable session state | extend `SessionEventMap` | `docs/subsystems/session.md` | **Model-visible ⟺ logged.** A new model-visible input requires a session event. Retrieval results shown to the model must be loggable |
| UI nodes | `ConversationNodeDefinition` + keyed renderer | `docs/subsystems/conversation.md` | How a provenance card would render |
| German UI | language-pack plugin over `packages/client/locale` | `LocaleId = string`, `LOCALE_IDS = ['zh','en']` | `BuiltInLocaleId` is closed; `LocaleId` is open and explicitly "accepted from language-pack plugins". Fallback chains must terminate at `en`. Desired, deferred past v1 (Q2); agent output language is independent and always German |
| Artifact preview + iterate loop | `dsh-client-ui-sidebar-documentpreview` + `dsh-tool-present` + `dsh-workspace-changes` | `packages/client/ui-sidebar-documentpreview`, `packages/deliverables/*` | Sidebar renderers: Markdown, code, image, PDF, HTML, Office→PDF (local conversion), plain-text fallback. `present` declares deliverable files (paths+descriptions, no content copy); users open sources in their default app. This is the revise loop of `20-decisions/0007`, mostly prebuilt |
| Command confinement | `dsh-sandbox-local` (Linux/macOS/Windows), `dsh-sandbox-policy`, `dsh-permission-presets` | `packages/sandbox/*`, `packages/interaction/permission-presets` | Fails `SANDBOX_UNAVAILABLE` rather than running unconfinced; reports `full`/`partial` per execution; presets bundle sandbox mode + approval policy (`workspace-write`+`ask`, `danger-full-access`+`never`). Her preset mounts no shell at all (`20-decisions/0009`), so this is defense-in-depth |

## Gaps — everything here is ours to build

| Gap | Evidence | Consequence |
|---|---|---|
| No document ingestion pipeline | no package scans a corpus folder into chunks | Write it |
| No text extraction for indexing | `office-to-pdf` is preview-only and creates no model-facing tool; no `pdf-parse`/OCR in `packages/*` | Choose and own an extraction stack (docx/odt/pdf/md). Unreadable scans are a real category in a teacher's folder |
| No chunking | absent | Write it; must respect German compound words and document structure |
| No embeddings seam in `ctx.llm` | no `embedding` surface in `packages/llm/llm/src` | Either keep embeddings private inside our plugin, or add a core seam. `20-decisions/0003` picks the former |
| No retrieval tool | absent | Write it on `ctx.tools` |
| No provenance/citation mechanism | absent | Write it. This is a charter requirement, not polish |
| No metadata extraction (Jahrgang, Thema, Textsorte, Kompetenzbereich, Operatoren) | absent | Write it; inferred at index time, correctable in a layer above the index |
| No German locale | `LOCALE_IDS = ['zh','en']` | Language-pack plugin |
| No pedagogy knowledge | absent | SH Fachanforderungen Deutsch: Operatoren (Anhang IV.1, Globaloperatoren IV.1.1), Anforderungsbereiche I Reproduzieren / II Zusammenhänge herstellen / III Reflektieren und Bewerten, zentrale Textformen (Anhang IV.2). These are **data files we vendor with source and date**, not model knowledge |

## Domain reference (verified 2026-09-20)

Source: *Fachanforderungen Deutsch Sekundarstufen I/II*, Schleswig-Holstein, fachportal.lernnetz.de.
Local extracts cached under this session's agent-tools directory; re-download before shipping.

- Anforderungsbereich I — Reproduzieren: wiedergeben, beschreiben, Verständnis sichern, geübte Verfahren anwenden.
- Anforderungsbereich II — Zusammenhänge herstellen: selbstständiges Erfassen, Einordnen, Strukturieren, Verarbeiten; Transfer.
- Anforderungsbereich III — Reflektieren und Bewerten: eigenständige Reflexion, Bewertung, eigene Lösungsansätze.
- Operatoren are **not** strictly mapped to one Bereich: "Alle Operatoren verlangen Leistungen im Anforderungsbereich II und zusätzlich in mindestens einem weiteren Anforderungsbereich." Any tool or prompt that assigns one Operator to exactly one Bereich is wrong by source.
- Published SH Abitur Erwartungshorizonte (e.g. Deutsch 2021, transparenz.schleswig-holstein.de) show the target structure: *Erwartete Leistungen* per Aufgabe with marked Anforderungsbereiche, *Bewertungskriterien*, percentage weighting per task, and the clause that sinnvolle begründete Abweichungen must be gewürdigt. That clause belongs in our house format — a generated Erwartungshorizont without it is not usable.

## SH AI-in-school context (verified 2026-09-20)

- **AIS.chat**: SH provides teachers a state-run DSGVO-compliant AI chatbot (login via VIDIS/Schulportal, governed by a Dienstvereinbarung, mandatory ~120-min AI-competency training citing Art. 4 EU AI Act). This is the free, blessed competitor named in `00-charter.md`.
- **IQSH Handreichung "KI@Schule"**: recommends AI-use disclosure as a Quellenangabe-style sentence ("Bei der Erstellung dieses Textes wurde … eingesetzt. Folgende Prompts wurden gestellt: …"), aimed primarily at *student* work; no watermark concept. Basis of `20-decisions/0010`.
- **EU AI Act Art. 50**: applicable since 2026-08-02; provider duty = machine-readable marking of synthetic content (exemption for assistive/standard-editing functions); deployer duty for text = public-interest text *without* human review — neither squarely binds a private tool whose output a teacher reviews and edits. Details and sources in `20-decisions/0010`.
- SH Leistungsnachweise guidance: when students use AI, Aufgaben and Bewertungskriterien must be adapted and Eigenständigkeit/Dokumentation assessed. Relevant to how generated Klassenarbeiten should look (the Erwartungshorizont must reflect adapted criteria), not to our legal posture.

## Invariants we adopt from dsh (cheap, and they bite if ignored)

- **Model-visible ⟺ logged.** Retrieval results reaching a request must be reconstructable from the session log.
- **Registrations are effects.** Every contribution goes through `ctx.effect()` / `ctx.on()`; `register()` returns the disposer.
- **Waterfall listeners must call `next()`.** Returning without it short-circuits the chain — a silent, brutal bug class.
- **Misconfiguration fails loud** at load when self-contained, otherwise at the earliest resolvable point. Never silently skip a missing referent.
- **No hardcoded tunables in plugins.** Deployment-varying choices are validated `Config` fields settable from `cordis.yml`. A `DEFAULT_*` constant is not configurability.
- **Opaque cross-boundary ids are branded**, never bare `string`.
- **Explicit > implicit at package boundaries.** Defaulting is an explicit `resolve(request): Spec` step, never a hidden `?? default` inside `run()`.
- **Prefer maintained dependencies over hand-rolling** when they genuinely delete owned code and tests.

Deliberately **not** adopted: per-file 100% coverage gates, bilingual `en`/`zh` docs, `doc-sync` gate machinery, snapshot-recorded e2e against a real API key. Out-of-tree, one user, one maintainer. Recorded in `20-decisions/0006`.
