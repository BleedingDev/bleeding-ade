# Architecture hypotheses to test

These are deliberately falsifiable starting points, not decisions.

## H1 — T3 Code remains the semantic control plane

Run one T3 server per execution environment. Extend its typed contracts, event-sourced orchestration, projections, reactors, provider/runtime adapters, and shared client runtime. Keep web and mobile clients connected through T3's authenticated RPC boundary.

**Why this is plausible:** T3 already owns projects, threads, provider sessions, terminals, git/filesystem operations, remote environments, authorization, event persistence, and shared nonvisual client state.

**Would be falsified by:** the required multi-runtime semantic model being impossible without replacing most of T3, or upstream churn making a coherent extension more expensive than a clean control-plane core.

## H2 — AgentBox provisions execution locations; it is not automatically the session database

BleedingADE asks AgentBox to create, start, pause, resume, checkpoint, and destroy isolated locations across local and cloud providers. A location may host a T3 server and one or more runtimes. Normal client continuity reconnects to that environment; true migration is a distinct capability.

**Would be falsified by:** AgentBox exposing a more authoritative, durable, semantic cross-machine session fabric that can subsume T3 without duplicated state or terminal parsing.

## H3 — Herdr is optional and capability-scoped

Use Herdr where its durable PTY/process control, named sessions, terminal observation/control, or agent detection adds value for runtimes lacking a better semantic API. Do not expose Herdr workspace/tab/pane topology as the product model.

**Would be falsified by:** Herdr proving to be the cleanest authoritative runtime/session API for all target runtimes without conflicting with T3's event sourcing and project/session identity.

## H4 — Semantic runtime boundaries beat terminal integration

Prefer T3 provider adapters, Oh My Pi ACP/headless/session surfaces, Prime Agent daemon/RPC modes, Claude hooks/workflow-side events, AgentBox APIs, and Herdr's typed socket protocol. Preserve terminal frames and raw provider data as provenance and fallback.

**Would be falsified by:** a target runtime exposing no stable semantic surface while terminal-only support still satisfies the chosen product promise honestly.

## H5 — BleedingADE owns a canonical cross-runtime event envelope, not every runtime's internal ontology

The core event envelope should provide identity, ordering, timestamps, causation/correlation, idempotency, replay, provenance, redaction, and capability metadata. Runtime-specific payloads remain lossless extensions. UI projections consume stable semantic categories rather than provider-specific wire formats.

**Would be falsified by:** the runtimes' semantics being too divergent for useful shared projections without destructive normalization.

## H6 — Beads remains authoritative for project task data; BleedingADE owns the product projection

For projects using Beads, its database remains the task/dependency source of truth. BleedingADE reads and writes through a typed Beads adapter, links tasks to projects/chats/agents/workflows, and implements its own responsive graph, triage, critical-path, bottleneck, and insight views.

Beads Viewer may be used as behavioral/reference evidence or an optional analysis oracle during discovery, but its UI or implementation is not embedded or copied.

**Would be falsified by:** the product job requiring a global task model Beads cannot synchronize, authorize, or query safely across environments.

## H7 — Cross-device continuity precedes cross-machine migration

The first coherent promise is: any authorized device can reconnect to and control the same server-owned work, with explicit controller ownership and multiple observers. Moving live execution between machines is a separate capability requiring provider/runtime-specific guarantees.

**Would be falsified by:** Petr's indispensable workflow requiring actual machine-to-machine migration in the first proof.

## H8 — One global attention projection can unify runtime-specific interruptions

Approvals, questions, failures, conflicts, blocked tasks, lease contention, and recoverable disconnections can be projected into a common actionable item model while retaining runtime provenance and action capabilities.

**Would be falsified by:** materially different interruption semantics that cannot share lifecycle, deduplication, urgency, ownership, and completion rules.

## H9 — Web/mobile are primary clients; desktop is a privileged helper only where browsers cannot act

Most semantic navigation and control should live in shared web/mobile client runtime and responsive UI. A desktop wrapper/helper is justified for local process launch, SSH/tunnel management, keychain integration, filesystem pickers, or OS notifications—not as a separate product architecture.

**Would be falsified by:** core workflows requiring privileged local APIs that cannot be safely delegated to the execution server or a thin helper.
