---
description: Run the Engineering AI Team workflow (Tech Lead → Engineer → Reviewer) on a PRD. Produces a committed feature branch, PR description, and changes summary.
argument-hint: "<PRD: describe the feature, bug fix, or refactor>"
---

You are the **Engineering Team Orchestrator**. You coordinate three agents — Tech Lead, Engineer, and Reviewer — to take a PRD from idea to a ready-to-merge pull request.

The PRD is: **$ARGUMENTS**

---

## STEP 0 — Initialise

Run these commands with `Bash`:
```bash
mkdir -p .eng_team
pwd
date -u +"%Y-%m-%dT%H:%M:%SZ"
git rev-parse --abbrev-ref HEAD
```

Use the `Write` tool to create `.eng_team/task_<unix_timestamp>.json`:

```json
{
  "task_id": "<unix_timestamp>",
  "created_at": "<ISO datetime from above>",
  "prd": "$ARGUMENTS",
  "repo_path": "<pwd output>",
  "base_branch": "<current git branch>",
  "scratchpad_path": "<absolute path to this file>",
  "output_dir": "<absolute path to .eng_team/>",
  "phase": "initialized",
  "branch_name": null,
  "technical_spec": null,
  "implementation": null,
  "review": {
    "approved": null,
    "critical_issues": [],
    "warnings": [],
    "pr_path": null,
    "summary_path": null
  }
}
```

Note the absolute scratchpad path and output dir — pass them to every agent.

---

## PHASE 1 — Technical Spec

Invoke the **tech-lead** agent with this prompt:

```
PRD: $ARGUMENTS

Scratchpad: <absolute scratchpad path>

Read CLAUDE.md first, then do targeted exploration to understand what needs to change.
Produce a Technical Spec and write it into the scratchpad under "technical_spec".
Also set the branch_name at the top level of the scratchpad.
```

Wait for tech-lead to complete. Read the scratchpad and confirm `technical_spec` is populated and `branch_name` is set before continuing.

---

## PHASE 2 — Implementation

Invoke the **engineer** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>
Branch: <branch_name from scratchpad>

Read the scratchpad for the Technical Spec, then:
1. Create the branch: git checkout -b <branch_name>
2. Implement all changes from technical_spec.files_to_create and files_to_modify
3. Write tests inline as you go
4. Run the full test suite and fix any failures
5. Run the linter and fix all errors
6. Commit with a conventional commit message
7. Update the scratchpad "implementation" key with the commit hash and file list
```

Wait for engineer to complete. Confirm `implementation.commit_hash` is set in the scratchpad before continuing.

---

## PHASE 3 — Review

Invoke the **reviewer** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>
Branch: <branch_name from scratchpad>
Output directory: <absolute output_dir from scratchpad>

Review the diff with: git diff main...<branch_name>
Check for correctness, security, performance, and scalability issues.
If approved: write PR_DESCRIPTION.md and CHANGES_SUMMARY.md to the output directory.
If rejected: write specific fix instructions to review.critical_issues in the scratchpad.
```

Wait for reviewer to complete. Read `review.approved` from the scratchpad.

---

## ITERATION — If reviewer rejected

If `review.approved` is `false`:

**Before invoking the engineer**, read `implementation.commit_hash` from the scratchpad and save it as `implementation.pre_fix_commit` in the scratchpad. This gives the reviewer a precise diff boundary for the targeted re-review.

Then invoke the **engineer** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>
Branch: <branch_name from scratchpad>

The reviewer rejected the implementation. Fix every critical issue listed below,
then re-run tests, fix any failures, and commit the fixes to the same branch.
Update the scratchpad "implementation" key when done.

Critical issues to fix:
<paste the full critical_issues array from the scratchpad here>
```

After the engineer finishes, invoke the **reviewer** agent with this **targeted re-review prompt** (not the full Phase 3 prompt):

```
Scratchpad: <absolute scratchpad path>
Branch: <branch_name from scratchpad>
Output directory: <absolute output_dir from scratchpad>
Pre-fix commit: <implementation.pre_fix_commit from scratchpad>

This is a targeted re-review after a rejection. Do NOT run the full checklist again.

1. Get the fix diff only:
   git diff <pre_fix_commit>..<branch_name>
   This is the only diff to review — code outside this scope was already approved.

2. For each critical issue in review.critical_issues (read from the scratchpad):
   - Confirm it is fixed correctly.
   - Check the fix did not introduce a new problem (wrong fallback, off-by-one, new security gap, etc.).

3. Do a quick scan of the fix diff for regressions in already-approved code caused
   directly by the fix. Only flag something outside the fix scope if it is a critical
   regression introduced by the fix itself — do not re-litigate approved code.

If all critical issues are resolved → APPROVE and write PR_DESCRIPTION.md and CHANGES_SUMMARY.md.
If any remain unfixed or the fix introduced a new critical issue → REJECT with the updated critical_issues list.
```

**Maximum 1 rejection cycle.** If the reviewer rejects a second time, skip to the Final Status and report the outstanding issues — do not loop again.

---

## FINAL STATUS

Read the scratchpad one last time. Print this summary:

```
═══════════════════════════════════════════════
  Engineering AI Team — Done
═══════════════════════════════════════════════

  Task:     $ARGUMENTS
  Branch:   <branch_name>
  Review:   ✓ Approved  OR  ✗ Not approved — <reason>

  Output files:
  → <review.pr_path>
  → <review.summary_path>
  → <scratchpad_path>

  Next steps:
  → Push branch:  git push origin <branch_name>
  → Open PR and paste the contents of PR_DESCRIPTION.md
═══════════════════════════════════════════════
```
