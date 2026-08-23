# BleedingADE agent instructions

## Phase gate

This repository is in **Wayfinding**, not implementation.

Do not add production application code, scaffold the final stack, fork T3 Code, implement adapters, or turn the decision map into an implementation backlog. Implementation begins only after **Approve the implementation handoff boundary** closes with explicit approval.

Cheap disposable probes are allowed only inside a `wayfinder:prototype` decision ticket and must not become a disconnected product prototype.

## Required reading

Before working:

1. Read this file completely.
2. Read `CONTEXT.md` and use its canonical vocabulary.
3. Read `docs/product/initial-vision.md`.
4. Read `docs/wayfinder/map.md`.
5. Read only the claimed ticket and directly relevant resolved tickets.
6. Consult Matt Pocock's Wayfinder skill: `https://github.com/mattpocock/skills/blob/main/skills/engineering/wayfinder/SKILL.md`.

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

## Domain language

`CONTEXT.md` is the canonical glossary. Do not reintroduce retired overloaded terms.

- Use **BleedingADE Server**, **Host**, **Placement**, **Project**, **Checkout**, **Worktree**, **Chat**, **Execution Segment**, **Transition**, **Branch**, **Ownership Epoch**, **Origin Event**, **Replica**, **Attention Item**, and **Task Authority** precisely.
- Never use bare **Session**. Qualify Provider Session, Client authorization session, or Terminal Attachment.
- Replace generic **Handover** with reconnect, provider resume, relocate, takeover, debug fork, recovery fork, checkpoint restore, or restart.
- Qualify **History** as Origin Event history, Chat Timeline, Terminal history, Bead audit history, Replica, search index, or memory.

## Architectural discipline

Treat named tools as candidates with capabilities, not mandatory layers.

The strongest prior is to extend T3 Code's semantic event-sourced model and keep runtime-specific complexity at adapter boundaries. Do not parse terminal output as the primary semantic integration. Terminal data may remain diagnostic provenance and fallback.

Reuse T3 Code's existing server-owned PTYs, terminal contracts, and first-party `libghostty-vt` web/mobile renderers. Do not add another terminal stack unless a focused later probe proves a gap.

Require capability negotiation and honest degradation. An unavailable binary, credential, service, Host, Runtime feature, or semantic event must be reported explicitly.

Do not let tmux, NTM, Herdr panes, or AgentBox boxes become the product's primary information architecture unless a resolved decision explicitly changes that boundary.

## Owner-set constraints

- Licensing is already handled and is outside this Wayfinder effort.
- The selected task system is `Dicklesworthstone/beads_rust` / `br`, not the Dolt-based Beads implementation.
- BleedingADE should provide native Beads graph, dependency, triage, statistics, and insight functionality inspired by Beads Viewer and the Wayfinder Obsidian plugin. Do not embed, clone, or copy their implementation/UI.
- Planning happens in ordinary BleedingADE Chats through user-installed skills. A thin operational `br` skill may be provided, but no planning methodology is baked in initially.
- Web and mobile quality matter more than a fully native desktop app.
- Each BleedingADE Server must remain independently useful; optional replication/aggregation is never central execution authority.
