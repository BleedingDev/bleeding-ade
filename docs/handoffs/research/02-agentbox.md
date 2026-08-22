# $handoff — AgentBox execution-location, checkpoint, and handover primitives

## Target

Resolve [Research AgentBox execution-location, checkpoint, and handover primitives](https://github.com/BleedingDev/bleeding-ade/issues/21).

Follow the [shared research protocol](README.md).

## Objective

Establish exactly what AgentBox can own for execution placement, isolation, lifecycle, checkpointing, relocation, takeover, and local debug forks in BleedingADE’s federated T3-style architecture.

## Investigate

Inspect current `madarco/agentbox` source and docs, including:

- public REST API, provider SDK, CLI, relay/hub/control plane;
- local Docker, remote Docker, Hetzner, Vercel, E2B, and other maintained backends;
- workspace/Git/uncommitted-state synchronization;
- pause/resume, stop/start, snapshots/checkpoints, restore;
- credentials, approvals, identity, failure recovery;
- host-local versus cloud execution differences.

Distinguish precisely:

- reconnect to unchanged remote execution;
- filesystem/worktree relocation;
- logical chat/workstream takeover;
- non-destructive local debug fork;
- checkpoint restore;
- provider-native session resume;
- live process migration.

## Resolution must additionally contain

- provider-specific capability matrix;
- authoritative owner for workspace, process, PTY, provider conversation, credentials, checkpoint, and identity;
- exact preserved and lost state for each operation;
- API/SDK/CLI stability;
- security and operational constraints;
- recommendation: embedded component, API/SDK service, CLI integration, optional placement adapter, or excluded;
- what BleedingADE must implement itself.

## Suggested skills

- `research`
- `wayfinder`
- `domain-modeling`
