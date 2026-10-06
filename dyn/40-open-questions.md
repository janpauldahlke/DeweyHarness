# 40 — Open questions (product level)

Status: agent draft, 2026-10-06. The questions only jan or Pauline can answer,
ranked by how much the plan bends around them. Each gets: what it is, who
answers it, my recommendation, and my best guess (explicitly a guess). The
draft's engineering questions (Q1–Q9 in `plan/30-open-questions.md`) are
tracked in §3 with their status — not re-litigated, only updated where the
reframe changed something.

## 1. Ranked product questions

### N1 — Is the Archive a *prerequisite* or a *core value*?
**Why it matters:** it is the single decision the reframe forces. If
prerequisite: curation is expensive plumbing and everything is justified by the
Partner. If core: the Archive has its own success bar, its own features (the
suggested-structure report, catalog v2), and the project survives even if the
Partner underwhelms. `30-risks-and-bets.md` §5 is written around this being
the question jan most needs to answer.
**Who:** jan (his overnight words may support only the weaker reading).
**Recommendation:** build as prerequisite (the order is right either way),
*claim* core value in the docs, and let the two-week window's Stewardship
proxy arbitrate. Cost of being wrong: low (flip-table H-archive).
**Guess:** both, but the core-value claim is a 60/40 bet — the archive is
*probably* worth it on its own for a desktop messi, which is precisely why
jan reframed the night before.

### N2 — What exactly is "the drive"?
One folder she assembled? Her whole desktop? A synced folder? How many
files/GB, what format mix, what scan share — and **is there a backup?**
**Why it matters:** it sizes P2 (the biggest phase), decides the first-run
cut, makes or breaks the OCR trigger, and the read-only promise should be
*informed* by whether her files have a second copy. A1 in `01` §3 is this
question.
**Who:** jan + Pauline (a 20-minute look at the drive answers most of it; P0's
slice answers the rest).
**Recommendation:** she assembles (or we walk her desktop once and she
confirms) **one folder** — the "school drive" — and P0's census reports
count/size/format/scan-share/backup. If the desktop is the real corpus and
she won't assemble a folder, the ingest must take multiple roots (A1 flips) —
that is a first-run-UX cost we only pay if we have to.
**Guess:** one folder, a few thousand files, mostly docx/pdf/md, scans a
minority (a teacher who works digitally), backup unknown (probably Time
Machine at best).

### N3 — What does "clean up the drive" mean *to her*?
Does she want the end state organized (and who performs it — her, over a
weekend, or Dewey, later)? Or is a good map + duplicate report enough?
**Why it matters:** it decides how much of `10-curation-first.md` §4 (the
suggested-structure report, the move capability) is v1 value or backlog.
**Who:** Pauline (directly), jan (from watching her work).
**Recommendation:** v1 = the map (catalog + duplicate report +
suggested-structure report as a document she can work through). Move
capability on explicit double-ask (the trigger in `20` §7). This is the
lowest-risk reading of her own words and it produces the end state she wants
without the trust-break surface.
**Guess:** she wants the *end state* (an organized drive) but will happily
execute the moves herself given a good plan — teachers are used to weekend
cleanup projects; they've been doing it by hand for years.

### N4 — Which artifact family does she live in?
**Why it matters:** B7 (`30` §1) — one artifact family is the thin slice's
produce scope; if we pick the wrong family, the thesis is tested on a sample
that misses her real work.
**Who:** Pauline's real prompts (P0) answer it better than any interview.
**Recommendation:** let the week of prompts decide; default to **Arbeitsblatt
+ its Erwartungshorizont** (the draft's family — it exercises both the
generate loop and the hardest idiom structure, the percentage-weighted table).
**Guess:** Arbeitsblätter dominate her volume; Erwartungshorizonte are lower
volume but higher stakes (her name on them) — both worth the one family.

### N5 — How does she actually experience AIS.chat?
Tried it? Uses it? What for? The draft's competitive framing ("why not
AIS.chat" = grounding/idiom/artifacts, not compliance) assumes she knows it
and finds it ungrounded.
**Why it matters:** if AIS.chat is *actually* her daily tool for some tasks,
Dewey's bar for those tasks is higher than "beats ChatGPT."
**Who:** Pauline, one conversation.
**Recommendation:** ask directly in the P0 conversation; treat her answer as
data for the Reach proxy's baseline (what does she give to AIS.chat today?).
**Guess:** she's tried it, found it compliant-but-generic, and still reaches
for ChatGPT for real work — the draft's framing holds.

### N6 — Does she care where corpus text goes?
Q1 is the draft's engineering framing (which endpoint sees text = config).
The product question underneath: does *she* have a preference (school context,
personal data, peace of mind), and would "your text goes to X" change her
trust?
**Why it matters:** if she says "no text to the cloud, ever," the model-route
design flips (local generation first, quality cost paid) — a P0 answer, not a
P6 surprise.
**Who:** Pauline.
**Recommendation:** default to the draft's two-route split (generation → best
available cloud; classification → local 27B/company endpoint), state the
data flow plainly at first run ("your documents are read here; only the parts
I send to the model for generation leave, to X"), and let her veto move the
design. The routing is per-request config, so the flip is cheap.
**Guess:** she's fine with cloud generation (ChatGPT already does exactly
that); she cares more that her *files* stay on her disk, which the design
guarantees regardless.

