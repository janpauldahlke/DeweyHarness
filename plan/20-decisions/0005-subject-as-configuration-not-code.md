# 0005 — Subject and Land are configuration, generalized only on the second instance

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan, as architect: "we need an abstraction for all the possible subjects ... most of them will have max 3 domains ... we build more a smart interface where this does not matter much, it could belong in the executing agent system prompt, you are teacher for xyz."

The instinct is right and the timing is the risk. Designing a subject abstraction from one instance (Deutsch-SH) means inventing an interface whose only evidence is a sample size of one. Every abstraction built that way encodes the accidental properties of its first case, and the second case pays for it.

Meanwhile his own second sentence contains the correct answer: most of the subject-specific behavior *is* prompt and data, not code.

## Decision

A subject is a **configuration triple**, not a code branch:

1. **Corpus folder(s)** — which files belong to this subject.
2. **Skills pack** — kebab-case `SKILL.md` authoring recipes for that subject's artifact types, discovered through `ctx.skills` with the layered rank order dsh already provides (project `.dsh/skills` < custom < user < bundled).
3. **Vendored reference data** — the subject/Land's authoritative vocabulary as dated files with a source URL: for Deutsch-SH that is the Operatoren list, the three Anforderungsbereiche, and the Erwartungshorizont structure from the *Fachanforderungen*.

Persona ("you teach Deutsch at a Gymnasium in Schleswig-Holstein") is a `dsh-persona` row plus `ctx.systemPrompt.section()` contributions — configuration, exactly as jan proposed.

**No subject-generic interface is written in v1.** The rule that replaces it:

> The abstraction is extracted when the second instance exists and the duplication is visible in code, not before.

The first extraction candidate is Deutsch-HH or Musik-SH, whichever jan wants second. Until then the only structural commitment is that subject-specific content lives in *data and skills*, so that adding a subject is adding files, not editing the retrieval engine, the index schema, or the generator.

## Alternatives considered

- **Subject plugin interface in v1** (`interface SubjectAdapter { operators(); bereiche(); textsorten() }`). Rejected: one implementation, so every method signature is a guess. The cost is not the interface, it is that the interface freezes before evidence arrives.
- **Hardcode Deutsch-SH everywhere.** Rejected: cheap now, and it produces exactly the "rewrite it for the second subject" outcome that eris extras demonstrate.
- **Subject as a per-session agent preset only.** Partially adopted — presets do carry the persona and skill scope. Insufficient alone, because the reference data and metadata schema are host-side index concerns, not session concerns.

## Consequences

- v1 code must avoid the trap by construction: the retrieval engine and index schema take Fach/Land/Jahrgang as **opaque indexed facets**, never as a `switch`. A `switch (fach)` anywhere in the pipeline is a review-blocker.
- Reference data files carry a source URL and a retrieval date, because curricula change and a stale Operatoren list is a silent correctness bug. The SH list is *not* model knowledge; it is vendored data.
- Adding a subject should be a documentation exercise. If it requires code, this decision was wrong and gets superseded — that is the test.
- Musik is explicitly out of v1 (`00-charter.md`) and is the harder case: sheet music needs symbolic metadata over PDF/MusicXML, which is a different extraction pipeline, not a different skills pack. If Musik-SH is chosen as the second instance, expect to discover that "subject as configuration" does not cover non-text corpora, and expect to supersede part of this record. That is a useful discovery, made cheaply, rather than a v1 bet.
