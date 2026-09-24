# Goaldy — docs moved

**Moved 2026-09-24.** No Goaldy documentation lives in this vault any more.

Everything is in the **`nisan-tagar/goaldy` app repository** (private), where a doc
change ships in the same pull request as the change it describes:

| Document | Path in the app repo |
|---|---|
| PRD — product requirements, features, UX decisions | `docs/project/PRD.md` |
| TDD — tech design, architecture, schema, data flow | `docs/project/TDD.md` |
| Market analysis, GTM strategy and the financial model | `docs/business/market-analysis-and-gtm.md` |
| Specs · phased plans | `docs/superpowers/specs/` · `docs/superpowers/plans/` |
| Bug register · QA punch lists · design notes | `docs/bugs/` · `docs/qa/` · `docs/design/` |

## Why the old copies were deleted rather than left in place

The PRD and TDD moved to the app repo on 2026-09-16 (PRD v2.15). The copies left
behind here were never updated again and had fallen roughly fifteen versions
behind — they still described `reported_value` as a live column, the metrics
engine as unbuilt, and both registered bugs as unfixed. All three statements are
now false.

A stale copy of a source-of-truth document is worse than no copy: a reader who
finds it has no signal that it is wrong. They are deleted; the history remains in
this repository's git log if it is ever needed.

The rationale and the full repository topology are recorded in the app repo at
**TDD §14, "Repository and Distribution Topology"**.

## What is still here

`docs/goaldy/` — the landing page artifact (`index.html` and its assets).
Per TDD §14 D4 it moves to a new **public** repository alongside the operator
guides and the issue tracker. Until that repo exists, this is the one Goaldy
artifact outside the app repo.
