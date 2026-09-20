# 0007 — Interaction model is iterative co-authoring over a live preview, not one-shot delivery

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan, correcting the implicit model in the v1 trace: "imagine it not like she gives task and the agent delivers and well done, she will iterate multiple times." And: "she needs a partner that helps her create, produce documents of any kind, reorder, reread, take images, add images, discuss contents, discuss literature, discuss Anforderungen."

He also uses the clearest product description yet: "it is basically a chatbot, but customized."

Two consequences the earlier draft missed:

1. She uses ChatGPT as a **rubber duck** — thinking partner for literature, requirements, lesson design — not only as a document factory. A large share of real usage is conversation that produces no artifact.
2. The artifact she does produce goes through **several rounds**: draft → read → change this → read → reorder → add an image → done. The loop, not the first draft, is the product.

Research confirmed dsh already ships most of this loop, which is why this is a decision and not a build plan:

- `packages/client/ui-sidebar-documentpreview` — right-sidebar preview with registered renderers for Markdown, code, images, PDF, HTML, and **Office documents converted locally to PDF**, plus plain-text fallback. Renderer selection, reload, wrap, file status.
- `packages/deliverables/tool-present` — the model-facing `present` tool declares final files as deliverables (records paths and descriptions, does not copy contents); users open source files in their default application.
- `packages/deliverables/workspace-changes` — tracks what a session changed in the workspace.
- The web client chat node surface (`ConversationNodeDefinition` + keyed renderer) is how a custom node — a source/provenance card — would render inline.

## Decision

The product is a **persistent chat session over her corpus** with an artifact loop beside it. Three modes, one agent, no mode switching by the user:

| Mode | Output | Mechanism |
|---|---|---|
| **Discuss** | conversation only — literature, Anforderungen, lesson design, "is this Aufgabe too hard for Klasse 7" | plain turns; retrieval tool available but not mandatory |
| **Produce** | a file: Arbeitsblatt, Klassenarbeit, Erwartungshorizont, table, slide | authoring skills + `present` + sidebar preview |
| **Revise** | edits to an existing artifact across rounds | file stays the working object; preview reloads; she keeps the last version she approved |

Requirements that follow:

- **An artifact is addressable across turns.** "Make Aufgabe 3 harder" must resolve to a file and a location, not to a chat message that may already be compacted away. Artifact identity lives in the session log and on disk, never only in context.
- **Every produced file is previewable in-product and openable in her real application** (Word/Keynote/LibreOffice). `present` + documentpreview already give both. We do not build a previewer.
- **Revision is non-destructive.** She must be able to go back to the previous version. Cheapest correct form: the agent writes a new file or a versioned copy, never silently overwrites the one she approved.
- **Discussion is a first-class mode, not a failure to produce.** A session that ends with no file is a successful session. Do not measure quality by artifact count, and do not make the agent push for a deliverable.
- **Images in both directions.** She attaches images (a photo of a textbook page, a score, a whiteboard); the agent may place images into artifacts. Both depend on an image-capable model — see `30-open-questions` Q7.

## Alternatives considered

- **One-shot generation with a download button** (the implicit v1-trace model). Rejected: it is exactly the ChatGPT workflow she is trying to leave, with better grounding.
- **A separate "document editor" surface** (form-based, WYSIWYG, her words: "at least it needs a form where pauline can view the result and reiterate"). Rejected as a *custom build*: dsh's sidebar preview plus `present` plus the chat loop already is that form. A custom editor UI is a product in itself and would swallow the project. Revisit only if the preview proves too coarse for real revision work.
- **Mode switching as an explicit UI control** (chat mode / document mode). Rejected: the agent can tell from the request whether a file is wanted, and an explicit switch is one more thing for a non-engineer to learn. Keep the tool set constant.

## Consequences

- The v1 trace in `00-charter.md` must be extended past step 6: the real acceptance is the *second* round ("mach Aufgabe 3 schwerer") and the preview updating. Updated there.
- Provenance (`0002`) needs a rendering home. A source card as a `ConversationNodeDefinition` is the natural fit; a plain markdown citation list is the acceptable v1 fallback.
- Long iterative sessions hit context limits. dsh's `ctx.compaction` handles pressure, but compaction can drop the artifact's history — which is why artifact identity must be durable rather than contextual. This interacts directly with `0008`: retrieval should be re-runnable on demand precisely because context gets compacted.
- Non-destructive revision means an output-directory convention (per `0004`, corpus stays read-only). Versioning policy is an implementation decision, not a charter one.
- "Discuss" mode raises the value of corpus grounding for conversation, not just for documents — she asks "what do I already have on Balladen?" mid-discussion. Retrieval must be cheap and callable at any time, which `0008` addresses.
