# 30 — Open questions

Unsettled items, each with options and a recommendation. Blocking questions are marked; a blocking question stops design work downstream of it.
When one is answered, it becomes a decision record in `20-decisions/` and this entry is struck, not silently edited.

## Q1 — Model routing (BLOCKING, re-scoped 2026-09-20)

**Why blocking:** it fixes the `dsh-llm-pi-ai` routes in the profile, the latency/quality bar in `00-charter.md`, and where the `0008` classify pass runs.

**Re-scoping after review.** The earlier framing over-weighted legal analysis; jan's correction, accepted: this is his project, the corpus stays on her disk (local folders or Schulcloud), and the remaining question is a plain engineering data-flow one: **which endpoints see corpus text, and is that what we want?** No compliance project. One factual note stays on the record and then we move on: personal data *can* enter the corpus even when she only prepares lessons, so if a customer story ("local AI in own datacenters") ever materializes, the routing design below is what makes it demonstrable — that is a product argument, not a legal one.

Candidates, all expressible as `dsh-llm-pi-ai` routes (settings-only, hot-swappable per request):

| Route | For | Against |
|---|---|---|
| OpenRouter → strong cloud model (jan's inclination: Qwen3.8-max class) | Best German pedagogy idiom; zero ops | Corpus chunks leave the machine; per-token cost on a large corpus |
| jan's company endpoint, Qwen 27B Q8 | Free; data stays in his infra; good enough for *classification* (`0008`) | Q1a: do its terms permit her school material? Unverified — check before wiring it even as test route |
| Local: 27B Q8 on the two external GPUs via llama.cpp-style server | Nothing leaves the machine; the demo for the "own datacenter" story | Weaker idiom exactly where it matters; ops burden on jan; her Mac may not have the GPUs attached |

Recommendation: **two routes from day one, roles split by task.** Generation and discussion → cloud (quality is the product). Catalog classification/dedupe summaries → the cheapest adequate endpoint (company or local; classification is an easier task than grounded pedagogy, `0008`). Because routing is per-request config in dsh, switching a role later is a settings change, not a rewrite. Decide by experiment: run 10 real tasks through both and compare, don't argue.

**Q1a (still open):** company endpoint terms — do they permit third-party (Pauline's) school material?

## Q2 — German locale: deferred by decision, scope TBD

Settled in review: **English UI chrome is acceptable for v1** and does not touch agent language (agent output is always German, `00-charter.md`). jan will still develop a `de` language pack because "why could [it] not help opensource after we developed something" — contributing it upstream to dsh is a plausible give-back and a forcing function for quality.

Remaining question: which namespaces the pack covers first (conversation, approval, deliverables, model-selection, workspace are the surfaces she touches). Non-blocking. `LocaleId` is open to language-pack plugins; fallback chains must terminate at `en` (verified).

Error surfaces follow the charter rule regardless of UI language: **typed error codes + structured payloads, display text via i18n dictionaries** — jan's "enums and i18n, clean code cmon" correction, adopted verbatim into `00-charter.md` and matching dsh's own convention (`StorageError` codes etc.).

## Q3 — `.docx` template stack (narrowed by 0009)

`0009` removed script-execution authoring from v1, so the question is now: **which in-process library renders her house format?** Candidates: `docx` (JS), `docxtemplater` (template-first), or raw OOXML via jszip for full control. Same question later for `.pptx`/`.xlsx` if she needs them ("good old python3 to make PowerPoints" was the alternative — rejected in v1 by `0009`, revisitable on evidence).

Decide by experiment: take one real Arbeitsblatt of hers + one real Erwartungshorizont ("in the end it is just another table" — jan; agreed, which makes tables the test case: Punktespalten, merged cells, Kopfzeile with Name/Klasse/Datum), reproduce both through the top two candidates. Also test round-trip: agent edits a file she already edited in Word, without destroying her formatting. That round-trip is harder than generation and is the real risk in the `0007` revise loop.

## Q4 — Corpus reality check (unchanged, still the cheapest high-information action)

Unknown: file count ("quite large" — jan), format mix, scan share, duplicate rate, folder organization. Get a representative slice (20-50 files) before designing extraction and chunking. The dedupe design (`0008`) especially is guesswork until we see how her duplicates actually differ (renamed? re-saved? annotated?).

## Q5 — Retrieval + interaction fixture (unchanged)

20-30 real requests from a week of her actual work — including pure discussion prompts, since `0007` makes discuss-mode first-class — plus which files she considered relevant. Becomes the CI fixture (`0006`) and doubles as product discovery.

## Q6 — Product name and first-run shape (unchanged, non-blocking)

## Q7 — Vision: images in and out (NEW)

`0007` requires images both directions: she attaches photos (textbook page, score, whiteboard) and artifacts may embed images (score sheets for Musik later, diagrams). Constraints:

- Inbound images need an image-capable model route — cloud routes vary; the local 27B may or may not have a vision sibling. dsh has an attachment seam (`packages/attachment`) and models declare `inputModalities`.
- Outbound images inside `.docx` are a template-layer question (Q3), not a model question.
- Inbound "take images" could also mean OCR-on-ingest for scanned PDFs (Q4 will tell us how many exist). That is a pipeline stage, not a chat capability, and may deserve a local-only OCR tool (e.g. tesseract class) rather than a vision LLM.

Decide after Q4 (how many scans actually exist) and Q1 (which routes are image-capable). Not blocking for the v1 trace, which is text-only, but blocking for Musik later.

## Q8 — Interaction memory: what, if anything, to take from eris (NEW)

jan: "steal the semantic brain from eris, the vectorDB that grows with all her interactions ... it should be a system that runs long."

The useful distinction: **corpus memory** (her documents — settled, `0008`) vs **interaction memory** (what she prefers, what we discussed, decisions across sessions — unsettled). The eris semantic-recall idea belongs to the second, and the second is genuinely valuable for a *long-running* partner: "in meinem Stil" only works if style preferences persist; a Diskussion about Effi Briest in March should inform one in November.

Concerns to resolve before adopting anything:

- jan's own question — does top-k context injection help or confuse? Same answer as `0008`: retrieval-on-demand as a tool beats ambient injection. An interaction-memory recall should be a `dig`-style call the agent makes when a session is long-running or references past work, not a per-turn dump.
- eris's tiered memory (staged/committed) is coupled to its vault-write model; Dewey's sessions already persist in dsh's session log. The cheapest v1 form of interaction memory may be: session log + a small "preferences" document the agent maintains (plain Markdown, human-readable, editable by her, indexed like any corpus file). No vector DB, no eris port. If measurement later shows recall failing, escalate — behind the same tool seam as `0003`.
- "Runs long" also means weeks/months of sessions: check dsh session storage growth and `session-query` behavior at that scale before promising it.

Recommendation: v1 = preferences-document pattern, deferred escalation to semantic recall with an explicit trigger (measured recall failure). Write the decision record when the trigger or the pattern is chosen — this entry stays open until then.

## Q9 — TypeScript comfort and the dsh type-magic surface (NEW, non-blocking)

jan: "a little nasty we write typescript again, but I see they do much type magic here."

Two mitigations, both already in the plan: `40-upstream-contract.md` exists precisely so we touch a *listed, verified* subset of dsh's surface instead of swimming in it; and `0001` keeps us out-of-tree, so their declaration-merging event maps and branded-id machinery are things we *consume at seams*, not reproduce. Our own code sets its own bar: strict mode, minimal type-level machinery, types that a tired maintainer can read at 23:00. If a dsh seam forces type wizardry we cannot follow, that is a `0001`-revisit trigger, and it gets a decision record.
