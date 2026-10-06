# Note 01 — Idiom from exemplars: her own artifacts are the style source

Status: agent note, 2026-10-06. Deep dive on the single biggest lever on the
fuzziest acceptance bar — "it sounds like her material, not ChatGPT."
Claim: the SH spec gets the model to *SH-correct*; only her own prior
artifacts get it to *hers*. The corpus already carries her style — the product
just has to feed it back.

## 1. The problem, precisely

"Sounds like her" decomposes into four channels, and the draft's plan only
fully funded two:

| Channel | What it carries | Draft's funding | Gap |
|---|---|---|---|
| **Spec data** (vendored SH Fachanforderungen) | the *allowed* vocabulary: Operatoren, Anforderungsbereiche, published Erwartungshorizont structure | funded (`0005`, data files with source+date) | none — but it only gets you to "SH-correct" |
| **House format** (templates) | layout: Kopfzeile (Name/Klasse/Datum), Punktespalten, merged cells, column structure, the *sinnvolle Abweichungen* clause | funded (P5; the stack itself is Q3) | none once templates exist |
| **Her voice** | wording, register, task phrasing ("Lies den Text und trage ein …" vs. textbook-generic phrasing), structure habits (number of Aufgaben, difficulty ramp I/II/III), recurring conventions | **not funded** — the persona row ("you teach Deutsch in SH") is a generic stand-in for *her* | **the gap** |
| **Her standing instructions** | taste and context: "two columns when there are images," "this class is weaker," "never use 'erörtern' in Sek I" | not funded (Q8 preferences doc, deferred) | smaller; grows over time |

The gap is "her voice." A model with perfect spec data and perfect templates
still writes in the voice of *a* German teacher, not *her* — and that is
exactly the ChatGPT tell she will feel in three seconds. (This is also why the
draft's P6 gate is *her verdict* and not a rubric: rubrics can measure the
spec channel; only she can measure the voice channel.)

## 2. The insight

