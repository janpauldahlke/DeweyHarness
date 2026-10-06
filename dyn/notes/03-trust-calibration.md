# Note 03 — The catalog surface as trust engine

Status: agent note, 2026-10-06. Deep dive on the curation surface
(`10-curation-first.md` §3, P3): why the catalog screen is not a feature but
the *trust engine* of the whole product, how trust is built in increments,
what breaks it, and why the UI is staged.

## 1. Why this surface is load-bearing

A non-engineer is being asked to act on inferences she cannot evaluate:
"this file is Klasse 9, Textsorte Gedicht, Operatoren: analysieren" — and
"these 12 files are duplicates of that one" — and "37 files I couldn't read."
The derived index (`0004`) is clever; if it is opaque, it is a black box, and
a black box that a non-engineer can't inspect is a black box she will quietly
stop trusting and eventually stop using.

But it is more than adoption. The catalog is the **trust engine** for three
reasons:

1. **It is where her corrections live.** Every "you got that wrong" she makes
   lands in the overrides plane and *persists across rebuilds*. A correction
   she can find again is a correction she believes in. Persistence is what
   turns "it's a report" into "it's *mine*" — the ownership moment that no
   other surface provides.
2. **It is the honesty surface.** The health view (unreadables, skips,
   low-confidence rows, clusters awaiting a decision) is the system telling
   her what it *doesn't* know. A system that confesses its gaps makes its
   confident claims credible. Honesty about limits is not a feature; it is
   the substrate under every confidence number.
3. **It feeds the Partner's quality ceiling.** The Partner's retrieval,
   grounding, and idiom all ride on catalog metadata quality
   (`notes/01` §5). Her corrections *are* the model's training signal for
   this user. The trust loop and the quality loop are the same loop:
   she trusts → she corrects → the Partner gets better → it's right more
   often → she trusts.

## 2. The trust loop, as increments

Trust is not a property you ship; it is a sequence of *events*, each one a
small bet she takes and wins:

```
see          → the catalog shows her own files, summarized, in her language
question     → a low-confidence row is phrased as a question, not a fact
correct      → she fixes one row in one action
verify       → the correction is visible again, and survives a full rebuild
delegate     → she answers a cluster's "which is the original?" and moves on
steward      → she corrects *without being asked* (the Stewardship proxy fires)
```

The design rules follow from the sequence — each rule exists to make the
*next* bet in the sequence cheap:

1. **Every inference carries its evidence.** A confidence number is
   meaningless to a non-engineer. The evidence line — "filename says Kl7;
   text mentions *acht Klässler*; folder is 2021/9" — makes the claim
   checkable in three seconds. Low confidence is *phrased as a question*:
   "Klasse 9, probably — the text doesn't say. Is that right?" Questions
   invite correction; confident facts invite silence (and rot).
2. **Corrections are one action, always findable, never silently dropped.**
   "Mark this as Klasse 9" in chat lands in the overrides plane (`0004`);
   "what have I corrected?" is always answerable; a correction whose file
   disappeared is *surfaced as dangling*, not deleted. Durable corrections
   are what make correcting worth the effort — and effort is the only thing
   a non-engineer spends to build trust.
