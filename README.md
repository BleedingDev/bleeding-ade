# BleedingADE

BleedingADE is a project-first control plane for coding agents: the best parts of T3 Code, AgentBox, Herdr, Beads, and selected agent runtimes, combined without exposing terminal/session-manager chaos as the product model.

## Current phase: Wayfinding

No production implementation yet. This repository is first resolving the product job, domain model, execution topology, session continuity, event fabric, runtime boundaries, web/mobile information architecture, and minimum coherent release.

Implementation starts only after the final Wayfinder handoff decision closes.

## Strong product priors

- Extend T3 Code coherently rather than build a disconnected prototype.
- Excellent web and mobile experience; native desktop is secondary unless discovery proves otherwise.
- Run code and agents on one or more execution machines while controlling them from any authorized device.
- Use AgentBox for isolated/resumable execution locations where it earns that role.
- Treat Herdr as a candidate capability, not a mandatory architecture layer.
- Support T3 Code providers plus Oh My Pi, Prime Agent/RLM, and Claude Code with UltraCode workflows through typed capability-negotiated adapters.
- Make Beads task state and a native graph/triage experience available inside BleedingADE. Reproduce the useful interaction model; do not embed or copy Beads Viewer.
- Keep terminal output as diagnostic provenance and fallback, never the primary information architecture.
- Agent Mail, CASS, and other Agent Flywheel components are optional until their product value is proven.

## Start here

- [Wayfinder map](docs/wayfinder/map.md)
- [Architecture hypotheses](docs/architecture/hypotheses.md)
- [First grilling round](docs/wayfinder/grilling-round-1.md)
- [Decision dependency graph](docs/wayfinder/dependency-graph.md)

GitHub issues are the canonical live decision record once published.
