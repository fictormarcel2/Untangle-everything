# Untangle: clean up AI-generated code

An agent skill for Claude Code, Cursor and Codex that audits and refactors a codebase AI built, without letting the AI make the decisions.

You shipped fast with a coding agent. Months later the repo has three ways to authenticate, tests that pass no matter what, files called `service_v2_final.ts`, and nobody who can explain why half of it exists. The code is rarely the real problem. The problem is forty decisions that were never made.

`untangle` walks an agent through making those decisions with you, one at a time, and leaves behind a codebase that agents can keep working in safely.

## When to use it

- You need to clean up a vibe-coded app or an AI-generated codebase before building more on it.
- You inherited AI-written code and want a code audit before you trust it.
- The repo has duplicate code, dead code and several patterns for the same job.
- The tests are green but nobody believes them.
- Every fix breaks something unrelated, and the technical debt keeps growing.
- You want coding agents to stop making the mess worse.

## What it does

| Phase | What happens | Stops for you? |
|---|---|---|
| **1. Audit** | Read-only. Runs your build and tests, maps the repo, hunts 13 families of tangles, breaks business rules on purpose to see if the tests notice, and scores a 14-point scorecard. | Yes, with a report |
| **2. Triage** | Marks every finding keep, fix or remove, ranked by risk. Recommends one winner wherever implementations compete. | Yes, for approval |
| **3. Refactor** | Works through approved items in small slices on a branch. Each slice gets a safety net, one change, a full check run and its own commit. | Only when a new decision is needed |
| **4. Sustain** | Turns decisions into lint rules and CI checks where a machine can enforce them, and into `AGENTS.md` and ADRs where it cannot. | No |

You can stop after any phase. The audit alone is useful.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills) (Claude Code, Cursor, Codex and others):

```bash
npx skills add fictormarcel2/untangle-everything
```

As a Claude Code plugin:

```
/plugin marketplace add fictormarcel2/untangle-everything
/plugin install untangle@untangle-everything
```

Manually: copy `skills/untangle/` into `~/.claude/skills/`.

## Use

```
/untangle                  start, or resume from UNTANGLE.md
/untangle audit            audit only
/untangle audit src/api    audit one area
/untangle refactor         continue approved slices
```

Or ask in plain language: "audit this repo", "untangle this codebase".

## What it leaves in your repo

- `UNTANGLE.md`: the ledger. What was read, every finding with evidence, every decision, every slice and its commit. The agent resumes from it, so a cleanup can span many sessions.
- Tests that fail when behaviour is wrong.
- `docs/adr/`: one short record per decision.
- `AGENTS.md`: the one way to do each thing, so the next agent does not regrow the tangle.
- Lint and CI rules that enforce the decisions.

## How it is designed

**The human decides.** The tool that created the inconsistencies cannot be trusted to pick which ones survive. The agent gathers evidence and recommends; nothing is deleted or merged without an approved row in the ledger.

**Audit before any edit.** A refactor without a map removes the wrong things. The first phase changes nothing.

**No refactor without a net.** In AI-built repos the tests are usually the least trustworthy part. Before touching a slice, the agent proves its tests can fail, or writes characterization tests that pin current behaviour.

**Evidence, not smells.** Every finding carries a file, a line and the command that showed it. Code counts as dead only after a search for dynamic references.

**State lives in a file.** Whole-codebase work outlasts a context window. The ledger keeps coverage claims honest and lets work resume where it stopped.

**Subtract.** No rewrite, no new dependencies, no compatibility shims. The line count should go down.

**Guardrails over good intentions.** A rule a machine can check becomes a check. Prose rules are the fallback.

## What it hunts

Competing implementations · tests that cannot fail · stubs and mock data in production paths · swallowed errors and silent fallbacks · suppressed type, lint and CI checks · dead code, unused dependencies and agent debris · god files and dumping-ground `utils` · business rules enforced nowhere or only in the UI · committed secrets and wildcard permissions · nonexistent or abandoned packages · READMEs and comments that are no longer true · schema drift · errors that reach nobody.

The full list with detection hints is in [catalog.md](skills/untangle/catalog.md).

## Limits

- The security check covers common mistakes in AI-generated code. It is not a penetration test.
- Branch protection, alert routing and production behaviour cannot be verified from a repository. The skill marks these "not verifiable" and asks you.
- On large repos the agent reads the riskiest areas closely and only maps the rest. The ledger states which is which.

## Structure

```
skills/untangle/
  SKILL.md       the process and its rules
  catalog.md     what to hunt, loaded during Audit
  templates.md   ledger, scorecard, ADR and AGENTS.md templates
```

## Credits

The four phases follow the [Untangle framework](https://untangle.work/framework/), and the scorecard builds on its [health checklist](https://untangle.work/checklist/). The problem framing in Triage draws on [caldoai/skills](https://github.com/caldoai/skills). This project is independent and not affiliated with either.

## License

[MIT](LICENSE)
