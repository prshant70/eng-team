# Quality Gates in a Multi-Repo / Microservices Environment

> Extending the single-repo quality philosophy to systems where logic is split across services, shared libraries, and utilities — and where one PRD may touch more than one repo.

---

## Why Single-Repo Gates Are Not Enough

The quality gates designed for a single repo assume one CLAUDE.md, one test suite, one diff to review. In a microservices environment, the failure modes multiply:

- An agent has full context on Service A but no idea Service B even exists
- A schema change in a shared library silently breaks three consumers
- Two engineer agents modify overlapping contracts in parallel with no coordination
- The reviewer only sees one diff but the bug lives in the interaction between two services

Each of these requires a gate that simply doesn't exist in the single-repo model.

---

## New Layer Required: System-Level Context

Each repo keeps its own `CLAUDE.md` for local conventions. But multi-repo changes require an additional layer — an org-level architecture document that every agent reads before planning.

This document covers:

- **Service inventory** — what each service owns, its public API surface, who calls it
- **Dependency graph** — which services depend on which, where contracts live (OpenAPI specs, proto files, shared types)
- **Cross-cutting conventions** — auth patterns, error formats, event schemas — things that must be consistent across all services

This is not a living wiki maintained by hand. It should be partially auto-generated from actual API specs, import graphs, and event bus subscriptions — so it reflects the real system, not someone's memory of it.

---

## Gate 1: Contract-First Planning

For any multi-repo PRD, the tech-lead's plan artifact must answer: *which service boundaries does this change cross?*

Any change that touches an API contract, event schema, or shared type must be declared upfront — before any engineer agent starts writing code. The plan names the contract change explicitly. All downstream service changes are derived from it.

This enforces the right order of operations: **contracts first, implementations second.** Agents cannot drift into incompatible assumptions if the contract is locked before they start.

---

## Gate 2: Sequenced Parallel Execution

Multiple engineer agents can work in parallel on separate services — but only after the contract is settled.

The coordination rule: **no agent touches a service that depends on a contract change until that contract change is finalized.**

This is a sequencing constraint, not a quality check. Violating it means two agents make independent assumptions about the same interface, and both may be wrong in ways that only surface at integration time.

---

## Gate 3: Cross-Repo Adversarial Reviewer

The single-repo reviewer reads one diff. In multi-repo, the reviewer must read all diffs together and specifically check:

- Are all contract changes backward compatible — or is there a coordinated breaking change with a migration plan?
- Does Service B's usage of the new API actually match what Service A implemented?
- Is there a service that calls the changed interface that wasn't included in the plan?

This reviewer requires the dependency graph from the system-level context layer to know which services to check. It cannot discover blast radius from the diffs alone.

---

## Gate 4: Contract Tests as the Quality Floor

In a single repo, the test suite is the quality floor. In multi-repo, the equivalent is **contract tests** — consumer-driven tests that run against the producer's implementation.

Every service that publishes an API should have contract tests defined by its consumers. These are the only automated checks that can catch cross-service incompatibilities before integration. Unit tests and linting within each service will not surface interface mismatches.

---

## Revised Implementation Priority (Multi-Repo)

| Priority | Gate | What it addresses |
|---|---|---|
| 1 | System-level architecture document | Gives agents cross-service context |
| 2 | Per-repo `CLAUDE.md` | Gives agents local conventions |
| 3 | Contract-first plan artifact | Prevents incompatible parallel implementations |
| 4 | Sequenced parallel execution | Enforces contract-before-consumer ordering |
| 5 | Full test suite per repo + contract tests | Sets the automated quality floor |
| 6 | Cross-repo adversarial reviewer | Catches interface mismatches across diffs |
| 7 | Feedback capture loop | Compounds quality improvements over time |

---

## The Unsolved Problem

The system-level architecture document is only as good as its maintenance discipline. In a fast-moving microservices environment, the dependency graph goes stale quickly.

The real answer is that this document needs to be auto-generated — derived from actual API specs, import graphs, and event bus subscriptions — not maintained by hand. Until that tooling exists, the document is a useful approximation, not a guarantee. Treat it as the best available context, and build agents that flag when they encounter service references not covered by it.

---

*Document authored from eng-team architectural discussion — May 2026.*
