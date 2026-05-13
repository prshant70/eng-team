# Engineering AI Team

A Claude Code setup that turns a plain-English feature request into a committed, reviewed feature branch — automatically.

## What is this?

This repo provides a three-agent AI engineering team built on Claude Code's agent and slash command system:

- **Tech Lead** — reads the PRD and your codebase, then produces a precise Technical Spec (file paths, function names, acceptance criteria) without writing any code.
- **Engineer** — implements exactly what the spec says, writes tests inline, runs the linter, and commits.
- **Reviewer** — diffs the branch against main, checks for correctness, security, performance, and scalability issues, then either approves (producing a PR description and changes summary) or rejects with specific, actionable fix instructions.

If the reviewer rejects, the engineer fixes the issues and the reviewer runs again — up to one iteration. The whole pipeline is orchestrated by a single slash command: `/eng-team`.

## Goal

The goal is to eliminate the manual back-and-forth of planning, implementing, and reviewing small-to-medium engineering tasks. You describe what you want; the team figures out how to build it, builds it, and tells you whether it's ready to ship.

## Directory structure

```
.claude/
  agents/
    tech-lead.md      # Tech Lead agent definition
    engineer.md       # Engineer agent definition
    reviewer.md       # Reviewer agent definition
  commands/
    eng-team.md       # /eng-team orchestrator slash command
CLAUDE.md             # Codebase guide (fill this in for your repo)
```

## How to use

### Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated
- This repo's `.claude/` directory copied into (or checked out at the root of) your own project

### 1. Fill in CLAUDE.md

`CLAUDE.md` is what the agents read instead of exploring your repo from scratch. It should document:

- Top-level directory layout
- How to run tests and the linter
- Code style conventions (indentation, naming, error response format)
- Git branch and commit message conventions
- Files and directories that must not be touched

A complete `CLAUDE.md` template is already in this repo — fill it in for your project. Accurate documentation here cuts agent run time by 30–50%.

### 2. Run the slash command

Open Claude Code in your project and type:

```
/eng-team <your PRD here>
```

**Examples:**

```
/eng-team Add rate limiting to the cart API — max 100 requests per minute per user, stored in Redis, fail-open if Redis is down

/eng-team Fix the null pointer in OrderService.processRefund when the original payment record is missing

/eng-team Refactor the auth middleware to extract token validation into a standalone utility so it can be reused in the webhook handler
```

### 3. Wait for the pipeline

The orchestrator runs three phases automatically:

| Phase | Agent | Output |
|-------|-------|--------|
| 1 — Spec | Tech Lead | Technical Spec written to `.eng_team/task_<id>.json` |
| 2 — Implementation | Engineer | Code committed to a new feature branch |
| 3 — Review | Reviewer | `PR_DESCRIPTION.md` and `CHANGES_SUMMARY.md` in `.eng_team/` |

When done, the orchestrator prints a summary:

```
═══════════════════════════════════════════════
  Engineering AI Team — Done

  Task:     Add rate limiting to the cart API
  Branch:   feat/cart-rate-limiter
  Review:   ✓ Approved

  Output files:
  → .eng_team/PR_DESCRIPTION.md
  → .eng_team/CHANGES_SUMMARY.md
  → .eng_team/task_1234567890.json

  Next steps:
  → Push branch:  git push origin feat/cart-rate-limiter
  → Open PR and paste the contents of PR_DESCRIPTION.md
═══════════════════════════════════════════════
```

### 4. Push and open the PR

```bash
git push origin <branch_name>
```

Then open a PR and paste the contents of `PR_DESCRIPTION.md` as the description.

## What gets produced

- A feature branch with working, tested code
- A conventional-commit message
- `PR_DESCRIPTION.md` — what changed, why, how to test it
- `CHANGES_SUMMARY.md` — files changed, verification steps, config surface
- `.eng_team/task_<id>.json` — full audit trail (PRD → spec → implementation → review)

## Tips

- **Be specific in your PRD.** "Add rate limiting" is harder to act on than "Add a sliding-window rate limiter to the cart API, max 100 req/min per user, backed by Redis, fail-open on Redis errors." More detail in → less ambiguity out.
- **Keep CLAUDE.md accurate.** If the test command is wrong, the engineer will commit with failing tests. Update it whenever you change the project structure or tooling.
- **Review the spec before the engineer runs.** The orchestrator runs the full pipeline automatically, but you can pause after Phase 1 to read the Technical Spec in the scratchpad JSON and catch misunderstandings before any code is written.
- **The reviewer is a quality gate, not a rubber stamp.** It checks correctness, security, and performance against a concrete checklist. If it rejects, read `critical_issues` in the scratchpad — those are real problems to fix.
