# Templates

## Ledger: `UNTANGLE.md`

Statuses: `open` → `approved` → `done`, or `wontfix` (with reason), or `suspect` (needs the human).

```markdown
# Untangle ledger

Phase: audit | triage | refactor | sustain | done
Branch: untangle/<topic>
Updated: <YYYY-MM-DD>

## Baseline

| Step | Command | Result |
|---|---|---|
| Install | | |
| Build | | |
| Test | | |
| Lint | | |
| Typecheck | | |

## Coverage

| Path | Depth | Notes |
|---|---|---|
| src/auth/ | read | |
| src/reports/ | mapped | |
| infra/ | not opened | |

## Scorecard

| # | Check | Before | After | Evidence |
|---|---|---|---|---|

## Findings

| ID | Risk | Finding | Evidence | Decision | Status |
|---|---|---|---|---|---|
| F-001 | high | Three auth paths reachable in production | src/api/a.ts:12, src/mw/auth.ts:40, src/legacy/jwt.ts:8 | fix → D-001 | approved |

## Decisions

| ID | Concern | The one way | Why | Resolves | Enforced by |
|---|---|---|---|---|---|
| D-001 | Auth | `requireUser` middleware | 41 of 52 call sites, only path with tests | F-001, F-007 | lint rule `no-direct-token-read`, ADR 0001 |

## Slices

| # | Slice | Findings | Net | Status | Commit |
|---|---|---|---|---|---|
| 1 | Characterization tests for checkout totals | F-003 | n/a | done | a1b2c3d |
| 2 | Delete unrouted v1 handlers | F-004 | slice 1 | approved | |

## Questions for the human

- Is branch protection on `main` requiring CI? (not verifiable from the repo)
```

## Scorecard

Score each check pass, fail, or `n/v` (not verifiable from the repo, with the question to ask). Which checks fail matters more than the count.

| # | Check | Passes when |
|---|---|---|
| 1 | Clean checkout | A fresh clone builds and runs by following the README alone |
| 2 | Tests notice | A deliberately broken business rule turns the suite red |
| 3 | One auth path | Exactly one authentication implementation is reachable in production |
| 4 | One data path | New work has one data-access route; no business rule is duplicated across query strings |
| 5 | No secrets | No live credential in the tree or anywhere in history |
| 6 | Nothing uncalled | No unused features, unrouted endpoints or stale flags |
| 7 | No dumping ground | `utils`-style folders are small and hold no business logic |
| 8 | CI blocks | Lint, typecheck, build and tests run in CI, and a failing run cannot merge |
| 9 | Invariants located | Each key business rule has one server-side enforcement point someone can name |
| 10 | Dependencies sound | Lockfile committed, no fixable vulnerabilities, nothing abandoned or nonexistent |
| 11 | Failures are loud | No swallowed errors on important paths; errors reach a person |
| 12 | Explainable | The five highest-churn files can be explained: what, why built this way, what breaks if deleted |
| 13 | Checks are on | No type, lint or CI suppression without a written reason |
| 14 | Agents aligned | One agent rules file exists, is the single source, and matches the code |

## ADR: `docs/adr/NNNN-title.md`

```markdown
# NNNN. <Decision in one line>

Status: accepted
Date: <YYYY-MM-DD>

## Context
What competed, with evidence (paths, call-site counts).

## Decision
The one way from now on.

## Consequences
What was removed or migrated. What gets harder.

## Enforced by
The lint rule, CI step or test that fails when this is violated. "Review only" if none is possible.
```

## Agent rules: `AGENTS.md`

Fill from the Decisions table. Every row must be true of the code today.

```markdown
# Agent rules

## Commands
- Install: `...`
- Test: `...` (must pass before any commit)
- Lint and typecheck: `...`

## One way to do each thing

| Concern | Use | Example | Never |
|---|---|---|---|
| Auth | `requireUser` middleware | src/auth/middleware.ts | read the token in a handler |

## Where things go
- <kind of code> → <directory>

## Before you add anything
- Search for an existing helper, type or pattern and reuse it.
- Ask before adding a dependency.
- Change the original file. No `_v2`, `_new` or `_old` copies.
- Fix the cause of a type, lint or test failure. Do not silence it.
- Let errors propagate. No empty catch, no fallback that hides a failure.
- A red test means the change is wrong until shown otherwise. Do not edit the assertion to pass.
- Decisions live in `docs/adr/`. A change that contradicts one needs a new ADR first.
```
