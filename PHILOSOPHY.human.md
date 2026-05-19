# Philosophy

Read this before you extend the workflow, not before you use it. The agents work without any of this. But if you're making decisions about what to change, what to add, or whether this approach fits your team at all — this is the reasoning behind the choices. Some of those choices look like constraints. They're not. They're opinions.

---

## The conviction

### Start at the bottom, earn the top

Every engineering team runs on a spectrum of judgment. Writing code, running tests, reviewing a diff — these sit close to the machine. There are ground truths. Tests pass or they fail. Logic is sound or it has a bug. A security issue either exists or it doesn't. Now go to the other end: deciding what to build, understanding what a user actually needs, making a design call that balances business constraints with technical reality. No ground truth there. Just experience, context, and taste accumulated over years.

AI should enter that spectrum from the end where it can be verified, not from the end where it can't.

This isn't a limitation on ambition. It's sequencing. When an AI ships code, an engineer can catch the mistakes with tools they already have. When an AI makes a product call, there's no equivalent check. Asking a team to trust the subjective before they trust the objective isn't bold — it's setting the system up to fail publicly on the hardest possible surface. You don't build trust that way. You destroy it.

The bottom-up approach works because every layer of trust you establish creates the foundation for the next one. A team that has watched AI reliably deliver correct, tested, reviewed code for a few months thinks differently about what the system could do next. A team that's never trusted the bottom has nothing to build on.

### Replace one role at a time

There's a temptation when building agentic systems to think big — automate product, design, engineering, QA, deployment, all at once. The problem is that when a system that complex breaks, and it will, nobody knows where to look. Teams don't iterate on systems they can't debug. They abandon them.

`eng-team` takes one slice: from a written requirement to a merged pull request. That's it. Everything above that — who decides what to build, how requirements get written — stays with humans. Everything below — how code gets deployed, how production gets monitored — stays with existing tooling. The team doesn't have to change anything else about how it works to get value from this.

The narrow insertion point is a feature. It means the team can observe the system before trusting it further, and extend it at a pace that matches their comfort rather than the system's ambition.

### Surface uncertainty, never absorb it silently

The worst thing an agent can do is proceed on a bad spec. Not because it'll get stuck, but because it'll succeed — it'll write code, pass tests, generate a PR, and everything will look fine until a human tries to use the feature and realizes the agent answered the wrong question.

Silent assumptions compound in exactly this way. The agent fills in a gap, builds confidently against its own interpretation, the reviewer sees clean code and approves, and the mistake doesn't surface until it's expensive to fix. Compare that to an agent that names the gap, stops, and asks. One of those conversations is awkward for five minutes. The other costs days.

The agents in this workflow are built to surface what they don't know rather than paper over it. That principle also points to something bigger: the quality of what this system produces is bounded by the quality of what goes into it. The interface where humans hand off requirements to agents is the most consequential design decision in the entire workflow. A tight, unambiguous handoff makes everything downstream better. A loose one — free-text, implicit, underspecified — forces agents to guess, and they will guess wrong often enough to matter.

---

## The workflow

Three agents, strict separation of concerns. A Tech Lead that reads the requirement and writes a technical spec. An Engineer that implements against the spec, writes tests, and commits. A Reviewer that reads the diff and either approves or comes back with specific things to fix. No agent makes decisions outside its lane. The Tech Lead doesn't implement. The Engineer doesn't review. The Reviewer doesn't rewrite — it points.

They communicate through a shared scratchpad rather than talking to each other directly. This matters for reliability: if anything breaks in the cycle, the scratchpad has the full reasoning trail — the spec, the gaps the engineer flagged, the reviewer's findings. It's not just state management, it's the audit log.

The Reviewer's job is worth saying clearly: it reads the diff, not the intent. It doesn't care what the agent was trying to do. Code has to stand on its own merits, because that's what ships. A review that approves well-intentioned code with a real security hole is not a good review. The Reviewer is built to treat the diff as the only thing that matters.

---

## The vision

### The interface moves, but doesn't disappear

Right now a human writes a requirement and hands it to the orchestrator. That's the interface. It works, and it requires nothing from the team — whoever writes requirements today keeps writing them exactly as they do.

But prose is imprecise by nature. Requirements written by humans carry implicit context, assumptions the author didn't know they were making, and gaps that only become visible when an agent hits them mid-implementation. The Tech Lead surfaces what it can, but some ambiguity always gets through.

The next step is an agent that sits above the Tech Lead — one that takes a rough requirement, asks the right clarifying questions, and produces a structured, complete spec before engineering starts. When that agent exists, the format of the spec is fully controlled. Every decision the Tech Lead would otherwise have to invent is already explicit. The ambiguity problem doesn't get managed better — it gets resolved before it ever enters the engineering pipeline.

This compounds as the system grows. Each layer added above the current top generates precise inputs for the layer below. Ambiguity gets pushed further upstream with each addition, until it lands where it belongs: with the person who actually knows what the business needs. That person doesn't disappear. Their job just shifts from writing specs to verifying that the system understood them correctly.

### The gate that stays human

No matter how many layers get added, one gate should stay manual: a human confirming the requirement is right before engineering begins. Not editing, not rewriting — just confirming the system captured the intent. That's a business judgment, not a technical one. It's the moment where organizational context and strategy get applied, and those don't live in any agent.

This gate probably outlasts every other human step in the workflow. Maybe it goes eventually. But it should be the last thing to go.

### Earning the roles above

As the lower layers prove themselves, the system earns the right to grow upward — into QA, deployment, production monitoring, and eventually into roles that assist with design and product decisions. The key word is assist. The higher up you go, the less the agent's job is to execute autonomously and the more it is to prepare the ground for a human decision: surface the options, make the tradeoffs explicit, flag what it doesn't know. A code reviewer can be fully autonomous because right and wrong are checkable. A product agent that makes calls on user needs without oversight is a different risk category entirely.

The conviction isn't that AI eventually owns every role. It's that AI should earn each role in turn — starting at the verifiable end, demonstrating reliability, and moving up only when the track record justifies it. That's how trust works between people. There's no reason it should work differently here.

---

This workflow reflects one team's best current thinking on how to build that trust without betting everything on it at once. Take what fits. Push back on what doesn't. The gaps in the system will surface on their own — they always do.
