# Code as Agent Harness — Implications for eng-team

Summary of how [Code as Agent Harness](https://arxiv.org/abs/2605.18747) (Ning et al., 2026) relates to **eng-team**, and a prioritized backlog for strengthening the harness.

---

## Article in one paragraph

The survey argues that in agentic systems, **code is not only output** — it is the **operational harness**: the executable substrate for reasoning, acting, environment modeling, and verification. A good harness makes behavior **executable, inspectable, stateful, and verifiable** over long horizons. Progress depends as much on harness engineering (tools, memory, oracles, control loops, multi-agent shared state) as on the base model.

---

## How eng-team already fits

eng-team is a **code-centric agent harness** for the slice PRD → spec → implementation → review → merge-ready PR:

| Paper layer | eng-team today |
|-------------|----------------|
| **Harness interface** | `CLAUDE.md`, `technical_spec`, Engineer edits, Reviewer `git diff` |
| **Harness mechanisms** | Orchestrator phases, bounded loops, `repo_context`, test/lint gates |
| **Multi-agent over code** | Tech Lead → Engineer → Reviewer via `.eng_team/task_*.json` (orchestrator-only; no peer chat) |
| **Verifiable closure** | Tests + linter + structured review checklist |

This aligns with **PHILOSOPHY.md**: bottom-up trust, narrow insertion point, diff-based review (output over intent).

The article does **not** suggest replacing this design. It names what to harden next: **oracle quality**, **shared state discipline**, **harness telemetry**, and **governed iteration**.

---

## Key upgrades (article → eng-team)

### 1. Scratchpad as program state

Extend `.eng_team/task_*.json` beyond narrative logging:

- `verification_evidence` (tests run, linter result, diff stats)
- `assumptions[]` with `verified_by` (test / diff hunk / reviewer item)
- Per-phase `read_set` / `write_set`
- Commit pins: `base_commit`, `spec_version`, `impl_commit`

*Paper: §2.3, §4.2, §5.2.4 — transactional shared program state.*

### 2. Verification stack (not only “tests passed”)

On approve, require an **evidence bundle** and explicit limits:

- What was checked (unit / integration / security hints / coverage on touched files)
- `untested_regions[]` — what the oracle does **not** prove
- For `complex` tasks: runnable `acceptance_checks` or test skeletons in the spec

*Paper: §5.2.1–5.2.2 — oracle adequacy and semantic verification beyond executable feedback.*

### 3. Harness-level evaluation

Log per-run **trajectory metrics** in the scratchpad:

- Phase durations, clarification/review cycles
- Recovery: each `critical_issue` linked to a fix commit
- `oracle_strength` (trivial vs full checklist, targeted re-review scope)

*Paper: §5.2.1 — evaluate the harness, not only final task success.*

### 4. Failure-type routing in the orchestrator

Route feedback by signal type:

| Signal | Action |
|--------|--------|
| `spec_gaps` | Tech Lead (max 1 cycle — existing) |
| Test failure | Engineer fix mode |
| Lint only | Engineer, narrow scope |
| Behavior vs spec | Tech Lead, not blind Engineer patch |
| Security/perf | Reviewer targeted re-review |

*Paper: §3.4 — plan → execute → verify with feedback-driven control.*

### 5. Action validation (lightweight harness boundary)

Pre-flight before Engineer acts:

- Edits only under `files_to_modify` / `files_to_create`
- No edits on `base_branch`
- Bash allowlist from `CLAUDE.md` (no destructive or secret-leaking commands)

*Paper: §2.2 — code mediates intent; filter invalid actions before execution.*

### 6. Human gates as durable state

Scratchpad fields: `human_gates` (`prd_approved`, `spec_approved`, `merge_approved`), `human_resolution` on escalation so later runs do not repeat the same failure.

*Paper: §5.2.5; **PHILOSOPHY.md** — the gate that stays human.*

### 7. Cross-task memory (optional, later)

`.eng_team/learnings.json` for recurring reviewer findings, flaky areas, repo-specific patterns — opt-in, governed.

*Paper: §3.2 — memory and context engineering.*

### 8. Harness evolution with regression discipline

Golden fixture repos + expected scratchpad phases; prompt/checklist changes only with held-out regression tasks and explicit change contracts.

*Paper: §5.2.3 — self-evolving harnesses without regression.*

---

## What to keep (already strong)

- Bottom-up, verifiable slice (code → tests → diff review)
- Orchestrator-owned control flow; bounded loops; targeted re-review
- Role/tool separation (Tech Lead no Edit; Reviewer judges diff not intent)
- `/eng-team-context` as environment bootstrapping
- Scratchpad as audit trail

---

## Prioritized backlog

| Priority | Change | Paper reference |
|----------|--------|-----------------|
| **P0** | Evidence bundle + `untested_regions` on approve | §5.2.2 |
| **P0** | Commit pins + `spec_version` on scratchpad | §4.2, §5.2.4 |
| **P1** | Trajectory / harness metrics in every task JSON | §5.2.1 |
| **P1** | Failure-type routing in orchestrator | §3.4 |
| **P2** | Engineer file-scope + bash policy enforcement | §2.2 |
| **P2** | `acceptance_checks` for `complex` specs | §2.1 |
| **P3** | Cross-task `.eng_team/learnings.json` | §3.2 |
| **P3** | Golden-repo harness regression tests | §5.2.3 |

---

## Bottom line

eng-team is already a **code-as-harness** system for software engineering. The survey’s main push is to evolve from **prompt orchestration that usually works** to **harness engineering**: every approval carries proof, every phase carries versioned assumptions, and harness failures improve the system with regression discipline — without widening scope beyond the PRD → PR slice until trust is earned.

---

## Reference

- **Paper:** [Code as Agent Harness: Toward Executable, Verifiable, and Stateful Agent Systems](https://arxiv.org/abs/2605.18747)
- **Related repo docs:** `PHILOSOPHY.md`, `README.md`, `.claude/commands/eng-team.md`
