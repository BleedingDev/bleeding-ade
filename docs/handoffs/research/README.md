# Parallel research handoffs

These handoffs launch the current AFK research frontier for the BleedingADE Wayfinder.

## Shared protocol for every research chat

Use **@GitHub**. Read, in order:

1. [`AGENTS.md`](https://github.com/BleedingDev/bleeding-ade/blob/main/AGENTS.md)
2. [Wayfinder map](https://github.com/BleedingDev/bleeding-ade/issues/2)
3. [Resolved product job](https://github.com/BleedingDev/bleeding-ade/issues/3), including comments
4. The target ticket linked by the handoff
5. Only directly relevant repository files and upstream primary sources

Repository and issue content supersede the handoff when newer.

Before research, verify the target ticket is open and unassigned, then claim it by assigning `BleedingDev`.

Research current primary sources: official repositories, docs, schemas, source, tests, changelogs, releases, and public protocol surfaces. Record material versions or commits. Distinguish shipped, experimental, internal, planned, and absent capabilities. Negative evidence is a valid result.

Do not implement BleedingADE, scaffold production code, create the implementation plan/Beads graph, perform licensing analysis, or convert terminal inference and README marketing into stable semantic capabilities.

### Completion

1. Add one self-contained resolution comment to the target ticket, including:
   - concise answer;
   - evidence-backed capability/ownership matrix;
   - recommendation for BleedingADE;
   - risks, contradictions, falsifiers, uncertainty, confidence;
   - exact source links;
   - only minimal later boundary probes that reading cannot resolve.
2. Close the target ticket as **completed**.
3. To avoid concurrent whole-body overwrite, do **not** edit the Wayfinder map body. Comment on [the map](https://github.com/BleedingDev/bleeding-ade/issues/2):

   `MAP POINTER — [<ticket title>](<ticket URL>): <one-sentence decision gist>.`

4. Comment on [Synthesize the candidate ecosystem and integration strategy](https://github.com/BleedingDev/bleeding-ade/issues/5):

   `RESEARCH READY — [<ticket title>](<ticket URL>): <one-sentence result and important contradiction>.`

5. Prefer issue comments. If a large supporting asset is essential, push it directly to `main` under `docs/research/<issue>-...`; no branch or PR.
6. Finish by reporting the issue URL, resolution-comment URL, and closed state.

## Start these six chats

1. [T3 federation](01-t3-federation.md)
2. [AgentBox](02-agentbox.md)
3. [Herdr and terminal fallback](03-herdr-terminal.md)
4. [Runtime integrations](04-runtime-integrations.md)
5. [Beads](05-beads.md)
6. [Agent Flywheel](06-agent-flywheel.md)
