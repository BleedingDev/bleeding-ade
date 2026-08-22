# $handoff — Semantic integration surfaces for OMP, Prime Agent, and Claude workflows

## Target

Resolve [Research semantic integration surfaces for OMP, Prime Agent, and Claude workflows](https://github.com/BleedingDev/bleeding-ade/issues/23).

Follow the [shared research protocol](README.md).

## Objective

Identify truthful programmatic boundaries for controlling and observing Oh My Pi, Prime Agent/RLM, Claude Code with UltraCode workflows, and T3’s existing providers without terminal parsing as the primary integration.

First resolve authoritative current names and repositories.

## Investigate per runtime

- RPC, daemon, server, ACP, SDK, structured CLI/JSON, and internal protocols;
- session identity, persistence, history, resume, reconnect, multi-client behavior;
- messages, tools, approvals/questions, artifacts, diffs, usage/cost;
- cancellation, retry, failure, raw provenance;
- parent/child agents and workflow stages;
- live versus historical events.

Classify each boundary:

1. stable documented API/protocol;
2. supported structured CLI mode;
3. internal usable protocol with maintenance risk;
4. terminal-only fallback;
5. unavailable semantic capability.

Do not design the final adapter or run broad prototypes. Establish capability truth and minimal adapter class first.

## Resolution must additionally contain

- per-runtime capability matrix;
- lifecycle/history/resume fidelity;
- agent-tree/workflow fidelity;
- approval/tool/artifact/interruption support;
- protocol stability and maintenance risk;
- recommended adapter class;
- explicit degraded/terminal-only capabilities;
- minimal later probes for unresolved boundaries.

## Suggested skills

- `research`
- `wayfinder`