3. **The system says what it doesn't know.** The health view is the catalog's
   conscience: named unreadables with reasons, skips, low-confidence rows,
   clusters awaiting her. This is the honest-exclusion category from
   `10` §1.4 made visible — and it is what makes J2 ("tell the truth about
   what's there") *demonstrable* instead of asserted.
4. **Nothing acts.** V1 curation is *surfacing*, not doing. It proposes
   clusters and a suggested structure; it never moves, renames, or deletes
   (the map-before-surgery rule, `10` §4). Trust is spent on *actions*, not
   on information — so the first action (consented file moves) arrives later,
   journaled, undoable, and with its own decision record.
5. **Progress is visible, not just complete.** The ingest feed (per-file
   names, running counts, the gap list forming) is what keeps her watching
   the first hour. A non-engineer's patience is a function of visible
   progress; a progress bar that says "indexing…" for 20 minutes is a
   non-engineer leaving the room.
6. **Her language.** German, plain, no jargon. No "chunk," no "confidence
   0.72" — "pretty sure" / "not sure," with the evidence underneath.
   Technical precision lives in the store and the logs; the surface speaks
   to a teacher, not to jan.

## 3. What breaks trust (the failure catalog)

Each of these is a named failure with a named mitigation — so the build can
be checked against them:

| Break | How it happens | Mitigation already in the plan |
|---|---|---|
| **Silent misclassification** | She discovers later that a row was wrong and was never shown as uncertain | confidence + evidence on every row; low confidence phrased as questions; health view re-asks |
| **Correction vanishes** | A reindex silently drops her override | two-plane store (`0004`); **override-survives-rebuild is an explicit test** (`20` §5) |
| **Quiet gaps** | the catalog is silent about what it skipped and she finds it by accident | honest-exclusion category + health view (J2 made demonstrable) |
| **Unwanted action** | the system moved/renamed/deleted something she didn't ask for | read-only corpus mount; audit; no move capability in v1 (rule 4) |
| **Jargon** | "confidence 0.72, chunk 14, BM25" in her face | rule 6; the surface is German plain-language by construction |
| **Slow, opaque first run** | 20 minutes of "indexing…" | rule 5; small first cut (N2); Opening B (`10` §5) |
| **Confident wrong Partner** | a generated artifact cites a file that doesn't say what it claims | provenance must resolve (`0002`); citations resolvable is a test invariant; honest-on-empty |

Note the pattern: the first four breaks are *trust-engine* failures (the
catalog), the last three are *honesty/precision* failures (system-wide). The
catalog is the place where the first four are caught — or where they become
visible.

## 4. Why the UI is staged (and what that costs)

The draft's plan had no catalog surface at all; `20` adds it staged:

- **v1 (thin slice): agent-rendered.** The agent produces a filterable table
  (Markdown/HTML in the preview); corrections happen in chat. *Cost:*
  correcting requires a sentence ("mark the Goethe one as Klasse 10") instead
  of a click. *What it preserves:* every trust rule in §2 — evidence,
  persistence, honesty, no actions, visible progress, her language. The
  trust invariants are *identical* in both stages; only the **cost of
  correcting** differs, and the corrections that matter (the first few) are
  cheap enough in chat.
- **v2 (full v1): custom `ConversationNodeDefinition`.** One-click override
  affordances on the rows (accept / correct / mark-as-duplicate / "keep both").
  Built *when stewardship events exist and chat-correction friction is
  visible* (the backlog trigger, `20` §7) — not before, because it is the
  biggest new UI surface in the project (dsh ships no catalog UI) and its
  scope should be set by observed use, not by anticipation.

Staging is not a compromise on trust: the moment of ownership (correction
persists and is findable) arrives in v1. v2 makes the loop *faster*, not
*real*.

## 5. The metric that proves it works

**Stewardship** (`01` §6): the count and *voluntariness* of her correction /
cluster / health-view events during the two-week window. Not "the catalog
exists" — *she touched it, and left the touches in*. A few voluntary
corrections plus one unprompted "what do I have on X?" is the evidence that
the engine is running. Zero events after a real invitation is the H-archive
failure signal (`20` §4) — and it is a signal that is *measurable*, which is
why the surface is the engine and not the decoration.

## 6. One paragraph for the morning

The catalog is not a view over the index; it is where a non-engineer learns
whether the thing can be trusted, one small bet at a time — see it know her
files, question one row, correct one row, watch the correction survive, and
finally correct one row *without being asked*. Build the surface so every
inference carries its evidence, every correction is durable and findable,
every gap is confessed, and nothing acts without her say-so — and the trust
that the Partner's grounding and idiom both depend on is not assumed, it is
*accrued*. Stage the clicks for later; the invariants are what matter, and
they fit in a chat.