### N7 — What personality does the discuss partner get?
Formal precise instrument vs. warm colleague who knows her? This shapes the
persona row, the interaction-memory scope (Q8), and how "rubber duck" time is
spent.
**Why it matters:** a non-engineer teacher who wants a *colleague* will
punish a clinical tool in discuss mode — and discuss is 40%+ of her real use
(`0007`'s observation).
**Who:** Pauline, from the two-week window (and the week of prompts).
**Recommendation:** start precise, warm, and *short* — a competent colleague
who gets the point, in plain German, no small-talk theater, no sycophancy.
Escalate warmth only if her prompts show she wants it (evidence, not taste).
**Guess:** competent colleague. She's a professional discussing literature and
requirements with an expert; she doesn't want a pet.

### N8 — How does she work with the drive day-to-day?
Finder? Schulcloud sync? Does the printing cycle start from a specific
folder? Does the drive live on the Mac or on a server?
**Why it matters:** the output-directory convention, the "corpus stays on her
disk" claim, and the first-run folder choice all sit on top of this; a
Schulcloud-synced corpus changes what "read-only" must tolerate (sync
conflicts, partial syncs).
**Who:** jan (he knows her machine) + one question to Pauline.
**Recommendation:** read the exact folder she works from; make the corpus
root a config value that can point at a synced folder without design changes;
the output dir stays local, next to (not inside) the corpus.
**Guess:** one local folder on her Mac, possibly mirrored to Schulcloud;
printing from Word; no exotic setup.

### N9 — What does "v1 done" mean to *her*?
**Why it matters:** anchors the MVP success test in her language, not ours.
**Who:** Pauline, at the thin-slice handoff ("here's what 'working' means to
us — is that your bar?").
**Recommendation:** the five items in `01` §10, restated to her as: "you can
ask it what you have on anything, correct it when it's wrong, and ask it for a
worksheet you can print." If her bar is higher or lower, the MVP boundary moves
*before* the two-week window starts.
**Guess:** "I can ask for a Balladen sheet and get something I'd print without
shame."

### N10 — Product name for her (draft Q6, still open)
**Why it matters:** small, but the name is the first trust signal ("what *is*
this thing").
**Who:** jan + Pauline, at P8.
**Recommendation:** keep **Dewey** as the working name — it is short,
explainable ("the Dewey catalog of your stuff," the archivist pun survives in
German: "der Dewey für deine Sachen"), and it already names the reframe's
center (classification/archive). Offer her three candidates at P8 and let her
keep, reject, or coin.
**Guess:** she'll coin her own and it'll be better.

## 2. The P0 conversation (the questions that fit in one chat with Pauline)

N2 (show me the drive), N3 (what does "clean up" mean to you), N4 (what do you
make most), N5 (what is AIS.chat like for you), N6 (is it okay if generation
text goes to the cloud), N7 (how do you want me to talk to you), N8 (where
does the drive live), plus: the backup question and the week of prompts. One
conversation, over coffee, no screens. Everything else in this doc is
derivable from it or from P0's slice.

## 3. Status of the draft's Q1–Q9

| Draft Q | Status after the reframe |
|---|---|
| **Q1 — Model routing** | Still open, still blocking for P6. **Updated:** the cheap classify route now has a concrete candidate — the local Qwen3.8-27B Q4 (jan's own words: the agent *is* that model). Two actions: (a) check Q1a (company endpoint terms) before wiring it as a test route; (b) run the **route probe** — one real Arbeitsblatt through the local 27B vs. the cloud route, judged against her exemplar (`50-first-build.md` §4) — before treating routes as settled. The probe also answers the live question "does the German idiom hold on a 27B." |
| **Q2 — German locale** | Unchanged; English chrome acceptable for v1; agent language German regardless. The catalog screen's German display text (summaries, nudges, error messages) is *agent-side* and German from day one — the locale pack remains a UI-chrome matter. |
| **Q3 — `.docx` stack** | Unchanged; P5 experiment. **Added:** P0's exemplars (one real Arbeitsblatt + one real Erwartungshorizont) are the experiment's test inputs — the round-trip test uses *her actual* house format, not a reconstruction. |
| **Q4 — Corpus reality** | **Upgraded to the top P0 question** (it is now N2, split). The census must include: file count, size, format mix, scan share, duplicate rate by eye, folder-shape, backup status. The suggested-structure report's design (A4) also depends on how nested the mess actually is. |
| **Q5 — Retrieval/interaction fixture** | Unchanged; **extended:** the two-week window's usage log (`20` §3) grows the fixture organically — her real queries during real use are the fixture's second harvest. |
| **Q6 — Name + first-run shape** | First-run shape is now specified (Opening B, `10` §5); name still open (N10). |
| **Q7 — Images** | Unchanged; Q4-gated (scan count) and Q1-gated (image-capable routes). Musik stays blocked on it. |
| **Q8 — Interaction memory** | Recommendation confirmed: preferences-document pattern, escalate only on measured recall failure. **Clarified:** exemplar retrieval (A1) answers half of "in meinem Stil" — *style* comes from her files, not from memory. What the preferences doc must carry is the other half: *taste and standing instructions* ("two-column tasks when there are images," "always the Name/Klasse/Datum Kopfzeile," "I never use the operator 'erörtern' in Sek I"). Different content, same cheap pattern — the reframe shrank Q8, which is good news. |
| **Q9 — TypeScript comfort** | Unchanged; non-blocking. The reframe adds no new type-magic surface (the catalog screen v2 is UI, not type machinery). |
