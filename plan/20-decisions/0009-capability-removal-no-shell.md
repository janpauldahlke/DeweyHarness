# 0009 — Capability removal over containment: her preset has no shell

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan: "how to keep her machine safe from a gone wild agent, sudo rm rf fantasies. I am uncertain if an agent for Pauline needs really coding, it needs markdown and office. Or should we give it good old python3 to make it generate PowerPoints or Excels."

The question contains its own answer. An agent that cannot execute arbitrary commands cannot execute `rm -rf`. Containment is a fallback; removal is a guarantee.

dsh makes removal cheap because the tool set is per-session composition, not a global registry (`dsh-agent-presets`: "give each session the tools, prompt sections, and skills named by one preset's `agent.cordis.yml`"). Verified containment machinery, for the record:

- `dsh-sandbox-local` — confines commands and descendants on Linux/macOS/Windows, auto-selects the platform runner, and **fails with `SANDBOX_UNAVAILABLE` when none is usable, so commands never silently run without confinement**. Reports `full` or `partial` enforcement per execution.
- `dsh-sandbox-policy` — one file-effect policy applied to every confined bash, filesystem, and terminal call; deployment default mode + fallback workspace root; per-session override; the model is told the effective policy before each request.
- `dsh-permission-presets` — bundles sandbox mode + approval policy; ships `workspace-write` (`workspace-write` + `ask`) and `danger-full-access` (`danger-full-access` + `never`).

## Decision

**Pauline's preset mounts no shell.** No `bash`, no `terminal`, no `computer-use`, no `browser-use`. Its tool set:

| Capability | Grant | Boundary |
|---|---|---|
| Read corpus | yes | `ctx.fs` scoped to corpus roots, read-only |
| Write output | yes | one output directory only; corpus is never writable (`0004`) |
| Retrieve / dig | yes | `catalog_find`, `corpus_read` (`0008`) |
| Author documents | yes | deterministic in-process templates — see below |
| Declare deliverables | yes | `present` |
| Ask her a question | yes | the ask tool, for genuine ambiguity |
| Execute commands | **no** | not mounted; not sandboxed-and-allowed, absent |
| Run background jobs | yes | `ctx.jobs`, ours (index rebuild), not model-authored code |

Document authoring therefore does **not** use `dsh-skill-office`'s script-execution path in v1. We generate `.docx` (and later `.pptx`/`.xlsx`) through deterministic in-process libraries and her house-format templates. This answers his Python question directly: *not* Python3, not model-authored code.

Rationale, in order of weight:

1. **It deletes a failure class instead of bounding it.** Model-authored Python that writes files is the mechanism by which an agent corrupts things you did not anticipate. A template cannot.
2. **It makes output reliable.** Her house format — Kopfzeile, Name/Klasse/Datum fields, Punktespalten, Quellenangabe — is a *template*, so it is correct by construction rather than correct if the model remembers. This matters more than it sounds: format drift is the thing that makes generated documents unusable without rework.
3. **It removes the interpreter dependency.** `dsh-skill-office` explicitly delegates interpreters, authoring libraries, execution tools, and file delivery to the deployment. On her Mac that is a Python environment we must install, version, and debug remotely. A JS/TS library ships in our bundle.
4. **Sandbox becomes defense-in-depth, not the primary control.** We still set `sandbox-policy` to a workspace-write default with the output dir as the workspace root, and approval `ask` for writes outside it — so a future capability addition does not silently arrive with full access.

Revisit if she needs layouts a template cannot express. That is a real possibility (Beamer-style slides, complex tables), and the trigger is measured: she asks for something the template cannot do, twice.

## Alternatives considered

- **Shell + `sandbox-local` in `workspace-write` + `ask`.** dsh's default developer posture, and genuinely well-engineered. Rejected for her: `partial` enforcement is a real reported state, macOS sandboxing shares the host kernel and filesystem, and an approval dialog is a decision she is not equipped to make ("allow `python3 /tmp/gen.py`?"). The safe answer for a non-engineer is no dialog at all.
- **`dsh-skill-office` with its bundled Python.** Rejected for v1 on points 3 and 4 above. Kept as the escalation path if templates prove insufficient — it is the more *capable* option, just the less *safe* and less *deterministic* one.
- **No write access at all; output only as chat text she copies.** Rejected: that is the ChatGPT workflow (`00-charter.md`), which is the thing we are replacing.
- **Full access, she has admin rights, trust the model.** Rejected without discussion. Admin rights are exactly why this needs a boundary.

## Consequences

- We own the template layer: house format as data (per `0005`), one renderer, versioned output files (`0007` non-destructive revision).
- Q3 in `30-open-questions.md` is narrowed by this decision: the experiment is now *template library choice*, not *script-execution vs no-script-execution*.
- A capability we do not mount cannot be jailbroken into use. This is the strongest safety property available and it costs us one preset file.
- If we later add code execution, it arrives as an explicit decision record with its own threat model — not as a convenience.
