# Wayfinder: Define BleedingADE before implementation

## Destination

An evidence-backed, decision-complete product and architecture definition for BleedingADE that can be handed to implementation without reopening material questions about user value, scope, UX, runtime ownership, continuity, event semantics, integrations, security, or acceptance.

## Notes

**Mode:** planning only. This map resolves decisions; it does not implement the product.

**Strong priors to test, not blindly enforce:** preserve T3 Code's client/server, semantic event-sourced architecture, terminal contracts, and first-party web/mobile terminal stack; federate independently operational BleedingADE servers; prioritize excellent web/mobile use; validate AgentBox for isolated/resumable execution placement and handover; integrate Jeffrey Emanuel's `beads_rust` through supported `br` surfaces while providing a native graph/triage experience; support Oh My Pi, Prime Agent/RLM, and Claude Code workflows through typed adapters where truthful; require Herdr and broader Agent Flywheel components to prove distinct value.

**Owner-set constraints:**

- Licensing is already handled and is outside this effort.
- Reproduce useful Beads Viewer / Wayfinder Obsidian graph, dependency, triage, and insight capabilities natively. Do not embed, clone, or copy those products.
- The selected task implementation is [`Dicklesworthstone/beads_rust`](https://github.com/Dicklesworthstone/beads_rust) / `br`, not the newer Dolt-based Beads architecture.
- Planning happens inside ordinary BleedingADE chats through user-installed planning skills. BleedingADE may ship a thin operational `br` management skill, but no planning methodology is baked in initially.
- Web and mobile are first-class; a native desktop application is not initially required.
- Reuse T3 Code's existing server-owned PTYs, terminal contracts, and `libghostty-vt` web/mobile renderers. `coder/ghostty-web` is not a required BleedingADE dependency.
- Each BleedingADE server must remain independently useful; optional replication/aggregation must not become a central execution authority or single point of failure.
- Reuse T3 Code's existing connection targets, pairing, scoped authorization, client sessions, revocation, WebSocket tickets, and DPoP behavior. Do not build a second networking or device-authentication subsystem.
- Full local/federated statistics and telemetry are required. External telemetry and organization/multi-tenant reporting are later product work.
- Trusted replicas and local search may store full-fidelity canonical history; v1 does not add redaction, pseudonymization, or replication-time mangling. Credentials remain server-local and must not be transferred through ordinary history or handover.
- Agent Mail, CASS, NTM, Herdr, and other optional components are included only when research demonstrates unique product value and a viable boundary.

**Standing rules:**

- Consult Matt Pocock's Wayfinder and domain-modeling/grilling methods.
- Ground HITL decisions in concrete workflows.
- Prefer official source code and executable boundary probes over summaries.
- Use the canonical vocabulary in [`CONTEXT.md`](../../CONTEXT.md).
- Do not let terminal, tmux, NTM, Herdr panes, or box topology define the primary UX.
- Require capability negotiation and honest degradation.
- Do not self-resolve HITL tickets.
- Refer to tickets by linked title, never bare issue numbers.
- The final frontier ticket explicitly approves or rejects implementation readiness.

## Decisions so far

- [Define the job BleedingADE must be hired to do](https://github.com/BleedingDev/bleeding-ade/issues/3): build a self-hostable, federated T3-style client/server control plane where one human plans through user-supplied skills, converts plans into Beads, and supervises or hands over 5–100+ long-running agent workstreams across 2–10 machines from any authorized device, with complete semantic history and no assumed role for Herdr.
- [Reconcile the product promises and set hard non-goals](https://github.com/BleedingDev/bleeding-ade/issues/4): v1 requires first-class web/mobile fleet control, global operations plus project-scoped work, T3 provider preservation, capability-gated external runtimes, native `br` graph/triage/statistics, and relocate/takeover/debug-fork; it excludes teams, mandatory native desktop, mobile IDE behavior, live process migration, central execution authority, exhaustive Viewer parity, and unproven optional components.
- [Synthesize the candidate ecosystem and integration strategy](https://github.com/BleedingDev/bleeding-ade/issues/5): use one federated T3-derived semantic core; keep AgentBox, CASS, Herdr/NTM, memory, and peer coordination replaceable at the edges; integrate OMP RPC, Prime RPC, Claude Agent SDK, and `br` through capability-tested boundaries while reusing T3's own terminal implementation.
- [Research T3 Code federation, replication, and extension seams](https://github.com/BleedingDev/bleeding-ade/issues/6): reuse T3's multi-environment client runtime, keep each server authoritative for local execution and history, and add an origin-qualified execution-segment DAG plus designated-peer read-only replication rather than merging local event sequences or centralizing execution.
- [Define deployment, trust, privacy, and ownership assumptions](https://github.com/BleedingDev/bleeding-ade/issues/7): v1 is one operator with many paired clients and independent servers; it reuses T3 access/auth, allows explicitly selected hosts to hold full source, keeps credentials server-local, lets any authorized client control work, resolves approvals only at their origin server, permits full-fidelity trusted replication/search, and keeps external telemetry for later.
- [Define the canonical BleedingADE domain language](https://github.com/BleedingDev/bleeding-ade/issues/8): establish the canonical glossary in [`CONTEXT.md`](../../CONTEXT.md), separating Server/Host/Placement, Project/Checkout/Worktree, Chat/Execution Segment DAG, runtime/agent/workflow/Bead concepts, origin events/replicas/projections, and durable attention; retire bare Environment, Session, Handover, and unqualified History.
- [Research AgentBox execution-location, checkpoint, and handover primitives](https://github.com/BleedingDev/bleeding-ade/issues/21): use AgentBox, if included, as an optional per-server placement adapter; BleedingADE must own semantic identity, portable handover, and fencing because provider-local checkpoints and resume do not migrate live execution.
- [Research Herdr's incremental value and the terminal fallback stack](https://github.com/BleedingDev/bleeding-ade/issues/22): defer Herdr and retain only a conditional leaf-adapter option for otherwise terminal-only runtimes; reuse T3's existing PTY contracts and first-party `libghostty-vt` renderers rather than adding a parallel terminal stack.
- [Research semantic integration surfaces for OMP, Prime Agent, and Claude workflows](https://github.com/BleedingDev/bleeding-ade/issues/23): the strongest primary boundaries are OMP RPC, Prime RPC, Claude Agent SDK, and T3 typed provider adapters; Prime daemon and runtime-native multi-client transports remain optional higher-risk surfaces.
- [Research beads_rust data, synchronization, and native graph-intelligence boundaries](https://github.com/BleedingDev/bleeding-ade/issues/24): `br` owns project-scoped task/workflow state in local SQLite with explicit JSONL/Git interchange; BleedingADE owns stable project identity, movable task authority, federated execution leases, minimal provenance, and fingerprinted native graph intelligence rather than inventing a nonexistent shared Beads service.
- [Research Agent Flywheel components for history, memory, coordination, and orchestration](https://github.com/BleedingDev/bleeding-ade/issues/25): keep Flywheel components optional and non-authoritative—CASS may index replicated T3 history, Eidetic Engine is a later memory probe, Agent Mail is peer coordination, and NTM is at most a terminal backend rather than product navigation or authority.
- [Define `br` task authority and native graph intelligence](https://github.com/BleedingDev/bleeding-ade/issues/13): give each Project one independently movable current Task Authority, route strong mutations through its capability-probed Server adapter, keep Replicas explicitly read-only or eventual, and compute fingerprinted native Graph Projections with runtime overlays.

## Not yet specified

- Exact screen structure, gestures, density, and visual hierarchy beyond the current project-first hypothesis.
- Concrete canonical event schemas and adapter conformance fixtures.
- Exact boundary probes required by the resolved candidate-stack synthesis.
- Migration, release automation, and upstream-sync mechanics.
- Detailed implementation sequencing and the Beads execution graph.

## Out of scope

- Production implementation, scaffolding, or a disconnected prototype during this map.
- Licensing analysis or redistribution decisions.
- Training, fine-tuning, or modifying foundation models.
- Multi-user collaboration, organizations, team roles, shared ownership, or multi-tenant SaaS in v1.
- A required native desktop application in v1.
- Full mobile source editing, IDE behavior, or merge-conflict resolution.
- A custom device-PKI, recent-authentication, or destructive-action confirmation subsystem in v1; reuse T3's scoped pairing/auth model and existing runtime approval behavior.
- A second terminal renderer or `coder/ghostty-web` dependency unless a focused later probe proves T3's existing terminal stack insufficient.
- A terminal/tmux/NTM dashboard as BleedingADE's primary information architecture.
- Copying or embedding Beads Viewer or the Wayfinder Obsidian plugin.
- Exhaustive Beads Viewer parity, universal Bead-to-every-entity relationships, or special multiple-epic orchestration in v1.
- Placement/provider restriction policies in v1.
- Replication-time redaction, pseudonymization, or history mangling in v1.
- Pretending unavailable integrations, semantic events, or migration capabilities exist.
- Live OS-process migration in the initial product; relocate, takeover, and debug-fork remain in scope.

## Ticket index

GitHub issue relationships and blocker links are canonical; this list is the readable frontier.

### Resolved

- [x] [Define the job BleedingADE must be hired to do](https://github.com/BleedingDev/bleeding-ade/issues/3)
- [x] [Reconcile the product promises and set hard non-goals](https://github.com/BleedingDev/bleeding-ade/issues/4)
- [x] [Synthesize the candidate ecosystem and integration strategy](https://github.com/BleedingDev/bleeding-ade/issues/5)
- [x] [Research T3 Code federation, replication, and extension seams](https://github.com/BleedingDev/bleeding-ade/issues/6)
- [x] [Define deployment, trust, privacy, and ownership assumptions](https://github.com/BleedingDev/bleeding-ade/issues/7)
- [x] [Define the canonical BleedingADE domain language](https://github.com/BleedingDev/bleeding-ade/issues/8)
- [x] [Define `br` task authority and native graph intelligence](https://github.com/BleedingDev/bleeding-ade/issues/13)
- [x] [Research AgentBox execution-location, checkpoint, and handover primitives](https://github.com/BleedingDev/bleeding-ade/issues/21)
- [x] [Research Herdr's incremental value and the terminal fallback stack](https://github.com/BleedingDev/bleeding-ade/issues/22)
- [x] [Research semantic integration surfaces for OMP, Prime Agent, and Claude workflows](https://github.com/BleedingDev/bleeding-ade/issues/23)
- [x] [Research beads_rust data, synchronization, and native graph-intelligence boundaries](https://github.com/BleedingDev/bleeding-ade/issues/24)
- [x] [Research Agent Flywheel components for history, memory, coordination, and orchestration](https://github.com/BleedingDev/bleeding-ade/issues/25)

### Current frontier

- [ ] [Choose the federated BleedingADE server and placement topology](https://github.com/BleedingDev/bleeding-ade/issues/9) — HITL, unclaimed

### Blocked decisions

- [ ] [Choose logical-chat identity, continuity, and handover semantics](https://github.com/BleedingDev/bleeding-ade/issues/10)
- [ ] [Choose the origin-qualified semantic event fabric and capability contract](https://github.com/BleedingDev/bleeding-ade/issues/11)
- [ ] [Prove runtime-adapter conformance for T3 providers, OMP, Prime, and Claude workflows](https://github.com/BleedingDev/bleeding-ade/issues/12)
- [ ] [Prototype the project-first web and mobile information architecture](https://github.com/BleedingDev/bleeding-ade/issues/14)
- [ ] [Define global attention inbox semantics and interruption policy](https://github.com/BleedingDev/bleeding-ade/issues/15)
- [ ] [Choose optional search, memory, coordination, and terminal adapters](https://github.com/BleedingDev/bleeding-ade/issues/16)
- [ ] [Define federation failure, recovery, concurrency, and security invariants](https://github.com/BleedingDev/bleeding-ade/issues/17)
- [ ] [Define the minimum coherent release and its acceptance proof](https://github.com/BleedingDev/bleeding-ade/issues/18)
- [ ] [Choose repository, fork, and upstream strategy](https://github.com/BleedingDev/bleeding-ade/issues/19)
- [ ] [Approve the implementation handoff boundary](https://github.com/BleedingDev/bleeding-ade/issues/20)
