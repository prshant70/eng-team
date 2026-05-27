# Quality Gates for Autonomous Engineering Teams

> How to ensure eng-team agents always produce high-quality code, stay on scope, and build human trust in AI-generated PRs.

---

## Why AI Code Quality Degrades

Before picking gates, it helps to name the failure modes:

- **Too little context** — the agent doesn't know the repo's conventions, so it invents patterns.
- **Too little scope discipline** — the agent over-engineers because nothing stops it.
- **No verification loop** — the agent writes code and hands it off without checking if it actually works or matches intent.
- **No adversarial review** — the same agent that wrote the code also "reviewed" it.

Each gate in this document targets one or more of these root causes.

---

## Stage 1: Pre-flight (Before a Single Line Is Written)

The highest-leverage point is *before* implementation starts. The agent must produce a **plan artifact** — a structured document that states:

- Which files will change
- Rough line count estimate
- Implementation approach
- How the approach maps to each acceptance criterion in the PRD

This costs almost nothing and surfaces the biggest risks before wasted compute.

The plan is checked against:

**Scope reasonableness** — If the plan touches more than a threshold number of files or LOC for a small feature, that's a flag to surface before implementation begins.

**Repo structure alignment** — Does the plan follow existing module boundaries, naming conventions, and architectural patterns? A well-maintained `CLAUDE.md` is the primary mechanism here — treat it as a constitution that all agents must read and cite in their plan.

**Test-first commitment** — The agent declares what tests it will write before writing any implementation. This forces real thinking about the contract, not just the code.

---

## Stage 2: In-flight Controls (While Implementing)

**Incremental, reviewable commits** — Rather than one giant diff at the end, each logical chunk (a new function, a schema change, a new component) should be a discrete commit. This makes the diff auditable incrementally and makes it far easier to spot drift.

**Self-critique step** — After writing each logical unit, the agent reads its own diff and answers:
- Is this the minimum change needed?
- Does it follow the pattern used elsewhere in the codebase?
- Am I introducing anything that wasn't in the PRD?

Catching drift mid-implementation is dramatically cheaper than catching it at review.

---

## Stage 3: Post-implementation Gates (Before PR Is Opened)

These are the mechanical, automated checks that form the quality floor.

### Tests Must Pass
The full existing test suite must pass before a PR is opened. If the agent breaks tests, the PR does not open. This is enforced mechanically, not left to the agent's judgment.

### Test Coverage on New Code
The agent is required to write tests for its own additions. Coverage thresholds apply to the **diff** — not just overall repo coverage — to catch cases where the agent ships logic with zero tests.

### Static Analysis and Linting
TypeScript strict mode, ESLint, formatters, and any other repo-configured tools must pass at zero-tolerance. The agent runs and fixes these locally before the PR opens.

### Diff Size Audit
Compare the size of the PR (files changed, LOC) against the stated complexity of the PRD. A one-sentence feature request that produces a 1200-line PR is a signal worth surfacing — it doesn't mean the PR is wrong, but it should trigger human scrutiny before merge.

### File Blast Radius Check
Which files were modified? If the agent touched a shared utility, a config file, or anything outside the expected module scope, that must be explicitly flagged in the PR description. Unexpected file changes are one of the most common sources of subtle regressions.

---

## Stage 4: The Adversarial Reviewer Agent

This is the highest-trust gate and the most important one to get right.

**The agent that writes the code must never be the sole reviewer.**

A separate agent instance — with fresh context and no attachment to the implementation — reads the PRD and the diff, then answers a structured checklist:

- Does every acceptance criterion have corresponding code and a test?
- Is there any code that wasn't required by the PRD?
- Are there patterns that diverge from the existing codebase?
- Are there obvious edge cases not handled?
- Is the PR description accurate and complete?

The output is a **structured review report** attached to the PR. When the human reviewer opens the PR, they see the AI reviewer's assessment alongside the diff — surfacing disagreements, flags, and open questions. This reduces the cognitive load on the human reviewer and focuses their attention where it matters.

---

## Stage 5: Building Trust Over Time

The gates above catch bad output in the moment. Sustained trust requires a feedback loop.

**Capture human corrections** — Every time a human reviewer modifies an AI-generated PR, that change should be captured — as an annotated example or a `CLAUDE.md` update. This creates a growing library of "this is what we do here and why," progressively calibrating future agents to the team's standards.

**Retrospective evals** — Periodically sample merged AI PRs, strip context, and ask a fresh agent: "How would you implement this PRD given this codebase?" If the approach diverges significantly from what was merged, the agents are drifting from what the team actually wants. Use those diffs to improve the `CLAUDE.md` and agent prompts.

---

## Implementation Priority

Implement in this order for the best return on investment:

| Priority | Gate | What it addresses |
|---|---|---|
| 1 | `CLAUDE.md` with explicit conventions | Gives agents repo context |
| 2 | Plan artifact + scope check | Catches over-engineering before it happens |
| 3 | Full test suite enforcement | Sets a non-negotiable quality floor |
| 4 | Adversarial reviewer agent | Builds human trust most directly |
| 5 | Diff size + blast radius audit | Catches subtle scope creep |
| 6 | Feedback capture loop | Compounds quality improvements over time |

---

## Summary

No single gate is sufficient because the failure modes are different at each stage. The combination of:

- A strong **pre-flight** (scope discipline + plan artifact)
- **Mechanical post-implementation gates** (tests, linting, diff audit)
- An **adversarial reviewer** (independent judgment on correctness and fit)

...covers the three biggest failure modes. The rest is refinement and iteration as the team builds its feedback corpus.

---

*Document authored from eng-team architectural discussion — May 2026.*
