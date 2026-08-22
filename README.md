# BleedingADE

BleedingADE is a project-first control plane for coding agents: the best parts of T3 Code, AgentBox, Herdr, Beads, and selected agent runtimes, combined without exposing terminal/session-manager chaos as the product model.

## Current phase: Wayfinding

No production implementation yet. This repository is first resolving the product job, domain model, execution topology, session continuity, event fabric, runtime boundaries, web/mobile information architecture, and minimum coherent release.

Implementation starts only after the final Wayfinder handoff decision closes.

## Live Wayfinder

- **[Canonical decision map](https://github.com/BleedingDev/bleeding-ade/issues/2)** — one map, 18 decision tickets, five-ticket initial frontier.
- **[Current claimed ticket: Define the job BleedingADE must be hired to do](https://github.com/BleedingDev/bleeding-ade/issues/3)**
- [All discovery issues](https://github.com/BleedingDev/bleeding-ade/issues?q=is%3Aissue+is%3Aopen+label%3Aphase%3Adiscovery)

GitHub issues are the canonical live decision record. Repository documents provide durable context and architecture evidence.

## Strong product priors

- Extend T3 Code coherently rather than build a disconnected prototype.
- Excellent web and mobile experience; native desktop is secondary unless discovery proves otherwise.
- Run code and agents on one or more execution machines while controlling them from any authorized device.
- Use AgentBox for isolated/resumable execution locations where it earns that role.
- Treat Herdr as a candidate capability, not a mandatory architecture layer.
- Support T3 Code providers plus Oh My Pi, Prime Agent/RLM, and Claude Code with UltraCode workflows through typed capability-negotiated adapters.
- Make Beads task state and native dependency, graph, triage, and insight functionality available inside BleedingADE. Reproduce useful capabilities; do not embed or copy Beads Viewer or the Wayfinder Obsidian plugin.
- Keep terminal output as diagnostic provenance and fallback, never the primary information architecture.
- Agent Mail, CASS, NTM, and other Agent Flywheel components are optional until their product value is proven.
- Licensing is already handled and outside the Wayfinder scope.

## Repository context

- [Initial product vision](docs/product/initial-vision.md)
- [Architecture hypotheses](docs/architecture/hypotheses.md)
- [Preliminary ecosystem baseline](docs/research/ecosystem-baseline.md)
- [First grilling round](docs/wayfinder/grilling-round-1.md)
- [Decision dependency graph](docs/wayfinder/dependency-graph.md)
- [Repository mirror of the map](docs/wayfinder/map.md)
