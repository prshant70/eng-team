# Agents for Multi-Repo Changes

> Documents the new agents introduced to handle multi-repo PRDs, what each one does, and how they integrate into the existing eng-team workflow.

---

## Updated Workflow

```
Orchestrator
  └── Tech Lead        (multi-repo plan + blast radius analysis)
  └── Contract Agent   (locks interface changes before anyone implements)
  └── Engineer × N     (parallel, one per repo, gated on contract finalization)
  └── Reviewer         (cross-repo, reads all diffs together)
```

The Orchestrator drives the full pipeline. The core sequence — Tech Lead → Engineer → Reviewer — stays the same. Two things change: the Contract Agent is inserted between planning and implementation, and the Engineer step becomes parallel across repos.

---

## Existing Agents — What Changes

### Orchestrator (extended, not replaced)

The Orchestrator already handles sequencing and agent coordination. For multi-repo changes it needs two behavioral additions:

- **Parallel engineer dispatch** — spins up one Engineer agent per repo rather than always one
- **Contract gate enforcement** — Engineer agents that depend on a contract change are blocked from starting until the Contract Agent has finalized and published the contract artifact

No new agent is needed here. This is a logic and configuration extension.

### Tech Lead (extended)

In addition to its existing responsibilities, the Tech Lead must:

- Read the **system-level architecture document** (service inventory, dependency graph, cross-cutting conventions) before writing the plan
- Identify which service boundaries the PRD crosses
- Produce a **cross-repo plan artifact** that maps each acceptance criterion to a specific repo and lists every file expected to change per repo
- Flag any public API surface, event schema, or shared type that will be modified — this is the signal that triggers the Contract Agent

The plan artifact is the Orchestrator's input for deciding whether to invoke the Contract Agent and how many Engineer agents to spin up.

### Engineer (extended)

Each Engineer agent operates on a single repo, same as before. The changes are:

- Multiple instances run in parallel, one per repo
- Instances that depend on a contract change receive the finalized contract artifact as additional context before starting
- Instances working on independent repos (no shared contract dependency) start immediately in parallel

### Reviewer (extended)

The Reviewer receives all diffs across all repos simultaneously and adds one additional check to its existing checklist:

- Does Service B's usage of the new API match what Service A implemented?
- Is there any service that calls the changed interface that was not included in the plan?
- Are all contract changes backward compatible, or is there a coordinated breaking change with an explicit migration plan?

The Reviewer requires the dependency graph from the system-level architecture document to know which services to check — it cannot discover blast radius from diffs alone.

---

## New Agent: Contract Agent

### When it is invoked

The Orchestrator invokes the Contract Agent when either of the following is true:

- The PRD touches more than one repo
- The PRD touches a single repo but the Tech Lead's plan flags a change to a public API surface, event schema, or shared library interface

For purely internal single-repo changes (business logic, UI, infra config with no public interface change) the Contract Agent is skipped entirely.

### What it does

The Contract Agent owns the interface boundary between services. Its job is to produce a **contract artifact** — a precise, versioned definition of what is changing at the service boundary — and lock it before any Engineer agent starts implementing.

Specifically it:

1. Reads the Tech Lead's plan artifact and the current interface definitions (OpenAPI specs, proto files, shared types, event schemas) for all affected services
2. Produces a diff of the contract change — what is being added, modified, or removed at the interface boundary
3. Checks backward compatibility — flags breaking changes and requires an explicit migration or versioning plan if any exist
4. Publishes the finalized contract artifact so downstream Engineer agents can use it as a source of truth
5. Blocks the Orchestrator from starting any dependent Engineer agent until the artifact is published

### What it does not do

- It does not write implementation code
- It does not modify business logic
- It does not review the final diffs — that is the Reviewer's job

### Output

A contract artifact containing:
- The precise interface change (structured diff of the API surface)
- Backward compatibility assessment (compatible / breaking + migration plan)
- A list of all services that consume the changed interface, derived from the dependency graph

---

## Sequencing Rules

| Condition | Contract Agent | Engineer agents |
|---|---|---|
| Single-repo, no public interface change | Skipped | One agent, starts immediately |
| Single-repo, public interface change | Invoked | Starts after contract is finalized |
| Multi-repo, independent services (no shared contract) | Skipped | All agents start in parallel immediately |
| Multi-repo, shared contract change | Invoked | Contract-dependent agents wait; independent agents start immediately |

---

## Summary

One new agent is introduced: the **Contract Agent**. It fills the gap that exists in the current workflow — there was no role responsible for locking interface changes before implementation begins. Without it, parallel Engineer agents make independent assumptions about the same interface, and mismatches only surface at review or integration.

Everything else — Orchestrator, Tech Lead, Engineer, Reviewer — retains its existing role and gains scoped extensions to handle multi-repo context and parallel execution.

---

*Document authored from eng-team architectural discussion — May 2026.*
