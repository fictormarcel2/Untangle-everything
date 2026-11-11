# Tangle catalog

What to hunt during Audit. The signals are prompts for where to look. A hit becomes a finding only with `file:line` evidence and a stated risk.

## Starting commands

| Question | Command |
|---|---|
| What changes most? | `git log --since=1.year --format= --name-only \| sort \| uniq -c \| sort -rn \| head -30` |
| What is biggest? | `git ls-files \| xargs wc -l \| sort -rn \| head -30` |
| What changes together? | Pairs of files that keep appearing in the same commits of `git log --name-only` |
| Secrets in history? | `gitleaks` or `trufflehog` over the full history, not just the working tree |
| Dead code? | `knip` (JS/TS), `vulture` (Python), `deadcode` (Go), `cargo machete` (Rust) |
| Vulnerable dependencies? | `npm audit`, `pip-audit`, `govulncheck`, `cargo audit` |

## 1. More than one way

The defining AI tangle: each prompt got a fresh solution to a problem the repo had already solved.

| Signal | Where to look |
|---|---|
| Several auth paths reachable in production | Search `session`, `token`, `cookie`, `jwt`, `middleware`, `guard`, `permission`; group the hits by approach |
| Several data-access paths | Raw queries in handlers, an ORM, a service layer, direct client calls in jobs |
| Several libraries for one job | HTTP clients, date libraries, validators, state managers, test runners in the manifest |
| Several config mechanisms | `.env*` files, config modules, hardcoded constants, env reads scattered through the code |
| Several error or logging styles | Exceptions vs result objects vs `null`; `print`/`console.log` beside a logger |
| Copies with suffixes | `*_v2`, `*_new`, `*_old`, `*_final`, `*_backup`, `*.bak`, `* copy.*`, `Dockerfile.prod` beside `Dockerfile.final` |
| Re-implemented helpers | Functions with near-identical bodies or names (`formatDate`, `format_date`, `dateFormat`) |
| One concept, several names | `user` / `account` / `member` / `customer` for the same entity |
| Several lockfiles or package managers | `package-lock.json` with `yarn.lock`; `requirements.txt` with `pyproject.toml` |

Usual decision: pick a winner, migrate call sites, delete the rest, record an ADR.

## 2. Tests that lie

| Signal | Where to look |
|---|---|
| Cannot fail | `assert True`, `expect(true)`, no assertion at all, assertions inside a `catch` that never runs |
| Tests the mock | The unit under test is itself mocked; assertions only on call counts or arguments |
| Mirrors the implementation | Expected value computed by the same formula as the code |
| Switched off | `skip`, `xfail`, `.only`, `xit`, `t.Skip`, commented-out test bodies |
| Silently skipped | Tests cancelled or skipped when a service (database, network) is missing, while the runner still reports zero failures |
| Bent to pass | Git history where the assertion changed in the same commit as the code it guards |
| Test branches in production code | `if TESTING`, `NODE_ENV === 'test'`, `isTest` flags |
| Snapshot of everything | Large snapshots that get regenerated on every failure |

Usual decision: delete tests that cannot fail; rewrite the rest to assert behaviour at a boundary.

## 3. Fake or unfinished

| Signal | Where to look |
|---|---|
| Stubs presented as done | `TODO`, `FIXME`, `placeholder`, "in a real implementation", "for now", `NotImplemented` |
| Hardcoded results | Functions that return a constant, sample data or a fixed ID on a production path |
| Mock data in production | `mockUsers`, `sampleData`, `fake*` imported outside tests |
| Half-wired features | UI that calls an endpoint which does not exist; endpoints nothing calls |
| Placeholder credentials | `changeme`, `your-api-key`, `admin/admin`, `localhost` URLs in shipped config |

Usual decision: finish it or remove it. The human decides which.

## 4. Silent failure

AI-written code often makes an error disappear instead of handling it.

| Signal | Where to look |
|---|---|
| Swallowed errors | Empty `catch`/`except`, `except Exception: pass`, `_ = err`, catch that only logs then continues |
| Masking defaults | `\|\| {}`, `?? []`, `.get(key, default)`, `getattr(obj, name, None)` on values that must exist |
| Lying health checks | Endpoints that return OK without checking a dependency |
| Timing as correctness | Fixed `sleep` before a dependency is assumed ready; retries with no limit |
| Unawaited work | Promises or tasks started and never awaited |

Usual decision: let it fail loudly at the point of failure; handle only where recovery is real.

## 5. Suppressed checks

