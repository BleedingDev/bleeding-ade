# Initial product vision

## Working name

**BleedingADE** — previously FrankenT3.

## Intent

Create a polished, project-first control plane for coding agents that works from any authorized web or mobile device while one or more machines or isolated execution locations perform the actual work.

The user should see projects, conversations, agents, tasks, workflows, attention items, tools, artifacts, and runtime state—not tmux panes or terminal topology.

## Desired experience

- Top-level switching between projects, environments, or machines.
- A project-local rail for chats, agent sessions, workflows, and relevant tasks.
- A global attention inbox for approvals, questions, failures, conflicts, blocked work, and other actionable events.
- Visual inspection of individual agents, parent/child trees, workflow stages, task dependencies, tools, artifacts, and runtime state.
- Terminal output as a diagnostic/fallback surface only.
- Excellent responsive web and mobile control, including meaningful steering—not merely passive monitoring.

## Strong priors to test

1. Extend T3 Code coherently rather than build a separate frontend or control plane.
2. Preserve T3 Code's existing providers, remote connectivity, authentication, orchestration, event sourcing, and shared client runtime where sound.
3. Use AgentBox for isolated and resumable execution locations where it fits.
4. Treat Herdr as an optional capability whose exact role must be earned.
5. Support T3 Code providers plus Oh My Pi, Prime Agent/RLM, and Claude Code with UltraCode workflows through typed, capability-negotiated adapters.
6. Make Beads task state and native dependency/triage/insight views first-class inside BleedingADE. Reproduce useful capabilities; do not embed or copy Beads Viewer.
7. Keep Agent Mail, CASS, NTM, and other Agent Flywheel pieces optional until their value is proven.
8. Prefer semantic APIs/events over terminal parsing. Preserve raw terminal/runtime provenance without making the UI infer semantics from it.
9. Report missing tools, credentials, services, hosts, and capabilities honestly.

## Desired continuity

A user should be able to leave one device, open another, find the same project and running work, inspect history, answer an approval or question, and continue steering it.

The phrase **same session** is deliberately unresolved. It may mean:

- another client controlling the same running process;
- the same provider conversation/session identity;
- the same transcript and tool history;
- the same worktree and uncommitted state;
- a restored AgentBox checkpoint;
- transferred control ownership;
- equivalent recreation on another execution location.

The Wayfinder map must make this promise precise.

## Candidate semantic fabric

The product may need one normalized semantic event stream covering:

- stable project, environment, machine, runtime, session, turn, agent, task, workflow, and artifact identity;
- session lifecycle and control;
- historical replay and live subscription;
- ordering, timestamps, causation/correlation, idempotency, and reconnect cursors;
- parent/child agents and workflow stages;
- messages, reasoning/workflow state where available, tool calls, approvals, questions, failures, retries, and completion;
- raw runtime provenance plus redaction boundaries;
- explicit capability metadata.

T3 Code already has an event-sourced orchestration model. The decision is what to reuse and extend—not whether to invent a second event system by default.

## Candidate acceptance proof

A coherent first release may need to demonstrate:

1. Switching projects/environments.
2. Switching chats/sessions/workflows within a project.
3. Programmatic session creation and control.
4. Live semantic events from multiple agents and sessions.
5. Historical replay and reconnect.
6. Parent/child agent visualization.
7. UltraCode workflow visualization.
8. Chat/agent/Bead relationships and native task graph views.
9. Global attention events and actions.
10. Cross-device continuity with honest handover semantics.
11. Explicit degradation when optional integrations are absent.

This is input to the minimum-release decision, not a frozen implementation checklist.

## Known tensions

- AgentBox is described as the backend, while T3 Code already treats its server as the execution boundary.
- Herdr is powerful and enjoyable but may duplicate T3 process/session ownership.
- Beads can be canonical task state, an optional adapter, or a projection; these produce different products.
- Cross-device continuity can mean simple reconnection or actual execution migration.
- Broad runtime and Flywheel support conflicts with a smallest coherent first release.
- Web/mobile-first conflicts with full native desktop parity.
- A personal single-user tool and a secure collaborative product imply different trust and lease semantics.
