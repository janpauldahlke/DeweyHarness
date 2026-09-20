# 0006 — Which dsh engineering gates we adopt out-of-tree

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

`0001` puts us out-of-tree, which means dsh's CI gates no longer apply to us automatically. That is a choice with a default value of *none*, and "none" is how a two-person project rots. jan named this risk himself: long-running agent projects become difficult, and he wants hygiene.

The gates worth keeping are the ones that catch classes of bug an agent reliably introduces. The gates worth dropping are the ones that exist to coordinate a large distributed team.

## Decision

**Adopt:**

| Gate | Why it earns its cost here |
|---|---|
| `strict: true`, `noImplicitAny`; every remaining `any` explains why | Cheap, catches the most common agent-introduced unsoundness |
| Unit tests for every owned module; behavior-describing, not correctness-proving | We own ingestion, retrieval, metadata, provenance — all bug-dense |
| **A recorded retrieval fixture** (her real queries → known-correct sources) run in CI | This is the acceptance bar in `00-charter.md` made executable. Without it, retrieval quality is unfalsifiable and the project drifts. Highest-value test we own |
| Append-only decision records (`20-decisions/`), written at decision time | The agent-hygiene mechanism. Dated, immutable, records rejected options |
| A `40-upstream-contract.md` listing every dsh seam/package touched | Turns a dsh upgrade into a mechanical re-verification checklist instead of archaeology |
| Model-visible ⟺ logged; registrations are effects; waterfalls call `next()`; fail-loud config; no hardcoded tunables; branded ids | dsh invariants that are cheap to honor and brutal to debug when violated. Listed in `10-dsh-capability-map.md` |
| Source files read-only; index rebuildable (`0004`) | Converts an unbounded class of state-drift bugs into "press rebuild" |
| Trailing-newline and lint hygiene via lefthook or a pre-commit hook | Trivial to keep, keeps diffs clean for review |

**Reject:**

| Gate | Why it is dropped |
|---|---|
| Per-file 100% coverage on `src` | dsh's bar for a large team with CI time to burn. For us it incentivizes low-value tests over the one high-value fixture above. Coverage target: meaningful paths, not a number |
| Bilingual `en`/`zh` docs and `doc-sync` gate machinery | We are not shipping to that audience. Docs are English for the repo; the *product UI* is German (`10-dsh-capability-map.md`) — different thing, and the locale work is product code, not doc translation |
| Snapshot-recorded e2e against a real API key | Requires committing key-gated fixtures and re-recording on model change. Our model routing is still open (`30-open-questions` Q1); snapshot e2e is premature until it is pinned |
| The `AGENTS.md` prose standard, doc budgets, ban-lists | Useful discipline for a 4000-PR repo's consistency; overhead for one maintainer plus agents. We keep the spirit (concrete prose, no metaphors) without the gate |

**Deferred, revisit before v1 ships:**

- Typecheck/lint/build wiring: adopt whatever dsh-compatible tooling makes `pnpm test` and `pnpm typecheck` work in our package set. Exact config is an implementation detail, not a decision record.
- Whether to run a subset of dsh's own gates when we bump the pinned version. Probably yes, as a smoke test. Decide at first upgrade.

## Alternatives considered

- **Adopt all dsh gates.** Maximum rigor. Rejected: 100% coverage + bilingual docs + snapshot e2e is a tax sized for a distributed team, and paying it on a one-user project means paying it out of the time that should go to retrieval quality and pedagogy data.
- **Adopt none, move fast.** Rejected: this is the "long-running agent project becomes difficult" failure jan explicitly wants to avoid. The decision records and the retrieval fixture are the two things that keep an agent-assisted codebase honest over months.

## Consequences

- The retrieval fixture is now a first-class deliverable, not an afterthought. It needs real queries from Pauline early — which means the first practical step after the spec is *collecting her actual requests*, not writing code.
- Every decision gets a record at decision time. That is a discipline on both of us, and it is the thing that makes a long agent collaboration reviewable.
- We are explicitly responsible for the gates we dropped. If a class of bug appears that 100% coverage would have caught, the fix is to add the targeted test, not to lament the dropped gate.
