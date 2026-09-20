# 0010 — AI-use marking: machine-readable metadata plus optional footer, never a visible watermark

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan: "should we imprint on all papers a small watermark where we openly say we use help of AI to create this document? ... I read a new law lately about informing about usage of AI in certain document types." He also describes Pauline accurately: she "would draft a Klassenarbeit with her LLM but never use it unchecked and unread."

The law he half-remembered is real and **already in force**. Verified 2026-09-20:

- **EU AI Act Art. 50** applies since **2 August 2026** (Commission guidelines C(2026) 5054, 20 July 2026; Digital Omnibus Reg. (EU) 2026/1744 set the 2 December 2026 grace period *only* for systems already on the EU market before 2 August — a new system has **no grace period**).
- Art. 50(2) obliges **providers** of systems generating synthetic text to mark outputs **machine-readable and detectable** — with an exemption where the system "performs an assistive function for standard editing or does not substantially alter the input data provided by the deployer".
- **Deployer** disclosure duties for text are narrower than commonly assumed: they cover AI-generated text *published to inform the public on matters of public interest* **that did not undergo human review or editorial control**. A teacher's reviewed, edited Arbeitsblatt is neither public-interest publication nor unreviewed.
- **Schleswig-Holstein specifics** (more relevant than the AI Act here): SH provides teachers a state-run DSGVO-compliant chatbot (**AIS.chat**, via VIDIS/Schulportal, with a Dienstvereinbarung and mandatory ~120 min AI-competency training per Art. 4 AI Act). The IQSH *KI@Schule* Handreichung recommends, for AI-assisted text, a **Quellenangabe-style sentence** — "Bei der Erstellung dieses Textes wurde (Name des KI-Tools) eingesetzt. Folgende Prompts wurden gestellt: …" — *not* a watermark.

## Decision

Three layers, cheapest first:

1. **Machine-readable marking, always on, invisible.** Every generated file carries AI-assistance metadata: `.docx` custom document properties (`dc:creator` = Dewey, a `dewey:model`, `dewey:sources` property), PDF XMP if we ever emit PDF. Costs ~20 lines in the template layer (`0009`), survives her edits, satisfies the *form* of Art. 50(2) should jan ever place this on a market.
2. **Optional visible footer on drafts, off by default for finals.** A removable one-liner in the house template — the IQSH-recommended Quellenangabe form, adapted: "Entwurf — mit Unterstützung von Dewey erstellt; Quellen: …". On for drafts (which is what the tool produces), removed by her act of finalizing, because a reviewed-and-edited document is her work. The toggle is one template field, not a settings maze.
3. **No watermark. Ever.** A diagonal "KI-GENERIERT" stamp on a Klassenarbeit is wrong on every axis: it defaces a printed exam, it survives into copies and scans where it misleads (the version she rewrote is not AI-generated), and no applicable rule requires it — the deployer text-disclosure duty does not cover human-reviewed non-public-interest documents, and the SH guidance explicitly models disclosure as a citation sentence, not a mark.

Provenance (`0002`) already lists her own source files on generated artifacts. That Quellenangabe does double duty: it is the pedagogically valuable part, and it is the SH-idiomatic form of AI-use transparency.

## Alternatives considered

- **Visible watermark on everything** (jan's proposal). Rejected above.
- **Nothing at all.** Rejected: metadata marking is nearly free, it is the future-proof choice the moment "maybe I win a customer" happens (a system placed on the market after 2 Aug 2026 has no grace period), and the footer option gives Pauline something to show a skeptical Fachschaftsleitung.
- **Full prompt-log disclosure per the IQSH example** ("Folgende Prompts wurden gestellt: …"). Rejected as a default: that guidance targets *student* work where the teacher must assess the student's steering of the tool. For her own teaching materials it is noise. Available as an export if she ever wants it.

## Consequences

- The template layer owns marking; marking is data, not per-document logic (`0005`, `0009`).
- If Dewey is ever commercialized, the provider-side analysis must be redone deliberately — this record covers a private tool for one teacher who reviews everything, and says so.
- Competitive note for the charter: AIS.chat is free, state-blessed, and DSGVO-compliant. Dewey's answer is not compliance — it is grounding in *her* corpus, *her* house format, *her* idiom (`0002`). That differentiation now has a named competitor and should stay honest about it.
