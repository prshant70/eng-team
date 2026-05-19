# Philosophy

This document explains the beliefs `eng-team` is built on, the approach it takes to integrating AI into an engineering team, and where it is going. It is not required reading to *use* the workflow — the agents work without it. It is required reading to *extend* it, to decide whether it fits your team, or to understand why a design decision that looks like a limitation is actually a deliberate choice.

---

## The conviction

### Start at the bottom, earn the top

An engineering team is a hierarchy of judgment. At the bottom sit roles that are closest to computer systems — writing code, running tests, checking security, reviewing for correctness. At the top sit roles that require human intelligence in its most irreducible form — understanding business context, making product decisions, exercising aesthetic judgment, navigating organizational constraints.

The conviction behind `eng-team` is that AI should enter this hierarchy from the bottom, not the top and not all at once.

This is not a statement about AI capability. It is a statement about *trust* and *verifiability*. When an AI writes code, engineers can verify it — tests pass or they don't, the logic holds or it doesn't, the diff makes sense or it doesn't. Engineers already have the tools, the expertise, and the instincts to evaluate code output. When an AI makes a product decision or generates a design direction, there is no equivalent verification apparatus. "Correct" is contextual, subjective, and depends on years of domain knowledge that the team has and the AI doesn't. Asking a team to trust AI with the subjective before they trust it with the objective is asking them to jump to the hardest problem first.

The bottom-up approach earns trust incrementally. A team that has watched AI reliably write correct, reviewed, and tested code is a team ready to think about what the agent above the engineer might look like. A team that has never trusted the bottom has no foundation for trusting anything higher.

### Replace one role at a time

The other dimension of this is disruption. Replacing the entire team workflow at once — product, design, engineering, QA, deployment — requires every part of the organization to change simultaneously, and creates a system so complex that when something goes wrong it is impossible to know where. Teams abandon workflows they can't debug.

`eng-team` replaces one slice: the path from a written requirement to a merged pull request. Everything above that (how the requirement gets written, who decides what to build) and everything below it (how the PR gets deployed, how production is monitored) remains exactly as the team already does it. The insertion point is narrow, the value is immediate, and the team retains full control over everything they already own.

This is not a limitation. It is the design.

### Surface uncertainty, never absorb it silently

An agent that proceeds on a bad spec is more dangerous than one that stops and asks. Silent assumptions — where the agent invents an interpretation of an ambiguous requirement and implements against it — compound quietly. The code looks correct, the tests pass against the agent's interpretation, the reviewer approves the diff, and the PR merges before anyone realizes the feature doesn't match what was intended.

Every agent in the workflow is built to surface uncertainty before it becomes code. When inputs are unclear, the right behavior is to name the gap and wait — not to fill it in and proceed. This is slower in the moment and faster over the full cycle, because the cost of fixing a wrong interpretation after a commit is always higher than the cost of a clarification before one.

The corollary: output quality is bounded by input quality. An agent cannot manufacture intent it wasn't given. The human-AI interface — the point where human requirements become agent inputs — is the most important design surface in the entire system. How that interface is structured determines how much ambiguity the downstream agents have to absorb.

---

## The workflow

`eng-team` implements a three-agent pipeline: a Tech Lead that reads the requirement and produces a technical spec, an Engineer that implements against the spec and ships tested code, and a Reviewer that evaluates the diff and either approves or returns specific, actionable fix instructions. No agent talks directly to another. Each reads its inputs from a shared scratchpad, does its work, writes its outputs back, and signals the orchestrator. The scratchpad is also the audit trail — if something goes wrong anywhere in the cycle, the full reasoning chain is recoverable.

The principles behind each agent are consistent with the conviction above. The Tech Lead does not implement; it specifies, and it asks rather than assumes when the requirement is underspecified. The Engineer does not review; it builds, tests, and commits, and flags what it cannot resolve rather than guessing. The Reviewer reads the diff — not the agent's intent, not the PRD — because code must stand on its own regardless of what was planned. A review that approves intent rather than output is not a review.

Together, the three agents convert a human-authored requirement into a committed branch with a full PR description — a unit of work that a human engineer can inspect, verify, and merge with confidence.

---

## The vision

### The interface moves, but doesn't disappear

Today the interface between human judgment and AI execution is the PRD — a free-text requirement handed to the orchestrator. This is the right starting point: it requires nothing new from the team, and whoever writes requirements today can keep writing them exactly as they do.

The limitation is inherent to prose. Natural language carries implicit context, unstated assumptions, and ambiguities the author doesn't know they're making. The Tech Lead absorbs as much of this as it can and surfaces the rest, but some always slips through.

The next evolution is a PRD Agent that sits above the Tech Lead. Its job is to take a requirement — stated informally, as a business goal or a user complaint or a product idea — and produce a structured, schema-enforced output in exactly the format the Tech Lead expects. When the PRD is AI-generated, the format is fully controlled. Every required decision is explicit. Ambiguity that currently enters at the human-written boundary gets resolved by the PRD Agent before the Tech Lead ever sees it.

This is the compounding property of building upward. Each new agent added above the current top generates precisely-formatted outputs for the agent below. Ambiguity gets pushed further upstream with every layer, until it reaches the raw human input — the business goal, the user need, the idea — which is the one place ambiguity is irreducible. That is the correct final state: humans own the messy, judgment-heavy inputs at the top, and the system handles everything downstream.

### The gate that stays human

Even as new agents are added above, one gate should remain human: the approval of the requirement before it enters the pipeline. Not to rewrite it — to confirm the system captured the intent correctly. This is where the business's intent gets verified, and that is inherently a human responsibility. The interface evolves from "human writes requirement" to "human approves AI-structured requirement" — substantially lower friction, but the checkpoint remains.

It is probably the last gate to ever be removed, if it is removed at all.

### Earning the roles above

As the lower layers stabilize, the system grows upward into roles that carry more human judgment — QA, deployment, production monitoring, eventually design and product assistance. The word "assistance" matters here. The further up the hierarchy you go, the more an agent's role shifts from *executing* to *supporting human decisions*. A code reviewer can be fully autonomous because correctness is verifiable. A product agent should surface options and tradeoffs and make its assumptions explicit — but the human makes the call, until the system has earned the trust to do otherwise.

The conviction is not that AI will eventually replace every role. It is that AI should earn each role from the bottom up, one at a time, demonstrating reliability before taking on more responsibility. That is how trust works between people. There is no reason it should work differently between people and systems.

---

The workflow is one team's attempt to build that trust systematically. Take what's useful. The gaps will make themselves known.
