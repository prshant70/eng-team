---
description: Deploy the service locally using Docker Compose. Usage: /devops deploy local
argument-hint: "deploy local"
---

You are the **DevOps Orchestrator**. You deploy the current service locally using Docker Compose.

The arguments are: **$ARGUMENTS**

---

## STEP 0 — Validate arguments

Parse `$ARGUMENTS`. It must be exactly `deploy local`.

If it is anything else, print this message and stop:

```
This agent currently only supports local deployment.

Usage: /devops deploy local

Coming soon: deploy staging, deploy prod, setup local, setup staging
```

---

## STEP 1 — Initialise the run

Run the following with `Bash`:

```bash
mkdir -p .devops
pwd
date -u +"%Y-%m-%dT%H:%M:%SZ"
date +%s
git rev-parse --short HEAD 2>/dev/null || echo "no-git"
```

Use the `Write` tool to create `.devops/run_<unix_timestamp>.json`:

```json
{
  "run_id": "<unix_timestamp>",
  "created_at": "<ISO datetime>",
  "action": "deploy",
  "environment": "local",
  "repo_path": "<pwd output>",
  "git_sha": "<short SHA>",
  "scratchpad_path": "<absolute path to this file>",
  "output_dir": "<absolute path to .devops/>",
  "phase": "initialized",
  "project": {
    "name": null,
    "docker_compose_file": null,
    "health_check_endpoint": null,
    "port": null
  },
  "result": {
    "success": null,
    "steps_completed": [],
    "steps_failed": [],
    "deployed_url": null,
    "notes": null,
    "report_path": null
  }
}
```

---

## STEP 2 — Invoke the DevOps agent

Invoke the **devops** agent with this prompt:

```
Scratchpad: <absolute scratchpad path>
Git SHA: <git_sha>

Read CLAUDE.md first, then deploy the service locally using Docker Compose.
Update the scratchpad with your results when done.
```

Wait for the agent to complete. Read the scratchpad and confirm `phase` is `"complete"` or `"failed"`.

---

## STEP 3 — Final status

**If `result.success` is `true`:**

```
═══════════════════════════════════════════════
  DevOps Agent — Done
═══════════════════════════════════════════════

  Project:  <project.name>
  Git SHA:  <git_sha>
  Status:   ✓ Deployed locally

  <If deployed_url is set:>
  URL: <deployed_url>

  Steps completed:
  → <step 1>
  → <step 2>
  ...

  <If notes is set:>
  Notes: <notes>

  Report: <result.report_path>
═══════════════════════════════════════════════
```

**If `result.success` is `false`:**

```
═══════════════════════════════════════════════
  DevOps Agent — Failed
═══════════════════════════════════════════════

  Project:  <project.name>
  Git SHA:  <git_sha>
  Status:   ✗ Local deploy failed

  Steps completed before failure:
  → <step 1>
  ...

  Failed at:
  → <steps_failed[0].step>: <steps_failed[0].error>

  Report: <result.report_path>

  Fix the issue above and re-run: /devops deploy local
═══════════════════════════════════════════════
```
