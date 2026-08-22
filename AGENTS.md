# BleedingADE agent instructions

## Phase gate

This repository is in **Wayfinding**, not implementation.

Do not add production application code, scaffold the final stack, fork T3 Code, implement adapters, or turn the decision map into an implementation backlog. Implementation begins only after **Approve the implementation handoff boundary** closes with explicit approval.

Cheap disposable probes are allowed only inside a `wayfinder:prototype` decision ticket and must not become a disconnected product prototype.

## Required reading

Before working:

1. Read this file completely.
2. Read `docs/product/initial-vision.md`.
3. Read `docs/wayfinder/map.md`.
4. Read only the claimed ticket and directly relevant resolved tickets.
5. Consult Matt Pocock's Wayfinder skill: `https://github.com/mattpocock/skills/blob/main/skills/engineering/wayfinder/SKILL.md`.

## Wayfinder rules

- GitHub issues are canonical.
- The issue labelled `wayfinder:map` is the canonical low-resolution map.
- Refer to tickets by linked title, never by a bare issue number.
- Claim an open, unblocked ticket before work.
- Resolve at most one ticket per session, except parallel research tickets.
- A `mode:hitl` ticket must be resolved with Petr; never answer his side.
- A prototype exists only to improve a decision.
- Record the answer as a resolution comment, close the ticket, then add a one-line context pointer to the map's **Decisions so far**.
- Add new tickets only when the question is precise. Keep the rest in **Not yet specified**.
- Preserve native GitHub sub-issue/dependency relationships where available; issue-body links are the fallback.

## Architectural discipline

Treat named tools as candidates with capabilities, not mandatory layers.

The strongest prior is to extend T3 Code's semantic event-sourced model and keep runtime-specific complexity at adapter boundaries. Do not parse terminal output as the primary semantic integration. Terminal data may remain diagnostic provenance and fallback.

Require capability negotiation and honest degradation. An unavailable binary, credential, service, host, runtime feature, or semantic event must be reported explicitly.

Do not let tmux, NTM, Herdr panes, or AgentBox boxes become the product's primary information architecture unless a resolved decision explicitly changes that boundary.

## Owner-set constraints

- Licensing is already handled and is outside this Wayfinder effort.
- BleedingADE should provide native Beads graph, dependency, triage, and insight functionality inspired by Beads Viewer and the Wayfinder Obsidian plugin. Do not embed, clone, or copy their implementation/UI.
- Web and mobile quality matter more than a fully native desktop app.
