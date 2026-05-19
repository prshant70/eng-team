# Executive Summary

## Most AI in engineering is still just faster autocomplete

Teams using AI today are mostly accelerating individual developers — a prompt here, a code suggestion there, a human accepting or rejecting each output. That's useful, but the ceiling is low. A human is still in the loop for every decision. The real opportunity is autonomous delivery: hand the system a requirement, get back working, tested, reviewed code.

## We start where AI can actually be verified

Engineering work sits on a spectrum. Writing code, running tests, reviewing a diff for correctness — these are close to computer systems. Either the tests pass or they don't. Engineers already have the tools to catch when something is wrong. Product decisions, design judgment, business tradeoffs — these require context and experience no system can verify yet. Starting there is the wrong bet.

`eng-team` starts at the verifiable end: the path from a written requirement to a pull request. A tech lead agent turns the requirement into a technical spec. An engineer agent implements and tests it. A reviewer agent checks the output the way a senior engineer would. The team doesn't change how it works. Product still owns the roadmap. Engineers still decide what merges. The only thing that changes is the coding and review cycle runs autonomously.

## Trust is earned, not declared

You don't ask a team to trust a system they've never seen work. A team that watches AI reliably ship correct, reviewed, tested code for a few months has a fundamentally different relationship with the technology than one that's only read about it. That track record is what earns the right to automate the next layer.

## The system grows upward as that trust accumulates

Right now the system takes a human-written requirement as its input. The next layer is an agent that converts a rough business goal into a structured, unambiguous spec before any engineering starts — eliminating most of the rework that comes from building against the wrong interpretation of a requirement.

Above that: QA, deployment, production monitoring, eventually agents that assist product and design decisions. Each new layer generates cleaner inputs for the layer below, pushing ambiguity further upstream — until it reaches the one place it belongs, the human who knows what the business actually needs.

## One gate always stays human

Before engineering work begins, a person confirms the system understood the goal correctly. Not to rewrite anything — just to verify intent was captured right. That's a business checkpoint, not a technical one, and it belongs to a person regardless of how far the system grows.

## This is not about replacing engineers

It's about teams that build AI trust incrementally — starting with what's verifiable, moving upward — pulling ahead of teams that either ignore AI or try to automate everything at once. The workflow is designed to be narrow enough to adopt today and extensible enough to grow into whatever the team is ready for next.
