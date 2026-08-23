# $handoff — beads_rust data, synchronization, and native graph intelligence

## Target

Resolve [Research beads_rust data, synchronization, and native graph-intelligence boundaries](https://github.com/BleedingDev/bleeding-ade/issues/24).

Follow the [shared research protocol](README.md).

## Critical correction

The product target is Jeffrey Emanuel's [`Dicklesworthstone/beads_rust`](https://github.com/Dicklesworthstone/beads_rust) and its `br` CLI—not `gastownhall/beads`, `bd`, or the newer Dolt architecture.

The issue contains an earlier Dolt-based resolution produced from the incorrect original handoff. Treat it only as comparative background. It is superseded for BleedingADE product decisions.

## Objective

Determine what `br` should own, what BleedingADE should consume or invoke, and which graph, visualization, triage, prioritization, statistics, and insight capabilities BleedingADE should implement natively.

## Investigate

Inspect current `beads_rust` source/docs:

- SQLite plus JSONL authority and synchronization rules;
- auto-flush, import, merge, reconcile, recovery, and corruption behavior;
- CLI JSON/TOON/robot/capabilities surfaces;
- MCP and skill surfaces;
- dependency, hierarchy, status, priority, label, comment, memory, and link semantics;
- ready/blocked calculation, claims, coordination state, and concurrency;
- repository, worktree, VCS, and cross-machine behavior;
- schema/version compatibility, backups, stale state, conflicts, and failure modes;
- compatibility with `Dicklesworthstone/beads_viewer`.

Inspect Beads Viewer only to inventory graph/list/board behavior, ready/blocked state, cycles, critical path, bottlenecks, actionable triage, project statistics, robot outputs, data assumptions, and interaction patterns. Do not copy or embed its implementation or UI.

Evaluate the locked workflow and scope:

- planning occurs in ordinary BleedingADE chats via user-installed planning skills;
- BleedingADE may ship a thin Beads-management skill that invokes the supported `br` CLI;
- BleedingADE must not create a competing task database or duplicate Beads mutation logic;
- native UI should provide the useful graph/list/board, dependency, status, ready/blocked, cycle, critical-path, bottleneck, triage, and statistics experience;
- v1 does not require relationships from every Bead to every chat, agent, execution segment, commit, and artifact;
- v1 does not require special multiple-epic product semantics;
- v1 does not require parity with every Beads Viewer analytic or robot output.

## Resolution must additionally contain

- authoritative `br` data model and source-of-truth boundaries;
- supported read/write integration surfaces and stability;
- exact role of a BleedingADE-provided `br` skill versus server adapter and native UI;
- concurrency, claims, worktree, and cross-machine sync semantics;
- minimum useful project/chat/workstream correlation without broad v1 relationship machinery;
- native BleedingADE graph/triage/statistics inventory;
- safely recomputable derived metrics;
- ownership split and honest degradation behavior;
- explicit comparison showing which Dolt-research conclusions do not apply;
- minimal later boundary probes.

## Suggested skills

- `research`
- `wayfinder`
- `domain-modeling`