Her corpus contains **exactly the artifact types she wants generated, in her
voice**: years of real Arbeitsblätter, real Erwartungshorizonte, real
Aufgaben. That is a style corpus with no training run required. Few-shot
exemplar retrieval — *at generation time, retrieve 2–3 of her own prior
artifacts of the target type and inject them as style exemplars* — transfers
her voice into the generation with retrieval machinery she already has
(P4's `catalog_find`/`corpus_read`) and no new model, no fine-tuning, no
vector DB.

It quietly resolves most of the "in meinem Stil" question that Q8 left open:
*style* does not need semantic interaction memory at all — it lives in her
files. What Q8's preferences doc still needs is the other half: *standing
instructions and context* (see `notes/02` — the edit-diff loop is what
converts recurring taste into standing instructions).

## 3. The mechanism

**At generation time, for artifact type T (e.g., Arbeitsblatt):**

1. **Select exemplars** from the catalog: `Textsorte = T`, `Fach = Deutsch`
   (v1), Jahrgang match if the request names one. Prefer, in order:
   high-confidence catalog rows → recent files (style drifts year over year;
   the 2024 sheet is closer to the 2026 sheet than the 2014 one is) →
   readable, non-corrupt, reasonably sized. Take 2–3 files. If the type is
   rare in her corpus, fall back to the nearest type (Arbeitsblatt →
   Aufgaben) and say so in the prompt framing.
2. **Truncate to the style-bearing parts.** Full documents waste the budget on
   content the generation must not copy. Keep: the Kopfzeile, the opening
   instruction, 1–2 full Aufgaben with their sub-item lettering, the
   Erwartungshorizont table (if present), and anything that shows layout
   conventions. Target: **~2–4k tokens of exemplar text per generation** —
   enough to carry voice, cheap enough to be default.
3. **Frame them explicitly as style, not content.** The prompt section says
   (in German, since the agent is German): that these are *their* prior
   documents of this type; that the new artifact must match their
   *conventions, structure, and wording habits*; that their *content* (texts,
   tasks, answers) must come from the current request and the retrieved
   sources — never copied from the exemplars; that the SH spec data is the
   floor for Operatoren/Anforderungsbereiche and the exemplars are the model
   for everything else.
4. **Log the exemplars used** — which files, which sections, which truncation.
   They are model-visible inputs, so the *model-visible ⟺ logged* invariant
   applies (they need a session event); the log also answers her inevitable
   "where did it get that phrasing?" with a resolvable pointer — provenance
   for style, which is a trust feature, not just an audit one.

**What the exemplars must NOT do:**
- **Not be cited as sources.** They are stylistic inputs. If a generated
  artifact reuses, say, a Textausschnitt from an exemplar file, that file is
  *also* a source and gets a normal `corpus:` citation. Two roles, two
  mechanisms, never conflated (otherwise the provenance contract of `0002`
  gets ambiguous: "sourced from" vs. "styled after" are different claims).
- **Not override the spec floor.** If an exemplar uses an Operatoren
  construction the vendor data says is wrong-by-source, the spec data wins
  (the validator checks against the data). Her files can be wrong; the Fach-
  anforderungen are not.
- **Not become a content leak.** Truncation + framing handle the common case;
  the residual risk (a very distinctive task phrasing getting recycled) is
  acceptable — she will *notice* and either be delighted ("it even sounds like
   my 2023 sheet") or correct it, and the correction is a preference entry.

## 4. Why this is the biggest lever

1. **It targets the residual, not the base.** Spec data + templates get most
   of the way; the *remaining* gap to "that's mine" is voice, and voice is
   precisely what no amount of spec data contains. The lever acts where the
   deficit actually is.
2. **It is nearly free.** Selection is a catalog query (P4), truncation is a
   string op, injection is a prompt section. No new retrieval tech, no new
   model, no embeddings, no fine-tuning. It composes entirely out of seams
   the plan already touches.
3. **It gets better as the corpus grows and she corrects it.** Better catalog
   metadata → better exemplar selection → better voice match. The trust
   engine (corrections) and the idiom engine (exemplars) feed the same
   plane — a compounding loop the draft's plan did not have.
4. **It is testable early.** The route probe (`50` §4) runs the exemplar
   condition explicitly: same request, with/without exemplars, judged against
   her exemplar document on the voice axis. One morning, and the lever's
   contribution is *measured* instead of believed.

## 5. Interaction with the rest of the plan

- **P6 (partner):** exemplar retrieval is a first-class prompt section of the
  skill, not a nicety. The P6 gate ("sounds like her material") is
  unachievable without it; with it, the gate is a real bet (B5) instead of a
  hope.
- **Q8 (interaction memory):** shrunk. Style → exemplars (this note). Taste +
  standing instructions → the preferences-doc stub (the Q8 pattern, kept).
  The semantic-brain escalation trigger is unchanged (measured recall
  failure) but less likely to fire, because half of what "recall" was for is
  already answered by retrieval.
- **`notes/02` (edit diff):** the diff loop is the *learning* half — recurring
  edits become preference entries, which become standing instructions, which
  the exemplar framing incorporates ("and: always …"). Exemplars = her voice
  today; preferences = her voice *as she's refining it*.
- **Risks:** overfitting to a bad exemplar (mitigated by recency +
  confidence ordering and by her corrections); style drift across years
  (recency ordering); token cost on long documents (truncation to
  style-bearing parts); exemplar selection quality (the catalog's metadata
  quality — another reason the Archive's J1 bar is load-bearing for the
  Partner's idiom).

## 6. One paragraph for the morning

Give the model *her* three last worksheets of that type, told honestly that
they are hers, told to match the way she writes and not what she wrote, and
the "SH-correct but generic" failure mode loses its home. It is retrieval,
not memory, not fine-tuning — and it is the cheapest version of "in meinem
Stil" the plan can build, because the training data was already sitting in
her drive.