| Signal | Where to look |
|---|---|
| Type escapes | `any`, `as unknown as`, `@ts-ignore`, `# type: ignore`, `interface{}` used to dodge a type |
| Lint escapes | `eslint-disable`, `# noqa`, `//nolint`, `#[allow(...)]` without a stated reason |
| CI escapes | `continue-on-error: true`, `\|\| true`, `--no-verify`, `--force`, steps commented out |
| Relaxed config | `strict: false`, `skipLibCheck`, rules set to `off`, coverage thresholds at zero |

Usual decision: fix the cause; keep a suppression only with a reason written on the same line.

## 6. Surplus

| Signal | Where to look |
|---|---|
| Unreachable code | Dead-code tool output, unrouted handlers, unused exports, old API versions |
| Unused dependencies | Manifest entries nothing imports |
| Abstraction with one user | Interfaces with one implementation, factories for one product, base classes with one child |
| Options never used | Config values and feature flags with one value everywhere; parameters every caller passes identically |
| Shims for nobody | Backwards-compatibility wrappers, re-exports and deprecated aliases in code that has no external users |
| Commented-out code | Blocks of code in comments |
| Agent debris | `SUMMARY.md`, `IMPLEMENTATION_NOTES.md`, `CHANGES.md`, `fix_*.py`, `test_manual.*`, scratch scripts in the root |
| Committed artefacts | Build output, vendored dependencies, generated files, `terraform.tfstate`, `.DS_Store`, logs |

Usual decision: remove, after the dynamic-reach proof in SKILL.md.

## 7. Coupling and structure

| Signal | Where to look |
|---|---|
| God files | Files over ~800 lines or mixing transport, business rules and persistence |
| Dumping grounds | `utils`, `helpers`, `common`, `shared`, `misc` holding business logic |
| Hidden links | Files that change together in git history but do not import each other |
| Needless links | One module edited for unrelated reasons; a change in one feature breaking another |
| Wrong direction | Stable core importing volatile edges; circular imports |
| Rules in the wrong layer | Business rules in route handlers, UI components or SQL strings |

Usual decision: move code to where its reason to change lives. Restructure last, after removals.

## 8. Invariants

List the rules the business depends on (an unpaid order cannot ship; a user cannot read another user's data) and find where each is enforced.

| Signal | Where to look |
|---|---|
| Enforced nowhere findable | Nobody can point to the line |
| Enforced only in the client | Validation or permission checks in UI code with nothing server-side |
| Enforced in several places that disagree | The same rule with different thresholds or conditions |
| Missing per-object authorization | Endpoints that check login but not ownership of the record requested |

Usual decision: one server-side enforcement point per invariant, listed in a plain-language file.

## 9. Security hygiene

A check for the mistakes AI-generated code makes often. It is not a penetration test, and the report must say so.

| Signal | Where to look |
|---|---|
| Secrets | Keys in the tree or history, committed `.env`, `.aws/credentials`, plaintext passwords in config |
| Injection | SQL, shell commands or HTML built by string concatenation |
| Wildcards | CORS `*`, IAM `*:*`, `Principal: "*"`, security groups open to `0.0.0.0/0` |
| Safety switched off | TLS verification disabled, debug mode on, CSRF disabled, auth bypass flags |
| Containers and infra | Running as root, unpinned base images, no `.dockerignore`, admin-level service roles |

## 10. Dependencies

| Signal | Where to look |
|---|---|
| Packages that do not exist or are look-alikes | Check each unfamiliar name against the registry: age, downloads, maintainer |
| Floating versions | Ranges with no committed lockfile; `latest` tags |
| Abandoned or vulnerable | Audit tool output; last release years ago |
| Deprecated API use | Calls to APIs the installed version has deprecated or removed |

## 11. Docs and truth

| Signal | Where to look |
|---|---|
| README that does not work | Follow it in a clean checkout |
| Comments that narrate history | "now uses", "fixed", "updated to", "as requested", "previously" |
| Comments that contradict the code | Docstrings describing parameters or behaviour that no longer exist |
| Conflicting agent rules | `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `copilot-instructions.md` that disagree with each other or the code |
| Undocumented environment | Env vars read in code but absent from any template |

## 12. Data and schema

| Signal | Where to look |
|---|---|
| Schema drift | Migrations that do not produce the models the code uses |
| Hand-copied types | The same shape declared separately in backend, frontend and validation |
| Magic values | Status strings and IDs repeated as literals instead of one definition |

## 13. Operability

| Signal | Where to look |
|---|---|
| Errors go nowhere | No error reporting; logs nobody reads |
| Unstructured logs | No request ID, no levels, secrets or personal data in log lines |
| No way to tell it is broken | No alerting that reaches a person. Usually not verifiable from the repo: ask |
