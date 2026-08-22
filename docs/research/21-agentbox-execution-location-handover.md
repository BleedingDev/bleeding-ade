# AgentBox execution-location, checkpoint, and handover primitives

**Issue:** [#21 — Research AgentBox execution-location, checkpoint, and handover primitives](https://github.com/BleedingDev/bleeding-ade/issues/21)
**Research date:** 2026-08-23
**AgentBox revision:** [`madarco/agentbox@1de12d36feac92ed5108a8c9fb3606d47d13d3ac`](https://github.com/madarco/agentbox/commit/1de12d36feac92ed5108a8c9fb3606d47d13d3ac), current `main` at research time
**Published packages at that revision:** CLI `@madarco/agentbox` `0.27.1`; provider SDK `@madarco/agentbox-provider-sdk` `2.4.0`, provider API version `2`
**Scope:** Current shipped source and official provider documentation. This is an architecture boundary decision, not an implementation plan.

## Decision

Use AgentBox only as an **optional, per-BleedingADE-server execution-placement adapter**.

AgentBox can reliably own, within one operational server:

- creating an isolated local or provider-backed execution location;
- mapping a local box record to the provider runtime;
- provider-specific start, stop, pause, resume, reconnect, attach, URL, file-transfer, and destruction mechanics;
- workspace seeding and selected Git/workspace synchronization;
- same-provider filesystem checkpoints and new-box restore;
- the enforcement point for AgentBox host-relay approvals and credential forwarding;
- opportunistic Claude/Codex provider-session-file resume.

AgentBox must **not** own:

- BleedingADE project, workstream, thread, chat, command, event, or user-visible session identity;
- federated execution ownership, leases, fencing, relocate/takeover transaction state, or split-brain prevention;
- canonical history or portable checkpoints;
- a cross-provider or cross-server relocation protocol;
- guaranteed provider-conversation continuity;
- live process, socket, memory, or PTY migration;
- the global BleedingADE control plane.

The preferred control order is:

1. the versioned public Hub REST API for the operations it actually exposes;
2. a narrow, exact-version CLI adapter for missing AgentBox operations only, with postcondition probes rather than human-output parsing;
3. upstream API additions for checkpoint, workspace export/import, recover/adopt, and provider-session transfer;
4. never the internal relay/admin/bridge protocol or direct mutation of AgentBox state files.

The provider SDK is a contract for **authoring trusted AgentBox provider plugins**, not a management SDK for consuming AgentBox. Embedding AgentBox internals would make BleedingADE inherit a fast-moving pre-1.0 state model, provider quirks, host-global files, and internal relay protocol. A per-server adapter preserves BleedingADE's federated T3-style authority while allowing AgentBox to be replaced or omitted.

Live process migration is excluded. Same-box freezer/VM pause is valuable local continuity, but it is not relocation.

## Status vocabulary

| Status | Meaning |
|---|---|
| **Shipped** | Reachable in current source through a supported command, public API, or provider path. |
| **Experimental** | Shipped AgentBox code relies on an upstream experimental/beta primitive or an explicitly unstable path. |
| **Internal** | Used by AgentBox itself but not a public compatibility contract. |
| **Planned** | Mentioned as a follow-up or represented only by comments/docs, not a usable boundary. |
| **Absent** | No current first-class primitive was found. |
| **Unknown** | The provider or plugin does not promise enough semantics to use safely without a boundary probe. |

A method name is not a semantic guarantee. In particular, `pause`, `checkpoint`, `reconnect`, and `restore` mean materially different things by provider.

## Required distinctions

The terms below are deliberately non-overlapping.

| Operation | Meaning in BleedingADE | AgentBox result |
|---|---|---|
| **Reconnect unchanged execution** | Rebuild transport and client attachment to the same provider runtime, without moving its filesystem or changing logical owner. | **Shipped.** `Provider.reconnect` and `agentbox recover` rebuild relay registration, provider transport, URLs/tunnels, daemons, and agent attachment. Live process continuity exists only if the provider runtime never stopped. |
| **Filesystem/worktree relocation** | Move a portable, verified workspace representation from source server to target server and start a new execution there. | **Composition only.** AgentBox has file download/upload, Git bundle/relay flows, and provider checkpoints, but no atomic cross-server export/import format or relocation transaction. |
| **Logical chat/workstream takeover** | Transfer authority for the same BleedingADE workstream to another server, preserving semantic history and fencing the source. | **Absent.** AgentBox has local `BoxRecord`, `lastAgent`, and provider-session pointers, not a federated semantic workstream or ownership lease. |
| **Non-destructive local debug fork** | Keep source execution valid while creating an independently mutable copy for investigation. | **Composition only.** A filesystem checkpoint plus new box/worktree can create a copy; no first-class fork contract carries BleedingADE history, safety fences, or side-effect isolation. |
| **Checkpoint restore** | Create execution from previously captured filesystem state. | **Shipped, provider-specific.** AgentBox restore means create a **new box** from a Docker image or provider snapshot; it is not in-place rewind. |
| **Provider-native conversation resume** | Ask Claude/Codex/OpenCode to continue a persisted provider session. | **Partial.** Claude and Codex session files can be teleported/resumed; exactness differs by path. OpenCode resume is absent in the inspected implementation. |
| **Live process migration** | Move running process memory, sockets, kernel state, and PTY to another execution location. | **Absent.** Same-runtime Docker/VM pause is not migration; restored boxes receive new processes and PTYs. |

## AgentBox state model and public surfaces

### Core provider contract

[`packages/core/src/provider.ts`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/core/src/provider.ts) exposes a provider-neutral operational seam:

- create, start, reconnect, pause, resume, stop, and destroy;
- inspect/probe, exec, URL resolution, attach, SSH target, file upload/download;
- optional workspace resync and transport;
- optional checkpoint create/list/remove;
- optional agent-credential extraction;
- optional provider preparation and base fingerprinting.

The state vocabulary is only `running | paused | stopped | missing`. That is too coarse for a federated product. It cannot distinguish:

- runtime definitively deleted;
- provider temporarily unreachable;
- credentials invalid;
- host tunnel broken;
- provider control plane degraded;
- local record stale;
- source intentionally fenced.

`probeState` implementations commonly collapse exceptions into `missing`. BleedingADE must add `unknown` and `unreachable` and must never destroy or relocate solely because AgentBox reports `missing`.

`Provider.reconnect` is explicitly a cheaper sibling of start: it re-establishes host-side connectivity and daemons without power-cycling a runtime that is already alive. This is the correct primitive for client/server reconnection, not for relocation.

The checkpoint contract contains create/list/remove. Restoration is indirect: a later create names a checkpoint/snapshot. There is no generic checkpoint export, import, integrity manifest, portable archive, or cross-provider conversion contract.

### Host-local box registry

[`packages/core/src/box-record.ts`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/core/src/box-record.ts) and [`packages/sandbox-core/src/state.ts`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-core/src/state.ts) persist `BoxRecord` values under the host's `~/.agentbox/state.json`.

A box record carries, among other operational values:

- AgentBox-local box ID and name;
- provider name and provider runtime/container ID;
- workspace/project paths and Git worktree mapping;
- provider URL/tunnel/snapshot fields;
- relay and bridge tokens;
- checkpoint lineage;
- `lastAgent`;
- last host-driven provider state.

The cloud `lastState` field is explicitly not live truth. It records the last host-initiated transition. Live inspection requires a provider probe.

The state file is an **internal operational registry**, not a distributed identity store. It is host-path-dependent and protected by a host-local lock. The lock path can eventually proceed after timeout, accepting possible lost-update risk. BleedingADE must not replicate or concurrently write this file as a federation protocol.

An AgentBox box ID must therefore be stored only as a placement handle:

`{ bleedingAdeServerId, adapterKind, agentBoxBoxId, provider, providerRuntimeId }`

It must never become the BleedingADE workstream or chat identity.

### Public Hub REST API

The Hub documents a versioned OpenAPI 3.1 surface under `/api/v1` in [`apps/web/content/docs/api.mdx`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/web/content/docs/api.mdx).

It currently exposes:

- boxes: list, inspect, create, start, pause, resume, stop, destroy, screen;
- project registration and branch discovery;
- provider credentials/preparation/status;
- Git operations;
- service status/restart;
- approval listing/answering;
- asynchronous jobs and SSE logs.

It currently does **not** expose:

- checkpoint create/list/remove or restore selection;
- portable workspace export/import;
- `recover` or cross-host adoption;
- PTY attach;
- provider-session teleport/resume;
- carry-file transfer;
- direct relocation/takeover/debug-fork transactions.

The Hub backend explicitly says API lifecycle start does not restore agent tmux sessions; that remains a CLI concern. Hosted/serverless Hub profiles also cannot perform host-local execution actions and return unavailable for host-only mutations. The REST API is therefore the best available external contract, but not a complete handover API.

### CLI

The CLI is the complete shipped control surface, but it is pre-1.0 and changing quickly:

- CLI version: `0.27.1`;
- remote Docker registration and addressing changed materially in `0.26`/`0.27`;
- Daytona's default class changed;
- provider SDK versions advanced repeatedly;
- `0.27.1` fixed a fresh-install regression in `0.27.0`.

Some commands provide JSON, but handover-critical commands such as checkpoint and recover are human-oriented and not a clean machine contract. Checkpoint creation also contains TTY-dependent confirmation behavior for provider paths that reboot the source.

If used, BleedingADE must pin the exact CLI version, invoke only a small allowlisted command set, validate exit codes and postconditions, and refuse an unknown version. Human text must not be parsed into semantic events.

### Provider SDK

[`packages/provider-sdk`](https://github.com/madarco/agentbox/tree/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/provider-sdk) is versioned `2.4.0` with provider API version `2`. It compatibility-gates provider plugins and helps plugin authors implement `CloudBackend`.

It is not a box-management client SDK. A provider plugin:

- runs inside the AgentBox host process;
- receives provider credentials and broad host capabilities;
- can execute arbitrary trusted code;
- still inherits AgentBox's local state, relay, checkpoint-manifest, and Hub limitations.

Use it only if BleedingADE eventually supplies a new **placement backend to AgentBox**. Do not use it to consume existing AgentBox installations.

### Relay, bridge, and control plane

[`docs/host-relay.md`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/docs/host-relay.md) and [`apps/hub/README.md`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/hub/README.md) divide the system into:

- a host Node relay with access to host Git, SSH agent, integrations, credentials, and approvals;
- a per-box relay token;
- a separate bridge token for host-to-cloud-box polling;
- loopback-only admin endpoints;
- cloud long-polling for host-only actions;
- an optional hosted Hub/plane for shared status, approvals, and short-lived GitHub App leases.

The relay/admin/bridge wire is explicitly internal and has no stable public schema. It must not be used as BleedingADE's federation protocol.

A Hub may run beside each BleedingADE server. A single global AgentBox Hub must not become required execution authority because that would contradict independently operational federated servers and would still not own T3 semantic history.

## Authoritative ownership

| Concern | Authoritative owner today | What AgentBox owns | Required BleedingADE rule |
|---|---|---|---|
| Project/workstream/thread/chat identity | Not represented by AgentBox | Cosmetic box labels, local box ID, `lastAgent` | BleedingADE owns globally stable semantic IDs. |
| Execution ownership | Host process and provider runtime, without a federated lease | Local `BoxRecord`, provider/runtime ID, last requested state | BleedingADE owns lease epoch, source/target intent, fencing, and takeover commit. |
| Provider sandbox identity | Provider control plane or Docker engine | Mapping from box to provider runtime | Treat as opaque placement identity scoped to server/provider account. |
| Local Docker committed Git objects/refs | Host `.git` mounted read-write | Creates a box branch/worktree and invokes Git | Never assume working-copy files are host-visible; distinguish Git store from worktree bytes. |
| Local Docker uncommitted working files | Box/container writable layer | Seeds host dirty state and can checkpoint/download it | Portable handoff must export and checksum these bytes explicitly. |
| Cloud/remote-Docker workspace and `.git` | Sandbox filesystem | Seeds clone/bundle/overlay; can relay or bundle changes back | Host checkout is not automatically current. Handoff must pull from the sandbox. |
| No-Git workspace | Box/container or sandbox filesystem | Tar seed/download | BleedingADE must define inclusion/exclusion, ignored-file, symlink, mode, and integrity rules. |
| Process, RAM, sockets | Container, VM, VPS, or provider session | Calls provider lifecycle operations | Never infer process continuity from filesystem continuity. |
| PTY/tmux | Process namespace inside the current runtime | Creates/attaches tmux and provider CLI sessions | PTY is an operational attachment, never product identity. |
| Provider conversation | Provider CLI/session files plus provider account/backend | Copies selected session files, stores minimal pointers, passes resume flags | BleedingADE owns semantic transcript/history and records provider resume as optional continuity. |
| Provider infrastructure credentials | Host environment or `~/.agentbox/secrets.env` | Validates/forwards them to provider adapter | Credentials remain node-scoped references; do not replicate secrets as workstream state. |
| Git credentials | Host relay/SSH agent/config, hosted GitHub App lease, or deliberately copied direct credentials | Enforces the selected mode | BleedingADE records policy and approval; direct credentials must never transfer implicitly. |
| Agent authentication | Shared Docker volumes/host backups, Daytona shared volume where usable, or per-create upload/in-box files | Seeds/extracts/fans out selected auth files | Treat as independent secret state with provider-specific portability and revocation. |
| Host-action approval | Local relay prompt state or hosted Postgres approval row | Enforcement point for AgentBox host RPC | BleedingADE projects the approval semantically and fences both sides during takeover. |
| Checkpoint bytes | Docker image or provider-native snapshot/image | Captures/deletes and names through provider adapter | Bytes are operational optimization, not canonical history. |
| Checkpoint name/lineage | Host-local manifest under `~/.agentbox` | Maps project/name to artifact and source metadata | Manifest and artifact are both required; back up/verify independently if relied upon. |
| Endpoint/tunnel | Provider plus host-local SSH/SDK/tunnel process | Re-resolves or recreates on reconnect | Endpoint is ephemeral; clients resolve through the owning BleedingADE server. |
| Durable semantic history | BleedingADE event store | None | Must remain portable and independently replicated from execution state. |

## Workspace and Git continuity

### Local Docker with Git

AgentBox's local Docker model deliberately separates two kinds of state:

1. the host's `.git` database is bind-mounted read-write;
2. the box's worktree files live in the container's writable filesystem.

Consequences:

- commits and refs created in the box become visible through the shared Git database;
- uncommitted file changes are not equivalent to host working-tree changes;
- the host's dirty tracked state is captured through `git stash create` and replayed into the box;
- host untracked files are transferred separately;
- checkpointing captures the box filesystem but not arbitrary mounted volumes.

This topology is efficient on one host but not relocatable merely by copying a host checkout.

### Remote Docker and cloud providers

Remote Docker is intentionally treated like a cloud provider because host bind mounts and relay sockets do not cross the network.

Cloud-style creation:

- transfers a shallow clone or Git bundle;
- replays host uncommitted tracked and untracked state;
- leaves the sandbox with its own `.git`;
- routes remote operations through host relay/lease/direct modes;
- requires explicit bundle/push-host/download to land box-local work back on the host.

A cloud source can therefore contain commits and files that do not exist in the host checkout. Relocation must read the source execution, not reconstruct from the target's repository alone.

### Resync is not byte-for-byte restore

AgentBox can merge a host branch and overlay host dirty state into a restored box. Current resync behavior is box-wins on conflicts and reports/skips conflicts rather than creating ordinary conflict markers.

That is useful for warm starts, but it means a checkpoint restore may be intentionally modified by current host state. BleedingADE needs two separate operations:

- **restore exact artifact** for audit/debug/forensics;
- **restore and rebase/overlay current project state** for continued development.

The UI and event log must not call both simply “restore.”

### `carry` files and ignored state

AgentBox can copy selected files outside the workspace into a box. The box record retains audit metadata, not the original content as a portable object. Git-only transfer also omits ignored but operationally required content unless separately selected.

A BleedingADE portable workspace bundle must define, at minimum:

- Git refs/objects and desired branch;
- tracked, untracked, and explicitly included ignored files;
- symlinks, file modes, sparse/submodule/LFS behavior;
- deleted paths and case sensitivity;
- required external files;
- content hashes and total manifest hash;
- encryption and secret classification;
- producer/consumer versions;
- checkpoint and provider-session references;
- verification result on target.

AgentBox does not currently provide this bundle.

## Provider lifecycle capability matrix

Legend: ✅ guaranteed by current AgentBox path; ◐ partial/provider-dependent; ❌ absent; ? not sufficiently promised.

| Provider | Isolation/runtime | Reconnect unchanged runtime | Pause semantics | Stop/start semantics | Live process/PTY continuity | Material host-local dependency |
|---|---|---|---|---|---|---|
| **Local Docker** | Linux container; shared host kernel | ✅ Re-register relay/endpoints and idempotently recover container | ✅ Real `docker pause`/unpause freezes container processes | Writable layer survives; processes/tmux die and daemons relaunch | ✅ across pause only; ❌ across stop | Local Docker engine, `~/.agentbox`, host `.git`, relay and volumes |
| **Remote Docker** | Docker container on SSH-reachable engine | ✅ if registered host alias and SSH path still resolve | ✅ Remote `docker pause` | Remote container layer survives; processes die | ✅ across pause only; ❌ across stop | Registered SSH alias/key, local record/tunnel, exact remote Docker engine |
| **Daytona Linux VM** | Dedicated Linux VM class | ✅ by sandbox ID plus daemon/URL re-ensure | ✅ Provider pause preserves VM memory/processes | Filesystem survives stop/start; memory is cleared | ✅ across provider pause; ❌ across stop | Daytona account/API credentials, local record; region/class constraints |
| **Daytona container** | Provider container class | ✅ by sandbox ID | ❌ No native pause; AgentBox uses stop/archive-style cold path | Filesystem survives; processes die | ❌ | Daytona account/API credentials and local record |
| **Vercel** | Firecracker microVM compute session with persistent sandbox identity/filesystem | ✅ by provider identity; stopped sandbox resumes into a new session | ❌ AgentBox maps pause to stop/persist | Filesystem auto-snapshots; resume boots new compute session | ❌ | Vercel team/project credentials, local record, snapshot retention |
| **E2B** | Firecracker microVM sandbox | ✅ `connect` by sandbox ID, including another caller/location | ◐ Provider pause/connect is shipped, but reviewed docs do not promise enough about exact RAM/process/PTY identity | AgentBox stop maps to pause | ? same-sandbox process continuity; do not rely without probe | E2B API key, sandbox ID, local record |
| **Hetzner** | One VPS per box | ✅ same VPS if SSH identity and firewall allow it | ❌ AgentBox pause equals power-off | Disk survives; process, memory, sockets, tmux die | ❌ | Host-local per-box private key, known-hosts, current egress firewall, token |
| **DigitalOcean** | One Droplet per box | ✅ same Droplet if SSH identity and firewall allow it | ❌ AgentBox pause equals power-off | Disk survives; process, memory, sockets, tmux die | ❌ | Host-local per-box private key, known-hosts, firewall, token |
| **Provider plugin** | Plugin-defined | ? | ? | ? | ? | Trusted in-process plugin, credentials, any plugin-specific local state |

Important implications:

- Docker and Daytona VM have strong **same-runtime pause**. That is useful for temporary source fencing and instant local resume.
- Vercel's “persistent sandbox” separates durable filesystem identity from replaceable compute sessions.
- Hetzner/DigitalOcean “paused” is a UI normalization of powered off, not a frozen process.
- A generic `supportsPause: true` flag would be unsafe. BleedingADE needs semantic capabilities such as `freezesMemory`, `killsProcesses`, `keepsRuntimeId`, `sourceRemainsRunnable`, and `requiresHostPrivateKey`.

## Provider checkpoint and restore matrix

| Provider | Artifact and manifest | Source effect during capture | Restore target | Preserved by current AgentBox path | Lost/not guaranteed | Portability |
|---|---|---|---|---|---|---|
| **Local Docker** | Local Docker image plus project-scoped host manifest; layered or flattened | `docker commit` briefly pauses source by default, then source continues | New container/box on the same Docker engine | Container root filesystem, `/workspace`, installed dependencies and caches not in mounts | Process/RAM/PTY, transient kernel state, independent volumes, host-only relay state | Engine-local unless manually exported; AgentBox has no export/import workflow |
| **Remote Docker** | Docker image on one remote engine plus local host manifest; ref embeds remote host | Source is committed; remains usable | New container on the **same** remote engine | Remote container root filesystem/workspace | Process/RAM/PTY and independent volumes | Explicitly refuses restore on a different remote host |
| **Daytona Linux VM** | Provider snapshot plus host-local cloud manifest | Current AgentBox cold path stops source, snapshots, starts/reconnects it | New Daytona sandbox | Filesystem/root image state | Current implementation does not preserve live process/PTY; upstream hot snapshots/fork are not exposed as the AgentBox checkpoint contract | Daytona-scoped; not a portable BleedingADE artifact |
| **Daytona container** | Provider snapshot plus manifest | Stops/archives, snapshots, restarts | New Daytona container sandbox | Filesystem/root state | Process/RAM/PTY | Daytona-scoped |
| **Vercel** | Vercel snapshot ID plus manifest | `snapshot()` stops source; source remains stopped until later resume | New Vercel sandbox/session from snapshot | Filesystem, installed packages, sandbox configuration represented by provider snapshot | Live process/RAM/PTY/session identity | Vercel project/provider scoped; retention/expiry applies |
| **E2B** | E2B snapshot ID plus manifest | Source is paused while snapshot is created and remains resumable | New E2B sandbox from snapshot | Provider promises the same filesystem and “state” | Exact process/RAM/socket/PTY continuity in the new sandbox is not explicit enough to treat as live migration | E2B-scoped; snapshot survives source sandbox deletion per provider docs |
| **Hetzner** | Hetzner server snapshot/image plus host manifest | AgentBox does not stop or quiesce source before provider image call | New Hetzner server from image | Provider disk/image contents | Process/RAM/PTY; application consistency is provider/workload-dependent | Hetzner-scoped; image/region/account constraints |
| **DigitalOcean** | Droplet snapshot plus host manifest | AgentBox invokes provider snapshot without an AgentBox quiesce phase | New Droplet from snapshot | Provider disk/image contents | Process/RAM/PTY; application consistency is provider/workload-dependent | DigitalOcean-scoped |
| **Provider plugin** | Plugin-defined artifact plus AgentBox manifest if checkpoint is implemented | ? | Usually new box | ? | ? | Must be declared and proven |

Every named checkpoint is two-part state:

1. the Docker/provider artifact contains the bytes;
2. a host-local manifest maps the project-scoped name to that artifact and records source/lineage/fingerprint information.

Losing the artifact leaves a dangling manifest. Losing the manifest leaves an orphan artifact that AgentBox cannot resolve by the user-facing checkpoint name. Neither part is a BleedingADE-portable checkpoint.

Checkpoint restore always creates a new AgentBox box with new operational identity and tokens. It does not restore a BleedingADE workstream lease, T3 event history, an attached PTY, or the previous process tree.

### Provider maturity notes

| Provider path | Classification at pinned revision | Evidence/constraint |
|---|---|---|
| Local Docker | **Shipped** | Core path with native Docker lifecycle and local image checkpoints. |
| Remote Docker | **Shipped, recently reworked** | Built-in backend; host-alias addressing and preparation changed materially in recent CLI releases. |
| Daytona lifecycle | **Shipped** | Both container and Linux VM classes are implemented. Their pause semantics differ. |
| Daytona checkpoint | **Shipped over experimental upstream API** | AgentBox invokes Daytona's `_experimental_createSnapshot` and intentionally uses a cold stop/snapshot/start path. |
| Vercel | **Shipped** | Uses current persistent sandbox/snapshot SDK model; provider persistence is GA, while AgentBox remains pre-1.0. |
| E2B | **Shipped with evolving upstream pause surface** | AgentBox implements pause/connect and snapshot; E2B SDK documentation has recently carried beta/deprecated pause naming. |
| Hetzner | **Shipped** | Full VPS backend is implemented despite stale comments elsewhere calling parts of the provider a stub. |
| DigitalOcean | **Shipped** | Full Droplet backend is implemented; cross-host adoption remains constrained by host-local SSH identity. |
| Provider plugins | **Shipped extension seam; semantics unknown** | SDK API version 2 compatibility-gates plugins, but each plugin remains trusted and provider-specific. |

## Exact state preservation by operation

### 1. Reconnect to unchanged remote execution

**AgentBox support:** shipped through `Provider.reconnect` and `agentbox recover`.

Preserved when the runtime remained alive:

- provider runtime/container/sandbox ID;
- filesystem and local Git state;
- running processes and tmux only where the runtime itself remained running or strongly paused;
- provider session files already on disk;
- provider-side resources.

Rebuilt or changed:

- host relay registration;
- cloud poller;
- preview URLs/tokens;
- SSH ControlMaster and local forwards;
- Portless aliases;
- in-box AgentBox daemons;
- local attachment.

Potentially lost:

- host-only in-memory relay events;
- old tunnels and URLs;
- pending transient UI attachment state;
- project linkage when using `recover --adopt`;
- live process continuity if the provider had stopped/expired.

`recover --adopt` can rebuild a minimal local record for some untracked cloud sandboxes and mint fresh relay/bridge tokens. It cannot reconstruct host project root/index, original host Git worktree paths, BleedingADE identity, or checkpoint manifests.

Hetzner adoption is rejected without the original per-box private SSH key. DigitalOcean has the same backend key dependency even though the recover command's explicit preflight/help is currently Hetzner-focused. This is operational adoption, not federated takeover.

### 2. Filesystem/worktree relocation

**AgentBox support:** absent as one operation; possible only as a higher-level composition.

Potentially preservable with explicit export:

- selected Git objects and refs;
- committed and uncommitted workspace files;
- installed dependencies/build caches if a filesystem archive is chosen;
- selected provider-session files;
- selected non-workspace carried files.

Lost unless BleedingADE separately captures them:

- process tree, RAM, sockets, locks, open file descriptors;
- PTY/tmux attachment;
- provider-native volumes and nested-Docker state;
- host relay state and approvals;
- host paths and local worktree topology;
- provider snapshot lineage;
- secrets and credentials intentionally excluded from transfer;
- files omitted by Git/tar rules;
- source execution ownership/fencing state.

A provider-native checkpoint is not a relocation format because restore is bound to the same provider or Docker engine. A Git-only move is not a full workspace move because ignored/untracked/runtime-required files may be omitted. A raw tar move is not sufficient without Git/ref, mode, symlink, secret, and integrity semantics.

### 3. Logical chat/workstream takeover

**AgentBox support:** absent.

AgentBox can preserve or reconstruct an execution location and can sometimes resume a provider conversation. It does not have:

- a globally stable workstream ID;
- an append-only semantic event history;
- source/target ownership intent;
- lease epoch/fencing token;
- atomic handover commit;
- rollback status;
- source prohibition after target activation;
- replicated authorization/approval decisions.

BleedingADE must transfer the semantic event stream and ownership independently of execution bytes. Provider conversation resume should be recorded as one optional continuity result:

- resumed exact provider session;
- resumed provider's latest session;
- started fresh with semantic history/context;
- unsupported/failed.

### 4. Non-destructive local debug fork

**AgentBox support:** no first-class semantic fork; feasible composition.

A safe composition can:

1. quiesce or strongly pause the source long enough to obtain a consistent filesystem point;
2. capture a checkpoint or portable workspace bundle;
3. resume the source;
4. create a new local box/worktree/branch from the capture;
5. mark the new BleedingADE workstream as a debug child.

Preserved:

- source execution remains available after capture;
- filesystem/Git state at the fork point;
- selected provider-session artifacts if deliberately copied.

Not preserved:

- live process/PTY identity in the child;
- source's exact in-memory state, except provider-native mechanisms not exposed by AgentBox;
- automatic semantic parent/child relation;
- side-effect safety.

Copying a provider session into both source and child can create two writers against the same remote conversation and can repeat tool/host actions. The default debug fork should therefore use a new semantic workstream and either a fresh provider session seeded from history or an explicitly read-only/restricted provider-session clone.

Daytona now has upstream VM fork/hot-state capabilities, but current AgentBox does not expose them as its generic checkpoint/debug-fork contract. They cannot be assumed.

### 5. Checkpoint restore

**AgentBox support:** shipped, same provider/engine, new box.

Preserved:

- whatever root filesystem/disk state the provider artifact captured;
- checkpoint metadata still present in the local manifest;
- warm dependencies/build state;
- provider session files only if they reside in captured paths rather than excluded/mounted credential stores.

Changed or lost:

- new AgentBox box ID;
- new relay and bridge tokens;
- new runtime/container/sandbox identity in most providers;
- new process tree, PID values, sockets, tmux server, and PTY;
- independent volumes;
- transient environment and tmpfs token files;
- source ownership and approvals;
- exact bytes if host resync/overlay is enabled after restore.

Restore is not rewind. The source box can continue to exist, and the restored box can diverge. BleedingADE must model a child placement and explicit workstream relation.

### 6. Provider-native conversation/session resume

AgentBox has two distinct mechanisms.

#### Host-to-box session teleport

[`apps/cli/src/session-teleport/claude.ts`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/session-teleport/claude.ts) and [`codex.ts`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/session-teleport/codex.ts) copy selected host session files into a box and invoke native resume.

| Provider agent | Teleport behavior | Exactness | Important loss/risk |
|---|---|---|---|
| **Claude** | Copies one JSONL session, rewrites matching host workspace/cwd metadata to `/workspace`, launches with explicit `--resume <id>` | Exact session ID in the supported file shape | Does not move live process/PTY; only selected path fields are rewritten; provider/backend validity still required |
| **Codex** | Copies one rollout JSONL, rewrites path strings in `session_meta` and `turn_context`, invokes `codex resume <uuid>` | Exact UUID on this path | Does not move process/PTY; transcript/tool records remain unchanged and may reference source assumptions |
| **OpenCode** | No inspected teleport implementation | Unsupported | Fresh session required |

#### Same-box recovery after stop/start

[`apps/cli/src/agent-sessions.ts`](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/agent-sessions.ts) restores sessions best-effort:

- Claude stores an exact session pointer and resumes that ID;
- Codex records an active marker and uses `resume --last` in the working directory, not an exact durable pointer;
- OpenCode has no corresponding resume path;
- failure logs a warning and does not fail box start;
- when no session is resumable, recovery can start `lastAgent` fresh.

Provider-native resume preserves only what the provider CLI persisted and accepts. It does not preserve:

- the old process or terminal;
- unflushed in-memory provider state;
- BleedingADE semantic event identity;
- source/target fencing;
- exact tool/environment assumptions;
- a guarantee that the provider permits two concurrent resumptions.

### 7. Live process migration

**AgentBox support:** absent.

No inspected path transfers:

- process address space;
- kernel task state;
- open sockets;
- file descriptors and locks;
- active network sessions;
- tmux server and PTY;
- host tunnels;
- in-flight tool process;
- runtime memory across providers or servers.

Docker pause, Daytona VM pause, and possibly E2B pause preserve or suspend the same provider runtime. They do not move it. A provider snapshot that starts a new sandbox is not evidence of live migration.

## Credentials and approvals

### Provider infrastructure credentials

Hub/API credential setup writes provider secrets to the AgentBox host. Provider operations run on that host or through its Hub backend. These credentials are node authority and should remain outside replicated workstream history.

### Agent credentials

AgentBox uses several provider-specific paths:

- shared Docker volumes and host backup files;
- a shared Daytona credentials volume where the sandbox class supports it;
- per-create credential uploads for backends without compatible volumes;
- environment forwarding for API-key-based authentication;
- extraction/fan-out of refreshed credentials back to the host.

A checkpoint may capture agent auth if it is in the root filesystem, but may omit it when it lives on an external mounted volume. BleedingADE must never infer credential portability from checkpoint success.

AgentBox strips selected per-instance secrets from flattened Docker image configuration, but that does not prove the workspace/root filesystem contains no secrets. Direct credential modes deliberately copy secrets into the box; provider snapshots can then retain them.

### Git credentials

AgentBox supports three materially different trust modes:

- **relay:** host retains credentials and executes remote Git operations;
- **lease:** a hosted plane mints a short-lived repository-scoped GitHub App token;
- **direct:** credentials are copied into the box.

Direct mode is explicitly dangerous and can make checkpoints secret-bearing. BleedingADE relocation must transfer credential **references/policy**, not blindly copy source credentials.

### Approvals

The host relay is the actual security boundary for host-only Git/integration operations; the in-box Git shim is only a guardrail. Local approvals can be transient; hosted approvals can be persisted in Postgres.

During relocate/takeover/debug fork:

- source and target must not both retain unrestricted host-action authority;
- pending source approvals must be cancelled or tied to the old lease epoch;
- target approvals must be issued under the new execution lease;
- a replayed provider session must not silently replay already-approved actions.

AgentBox can enforce an individual host action. BleedingADE must own the cross-server authorization event and lease binding.

## Security and operational constraints

1. **AgentBox is privileged orchestration.** The relay runs with host Git/SSH/integration credentials. Provider plugins execute as trusted in-process code. Treat both as part of the server trust boundary.

2. **Local Docker is not a hostile multi-tenant boundary.** It shares the host kernel and mounts the host Git database read-write. It is suitable for user-owned coding workloads, not for mutually hostile tenants without stronger isolation.

3. **Provider semantics differ despite one interface.** “Pause” ranges from cgroup freezer to VM memory freeze to power-off. “Checkpoint” ranges from Docker rootfs image to cold VM image to provider snapshot. Capability negotiation must describe effects, not only method presence.

4. **Host-local SSH identity can strand VPS boxes.** Hetzner and DigitalOcean keep per-box private keys on the creating host. A second host cannot adopt/control the VPS merely from provider account access. Firewall rules can also bind reachability to the old host's egress IP.

5. **Provider/network errors can appear as missing.** Do not auto-destroy, fail over, or declare source absent on one AgentBox probe. Require provider-independent confirmation and a grace/unknown state.

6. **Checkpoint consistency is workload-dependent.** Docker pauses processes during commit, but mounted volumes are excluded. Daytona/Vercel use provider-controlled stop/snapshot paths. Current Hetzner/DigitalOcean capture does not perform an AgentBox application-quiesce phase. Databases and services need explicit flush/freeze hooks.

7. **Checkpoint retention is not durability.** Provider snapshots can expire, be deleted out of band, or become stale relative to a rebuilt AgentBox base image. Monitor artifact liveness and retain a portable workspace/history independently.

8. **Destroy is irreversible for uncaptured box-local state.** Cloud/remote work can exist only in the sandbox. A successful Git push does not prove ignored/untracked/runtime files were preserved.

9. **Credential fan-out and dual execution create races.** “Newest wins” credential updates, token rotation, and two resumed sessions can invalidate or overwrite each other. Fence credentials and host actions by execution lease.

10. **The hosted Hub is not a universal executor.** Serverless/plane paths deliberately reject host-local actions. Do not make it a required global authority for independently operational BleedingADE servers.

11. **Internal state and docs show churn/drift.** Current code has stale comments around some provider implementations and older control-plane documentation. Pin executable source and probe behavior rather than relying on marketing or an old architecture note.

12. **Snapshot source effects vary.** Some captures briefly freeze and continue the source; some stop it; some leave it paused; some do not quiesce it. BleedingADE must record the effect and explicitly restore or retain the source.

## Recommended BleedingADE integration boundary

### Role classification

| Candidate role | Decision | Reason |
|---|---|---|
| **Embedded component** | Reject | Couples BleedingADE to AgentBox internals, host-global state, relay protocol, provider package graph, and pre-1.0 churn. |
| **External API/SDK service** | Partial | Public REST `/api/v1` is preferred for exposed operations, but lacks checkpoint, transfer, recover, attach, and session handover. Provider SDK is the wrong direction. |
| **CLI integration** | Transitional only | Broadest shipped surface, but many critical commands are human-oriented and unstable. Exact pin plus postcondition probes required. |
| **Optional placement adapter** | **Adopt** | Keeps execution mechanics replaceable and local to each federated server while BleedingADE retains semantic authority. |
| **Excluded** | Live migration and internal protocol integration | No reliable primitive and not required by the product job. |

### Adapter authority

Each BleedingADE server may run its own AgentBox installation and optional Hub. The adapter may return operational handles and observations, but BleedingADE remains authoritative for:

- project/workstream/chat/event IDs;
- which server currently owns execution;
- source/target handover state;
- lease/fencing epoch;
- checkpoint intent and semantic checkpoint record;
- user-facing approvals and audit;
- retry/rollback/orphan policy;
- portable workspace/history;
- provider-session continuity result.

The adapter must expose semantic capabilities discovered from the exact AgentBox/provider combination. At minimum:

- unchanged-runtime reconnect;
- strong memory-preserving pause vs process-killing stop;
- source effect of checkpoint;
- filesystem checkpoint consistency;
- exact restore scope;
- workspace export/import scope;
- independent volume inclusion;
- provider-session resume by agent and exactness;
- host-key portability;
- checkpoint retention;
- public API coverage;
- failure observation quality.

Provider method presence is insufficient.

### Control transport

Use the following hierarchy:

1. **REST `/api/v1`** for box creation/listing/lifecycle, projects, providers, jobs/logs, approvals, Git, and service operations.
2. **Exact-version CLI bridge** for checkpoint/recover/session/file operations not represented in REST, only behind a server-local adapter.
3. **Postcondition verification** after every mutation: inspect provider runtime identity/state, target filesystem manifest, source state, and AgentBox record.
4. **No direct reads/writes** of `~/.agentbox/state.json` or checkpoint manifests as a supported integration.
5. **No calls** to relay `/admin`, `/rpc`, or `/bridge` as a BleedingADE protocol.
6. Prefer upstreaming public checkpoint/export/recover/session endpoints before making AgentBox a default backend.

## What BleedingADE must implement itself

AgentBox does not remove the need for these product primitives.

### Semantic state

- canonical event-sourced project/workstream/chat history;
- stable globally scoped IDs;
- provider-neutral message/tool/approval/checkpoint events;
- mappings to every placement and provider session;
- replication and client projections independent of execution location.

### Ownership and handover transaction

- execution lease with monotonically increasing fencing epoch;
- source freeze/quiesce intent;
- target preparation and verification;
- atomic semantic owner switch;
- cancellation and rollback;
- stale-source denial after takeover;
- orphan detection and cleanup;
- explicit distinction between relocate, takeover, restore, and debug fork.

### Portable workspace checkpoint

- provider-neutral content manifest and hashes;
- Git refs/objects plus complete selected working state;
- ignored/external file policy;
- symlink/mode/submodule/LFS semantics;
- encryption and secret redaction/classification;
- chunked transfer/retry;
- target verification;
- provenance linking source server, source placement, semantic event position, and provider artifact;
- exact-restore and overlay/rebase modes.

Provider-native snapshots can be recorded as acceleration artifacts attached to this checkpoint, never as the only canonical representation.

### Provider conversation continuity

- per-agent capability records;
- export/import adapters;
- exact vs latest vs fresh-resume result;
- path/environment rewrite audit;
- duplicated-session warning and side-effect policy;
- fallback context reconstruction from BleedingADE semantic history;
- provenance mapping from provider session ID to workstream and placement.

### Health and safety

- `running`, `strongly_paused`, `cold_stopped`, `unreachable`, `unknown`, `missing`, `destroyed`, and `fenced` states;
- multi-signal failure confirmation;
- credential/approval rebinding to lease epoch;
- source-side action denial during takeover;
- snapshot retention monitoring;
- audit of every adapter command and observed postcondition.

### Debug fork semantics

- parent/child workstream relation;
- branch/workspace copy provenance;
- read-only or restricted host-action policy;
- optional fresh provider session seeded from semantic history;
- explicit promotion/merge back, independent from source execution.

## Contradictions and negative evidence

1. **“Pause” is not one capability.** Local/remote Docker and Daytona VM can preserve live execution; Daytona container, Vercel, Hetzner, and DigitalOcean cannot under current AgentBox paths.

2. **“Snapshot” is not migration.** Every AgentBox restore path creates a new box. No generic contract restores the old process/PTY.

3. **AgentBox now has provider-session teleport, but not workstream handover.** Copying Claude/Codex files can preserve provider conversation continuity while leaving BleedingADE identity, execution ownership, and side-effect fencing completely unresolved.

4. **The public REST API is versioned but incomplete.** The most important handover operations remain CLI/internal concerns.

5. **The provider SDK is stable in a different direction.** It helps AgentBox load provider backends; it does not offer an application management client.

6. **Cloud checkpoints are not centrally portable.** A local manifest points to a provider-owned artifact and can be invalidated by retention/deletion.

7. **Adoption is not takeover.** It rebuilds a minimal local record and fresh relay tokens but loses host project linkage and cannot overcome host-local VPS SSH keys.

8. **Provider-native capabilities can exceed AgentBox's contract.** Daytona VM hot snapshots/forks exist upstream, but current AgentBox checkpoint intentionally follows a cold filesystem path. BleedingADE must negotiate what AgentBox actually exposes, not what the provider could theoretically do.

9. **A live box can be misreported as missing.** Provider probes often convert connection/credential errors to `missing`, making automatic failover unsafe.

## Falsifiers and minimal boundary probes

These probes are the smallest remaining empirical checks that source reading cannot settle. They should be run against the exact pinned AgentBox version before enabling a provider in production.

### 1. Lifecycle continuity canary

For each provider:

- run a process with a stable PID, open PTY/tmux session, in-memory counter, open listening socket, and periodically flushed file;
- pause/resume, then stop/start;
- record whether PID, PTY, socket, memory counter, filesystem counter, provider runtime ID, and endpoint survive.

**Falsifier:** any provider differs from the semantic capability advertised by the adapter.

### 2. Checkpoint completeness and consistency

Create a source containing:

- committed, modified tracked, untracked, ignored-required, executable, symlink, and deleted files;
- a mounted volume;
- an active SQLite or database workload;
- provider-session and credential files in both rootfs and mounted paths.

Capture/restore and compare a cryptographic manifest.

**Falsifier:** AgentBox/provider captures more or less than the matrix, or restore resync mutates an expected exact artifact.

### 3. Provider-session exactness

Create two Claude sessions and two Codex sessions in one workspace, then test:

- host-to-box teleport by explicit ID;
- stop/start recovery;
- cross-workspace path rewrite;
- source and target concurrent resume;
- OpenCode negative path.

**Falsifier:** Claude does not resume the explicit ID, Codex recovery chooses an unexpected session, or tool records break after path rewrite.

### 4. Cross-host recovery/adoption

From a second host with provider account credentials but without the source host's `~/.agentbox`:

- adopt Daytona, Vercel, E2B, Hetzner, and DigitalOcean sandboxes;
- verify project linkage, Git host actions, checkpoint discovery, relay, and agent resume;
- repeat after deliberately copying only documented portable state.

**Falsifier:** current source assumptions about VPS host-key dependence or cloud adoption loss are wrong.

### 5. Failure classification

For a known-running box, independently induce:

- provider API outage;
- invalid/expired provider credential;
- SSH firewall drift;
- local relay death;
- stale snapshot manifest;
- deleted provider runtime.

Compare `probeState`, Hub API, provider console, and source heartbeat.

**Falsifier:** a single AgentBox `missing` can safely distinguish deletion from temporary unreachability.

## Confidence and open uncertainty

| Finding | Confidence | Reason |
|---|---|---|
| Optional placement adapter is the correct BleedingADE role | High | Follows directly from AgentBox's operational provider boundary and absence of federated semantic ownership. |
| Public API coverage and omissions | High | Verified against API docs, route tree, and Hub backend interface at pinned revision. |
| Docker/remote-Docker pause and checkpoint semantics | High | Direct source plus Docker's documented freezer/commit behavior. |
| Hetzner/DigitalOcean host-key and power lifecycle constraints | High | Explicit backend source and failure messages. |
| Daytona class split and AgentBox cold checkpoint behavior | High for AgentBox; medium for upstream edge behavior | Direct AgentBox source; upstream VM snapshot/fork APIs are evolving. |
| Vercel filesystem persistence and process discontinuity | High | Current provider docs and AgentBox backend both separate filesystem persistence from compute sessions. |
| E2B exact RAM/process/PTY continuity through pause/snapshot | Medium-low | APIs promise pause/connect and “filesystem and state,” but reviewed docs do not define enough kernel/process detail for a product guarantee. |
| Arbitrary provider-plugin semantics | Unknown by design | Plugin methods are optional and trusted; each plugin needs declared capabilities and probes. |
| Provider-session resume robustness across versions/accounts | Medium | Source paths are clear, but provider CLIs and remote session semantics can change independently. |

## Source index

### AgentBox pinned source

- [Pinned commit](https://github.com/madarco/agentbox/commit/1de12d36feac92ed5108a8c9fb3606d47d13d3ac)
- [CLI package/version](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/package.json)
- [CLI changelog](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/CHANGELOG.md)
- [Provider contract](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/core/src/provider.ts)
- [Box record](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/core/src/box-record.ts)
- [Host state persistence](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-core/src/state.ts)
- [Workspace and Git sync](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/web/content/docs/sync-and-git.mdx)
- [Checkpoints and pausing](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/web/content/docs/checkpoints-and-pausing.mdx)
- [Docker provider](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-docker/src/docker-provider.ts)
- [Docker checkpoint implementation](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-docker/src/checkpoint.ts)
- [Remote Docker backend](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-remote-docker/src/backend.ts)
- [Generic cloud provider/checkpoint implementation](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-cloud/src/cloud-provider.ts)
- [Cloud checkpoint manifests](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-cloud/src/checkpoint.ts)
- [Daytona backend](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-daytona/src/backend.ts)
- [Daytona checkpoint adapter](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-daytona/src/checkpoint.ts)
- [Vercel backend](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-vercel/src/backend.ts)
- [E2B backend](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-e2b/src/backend.ts)
- [Hetzner backend](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-hetzner/src/backend.ts)
- [DigitalOcean backend](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/sandbox-digitalocean/src/backend.ts)
- [Provider registry and maintained built-ins](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/provider/loaders.ts)
- [Provider SDK package](https://github.com/madarco/agentbox/tree/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/packages/provider-sdk)
- [Public REST API](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/web/content/docs/api.mdx)
- [Hub backend contract](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/hub/lib/boxes/backend-types.ts)
- [Hub deployment/topologies](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/hub/README.md)
- [Host relay/security model](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/docs/host-relay.md)
- [`recover` and adoption](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/commands/recover.ts)
- [Agent-session restart restoration](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/agent-sessions.ts)
- [Claude session teleport](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/session-teleport/claude.ts)
- [Codex session teleport](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/session-teleport/codex.ts)
- [Checkpoint CLI](https://github.com/madarco/agentbox/blob/1de12d36feac92ed5108a8c9fb3606d47d13d3ac/apps/cli/src/commands/checkpoint.ts)

### Provider primary documentation

- Docker: [`docker container pause`](https://docs.docker.com/reference/cli/docker/container/pause/), [`docker container commit`](https://docs.docker.com/reference/cli/docker/container/commit/)
- Daytona: [Sandboxes/lifecycle](https://www.daytona.io/docs/sandboxes), [Persistence](https://www.daytona.io/docs/en/persistence/), [Snapshots](https://www.daytona.io/docs/snapshots/)
- Vercel: [Sandbox](https://vercel.com/docs/sandbox), [Snapshots](https://vercel.com/docs/vercel-sandbox/concepts/snapshots), [Duration and persistence](https://vercel.com/kb/guide/vercel-sandbox-duration-and-persistence)
- E2B: [JavaScript sandbox reference](https://e2b.dev/docs/sdk-reference/js-sdk/v2.10.5/sandbox), [Python sandbox reference with pause/snapshot](https://e2b.dev/docs/sdk-reference/python-sdk/v2.15.2/sandbox_async)

## Validation and limitations

Research validation performed:

- verified AgentBox `main` still resolved to pinned commit `1de12d36feac92ed5108a8c9fb3606d47d13d3ac` on 2026-08-23;
- traced each conclusion to current source, not only README marketing;
- cross-checked provider lifecycle/snapshot claims against official provider documentation;
- distinguished shipped, experimental, internal, planned, absent, and unknown behavior;
- checked the public API route/documentation against the Hub backend interface;
- reviewed all maintained built-in provider loaders: Docker, remote Docker, Daytona, Vercel, E2B, Hetzner, and DigitalOcean;
- included negative evidence and provider-specific contradictions.

Repository clone/build tests were not run. A shallow HTTPS clone from the execution environment failed with `Could not resolve host: github.com`. Research continued through authenticated GitHub source access at the immutable commit. No live Docker or paid cloud-provider accounts were available, so runtime-specific ambiguities are isolated in the boundary probes above rather than presented as guarantees.
