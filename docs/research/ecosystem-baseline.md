# Preliminary ecosystem baseline

This is evidence for Wayfinder tickets, not a final architecture decision. Every integration must later be verified against a pinned revision and an executable boundary probe.

| Candidate | Authoritative source | Preliminary programmatic surface | Primary uncertainty |
|---|---|---|---|
| T3 Code | [`pingdotgg/t3code`](https://github.com/pingdotgg/t3code) | Authenticated Effect RPC WebSocket; event-sourced orchestration; provider drivers; shared web/mobile client runtime; remote execution environments. | Which extensions remain upstream-friendly and which require a durable fork? |
| AgentBox | [`madarco/agentbox`](https://github.com/madarco/agentbox) | Local/cloud box provider abstraction; lifecycle, checkpoints, pause/resume; Hub REST API for boxes, jobs, approvals, projects, and logs. | Should T3 run inside each box, and which control-plane writes are stable across all providers? |
| Herdr | [`herdrdev/herdr`](https://github.com/herdrdev/herdr) | Persistent server-owned PTYs; local socket JSON schema; snapshots/events; agent control; terminal observer/controller; named and remote sessions. | Does it provide a narrow capability behind T3, or become a conflicting second owner? |
| Beads Rust (`br`) | [`Dicklesworthstone/beads_rust`](https://github.com/Dicklesworthstone/beads_rust) | Local-first SQLite task graph; JSONL git interchange/synchronization; structured JSON/TOON/robot CLI; capabilities, MCP, skills, coordination and recovery tooling. | Exact authority/sync contract across federated servers, and which mutations belong behind a thin `br` skill or server adapter? |
| Beads Viewer | [`Dicklesworthstone/beads_viewer`](https://github.com/Dicklesworthstone/beads_viewer) | Reads Beads Rust-compatible JSONL; robot JSON/TOON; graph, critical path, PageRank, alerts, planning, and HTML export. | Which behaviors/algorithms should BleedingADE reproduce natively and which should remain optional analysis? |
| Oh My Pi | [`can1357/oh-my-pi`](https://github.com/can1357/oh-my-pi) | Rich coding runtime; subagents; Agent Hub; ACP; collaboration; session restore and multiple provider/tool surfaces. | Stable headless lifecycle, transcript, tool, approval, and child-agent event boundary. |
| Prime Agent | [`PrimeIntellect-ai/prime-agent`](https://github.com/PrimeIntellect-ai/prime-agent) | Daemon and RPC modes; persistent execution; RLM/recursive child-agent behavior. | Lossless parent/child lifecycle, tool, approval, and reconnect semantics. |
| UltraCode workflows | Candidate Claude Code workflow layer; exact authoritative source must be pinned by research. | Workflow definitions, stages, progress/status, observation hooks, diagrams. | Whether progress can be observed semantically without parsing terminal output. |
| Agent Mail | Candidate Agent Flywheel messaging/file-reservation component. | Inboxes, messages, coordination, file reservations. | Whether it adds product value beyond T3 events + Beads ownership. |
| CASS | Candidate Agent Flywheel history/search component. | Cross-agent session/history search. | Whether BleedingADE's own event store and search satisfy the primary job. |
| NTM | Candidate tmux-based orchestration backend. | Process/session orchestration. | Optional backend only; must never define navigation or product identity. |
| Wayfinder | [`mattpocock/skills`](https://github.com/mattpocock/skills/tree/main/skills/engineering/wayfinder) | Decision map, child decision tickets, blocking frontier, HITL/AFK workflow, fog of war. | None; this is the planning method for this repository. |

## Preliminary architectural observations

### T3 Code already contains much of the desired semantic control-plane skeleton

Its documented architecture has one server execution boundary, authenticated method-level RPC, shared nonvisual web/mobile state, event-sourced orchestration, provider adapters, idempotent commands, remote environment identity, and reconnecting clients.

Therefore **extend T3** is a credible default. It also means replacing T3's server with AgentBox or Herdr would discard useful semantics unless those tools are placed behind narrower capabilities.

### AgentBox and Herdr solve different layers, but both overlap T3

A plausible split to test:

- AgentBox owns **where an isolated execution location exists** and its provider-specific lifecycle.
- Herdr optionally owns **durable PTY/process control** for terminal-native runtimes or diagnostics.
- T3/BleedingADE owns **projects, threads, semantic sessions, commands, events, authorization, and client projections**.

The execution-topology decision must assign exactly one authoritative owner for every lifecycle.

### Beads Viewer is not the Beads database

`beads_rust` keeps its working task state in SQLite and exports/synchronizes through JSONL under explicit `br` commands. Beads Viewer consumes the compatible JSONL and derives analysis; it is not the task authority. BleedingADE should use supported `br` surfaces, provide a thin management skill/adapter, and build its own responsive projections rather than write Viewer output or maintain a competing task database.

The separate `gastownhall/beads` Dolt architecture is comparative context only and is not BleedingADE's selected Beads implementation.

### Reconnection and migration are not synonyms

T3, AgentBox, Herdr, and provider runtimes each expose different forms of persistence or resume. The continuity decision must distinguish:

- reconnecting a new client to the same live server/process;
- transferring control ownership;
- resuming a provider-native conversation;
- restoring an execution snapshot;
- recreating equivalent work elsewhere;
- actually moving live execution.

The UI must name these honestly.

## Research standard

For every candidate integration, record:

1. Pinned repository and revision.
2. Maintained installation/runtime assumptions.
3. Typed API, RPC, event, hook, or structured CLI surface.
4. Stable identity and resume behavior.
5. Live and historical observability.
6. Approval/question/tool/child-agent/workflow fidelity.
7. Failure and degradation behavior.
8. Minimal executable probe proving the boundary.