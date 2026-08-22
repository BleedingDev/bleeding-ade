# $handoff — Agent Flywheel history, memory, coordination, and orchestration

## Target

Resolve [Research Agent Flywheel components for history, memory, coordination, and orchestration](https://github.com/BleedingDev/bleeding-ade/issues/25).

Follow the [shared research protocol](README.md).

## Objective

Identify which currently maintained Jeffrey Emanuel / Agent Flywheel components add distinct value to BleedingADE and expose viable programmatic boundaries.

Resolve authoritative repositories and current names first. Cover at least CASS/session search, current CASS memory component or successor, MCP Agent Mail, NTM, and directly relevant adjacent tools found through primary sources.

## Evaluate against BleedingADE

- one human, many devices and independently operational servers;
- complete T3 semantic history, optionally replicated for backup/search;
- many chats, worktrees, agents, and child-agent graphs;
- Beads as structured project/task state;
- no terminal/tmux/NTM organization in primary UX;
- no mandatory central point of failure.

For each component inspect:

- data ownership and identity;
- API/CLI/MCP/event/history surface;
- deployment and synchronization assumptions;
- scale, failure, security, and operational burden;
- overlap with T3, Beads, AgentBox, and runtime-native coordination;
- honest behavior when absent.

## Resolution must additionally contain

- authoritative inventory and maintenance status;
- capability/overlap matrix;
- integration boundary and stability;
- v1, optional-later, deferred, or excluded recommendation;
- concrete use of replicated T3 history with CASS/search/memory without making it authoritative;
- Agent Mail value relative to Beads claims and child-agent coordination;
- possible NTM backend role and why it must not define navigation;
- federation and security risks;
- minimal later probes.

## Suggested skills

- `research`
- `wayfinder`
