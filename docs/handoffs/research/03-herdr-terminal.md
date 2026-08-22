# $handoff — Herdr’s incremental value and the terminal fallback stack

## Target

Resolve [Research Herdr's incremental value and the terminal fallback stack](https://github.com/BleedingDev/bleeding-ade/issues/22).

Follow the [shared research protocol](README.md).

## Objective

Decide whether Herdr contributes unique necessary value beyond T3 Code’s runtime/server architecture, runtime-native semantic APIs, basic PTY infrastructure, and `coder/ghostty-web` for browser terminal rendering.

Herdr has no presumed role. Inclusion requires proof.

## Investigate

Inspect current Herdr source/docs for:

- server/client split and socket schema;
- session snapshot/events;
- named sessions and persistence;
- PTY/process ownership;
- terminal observe/control;
- remote SSH attach and handoff;
- plugins, agent detection, recovery, platform support, protocol stability.

Inspect `coder/ghostty-web` and Ghostty/libghostty direction. Separate browser terminal emulation from server-side PTY/process/session lifecycle.

Compare:

1. no Herdr;
2. optional diagnostic terminal adapter;
3. bridge for otherwise terminal-only runtimes;
4. generic PTY substrate;
5. externally controlled orchestrator.

Treat screen-based agent detection as inferred, lower-fidelity state unless a semantic source proves otherwise.

## Resolution must additionally contain

- T3 vs Herdr vs basic PTY vs ghostty-web matrix;
- genuinely unique Herdr capabilities;
- coupling, performance, platform, protocol, and maintenance cost;
- whether it materially improves federation, reconnect, takeover, or handover;
- recommendation: required, optional, deferred, or excluded;
- explicit inclusion criteria and falsifiers.

## Suggested skills

- `research`
- `wayfinder`
