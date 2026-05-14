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

## PHASE 1.5 — Spec clarification (conditional)

Read `technical_spec.complexity` and note it — you will pass it to the engineer and reviewer.

After the engineer runs (Phase 2), check if `phase` is `"spec_needs_clarification"`. If it is, run this phase **before** looping back to Phase 2:

Invoke the **tech-lead** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>

Spec gaps to clarify:
<paste the full spec_gaps array from implementation in the scratchpad>

Read the relevant source files and update the technical_spec in the scratchpad to resolve each gap.
Set phase to "spec_clarified" when done.
```

Wait for tech-lead to complete. Read the updated spec, then re-invoke the engineer (Phase 2 prompt below) with the clarified spec.

**Maximum 1 clarification cycle.** If the engineer flags gaps a second time, proceed with implementation — the engineer must document assumptions in `implementation.notes` instead.

---

## PHASE 2 — Implementation

Invoke the **engineer** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>
Branch: <branch_name from scratchpad>
Complexity: <technical_spec.complexity from scratchpad>

Read the scratchpad for the Technical Spec, then:
1. Check for spec gaps before implementing — if any exist, write them to the scratchpad and stop
2. Create the branch: git checkout -b <branch_name> 2>/dev/null || git checkout <branch_name>
3. Implement all changes from technical_spec.files_to_create and files_to_modify
4. Write tests inline as you go
5. Run the full test suite and fix any failures
6. Run the linter and fix all errors
7. Commit with a conventional commit message
8. Update the scratchpad "implementation" key with the commit hash and file list
```

Wait for engineer to complete. Read the scratchpad:
- If `phase` is `"spec_needs_clarification"` → run Phase 1.5
- Otherwise confirm `implementation.commit_hash` is set before continuing

---

## PHASE 3 — Review

Invoke the **reviewer** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>
Branch: <branch_name from scratchpad>
Base branch: <base_branch from scratchpad>
Complexity: <technical_spec.complexity from scratchpad>
Output directory: <absolute output_dir from scratchpad>

Review the diff with: git diff <base_branch>...<branch_name>
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
Base branch: <base_branch from scratchpad>
Complexity: <technical_spec.complexity from scratchpad>
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

**Maximum 1 rejection cycle.** If the reviewer rejects a second time, do not loop again — the task needs human review. In the Final Status, clearly surface the unresolved issues so a human engineer can take over.

---

## FINAL STATUS

Read the scratchpad one last time. Print this summary:

**If approved:**
```
═══════════════════════════════════════════════
  Engineering AI Team — Done
═══════════════════════════════════════════════

  Task:    $ARGUMENTS
  Branch:  <branch_name>
  Review:  ✓ Approved

  Output files:
  → <review.pr_path>
  → <review.summary_path>
  → <scratchpad_path>

  Next steps:
  → Push branch:  git push origin <branch_name>
  → Open PR and paste the contents of PR_DESCRIPTION.md
═══════════════════════════════════════════════
```

**If not approved after the full rejection cycle:**
```
═══════════════════════════════════════════════
  Engineering AI Team — Needs human review
═══════════════════════════════════════════════

  Task:    $ARGUMENTS
  Branch:  <branch_name>
  Review:  ✗ Not approved — automated fix cycle exhausted

  Unresolved issues (from scratchpad review.critical_issues):
  → <issue 1: file, line, description>
  → <issue 2: file, line, description>
  ...

  The branch has been committed and is ready for a human engineer to
  resolve the above issues before merging.

  Scratchpad for full context:
  → <scratchpad_path>
═══════════════════════════════════════════════
```
