# Lima, Incus, and Locki as a local Placement foundation

**Issue:** [Research Lima/Incus and Locki as a local Placement foundation](https://github.com/BleedingDev/bleeding-ade/issues/26)  
**Research date:** 2026-08-23  
**Decision status:** resolved  
**Follow-up:** [Prototype the direct Lima/Incus Placement boundary on Apple Silicon and Linux KVM](https://github.com/BleedingDev/bleeding-ade/issues/27)

## Decision

**Do not adapt Locki itself as a BleedingADE Placement provider. Preserve its architecture as evidence, and defer a direct Lima/Incus adapter until the cross-platform conformance prototype passes.**

The useful result is narrower than either “adopt Locki” or “reject Lima”:

1. **Jan Pokorný's maturity hypothesis is substantially verified at the substrate level.** Lima is a mature local Linux-VM foundation with supported VM drivers, structured lifecycle surfaces, official container and Kubernetes templates, automatic networking/port forwarding, and adoption by larger developer-container products. It can run complex applications because it provides a real Linux guest with its own kernel, systemd, kernel modules, container runtime, networking, and persistent virtual disk. That does not make its mounts transparent, its snapshots portable, or its instance ID a BleedingADE Placement contract.
2. **One shared Lima VM plus one Incus container per Placement is a credible density pattern, not a hostile-sibling security boundary.** The VM isolates the physical Host kernel. Incus containers inside it share the Lima kernel. Locki weakens that sibling boundary further by making every container privileged and sharing writable credentials, home, package caches, BuildKit, registry state, and bridge key.
3. **Locki is not a safe external adapter seam.** It is a useful beta developer tool whose selected commands expose JSON, but it has no versioned management API, complete lifecycle event contract, durable placement identity, portable Checkpoint contract, copy/relocate/takeover/debug-fork semantics, or PTY reattachment contract. Its private Python services, XDG files, eight-character Worktree IDs, shared home, and Claude transcript conventions must not become BleedingADE dependencies.
4. **The correct future seam is direct composition:** Lima's supported exact-version structured CLI/events for the VM boundary plus Incus's documented REST API, API-extension negotiation, asynchronous operations, notifications, and ETags for the sandbox boundary.
5. **AgentBox remains the approved first optional Placement adapter.** Lima/Incus is not automatically “adapter number two.” It becomes the next local Host-boundary candidate only if the precise prototype proves mount correctness, recovery, nested-workload support, hostile-sibling defaults, and useful density on Apple Silicon VZ/virtiofs and Linux KVM/QEMU.

No BleedingADE authority changes. A Lima instance, Incus project, Incus instance, Locki sandbox ID, Worktree path, PID, PTY, snapshot, and provider-session file are operational locators or artifacts. BleedingADE continues to own Server, Project, Chat, Execution Segment, Worktree, Checkpoint, Ownership Epoch, semantic history, fencing, relocate/takeover/debug-fork transactions, and user-visible identity.

## Evidence vocabulary

Every conclusion below is labelled by origin:

| Label | Meaning |
|---|---|
| **Measured** | Executed on the available research Host and retained as evidence. |
| **Source-derived** | Directly established from pinned official code, documentation, tests, or contracts. |
| **Inferred** | A stated consequence of source-derived topology or lifecycle, not directly executed here. |
| **Unverified — Host/platform limitation** | The requested executable boundary could not run on the available Host and remains a required conformance probe. |

A source test that prints a duration is not a measurement from this research Host. A command or method named `pause`, `snapshot`, `copy`, `resume`, or `restore` is not by itself evidence for process, RAM, socket, PTY, Provider Session, semantic-history, or ownership continuity.

## Pinned source baseline

| Component | Immutable research revision | Relevant status/surface |
|---|---|---|
| Lima | [`lima-vm/lima@183a60d721bf9cd969f366301bf90710b8c6a28e`](https://github.com/lima-vm/lima/commit/183a60d721bf9cd969f366301bf90710b8c6a28e) | Current source inspected for VM drivers, mounts, lifecycle events, networking, templates, and experimental snapshots. |
| Incus | [`lxc/incus@a1ece14328058ba695ca0c7d4a83037bbf57f3e0`](https://github.com/lxc/incus/commit/a1ece14328058ba695ca0c7d4a83037bbf57f3e0) | Current v7 source/docs inspected for REST/OpenAPI, API extensions, security, projects, lifecycle, snapshots, copy/move, and requirements. |
| Locki | [`JanPokorny/locki@e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b`](https://github.com/JanPokorny/locki/commit/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b) | `0.0.27`, declared Beta; Python 3.11–3.14; macOS/Linux; x86_64/aarch64. |
| AgentBox comparison | [`madarco/agentbox@1de12d36feac92ed5108a8c9fb3606d47d13d3ac`](https://github.com/madarco/agentbox/commit/1de12d36feac92ed5108a8c9fb3606d47d13d3ac) | Prior approved first-adapter research: [full evidence](https://github.com/BleedingDev/bleeding-ade/blob/main/docs/research/21-agentbox-execution-location-handover.md). |
| Docker comparison | [`docker/docs@a3fd0352216480db9becce4878e8e79caae46a45`](https://github.com/docker/docs/commit/a3fd0352216480db9becce4878e8e79caae46a45) | Bind mounts, user namespaces, and rootless execution. |
| Podman comparison | [`podman-container-tools/podman@8efac905d6a1a6eef672a714efb1c939ea1d597c`](https://github.com/podman-container-tools/podman/commit/8efac905d6a1a6eef672a714efb1c939ea1d597c) | Rootless/daemonless OCI lifecycle, REST API, VM use on non-Linux Hosts, CRIU checkpoint/restore. |
| gVisor comparison | [`google/gvisor@3c5eee17dc45659fb86843531074f38e78e0cc35`](https://github.com/google/gvisor/commit/3c5eee17dc45659fb86843531074f38e78e0cc35) | OCI `runsc` userspace application-kernel boundary; not a VM. |

Locki installs the Fedora Incus package from the guest distribution rather than pinning an Incus release or API-extension set. That is acceptable for an interactive beta tool, but not sufficient Capability Evidence for a BleedingADE adapter.

## Executable probe record

**Measured:** the available runner was Debian 13, Linux 6.18.35 x86_64, five CPUs, 6,219,544 KiB RAM, cgroup v2, ext4, and `supervisord` as PID 1. It had no `/dev/kvm`, `/dev/vhost-vsock`, `/dev/net/tun`, Lima/QEMU, Incus/LXC, Docker/Podman, BuildKit, k3s, or kubectl. Systemd was unavailable, mount namespaces were denied, and `github.com` DNS resolution failed. An unprivileged user namespace did work.

The exact output is retained in [the Host capability probe record](https://github.com/BleedingDev/bleeding-ade/blob/main/docs/research/evidence/26-host-capability-probe.txt).

**Unverified — Host/platform limitation:** cold Lima startup, warm Incus creation, idle/restart behavior, memory and disk overhead, cross-VM mount correctness, permissions, symlinks, file watching, concurrent Host/guest writes, colliding forwarded ports, nested Compose/BuildKit, k3s/Kubernetes, Host reboot, VM recreation, stale identity, partial cleanup, and hostile-sibling attacks could not execute here. No native-filesystem substitute is presented as evidence for those boundaries.

## Lima maturity hypothesis

### What is verified

**Source-derived:** Lima describes itself as a Linux virtual-machine system with automatic file sharing and port forwarding and lists production developer tools built on it in its [pinned README](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/README.md). Its current [VM-driver selection](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/website/content/en/docs/config/vmtype/_index.md) selects VZ on supported modern macOS, QEMU on Linux, and the WSL2 path on Windows; VM type is fixed at instance creation.

**Source-derived:** the official [Kubernetes template](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/templates/k8s.yaml) provisions a full kubeadm/containerd/systemd stack, required kernel modules and sysctls, automatic Host access to the API server, and an optional multi-node Lima network. The default templates similarly support containerd/Docker-style workloads. Complex applications work because Lima provides a complete Linux guest, not because Lima implements Kubernetes semantics itself.

**Verdict:** Jan Pokorný's claim that Lima is the most mature local foundation among the candidates is credible for “give a developer Host a durable, automated Linux VM capable of container and Kubernetes workloads.” It is not evidence that one Lima configuration behaves identically across macOS VZ, Linux QEMU/KVM, and Windows WSL2, nor that Lima alone supplies a provider-neutral Placement API.

### What remains materially conditional

**Source-derived:** Lima's [mount documentation](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/website/content/en/docs/config/mount.md) documents driver-specific behavior and caveats:

- QEMU defaults to 9p while VZ defaults to virtiofs;
- reverse-SSHFS depends on the SSH transport and has a different failure/security model;
- 9p security modes have ownership/symlink/cache trade-offs and documented kernel compatibility history;
- Linux virtiofs and WSL2 mount paths remain platform-sensitive;
- cross-boundary inotify support is experimental, and some nested events and Host-side removals are not reliably represented.

Those caveats affect T3/BleedingADE directly: a successful build is insufficient if the Server's watcher misses nested creates, renames, or deletions.

**Source-derived:** Lima has useful structured lifecycle observation through [`limactl watch --json`](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/cmd/limactl/watch.go), including instance-state and port-forward events, and structured list output. It does not expose a versioned public management REST API equivalent to Incus. Its Go store/driver/host-agent packages are usable source, but an external adapter depending on them would inherit internal compatibility risk.

**Source-derived:** Lima's [`snapshot` command](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/cmd/limactl/snapshot.go) is explicitly experimental. The QEMU implementation uses [`savevm`/`loadvm` for a running VM and qcow2 snapshots for a stopped VM](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/pkg/driver/qemu/qemu.go). It is QEMU-specific and does not include Host-mounted Worktrees or the Host-mounted Locki home. It must not be advertised as a portable BleedingADE Checkpoint.

## Locki topology as implemented

The diagram below is implementation topology, not a proposed BleedingADE authority model.

```mermaid
flowchart TB
  subgraph H[Physical Host — macOS or Linux]
    CLI[Locki CLI]
    D[Detached Locki Python daemon\nPID/port/version + idle sweep]
    REPO[Original repositories\nreal .git databases]
    WT[Host-visible Git Worktrees\n~/.local/share/locki/worktrees]
    META[Trusted Worktree metadata\n~/.local/share/locki/worktrees-meta]
    HOME[One shared sandbox home\n~/.local/share/locki/home\ncredentials + Provider Sessions]
    STATE[Locki/Lima XDG state\nkeys, logs, cleanup records, VM disk]
  end

  subgraph VM[One shared Lima VM named locki — Fedora]
    INCUS[Incus daemon + default project\nbtrfs storage pool + incusbr0]
    BK[Shared BuildKit daemon/socket/cache]
    REG[Shared nginx registry/GitHub/k3s caches\nVM-local CA/key]
    BEES[bees btrfs deduplication]
    PKG[Shared writable package caches]

    subgraph C1[Privileged Incus instance — sandbox A]
      P1[Agent/processes/PTY/systemd]
      R1[Container rootfs\nnested Docker/images/k3s state]
      M1[Mounted Worktree A]
      H1[/root = shared Host home]
    end

    subgraph C2[Privileged Incus instance — sandbox B]
      P2[Agent/processes/PTY/systemd]
      R2[Container rootfs\nnested Docker/images/k3s state]
      M2[Mounted Worktree B]
      H2[/root = same shared Host home]
    end
  end

  CLI --> D
  CLI --> VM
  D --> INCUS
  STATE --> VM
  WT -->|Lima writable mount| VM
  HOME -->|Lima writable mount| VM
  REPO --> WT
  META --> WT
  INCUS --> C1
  INCUS --> C2
  WT --> M1
  WT --> M2
  HOME --> H1
  HOME --> H2
  BK --> C1
  BK --> C2
  REG --> C1
  REG --> C2
  PKG --> C1
  PKG --> C2
```

### Physical Host

**Source-derived:** Locki's [path model](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/paths.py), [configuration](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/config.py), [Worktree service](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/worktree.py), and [home service](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/home.py) keep these on the physical Host:

- original repositories and their real `.git` databases;
- one Git Worktree per sandbox, visible and mutable from the Host;
- a trusted copy of each Worktree's `.git` pointer and original-repository metadata;
- one shared home used by every sandbox;
- Locki config, logs, SSH bridge keys, daemon runtime records, cleanup JSON, and Lima instance state/disk.

The Worktree contains a `.git` pointer, but the referenced original Git database is not mounted wholesale into the sandbox. Git/GitHub operations are intended to cross a constrained bridge instead.

### Shared Lima VM

**Source-derived:** Locki's [VM service](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/vm.py) creates one instance named `locki`, assigns all visible Host CPUs and memory, requests a 200 GiB sparse disk, mounts Worktrees and the shared home writable, and uses Fedora. Interactive operations may auto-start the VM; daemon-safe Incus calls deliberately do not.

**Source-derived:** [`vm-setup.sh`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/vm-setup.sh) creates the Incus/btrfs substrate, BuildKit, caching registry/proxies, certificate material, and bees deduplication. These are VM-local acceleration state. Deleting/recreating the Lima VM preserves Host Worktrees/home/config but discards these services, their stores, all Incus instances, Incus images, and container root filesystems.

### Incus instances

**Source-derived:** Locki's [container service](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/container.py) uses the eight-character Worktree ID as the Incus instance name, mounts only that Worktree at the same absolute path, attaches shared cache devices, and starts or provisions the container.

**Source-derived:** [`container-setup.sh`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/container-setup.sh) enables nested development workloads. Each instance is privileged, enables nesting, clears capability drops, receives broad `/proc` and `/sys` access, and may receive KVM-related devices. Each has its own root filesystem and process namespace, but all share the Lima kernel and the shared writable acceleration/secret surfaces.

### Why this topology exists

**Source-derived/inferred:** one VM amortizes hypervisor startup and memory; Incus provides cheap per-sandbox root filesystems, process namespaces, networking, and lifecycle; shared BuildKit/registry/package caches plus btrfs dedup reduce repeated downloads and disk use; Host Worktrees preserve editor visibility and survive VM loss. These are strong developer-experience choices inside one trusted personal environment. They are not safe defaults for mutually hostile agents.

## Locki identity, lifecycle, and stale-state behavior

**Source-derived:** [`new.py`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/new.py) and the Worktree service create:

- random eight-character Locki ID;
- Host Worktree path `<repository>-locki-<id>`;
- branch `<stem>#locki-<id>`;
- Incus instance named by the same ID.

This correlation is convenient local convention. It is not a globally stable Placement, Chat, Execution Segment, Worktree, Checkpoint, or Ownership Epoch ID.

**Source-derived:** the detached [daemon](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/daemon.py) stores PID/port/version, restarts when the installed Locki version changes, sweeps approximately every minute, stops containers after the configured idle interval, and later stops the VM when no containers run. Malformed or missing activity JSON is treated as empty state.

**Source-derived:** an active foreground Incus exec operation prevents idle stop. **Inferred:** a useful detached workload that no longer has an active Incus operation can be judged idle and stopped, killing its processes and PTYs.

**Source-derived:** while the VM is running, the daemon maps containers to Worktrees through mounted sources and deletes orphan containers whose Worktree is missing. It does not constitute a complete durable reconciliation protocol: a surviving Worktree with a missing container, stale trusted metadata, partial Git cleanup, a lost VM, or an unreachable Incus daemon has no federated identity or ownership transaction.

**Source-derived:** [`remove.py`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/remove.py) deletes the container, scoped cache, Worktree metadata, Git Worktree, and optionally its branch. [`vm.py`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/vm.py) can stop or delete the shared VM; VM deletion intentionally leaves Host Worktrees, home, and settings behind.

## Git/GitHub bridge and forced-command security

**Source-derived:** Locki's [bridge service](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/bridge.py), [forced-command implementation](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/internal.py), and sandbox [AGENTS guidance](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/AGENTS.md) provide a meaningful defense-in-depth layer:

- allowlisted command grammar rather than an arbitrary Host shell;
- canonical Worktree/cwd checks;
- rejection of symlinked `.git` pointers;
- restoration from trusted metadata using no-follow behavior;
- command arguments tied to the Worktree ID, expected repository/remotes, branches, and stash refs;
- no SCP, agent forwarding, or X11;
- denied-command logging.

It is not a security boundary:

- every sandbox can read the same bridge client private key from the shared home;
- every sandbox has direct network access and shared credentials/configuration;
- every sandbox is a privileged container inside the shared VM;
- the [Claude branch guard](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/claude-branch-guard.sh) deliberately fails open when it cannot establish state;
- compromise of the shared VM kernel bypasses all sibling-container assumptions.

The bridge reduces accidental and unsophisticated direct mutation of the original Host Git database. It cannot make shared secrets, shared caches, the VM kernel, or writable Host Worktrees safe against a hostile sibling.

## Networking and shared acceleration state

**Source-derived:** [`port_forward.py`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/port_forward.py) adds an Incus proxy device that binds a selected port on `0.0.0.0` and forwards to a container's loopback. Two sandboxes can use the same internal service name and port, but the same published port collides in the shared VM/Host forwarding path. Random allocation performs a bind/check followed by a separate device-add operation, leaving a time-of-check/time-of-use race.

**Source-derived:** shared writable BuildKit, registry/proxy caches, package caches, image tags, and cache keys improve warm performance. **Inferred:** they also permit accidental or malicious cross-sandbox cache poisoning, tag replacement, secret-bearing build-cache leakage, and noisy-neighbour effects unless namespaced and access-controlled.

**Source-derived:** Locki caches the k3s installer and release assets and configures nested Docker/BuildKit support. This establishes implementation intent, not a measured guarantee that a systemd/networked k3s cluster survives every Locki lifecycle transition.

## Complete authority matrix

| Concern | Physical location in Locki | Operational owner in Locki | Persistence/continuity | Required BleedingADE authority rule |
|---|---|---|---|---|
| BleedingADE Server identity and event store | Not represented | None | None | BleedingADE Server owns it; never derive it from a VM/container. |
| Project, Chat, Execution Segment, Branch, Ownership Epoch | Not represented semantically | None | Locki labels/branch names only | BleedingADE owns stable IDs, lease epoch, fencing, and transition history. |
| Locki sandbox ID | Host metadata, Worktree name, Incus instance name | Locki | Survives as Host naming even if instance is lost | Store only as an adapter-scoped opaque locator component. |
| Lima VM identity/config/disk | Host Lima state | Lima/Locki | Disk normally survives stop/reboot; not VM deletion | Locator component and failure domain, not Placement identity. |
| Incus project/instance identity | Incus database inside Lima VM | Incus | Survives stop; lost with VM recreation unless separately backed up | Opaque provider runtime ID scoped to one Lima/Incus installation. |
| Original repository and real `.git` | Physical Host | Host Git/user | Survives container/VM deletion | BleedingADE Project/Checkout authority; never assume Incus snapshots include it. |
| Host-visible Worktree bytes | Physical Host, mounted through Lima into one instance | Shared mutation by Host and sandbox | Survive container and VM recreation; no process continuity | BleedingADE Worktree identity remains independent; record exact mount path and integrity. |
| Worktree `.git` pointer and trusted copy | Worktree plus Host metadata | Locki bridge/Host Git | Survives VM loss; may become stale after partial cleanup | Validate as workspace metadata, never as global identity. |
| Container root filesystem | Incus storage pool in Lima VM | Incus | Survives stop/restart; lost on instance/VM deletion | Provider-owned ephemeral/persistent filesystem artifact. |
| Processes, RAM, sockets, service state | Incus instance and shared Lima kernel | Runtime/Incus | Survive client disconnect; die on container stop, VM stop, recreation, or ordinary Host reboot | Effect-specific continuity only; never infer from filesystem persistence. |
| PTY/terminal attachment | Incus exec process plus Host client | Locki/Incus transport | Current attachment only; no Locki reattach contract | Operational attachment, never Chat/Execution Segment identity. |
| Shared sandbox home | Physical Host mounted as `/root` everywhere | Locki/user | Survives container/VM recreation | Must not be shared by default in BleedingADE hostile-sibling profile. |
| Infrastructure and agent credentials | Shared Host home/config copied by setup | User/Locki | Files survive; revocation/backend state varies | Node-scoped secret references; never implicit Placement state. |
| Provider Session files | Shared Host home plus provider backend | Provider CLI/backend | Files survive container/VM recreation; exact resume is provider-specific | BleedingADE owns semantic transcript; session resume remains optional Capability Evidence. |
| Global package caches | Lima VM shared storage | Locki | Survive container deletion; lost with VM recreation | Declare shared writable trust domain and poisoning risk. |
| Scoped dependency caches | Lima VM per-sandbox paths | Locki | Removed with sandbox cleanup; lost with VM | Optional acceleration, not Worktree or Checkpoint. |
| BuildKit daemon/socket/cache | Lima VM | Locki/BuildKit | Survives container restart; lost with VM recreation | Never shared across untrusted Placements by default. |
| Registry/proxy cache, tags, CA/key | Lima VM | Locki/nginx/registry | Survives container restart; lost with VM recreation | Namespace and pin; never use mutable tag as checkpoint identity. |
| Incus base images | Incus pool in Lima VM | Incus | Shared/deduplicated; lost with VM recreation | Provider artifact identified by immutable digest and version evidence. |
| Nested Docker images/volumes | Container root filesystem unless explicitly externalized | Nested Docker | Survive container stop; lost on container recreation | Included/excluded state must be explicit. |
| Kubernetes/k3s state | Container root filesystem and processes | k3s/systemd | Disk may survive stop; control-plane processes die; lost on recreation | Workload state, not Placement identity; must be quiesced/probed. |
| Host bridge key and server | Key in shared Host home; server on Host | Locki | Key survives; server/connection restarts | Privileged Host integration; must be per-Placement or trust-domain scoped. |
| Incus administrative socket/API | Lima VM | Incus/Locki | Survives daemon restart with VM disk | Root-equivalent substrate control; only owning BleedingADE Server may access it. |
| Lima snapshot | QEMU VM state/disk, where supported | Lima/QEMU | Driver-local; external Host mounts excluded | Optional provider acceleration artifact, not canonical Checkpoint. |
| Incus snapshot/export/copy | Instance-owned volume/config, optional stateful VM state | Incus | External Worktree/custom volumes excluded; container stateful restore unreliable | Attach truthful inclusion manifest; never imply portable process/PTY continuity. |
| Canonical BleedingADE Checkpoint | Not represented | None | None | BleedingADE owns name, lineage, portable manifest, verification, and restore semantics. |

## Lifecycle matrix

The matrix describes Locki's implemented topology. “Filesystem survives” never means the process, PTY, semantic Chat, or ownership lease survives.

| Operation | Host Worktree and real Git | Shared home/credentials/Provider Sessions | Incus rootfs and VM-local caches | Processes/RAM/sockets/PTY | Identity/recovery result |
|---|---|---|---|---|---|
| Client disconnect | Unchanged | Unchanged | Unchanged | Foreground attachment ends; underlying detached work may continue | No reconnect/reattach contract for the old PTY. Locki ID remains local convention. |
| Idle timeout | Unchanged | Unchanged | Container disk persists; later VM disk persists after clean stop | Container stop kills all sandbox processes and PTYs; later VM stop kills remaining guest processes | Activity is operational, not semantic. Detached work may be stopped if it no longer owns an active Incus operation. |
| Container stop | Unchanged | Unchanged | Rootfs and shared VM caches persist | All container processes, RAM, sockets, PTYs lost | Later start is restart, not reconnect or restore. |
| Container restart/start | Unchanged | Re-mounted | Existing rootfs and VM caches reused | New process/service instances; old PTY gone | Same Incus locator, new runtime epoch; BleedingADE must emit truthful lifecycle effects. |
| Lima VM stop | Unchanged | Host copy survives | VM disk, Incus DB, instances, images, BuildKit/registry/cache bytes persist | All VM/container processes and PTYs lost | Next start can rediscover instances if disk is intact; not process continuity. |
| Physical Host reboot | Host files and Lima disk normally persist | Host files persist | VM disk normally persists | Host, VM, and container processes/PTYs lost | **Inferred/unverified:** on-demand restart/reconciliation is required; no measured reboot evidence here. |
| Lima VM recreation/delete | Worktrees and real Git survive | Shared Host home/settings survive | All VM-local Incus instances/rootfs/images, BuildKit, registry, CA/key, package caches, nested images, and k3s state are lost | Lost | Locki leaves Host state that can look valid while runtime locators are stale. Recreate is new execution, not resume. |
| Incus container recreation | Worktree survives | Shared home survives | Old rootfs, nested Docker images/volumes, and k3s state lost; global VM caches survive; scoped cache is removed on deletion | Lost | Same Worktree may be attached to a new instance, but it is a new runtime/Execution Segment. |
| Locki upgrade | Worktrees/home normally survive | Survive | Existing VM/containers may carry old provisioning; Fedora/Incus packages and setup scripts can drift | Daemon restarts when Locki version changes; running guest work may continue until another lifecycle action | **Inferred:** exact-version reconciliation is incomplete because Locki and Incus are not jointly pinned as an adapter protocol. |
| Lima snapshot | External Host Worktree/home excluded | Excluded as Host mounts | QEMU VM disk/RAM may be captured by experimental `savevm`; driver-dependent | Guest process state may be restored only within supported QEMU conditions; Host attachment is not portable | Not a complete Locki or BleedingADE Checkpoint. |
| Incus snapshot | External Worktree excluded; real Git excluded | Shared Host home excluded | Instance-owned rootfs captured; custom volumes separate | VM stateful snapshots supported; container stateful behavior constrained by CRIU | Snapshot artifact remains Incus-local unless exported/copied; no semantic identity. |
| Incus copy/move | External Worktree remains a separate path/binding | External shared home remains separate | Copies/moves instance-owned volume/config; source stop usually required for containers | Ordinary coding containers lose process/PTY continuity | New provider runtime ID; BleedingADE must transact relocate/fork separately. |
| `locki rm` deletion | Worktree and metadata removed; real repo remains; branch optionally deleted | Shared home remains for all other sandboxes | Container and scoped cache removed; global VM state remains | Lost | Destructive local cleanup, not semantic Chat deletion. Partial failure can leave stale Git/metadata/runtime pieces. |
| `locki vm delete` | Worktrees and repos intentionally remain | Shared home/settings intentionally remain | Entire VM-local substrate lost | Lost | All Incus locators stale. Host-visible Worktrees are recoverable bytes, not recovered execution. |

## Incus capability and security boundary

### Stable integration surface

**Source-derived:** Incus's [REST API contract](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/rest-api.md) is the strongest integration surface in this study:

- all Incus clients communicate through REST over a local Unix socket or TLS;
- major API versions change only for backward-incompatible changes;
- additive features are discoverable through `api_extensions` documented in the [extension registry](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/api-extensions.md);
- stable numeric statuses accompany text statuses;
- long operations return IDs and support waiting/notification;
- WebSocket notifications avoid blind polling;
- ETags/`If-Match` protect concurrent updates;
- a generated OpenAPI specification describes endpoints.

The supported CLI also provides JSON/YAML output—see [`incus list --format`](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/cmd/incus/list.go)—and `incus query`/monitoring. A direct adapter should use REST first and structured CLI only as a narrow fallback.

The local Incus administrative socket is effectively root-equivalent substrate control. It must never be mounted into an agent sandbox or exposed as an agent tool.

### Sibling isolation

**Source-derived:** Incus defaults to unprivileged containers and supports isolated ID maps, resource limits, filtered networking, confined projects, and project-specific images/profiles/networks/storage. Its [project model](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/explanation/projects.md) can separate names and resources and restrict users to projects.

**Source-derived:** Incus's [security policy](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/SECURITY.md) explicitly does not consider privileged containers root-safe. Its [FAQ](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/faq.md) warns that privileged containers can affect the entire Host—for Locki, the “Host” at this layer is the shared Lima VM. Locki therefore gains a physical Host kernel boundary from Lima but does not provide a credible hostile sibling boundary inside that VM.

A direct BleedingADE profile must default to unprivileged Incus containers, isolated maps where available, resource limits, network filtering, and no shared secret-bearing writable surfaces. A dedicated Lima VM per Placement or per high-risk trust domain remains the stronger option when shared-kernel compromise is in scope.

### Nested Docker/BuildKit/Kubernetes

**Source-derived:** the Incus FAQ documents Docker-in-Incus with `security.nesting=true`; required kernel modules must be loaded by the Incus Host because a container cannot load them. This proves feasibility, not the necessity of privileged containers. Locki's broad privileged profile is an implementation convenience that a BleedingADE prototype must attempt to avoid.

**Source-derived:** current Incus source requirements include a modern Linux kernel, cgroups, namespaces, seccomp, LXC, and optional CRIU/QEMU capabilities in [the pinned requirements](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/requirements.md). Capability Evidence must pin both the Lima guest image/kernel and Incus API extension set; “Incus installed” is insufficient.

### Snapshot, copy, and migration truth

**Source-derived:** [Incus backup/snapshot documentation](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/howto/instances_backup.md) distinguishes instance snapshots, exports, and copies. Custom storage volumes are not part of an instance backup and must be handled separately. In the Locki topology, the Host-mounted Worktree and shared home are external to the instance snapshot.

**Source-derived:** Incus supports stateful snapshots for VMs. Container stateful snapshots are not fully supported because of CRIU limitations. Its [move documentation](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/howto/move_instances.md) says container live migration is reliable only for very basic non-systemd containers without a network device; real-world containers should normally stop, move, and start. A coding Placement running systemd, Docker, BuildKit, or k3s must assume process/RAM/socket/PTY loss.

## Integration-surface classification

Classification:

1. documented stable API;
2. supported structured CLI;
3. internal but usable surface with maintenance risk;
4. terminal/prose fallback;
5. unavailable.

| Component/surface | Class | Result for BleedingADE |
|---|---:|---|
| Incus REST/OpenAPI, API extensions, operations, notifications, ETags | **1** | Preferred sandbox-management seam. Pin version/extensions and validate postconditions. |
| Docker Engine API; Podman REST API | **1** | Strong direct-container control seams, but do not create a VM Host boundary on Linux. |
| AgentBox Hub `/api/v1` | **1, incomplete** | Approved first-adapter seam where covered; checkpoint/recovery/session gaps remain as documented in the AgentBox resolution. |
| Lima `list --format`, `watch --json`, lifecycle commands with exact-version exit/postcondition checks | **2** | Preferred current VM seam. There is no equivalent public management REST API. |
| Incus JSON/YAML CLI, `query`, monitor | **2** | Supported fallback or diagnostic path, not necessary when REST is available. |
| Locki `new --json`, `list --json`, VM status JSON, selected port/removal JSON | **2, partial** | Discovery only. It does not cover complete lifecycle, events, recovery, Checkpoint, copy, or identity. |
| Docker/Podman JSON-capable CLI | **2** | Usable fallback behind pinned adapters. |
| Lima Go store/driver/host-agent/event packages | **3** | Technically usable but internal compatibility risk; avoid until a public library contract exists. |
| Locki Python services and XDG metadata | **3** | Reject as adapter contract. Do not import or write directly. |
| AgentBox internal relay/state/checkpoint files | **3** | Already rejected as federation/adapter contract. |
| Locki interactive `exec`, VM stop/delete prose, shell orchestration | **4** | Human fallback only; no semantic event parsing. |
| Lima QEMU snapshot list/human output | **4, experimental** | Diagnostic/experimental only. |
| Locki lifecycle event stream, durable API version negotiation | **5** | Unavailable. |
| Locki portable Checkpoint/export/import/copy/relocate/takeover/debug fork | **5** | Unavailable as first-class semantics. |
| Locki PTY reconnect/reattach | **5** | Unavailable. |
| Reliable Incus stateful migration of systemd/networked coding containers | **5 for product claims** | Officially constrained by CRIU; require stop/copy/start semantics. |
| Portable live process/RAM/socket/PTY migration across Lima/Incus Hosts | **5** | Unavailable. |

## Eight-design comparison

| Design | Physical Host boundary | Sibling-Placement boundary | Persistence versus continuity | Control/authority consequences | Decision |
|---|---|---|---|---|---|
| 1. Ordinary BleedingADE Server directly on Host | None | Host user/process permissions only | Host files/processes may persist independently; no substrate abstraction | Lowest overhead and simplest default; Server directly owns local execution | Required baseline; independently useful without adapters. |
| 2. AgentBox local Docker | No separate Linux kernel on Linux | OCI namespaces/cgroups; same Host kernel | Docker pause can freeze same container; stop/recreate kills processes; Host Git/mount state has separate semantics | Approved first optional adapter; AgentBox IDs remain opaque locators | Keep approved first status. Not a hostile Host boundary. |
| 3. Direct Docker/Podman, optionally gVisor | Docker/Podman share Host kernel on Linux; non-Linux tools already add their own VM | Rootless/user namespaces improve confinement; gVisor adds a userspace application kernel but is not a VM | Bind mounts persist; ordinary stop/recreate loses process/PTY; CRIU/runtime support is conditional | Excellent APIs and low density cost; BleedingADE would own more lifecycle code than with AgentBox | Useful baseline/possible later adapter; not selected by this ticket. |
| 4. One Lima VM per Placement | Stronger VM kernel boundary from physical Host | Separate VM kernel per Placement | VM disk persists through stop; process continuity depends on VM pause/snapshot semantics; Host mounts remain external | Clearest isolation/failure domain and easiest cache/secret separation | Security profile for high-risk Placements; expensive at 10–50. |
| 5. One shared Lima VM + one Incus container per Placement | One VM boundary from physical Host | Shared Lima kernel; unprivileged Incus can improve sibling confinement | Host Worktree can persist while container/VM processes die; VM-local caches amortized | Best candidate density pattern if trust domain and shared state are explicit | Architecture candidate, conditional on prototype. |
| 6. Adapt Locki externally | Same topology as 5 | Locki uses privileged siblings and broad shared writable state | Worktrees/home survive; processes/PTYs do not survive idle/stop/recreation | Locki IDs/CLI/state are incomplete and unstable for BleedingADE semantics | **Reject adapter. Learn from architecture only.** |
| 7. Direct BleedingADE Lima/Incus adapter | Configurable shared or dedicated VM boundary | Configurable unprivileged Incus/dedicated VM profiles | Can report exact effects and treat Host Worktree separately from runtime/rootfs | Server keeps authority; Incus REST + Lima structured CLI are viable seams | **Deferred optional adapter pending prototype.** |
| 8. Full BleedingADE Server inside Lima VM or each Incus instance | Server lives behind that substrate boundary | Depends on VM/container profile | It owns its own event store, projects, runtime, credentials, and reconnect endpoints | This is an independently operational federated Server, not a subordinate Placement of the Host Server | Valid deployment contrast only; never dual authority. |

## Security and performance comparison

| Property | AgentBox local Docker | Direct Docker/Podman (+ optional gVisor) | Locki shared Lima/Incus | Remediated direct Lima/Incus candidate |
|---|---|---|---|---|
| Physical Host kernel isolation | No on Linux | No with runc/crun; gVisor reduces kernel surface but is not a VM | Yes, one shared Lima VM | Yes, shared or dedicated Lima VM |
| Sibling kernel isolation | Shared Host kernel | Shared Host kernel; gVisor sandbox grouping is runtime-dependent | Shared Lima kernel; Locki containers privileged | Shared Lima kernel with unprivileged isolated Incus, or separate VM per trust domain |
| Host filesystem exposure | AgentBox-specific Host Git/data mounts | Explicit bind mounts, writable by default per [Docker docs](https://github.com/docker/docs/blob/a3fd0352216480db9becce4878e8e79caae46a45/content/manuals/engine/storage/bind-mounts.md) | Assigned Worktree plus one shared Host home; VM compromise reaches mounted paths | Only assigned Worktree by default; exact rw/ro and mount semantics recorded |
| Rootless/user namespace | Provider-specific | Docker [userns-remap](https://github.com/docker/docs/blob/a3fd0352216480db9becce4878e8e79caae46a45/content/manuals/engine/security/userns-remap.md), [rootless mode](https://github.com/docker/docs/blob/a3fd0352216480db9becce4878e8e79caae46a45/content/manuals/engine/security/rootless/_index.md), and [rootless Podman](https://github.com/podman-container-tools/podman/blob/8efac905d6a1a6eef672a714efb1c939ea1d597c/README.md) improve confinement | Not used for sandbox boundary; containers privileged | Required default: unprivileged + isolated maps where supported |
| Secret/Provider Session scope | Host/AgentBox policy-specific | Adapter-defined | Same shared home visible to every sandbox | Per-Placement secret injection/reference; no shared writable session home by default |
| Build/cache scope | Docker engine and AgentBox provider-specific | Usually shared engine stores unless separated | Shared writable BuildKit, registry, package caches, image namespace | Per trust-domain namespace; no untrusted shared socket/tag/cache keys |
| Nested Compose/BuildKit | Natural Docker path | Natural; gVisor compatibility must be proved | Explicitly enabled; performance optimized | Must pass unprivileged profile or be declared unavailable |
| Nested k3s/Kubernetes | Not a Local Docker guarantee | Usually requires additional privileges/kernel features | Support intent via privileged nested environment and cached assets | Must be measured; no privileged-default claim |
| Checkpoint truth | Provider-specific AgentBox contract; local Docker image excludes mounts/volumes | Runtime/CRIU-specific; external mounts separate | No Locki contract; manual Lima/Incus artifacts exclude Host mounts | Effect-specific evidence and BleedingADE portable manifest remain separate |
| Warm density | Container-efficient | Container-efficient | One VM amortized across cheap containers and shared caches | Expected benefit, but unverified until 1/10/50 measurements |
| Failure blast radius | Host Docker engine/kernel | Host engine/kernel | Shared VM kernel, Incus daemon, BuildKit, registry, caches | Explicit isolation domain; dedicated VM option for higher risk |

[gVisor's pinned README](https://github.com/google/gvisor/blob/3c5eee17dc45659fb86843531074f38e78e0cc35/README.md) correctly states that ordinary containers are not a sandbox for hostile code and that gVisor adds an OCI-compatible userspace application kernel. It is a meaningful direct-container hardening option, but mounted data still remains deliberately exposed and gVisor compatibility/performance are workload-specific.

## Stress-case results

| Stress case | Result | Evidence class |
|---|---|---|
| Hostile agent attacks sibling Worktree | Direct mount exposes only its assigned Worktree, but shared home, bridge key, caches, BuildKit, registry, and privileged shared-VM kernel create multiple sibling attack paths | Source-derived/inferred; runtime exploit unverified |
| Mutation of shared home/credentials/Provider Sessions | Every container receives the same writable `/root`; copied Claude/Codex/Gemini/OpenCode/Copilot material can be read or changed | Source-derived from home/setup paths |
| BuildKit/cache/registry/image-tag poisoning | Shared writable daemon/socket/caches and mutable names create cross-sandbox integrity risk | Source-derived topology; attack unverified |
| Git bridge bypass | Original `.git` is not mounted and forced commands are constrained, but shared bridge key, direct network, fail-open guard, and VM compromise prevent treating it as a hard boundary | Source-derived/inferred |
| Host filesystem access | Assigned Worktree and shared home are intentionally writable Host mounts. Other Host paths depend on Lima mount/bridge configuration and VM compromise | Source-derived; escape unverified |
| Shared Lima-kernel compromise | Compromises every Incus sibling and all VM-visible mounts/state; VM still limits direct physical Host-kernel access | Source-derived boundary model |
| Identical internal service/cluster names | Namespaces allow many internal duplicates | Source-derived/inferred; unverified workload probe |
| Identical published Host port | Incus proxy/Lima forwarding collides; random allocation has a race | Source-derived |
| Identical cache key or image tag | Shared cache/tag namespace permits collision or poisoning | Inferred from source-derived sharing |
| BleedingADE Server crash while sandbox continues | Incus/guest process may continue independently; Locki idle daemon may later stop it | Inferred; requires prototype reconciliation and ownership fencing |
| Lima VM loss while Host Worktrees remain | Worktrees/home survive, all VM-local runtime and acceleration state is lost | Source-derived |
| Incus instance loss with stale Locki records | Host Worktree/metadata can remain while runtime is missing; current cleanup is partial and VM-dependent | Source-derived |
| Unsupported Host/architecture | Locki declares macOS/Linux x86_64/aarch64; current Lima paths differ materially across VZ, QEMU, and experimental WSL2 | Source-derived |

## Platform and scale limits

### Platform tuples are separate capabilities

A capability record must include `{Host OS/build, Host architecture, Host filesystem, Lima revision, VM driver, mount type, guest image digest/kernel, Incus version, api_extensions, adapter revision}`. Results from macOS arm64 VZ/virtiofs cannot be copied to Linux x86_64 QEMU/9p, Linux virtiofs, or Windows WSL2.

Locki currently declares macOS and Linux on x86_64/aarch64, not Windows. Linux acceleration depends on QEMU/KVM availability. Nested KVM, Docker, BuildKit, and Kubernetes additionally depend on Host/guest kernel features and devices.

### Approximate Placement counts

No numeric latency, RAM, or disk claim was measured on this Host.

| Scale | Source-derived expectation | Binding limits and decision |
|---:|---|---|
| **1 Placement** | A single shared or dedicated Lima VM can provide a useful physical Host boundary; Incus provides a complete Linux sandbox and persistent rootfs | Likely viable, but mount watchers, nested workloads, and recovery still need both target Hosts. Dedicated VM is acceptable for high-risk work. |
| **10 Placements** | A shared VM amortizes boot, kernel, images, BuildKit, registry, and caches; Incus containers are materially cheaper than ten VMs | Must add CPU/memory/PID/disk/network limits and isolate secret/cache namespaces. Locki currently assigns all Host CPU/RAM to one VM and does not establish per-Placement admission control. |
| **50 Placements** | Incus can represent many stopped/running instances and btrfs/dedup can reduce repeated disk data | A developer Host may become unusable from RAM/process/I/O/watcher/network/cache pressure. One VM per Placement is generally impractical. The shared-VM pattern is not shippable at this scale without measured admission, suspension, reconciliation, and p50/p95 evidence. |

Locki's E2E suite in [`test/e2e.sh`](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/test/e2e.sh) exercises cold/warm/hot paths and nested workloads, but prints timings rather than enforcing a performance contract. It is upstream test intent, not Capability Evidence from this research.

## Required Placement capability-contract additions

The provider-neutral authority model does **not** change. The Placement capability/evidence record needs additive fields so a future Lima/Incus adapter cannot collapse materially different boundaries into booleans.

### 1. Composite operational locator

Store an opaque provider locator such as:

```text
{
  serverId,
  adapterKind: "lima-incus",
  limaInstance,
  incusProject,
  incusInstance,
  worktreeBindingId,
  providerRuntimeGeneration
}
```

Only `serverId` is BleedingADE identity. The rest is adapter-scoped and replaceable. Locki's eight-character ID is not reused as a BleedingADE ID.

### 2. Boundary topology

Capability Evidence must state:

- physical Host kernel boundary: none / gVisor-like / VM;
- sibling kernel boundary: shared / dedicated;
- isolation-domain ID and declared trust level;
- VM scope: shared trust domain or dedicated Placement;
- shared failure domains: Lima VM, Incus daemon, network, storage, BuildKit, registry;
- privilege profile: unprivileged/privileged, ID-map mode, capabilities, devices, kernel modules.

### 3. Host filesystem exposure

For every mount, record:

- Host and guest path/binding identity;
- read-only/read-write;
- Lima mount implementation and driver;
- ownership/ID-map behavior;
- whether it is included in Incus/Lima snapshots, copy, export, or migration;
- symlink, mode, xattr, sparse-file, atomic-rename, fsync, and watcher evidence;
- concurrent Host/guest write behavior and integrity policy.

### 4. Shared writable-state inventory

Explicitly enumerate home, credentials, Provider Sessions, SSH/bridge keys, package caches, dependency caches, BuildKit sockets/cache, registry/tag namespace, base images, nested engine state, Kubernetes state, network/kernel, and Host mounts. Each item declares scope `{placement, trusted-domain, VM-global, Host-global}`, mutability, secret-bearing status, poisoning risk, and recreation behavior.

### 5. Effect-specific lifecycle

For disconnect, adapter idle, container stop/start/recreate, VM stop/start/recreate, Host reboot, adapter/Incus upgrade, snapshot, copy, move, and deletion, report independently:

- same provider runtime ID or new generation;
- filesystem/rootfs survival;
- external mount survival;
- process/RAM/socket/TCP/PTY survival;
- Provider Session file/backend survival;
- endpoint/port-forward survival;
- source-side effect;
- reconciliation confidence.

### 6. Checkpoint semantics

Every checkpoint-like artifact declares:

- included instance volumes and excluded custom/Host mounts;
- application quiesce behavior;
- process/RAM/device/socket/TCP/PTY capture;
- driver/API extension and portability scope;
- restore preconditions and target identity;
- whether restore creates a new runtime generation;
- source effect and retention;
- cryptographic manifest linking optional provider artifacts to the canonical BleedingADE Checkpoint.

### 7. Workload and networking capabilities

Record measured support for systemd, nested Docker/Podman, Compose, BuildKit, k3s/Kubernetes, KVM, and required privileged devices/modules. Port publication declares bind scope, allocator ownership, collision behavior, stale-forward cleanup, and whether identical internal names remain isolated.

### 8. Health, reconciliation, and evidence

Health must distinguish `running`, `stopped`, `unreachable`, `missing`, `stale`, `orphaned`, `drifted`, and `fenced`. No destructive cleanup may follow from `missing` alone.

Capability Evidence records exact versions/API extensions, Host tuple, probe revision, timestamp, raw evidence pointers, pass/fail/unknown, and expiry/reprobe conditions. `isolationDomainId`, VM name, Incus instance, PID, PTY, Worktree path, snapshot name, and Provider Session ID remain operational metadata, never semantic identity.

## Recommended default and trusted-domain profiles

A future direct adapter, if the prototype passes, should expose at least two truthful profiles:

### Default hostile-sibling profile

- unprivileged Incus container;
- isolated UID/GID map where supported;
- CPU, memory, PID, disk, and network limits;
- MAC/IP filtering;
- only assigned Worktree mounted read-write;
- no shared writable home, credentials, Provider Sessions, bridge key, BuildKit socket, registry tag namespace, package cache, nested engine store, or Kubernetes state;
- Server-only Incus administrative API;
- explicit port allocator and stale-forward reconciliation;
- optional dedicated Lima VM when the risk requires separate kernels.

### Trusted developer-domain profile

- shared Lima VM and Incus kernel;
- selected shared caches/registry/BuildKit only after namespacing and secret review;
- explicit statement that sibling compromise can affect shared VM-global state;
- never market this profile as hostile multi-tenant isolation.

Locki resembles the second profile, but additionally uses privileged containers and a globally shared secret-bearing home. It must not be copied unchanged.

## Recommendation relative to AgentBox

AgentBox's approved first-adapter status is not reopened:

- AgentBox already offers a broad local/cloud/provider placement surface and is the first optional adapter.
- Lima/Incus offers distinct local value: a physical Host VM boundary, complete Linux/systemd environment, nested container/Kubernetes capability, and potentially higher-density per-Placement Incus isolation than one VM each.
- That distinct value justifies the single conformance prototype, not immediate implementation priority.
- If the prototype passes, implement a **direct optional Lima/Incus adapter**, not a Locki adapter or fork.
- If it fails mount, security, recovery, or density falsifiers, retain Lima/Locki only as an architecture pattern and continue with AgentBox/direct Host execution.

## Falsifiers and minimum follow-up

The only new ticket created is [Prototype the direct Lima/Incus Placement boundary on Apple Silicon and Linux KVM](https://github.com/BleedingDev/bleeding-ade/issues/27). It is deliberately a disposable conformance boundary, not a generic implementation backlog.

Defer or reject the direct adapter if the prototype cannot disprove any of these:

1. Host/guest watcher or concurrent-write integrity is unreliable with no documented safe fallback.
2. The default profile lets a compromised sibling reach another Worktree or secret-bearing state without a shared-VM-kernel exploit.
3. Nested Compose/BuildKit or small k3s workloads require privileged containers or broad VM-global writable state in the default profile.
4. Host reboot, VM recreation, container recreation, upgrade skew, stale locators, or partial cleanup require destructive guessing.
5. Ten realistic Placements make either target developer Host operationally unusable.
6. Lima/Incus versions and API-extension skew cannot be safely gated.
7. Published-port ownership cannot prevent collisions and stale forwarding.
8. Snapshot/copy labels cannot be represented without implying portable process/RAM/socket/PTY or semantic continuity.

No additional implementation, fork, adapter, cache, security-hardening, or UX tickets are justified until that boundary evidence exists.

## Primary source index

### Lima

- [README and ecosystem](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/README.md)
- [VM-driver selection](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/website/content/en/docs/config/vmtype/_index.md)
- [Mount implementations and caveats](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/website/content/en/docs/config/mount.md)
- [Default template and networking/resource defaults](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/templates/default.yaml)
- [Kubernetes template](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/templates/k8s.yaml)
- [`limactl watch --json`](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/cmd/limactl/watch.go)
- [Experimental snapshot command](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/cmd/limactl/snapshot.go)
- [QEMU snapshot implementation](https://github.com/lima-vm/lima/blob/183a60d721bf9cd969f366301bf90710b8c6a28e/pkg/driver/qemu/qemu.go)

### Incus

- [REST API and versioning](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/rest-api.md)
- [API extension registry](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/api-extensions.md)
- [Security model](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/explanation/security.md)
- [Security policy for privileged containers](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/SECURITY.md)
- [Projects and confined isolation](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/explanation/projects.md)
- [Docker nesting and privileged-container FAQ](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/faq.md)
- [Current requirements](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/requirements.md)
- [Snapshots, exports, and backups](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/howto/instances_backup.md)
- [Copy/move and live-migration limits](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/doc/howto/move_instances.md)
- [Structured CLI list output](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/cmd/incus/list.go)
- [Stateful snapshot CLI contract](https://github.com/lxc/incus/blob/a1ece14328058ba695ca0c7d4a83037bbf57f3e0/cmd/incus/snapshot.go)

### Locki

- [README and declared product topology](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/README.md)
- [Package metadata/version/platforms](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/pyproject.toml)
- [Lima VM lifecycle/configuration](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/vm.py)
- [Incus container lifecycle/mounts/caches](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/container.py)
- [Worktree creation and metadata](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/worktree.py)
- [Git/GitHub bridge server](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/bridge.py)
- [Forced-command implementation](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/internal.py)
- [Daemon, idle, and orphan cleanup](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/daemon.py)
- [Shared home and agent/provider state](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/services/home.py)
- [Host path/state layout](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/paths.py)
- [Port-forward implementation](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/cmd/port_forward.py)
- [VM provisioning, btrfs, BuildKit, registry, and caches](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/vm-setup.sh)
- [Container provisioning and nested workloads](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/container-setup.sh)
- [Sandbox operating/security guidance](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/AGENTS.md)
- [Fail-open Claude branch guard](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/src/locki/data/claude-branch-guard.sh)
- [E2E lifecycle/nested-workload test intent](https://github.com/JanPokorny/locki/blob/e5ca22c6e33f83e950c6c6d3dcca4336329a6d6b/test/e2e.sh)

### Container baselines

- [Docker bind-mount authority and Host coupling](https://github.com/docker/docs/blob/a3fd0352216480db9becce4878e8e79caae46a45/content/manuals/engine/storage/bind-mounts.md)
- [Docker user-namespace remapping](https://github.com/docker/docs/blob/a3fd0352216480db9becce4878e8e79caae46a45/content/manuals/engine/security/userns-remap.md)
- [Docker rootless mode](https://github.com/docker/docs/blob/a3fd0352216480db9becce4878e8e79caae46a45/content/manuals/engine/security/rootless/_index.md)
- [Podman lifecycle, REST, rootless, VM, and CRIU overview](https://github.com/podman-container-tools/podman/blob/8efac905d6a1a6eef672a714efb1c939ea1d597c/README.md)
- [gVisor boundary and OCI integration](https://github.com/google/gvisor/blob/3c5eee17dc45659fb86843531074f38e78e0cc35/README.md)
