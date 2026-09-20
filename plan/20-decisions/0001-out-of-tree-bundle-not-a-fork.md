# 0001 — Out-of-tree bundle set against pinned dsh, not a fork

Date: 2026-09-20
Status: proposed
Deciders: jan, agent

## Context

jan's stated plan was "a fork or my own version of the deepseek-harness".
dsh is MIT-licensed, at `0.1.6-alpha.2`, and its own README says: "**THERE WILL BE COMPATIBILITY-BREAKING CHANGES.**"
The checkout at `/home/hagbard/dev/deepseek-harness` has ~4471 merged PRs, vendored Cordis under `vendor/`, and a CI surface that includes per-file 100% coverage, `doc-sync` gates, bilingual docs, and snapshot-recorded e2e.

dsh also documents a supported out-of-tree extension path: a **bundle** npm package declaring `dsh.bundle.patch` → `cordis.patch.yml`, composed into a **profile** declaring `dsh.profile`, installed with `dsh plugin add`. Patch rows are applied in layer order and any row can be replaced by id from a higher layer.

## Decision

`deweyHarness` is an out-of-tree bundle set plus a profile, **in this repo**, against dsh consumed as a dependency. dsh is pinned to an exact version; we never develop inside a dsh branch or fork.

*Clarification added during review (record was still `proposed`): jan read this as "develop within a branch of deepseek-harness". It is the opposite — this repo stays standalone. His stated instincts are exactly the mechanism: we control our own package and versioning; if upstream moves faster than we can follow, we simply stay on the pinned lower version until we choose to walk the upgrade checklist in `40-upstream-contract.md`. "Accepting breaking changes" is true but passive: the pin means they cannot reach us until we bump it.*

Concretely:

- One npm package per capability we own (retrieval service, ingestion, metadata extraction, German language pack, Dewey bundle, Dewey profile). Naming and count are open; the constraint is that a bundle is authored and distributed and a profile is booted, and nothing is both.
- `package.json` pins dsh to an exact version. Upgrading is a deliberate act with a checklist, not a `pnpm up`.
- A `40-upstream-contract.md` enumerates every dsh seam and package we touch, so an upgrade is mechanical re-verification rather than archaeology.

## Alternatives considered

- **Fork.** Full control, no upstream churn surprises. Cost: we own 4471 PRs of merge debt against a codebase that explicitly promises breakage, plus their CI gates, plus the fact that the interesting work (retrieval, pedagogy data) is 100% additive and 0% in conflict with core. Rejected: the cost is structural and permanent, the benefit is hypothetical.
- **Patch upstream with our features.** Ideal in principle — retrieval is a genuine dsh gap. Rejected for v1: alpha API, their review process, and our timeline is "Pauline needs this". Revisit once the design has survived her actual use; a well-shaped retrieval seam is a plausible upstream contribution later.
- **MCP server only.** Fastest path: write a retrieval MCP server, mount it with `dsh-mcp-client`, zero bundle authoring. Rejected as the *whole* answer because we also need a German locale, a persona, presets, skills, and an ingestion job — those are bundle-side. Kept as a possible implementation detail for the retrieval tool alone; see `0003`.

## Consequences

- We can upgrade dsh without a merge, but we can also be broken by an upgrade without a merge conflict to warn us. Hence the pinned version and the upstream-contract checklist.
- No ability to change core behavior. If a needed change is core-side, we are blocked until we either find a seam or reverse this decision.
- We must declare all dsh-facing dependencies explicitly; `cordis.yml` bare plugins must appear in the resolver manifest's `dependencies`.
- We escape dsh's gates by default and must choose which to keep. That choice is `0006`.
