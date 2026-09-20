# 40 — Upstream contract

What we depend on in deepseek-harness, pinned. The purpose of this file is to make an upgrade mechanical: after bumping the pin, re-verify every row and nothing else.
Companion to `10-dsh-capability-map.md` (what exists) — this file is what *we* consume.

## Pinned version

| Item | Value |
|---|---|
| Host harness | deepseek-harness (`dsh`) |
| Pinned version | `0.1.6-alpha.2` (not yet declared in our `package.json` — do so at scaffold time) |
| License | MIT |
| Local checkout used for research | `/home/hagbard/dev/deepseek-harness` @ `ddefc45fbc` |
| Stability warning from upstream | "THERE WILL BE COMPATIBILITY-BREAKING CHANGES" |
| Node engines | `^22.19 \|\| >=24` |
| Package manager | pnpm workspaces |

## Seams and services we consume

Re-verify each on upgrade. "Verify" means: the symbol exists, its signature is unchanged, and the documented semantics still hold.

| # | Dependency | Kind | Used for | Verify |
|---|---|---|---|---|
| 1 | `ctx.tools` (`register`, `restrict`, `guard`) | service | retrieval tool, provenance tool | registration signature; scoped-tool registry layering |
| 2 | `tools/pre-execute` waterfall → `PreToolDecision` | event | write-allowlist, ingestion-side data classification gate | waterfall semantics; decision kinds `allow`/`deny`/`ask` |
| 3 | `tools/post-execute`, `tools/result` | events | result transformation / audit | ordering rules vs `tools/execute` |
| 4 | `ctx.systemPrompt.section()` | service | Operatoren list, house-format rules, persona sections | ordering + scope-local shadowing semantics |
| 5 | `dsh-persona` | plugin row | "you teach Deutsch, Gymnasium, Schleswig-Holstein" | persona row config fields |
| 6 | `dsh-agent-presets` (`agent.cordis.yml`) | plugin | per-session tool/prompt/skill composition | preset resolution; "only an empty session may switch presets" |
| 7 | `ctx.skills` (`registerProvider`, `register`, `list`, `snapshot`, `get`) | service | subject skills packs (`0005`) | rank order 100/200/300/400/500/600; `SkillProviderObservation.complete` semantics |
| 8 | `dsh-tool-skill` | plugin | model-facing `skill` invocation | catalog digest re-injection via `agent.inject()` |
| 9 | `dsh-skill-office` | plugin | `.docx` authoring (Q3) | asset layout; what the deployment must supply |
| 10 | `ctx.workspaceRegistry` | service | her corpus folder as a durable project | header validation of session membership |
| 11 | `ctx.fs` / `dsh-fs-local` / `dsh-fs-sandbox` | service | read corpus, write output dir only (`0004`) | `fs/*` events; sandbox backend interface |
| 12 | `dsh-storage-sqlite` (`node:sqlite`) | plugin | index + overrides storage (or our own DB file) | `StorageError` codes; version-mismatch rejection stance (no migration, pre-release) |
| 13 | `dsh-llm-pi-ai` | plugin | model routes: cloud gateway and/or self-hosted local server (Q1) | `providers` config surface; `apiKeyEnv` credential reference; atomic route re-registration; `GenerateOptions.stop` is rejected `UNSUPPORTED_OPTION` |
| 14 | `ctx.jobs` | service | long corpus rebuild in background | `job_*` tool contract |
| 15 | `ctx.compaction` | service | large retrieval results in context | when auto-compaction runs (`agent/pre-step`) |
| 16 | `SessionEventMap` extension | type | retrieval results reaching the model must be logged (model-visible ⟺ logged) | required-on-read semantics; `ignorable: true` envelope flag; when `SESSION_FORMAT_VERSION` bumps |
| 17 | `packages/client/locale` language-pack | plugin surface | German UI (Q2) | `LocaleId = string` openness; fallback chain must terminate at `en`; `LOCALE_IDS = ['zh','en']` |
| 18 | `ConversationNodeDefinition` + keyed renderer | UI | provenance/source card | `docs/subsystems/conversation.md` |
| 19 | Bundle/profile manifests | distribution | `dsh.bundle.patch`, `dsh.profile`, `dsh plugin add` | layer order: bundles in profile order → profile patch → home patch → `--patch` |
| 20 | `cordis.yml` loader | config | composition files | `!!js` allowed only under plugin `config` and entry `disabled`; bare plugins must appear in the resolver manifest `dependencies` |
| 21 | `dsh-client-ui-sidebar-documentpreview` | client plugin | artifact preview in the revise loop (`0007`) | renderer registry (Markdown/code/image/PDF/HTML/Office→PDF); Office preview reuses `ctx.officeToPdf` |
| 22 | `dsh-tool-present` + `dsh-workspace-changes` | plugins | deliverables declaration; tracking what a session changed | `present` records paths/descriptions without copying contents |
| 23 | `dsh-sandbox-local` / `dsh-sandbox-policy` / `dsh-permission-presets` | plugins | defense-in-depth under the no-shell preset (`0009`) | `SANDBOX_UNAVAILABLE` fail-closed; `full`/`partial` enforcement reporting; preset table keys |
| 24 | `packages/attachment` | plugin | inbound images (Q7) | attachment seam; model `inputModalities` declaration |

## Invariants we must not violate

Listed with rationale in `10-dsh-capability-map.md`. On upgrade, re-read dsh's `AGENTS.md` "Conventions" section — it is the authoritative list and it changes.

Highest-risk three, because violating them fails silently rather than loudly:

1. **Waterfall listeners must call `next()`** — returning without it short-circuits the chain and nothing errors.
2. **Model-visible ⟺ logged** — a runtime invariant asserts it; retrieval results need a session event.
3. **Registrations are effects** — a `register()` that is not disposed through `ctx.effect()` leaks across HMR reloads and produces duplicate tools.

## Upgrade procedure

1. Read dsh's release notes / `docs/session-format-status.md` for the target version.
2. Bump the pin in our `package.json` (exact version, no range).
3. Walk rows 1-20 above; record failures.
4. Re-read dsh root `AGENTS.md` conventions diff.
5. Run our test suite, including the retrieval fixture (`0006`).
6. Boot the profile with `--dump-config` and diff the composed tree against the pre-upgrade dump.
7. If a seam is gone and no replacement exists: file a decision record, and consider whether `0001` (no fork) must be revisited.
8. Only then let her installed version change. Her machine upgrades last and deliberately.

## Known risks from upstream churn

- **Session format versions.** dsh has a released-format migration chain and refuses future versions. If we ever persist retrieval state inside session events, a format bump can strand data. Mitigation: keep the index in our own store (`0004`), never inside session generations.
- **`dsh-skill-office` interpreter supply.** Its contract explicitly delegates interpreters and authoring libraries to the deployment, so a change there breaks `.docx` output invisibly. Mitigation: Q3 experiment pinned in our own CI.
- **Client UI namespaces.** `verify-client-ui-i18n` rejects hardcoded copy upstream; a language pack that lags a namespace change shows raw keys. Mitigation: translate only the namespaces she sees, and fail loud on a missing dictionary at load.
