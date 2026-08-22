# $handoff — T3 federation, replication, and extension seams

## Target

Resolve [Research T3 Code federation, replication, and extension seams](https://github.com/BleedingDev/bleeding-ade/issues/6).

Follow the [shared research protocol](README.md).

## Objective

Determine how BleedingADE can preserve T3 Code’s client/server and semantic event-sourced architecture while federating independently operational servers, presenting them through one client, optionally replicating history/read models, and supporting relocate/takeover/debug-fork without a central execution authority.

## Investigate

Inspect current `pingdotgg/t3code` code and history around:

- Effect RPC/WebSocket contracts and per-method authorization;
- environment identity, pairing, known environments, relay, SSH, reconnect;
- connection supervision and shared web/mobile/desktop runtime;
- orchestration commands/events, event store, receipts, projectors, reactors, replay;
- provider adapters and provider-session ownership;
- repository/project/worktree/checkpoint/diff models;
- persistence, migrations, updates, terminal support, upstream churn.

Compare:

1. client-side aggregation of independent servers;
2. peer/designated-peer replication;
3. optional non-authoritative backup/search aggregator;
4. centralized designs only as contrast.

Preserve logical chat identity while making execution segments and forks explicit. Test the locked trust breakers: wrong-target commands, lost attention events, and false linearization of branched history.

## Resolution must additionally contain

- component and authoritative-ownership map;
- extension-seam matrix: supported, narrow fork, invasive fork, unsuitable;
- federation/discovery/connection recommendation;
- replication and conflict rules;
- offline/reconnect/replay/staleness implications;
- relocate/takeover/debug-fork feasibility;
- scale implications for 20 projects, 100 chats/top-level agents, thousands of children, 10 machines;
- upstream maintenance budget and falsifiers.

## Suggested skills

- `research`
- `wayfinder`
- `domain-modeling` after the source investigation
