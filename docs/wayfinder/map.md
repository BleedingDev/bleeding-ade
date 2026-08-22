# Wayfinder: Define BleedingADE before implementation

## Destination

An evidence-backed, decision-complete product and architecture definition for BleedingADE that can be handed to implementation without reopening material questions about user value, scope, UX, runtime ownership, continuity, event semantics, integrations, security, or acceptance.

## Notes

**Mode:** planning only. This map resolves decisions; it does not implement the product.

**Strong priors to test, not blindly enforce:** extend T3 Code coherently; prioritize excellent web/mobile use; use AgentBox for isolated/resumable execution locations; expose Beads task state through a native BleedingADE graph/triage experience; preserve semantic events; support Oh My Pi, Prime Agent/RLM, and Claude Code with UltraCode workflows through typed adapters; keep Herdr and broader Agent Flywheel components only where they earn a clear role.

**Owner-set constraints:**

- Licensing is already handled and is outside this effort.
- BleedingADE should reproduce useful Beads Viewer / Wayfinder Obsidian graph, dependency, triage, and insight capabilities natively. Do not embed, clone, or copy those products.
- Web and mobile quality matter more than full native desktop parity.
- Agent Mail, CASS, NTM, and other Agent Flywheel components remain optional until proven valuable.

**Standing rules:**

- Consult Matt Pocock's Wayfinder and domain-modeling/grilling methods.
- Ground HITL decisions in concrete recent workflows.
- Prefer official source code and executable boundary probes over summaries.
- Do not let terminal, tmux, NTM, Herdr panes, or box topology define the primary UX.
- Require capability negotiation and honest degradation.
- Do not self-resolve HITL tickets.
- Refer to tickets by linked title, never bare issue numbers.
- The final frontier ticket explicitly approves or rejects implementation readiness.

## Decisions so far

- [Define the job BleedingADE must be hired to do](https://github.com/BleedingDev/bleeding-ade/issues/3): build a self-hostable, federated T3-style client/server control plane where one human plans through user-supplied skills, converts plans into Beads, and supervises or hands over 5–100+ long-running agent workstreams across 2–10 machines from any authorized device, with complete semantic history and no assumed role for Herdr.

## Not yet specified

- Exact screen structure, gestures, density, and visual hierarchy beyond the current project-first hypothesis; these become specifiable after workflow and domain decisions.
- Concrete canonical event schemas and adapter conformance fixtures; these become specifiable after topology and event-fabric decisions.
- Exact boundary probes required for individual runtimes; these graduate from ecosystem research and topology decisions.
- Migration, release automation, and upstream-sync mechanics; these graduate after the minimum release and repository strategy are chosen.
- Detailed implementation sequencing and the Beads execution graph; implementation planning starts only after the handoff gate closes.

## Out of scope

- Production implementation, scaffolding, or a disconnected prototype during this map.
- Licensing analysis or redistribution decisions; the owner considers them settled.
- Training, fine-tuning, or modifying foundation models.
- A terminal/tmux/NTM dashboard as BleedingADE's primary information architecture.
- Copying or embedding Beads Viewer or the Wayfinder Obsidian plugin.
- Pretending unavailable integrations, semantic events, or migration capabilities exist.
- Mandatory SaaS or multi-tenant operation unless the deployment/trust decision explicitly brings it into scope.
