# Engineering AI Team

A Claude Code setup that turns a plain-English feature request into a committed, reviewed, and deployed feature — automatically.

## What is this?

This repo provides an AI engineering team built on Claude Code's agent and slash command system:

- **Tech Lead** — reads the PRD and your codebase, then produces a precise Technical Spec (file paths, function names, acceptance criteria) without writing any code.
- **Engineer** — implements exactly what the spec says, writes tests inline, runs the linter, and commits.
- **Reviewer** — diffs the branch against main, checks for correctness, security, performance, and scalability issues, then either approves (producing a PR description and changes summary) or rejects with specific, actionable fix instructions.
- **DevOps** — deploys the service to an environment (local via Docker Compose, staging/prod via AWS ECR + ECS) or provisions a new environment from scratch.

If the reviewer rejects, the engineer fixes the issues and the reviewer runs again — up to one iteration. The build pipeline is orchestrated by `/eng-team`; deployment is a separate `/devops` command you run manually.

## Goal

The goal is to eliminate the manual back-and-forth of planning, implementing, and reviewing small-to-medium engineering tasks. You describe what you want; the team figures out how to build it, builds it, and tells you whether it's ready to ship.

## Directory structure

```
.claude/
  agents/
    tech-lead.md         # Tech Lead agent definition
    engineer.md          # Engineer agent definition
    reviewer.md          # Reviewer agent definition
    devops.md            # DevOps agent definition
  commands/
    eng-team.md          # /eng-team orchestrator slash command
    eng-team-context.md  # /eng-team-context context generator
    devops.md            # /devops deploy/setup command
CLAUDE.md.sample         # Template & documentation for CLAUDE.md (includes ## Deployment section)
```

## How to use

### Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated
- This repo's `.claude/` directory copied into (or checked out at the root of) your own project

### 1. Generate CLAUDE.md with `/eng-team-context`

Instead of manually filling in `CLAUDE.md`, use the context generator to automatically analyze your repo structure:

```bash
/eng-team-context /path/to/your/repo
```

This command will:
- Analyze your directory structure, language, frameworks, and tooling
- Detect test frameworks, linters, entry points, and external dependencies
- Generate a high-quality `CLAUDE.md` in your target repo that matches the template format

**Example:**
```
/eng-team-context ./
```

The generated `CLAUDE.md` is optimized for the eng-team workflow and can be re-run anytime your repo structure or tooling changes to keep it up-to-date.

### 2. (Optional) Customize CLAUDE.md

If needed, manually refine the generated `CLAUDE.md`:
- Add project overview details
- Clarify architecture diagrams or patterns
- Document any team-specific conventions
- List files or directories that should never be touched

For reference, see `CLAUDE.md.sample` which contains the template structure and documentation.

### 3. Run the eng-team workflow

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

### 4. Wait for the pipeline

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

### 5. Push and open the PR

```bash
git push origin <branch_name>
```

Then open a PR and paste the contents of `PR_DESCRIPTION.md` as the description.

### 6. Deploy with `/devops`

Once your PR is merged (or whenever you want to deploy), use the DevOps agent:

```
/devops setup local       # first-time: start local Docker services
/devops deploy local      # redeploy locally after changes
/devops deploy staging    # build image, push to ECR, deploy to ECS staging
/devops deploy prod       # deploy to ECS production
```

**Prerequisites for cloud deploys:**
- AWS CLI configured (`aws configure` or env vars)
- Docker installed and running
- `## Deployment` section filled in your repo's `CLAUDE.md` (auto-generated by `/eng-team-context`)

The agent writes a full run log to `.devops/DEVOPS_REPORT_<timestamp>.md` and a JSON scratchpad to `.devops/run_<timestamp>.json`.

**Setup a new environment from scratch:**

```
/devops setup staging
```

If no IaC is found, the agent scaffolds `.devops/aws-setup.sh` (ECR repo + ECS cluster creation), a task definition template, and step-by-step instructions — then defers to you to run them.

## What gets produced

- A feature branch with working, tested code
- A conventional-commit message
- `PR_DESCRIPTION.md` — what changed, why, how to test it
- `CHANGES_SUMMARY.md` — files changed, verification steps, config surface
- `.eng_team/task_<id>.json` — full audit trail (PRD → spec → implementation → review)

## Tips

- **Be specific in your PRD.** "Add rate limiting" is harder to act on than "Add a sliding-window rate limiter to the cart API, max 100 req/min per user, backed by Redis, fail-open on Redis errors." More detail in → less ambiguity out.
- **Keep CLAUDE.md up-to-date.** Whenever your project structure, dependencies, or tooling changes, re-run `/eng-team-context` to regenerate CLAUDE.md. A fresh, accurate CLAUDE.md cuts agent run time by 30–50%.
- **Review the spec before the engineer runs.** The orchestrator runs the full pipeline automatically, but you can pause after Phase 1 to read the Technical Spec in the scratchpad JSON and catch misunderstandings before any code is written.
- **The reviewer is a quality gate, not a rubber stamp.** It checks correctness, security, and performance against a concrete checklist. If it rejects, read `critical_issues` in the scratchpad — those are real problems to fix.

## Workflow summary

```
1. Add eng-team to your repo:
   → Copy .claude/ directory to your project root

2. Generate CLAUDE.md (includes ## Deployment section):
   → /eng-team-context ./

3. Run eng-team workflow:
   → /eng-team <your PRD>

4. Review & merge:
   → Push branch, open PR, paste PR_DESCRIPTION.md

5. Deploy:
   → /devops deploy staging
   → /devops deploy prod

6. (Repeat) Keep CLAUDE.md fresh:
   → /eng-team-context ./ (whenever structure changes)
   → /eng-team <next PRD>
```
