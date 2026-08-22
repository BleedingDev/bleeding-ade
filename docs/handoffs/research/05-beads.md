# $handoff — Beads data, synchronization, and native graph intelligence

## Target

Resolve [Research Beads data, synchronization, and native graph-intelligence boundaries](https://github.com/BleedingDev/bleeding-ade/issues/24).

Follow the [shared research protocol](README.md).

## Objective

Determine what Beads should own, what BleedingADE should read/write, and which graph, visualization, triage, prioritization, and insight capabilities BleedingADE should implement natively.

## Investigate

Inspect current Beads source/docs:

- Dolt embedded/server storage and source of truth;
- schema, migrations, backups, exports;
- CLI, JSON, API, MCP surfaces;
- dependency/hierarchy/link semantics;
- claims, concurrency, worktrees, cross-machine sync;
- messages/memory and repository identity;
- unavailable, stale, schema-skew, conflict, and recovery behavior.

Inspect Beads Viewer only to inventory user-visible capabilities, robot outputs, graph algorithms, data assumptions, refresh model, and interaction patterns. Do not copy/embed its implementation or UI.

Evaluate the locked workflow:

- planning occurs in ordinary BleedingADE chats via user-installed skills;
- a plan creates one or more Beads/epics;
- chats/workstreams/agents relate to and claim Beads;
- large graphs execute across worktrees and machines;
- Beads are project-scoped while multiple execution workstreams may run independently.

## Resolution must additionally contain

- authoritative Beads data model and integration surfaces;
- concurrency and cross-machine sync semantics;
- candidate project/chat/workstream/agent/workflow ↔ Bead relationships;
- native BleedingADE graph/triage/insight inventory;
- safely recomputable derived metrics;
- ownership split and degradation behavior;
- minimal later probes.

## Suggested skills

- `research`
- `wayfinder`
- `domain-modeling`
