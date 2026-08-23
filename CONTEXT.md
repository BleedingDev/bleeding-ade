# BleedingADE domain language

This glossary defines the canonical product language for BleedingADE. It describes domain meaning and identity, not implementation schemas.

## People, devices, and connectivity

### Operator

The single human who owns and controls a BleedingADE installation in v1.

### Client

A web, mobile, or desktop application that connects to one or more BleedingADE Servers.

### Authorized Client

A Client with a valid server-issued authorization session. Several Authorized Clients may represent the same Operator simultaneously.

### Host

The physical machine, VM, VPS, or container substrate on which a BleedingADE Server or execution placement runs.

### BleedingADE Server

An independently useful backend with stable identity and durable local state. A Server owns the execution and canonical local records that originate there.

### Connection

One live Client-to-Server transport. Losing a Connection does not change Server identity or execution ownership.

### Connection Endpoint

A reachable address and access method for one Server. A Server may have several endpoints.

### Placement

The location and substrate on which a Server or execution runs. A local machine, Windows Dev Kit, remote VPS, or AgentBox box may be a Placement; none is the product identity of the work.

### Replica

A non-authoritative copy of state originating elsewhere. A Replica may support backup, search, or read models, but does not silently become execution or history authority.

## Projects and filesystem state

### Repository Identity

The best-effort identity of the same logical version-control repository across Servers and Checkouts.

### Project

The user-facing unit that groups Chats, Beads, history, statistics, artifacts, and one or more server-local Checkouts. A Project may have reduced VCS capabilities when it is not backed by Git.

### Checkout

One Server-local clone or working copy belonging to a Project.

### Worktree

An isolated working directory used by an executing Chat. A Chat usually has one Worktree while executing, but may exist without one.

### Checkpoint

A captured restorable state of a Checkout, Worktree, provider conversation, or placement-specific execution state. A Checkpoint is not itself proof of portable continuation.

### Artifact

A durable file, diff, report, image, build output, or other result associated with a Project, Chat, Execution Segment, runtime operation, or Bead.

### Credential Reference

A reference to a credential capability available on a Server. Credential values remain server-local and are not ordinary history or handover state.

## Conversation and execution

### Chat

A persistent conversational and work unit inside a Project. A Chat may be informational or planning-only and therefore require no Worktree, process, or Agent.

`Workstream` is not a separate canonical entity in v1. Where useful, it is a descriptive label for a Chat doing sustained work.

### Execution Segment

A contiguous period of execution for one Chat on one Server, with one runtime/provider-process set and normally one Worktree. A new Server, restarted process, provider resume, restore, relocation, takeover, or fork may create a new Execution Segment.

### Transition

An explicit relationship between Execution Segments, such as resume, relocate, takeover, checkpoint restore, or restart.

### Branch

A non-linear continuation in a Chat's execution history, created by a debug fork, recovery fork, or another explicit divergence.

### Execution Segment DAG

The directed acyclic graph of a Chat's Execution Segments, Transitions, and Branches. It is the truthful execution-history shape and must not be flattened into a false linear run.

### Execution Owner

The one BleedingADE Server currently authorized to continue an active Branch at a particular Ownership Epoch.

### Ownership Epoch

A monotonically advancing ownership generation used to distinguish current authority from stale commands or reappearing prior owners.

### Observer

An Authorized Client that reads state without exclusive interactive control.

### Controller Lease

Temporary exclusive control used only for inherently unsafe concurrent surfaces, such as raw terminal input, one interactive prompt controller, or the commit phase of an ownership transfer. Ordinary semantic commands do not require a global Client lock.

## Runtime concepts

### Runtime

The software harness that executes an Agent and exposes lifecycle, message, tool, approval, workflow, or history capabilities.

### Provider

The model or provider integration selected through a Runtime. Runtime and Provider are separate concepts even when a product exposes them together.

### Provider Session

A provider- or runtime-native persistent conversation identity. It may be resumable without preserving the original process.

### Process

A live operating-system process owned by one Server.

### Terminal

A Server-owned PTY-backed interactive byte stream associated with a process or shell.

### Terminal Attachment

One Client's observation or control connection to a Terminal.

### Turn

One user-to-runtime interaction cycle inside a Chat or Provider Session.

### Message

A semantic communication record from a user, assistant, system, Agent, or runtime.

### Tool Call

A runtime-reported invocation of a tool, including its lifecycle and result where available.

### Approval

A durable request for permission before an action proceeds. The originating Server owns whether it remains pending and accepts the first valid answer.

### Question

A durable runtime or Agent request for Operator input. The originating Server owns whether it remains pending and accepts the answer.

### Failure

A semantic record that an intended operation did not complete successfully.

### Completion

A semantic record that an Agent, Runtime Task, Workflow Stage, Turn, or operation reached its declared terminal success state.

## Agents and workflows

### Agent

A runtime-reported actor performing work.

### Top-level Agent

An Agent directly started for a Chat's Execution Segment.

### Child Agent

An Agent that the runtime reports as a child of another Agent.

### Runtime Task

A unit of delegated runtime work, such as a Claude Task. A Runtime Task is not automatically a Bead.

### Workflow

A runtime-reported structured process containing ordered, branching, or parallel work.

### Workflow Stage

One runtime-reported stage inside a Workflow.

## Beads Rust concepts

### Task Authority

The one Server-local `br` workspace currently authoritative for Project Bead mutations.

### Bead

One task or project-work record owned by `beads_rust` (`br`).

### Dependency

A directed relationship between Beads that affects readiness, blocking, hierarchy, or another `br`-defined relation.

### Task Claim

A `br` mutation recording that work has been claimed. It does not replace BleedingADE execution ownership.

### Task Replica / Interchange State

JSONL, Git, backup, or other exported state used to synchronize, recover, or inspect Beads outside the authoritative `br` workspace. It is not automatically current Task Authority.

### Graph Projection

A rebuildable BleedingADE view or derived analysis of Beads, dependencies, status, readiness, cycles, critical path, bottlenecks, or statistics.

## Events, projections, and attention

### Origin Event

A canonical semantic event stored by the BleedingADE Server that produced it.

### Local Event Reference

The identity of an Origin Event within its producing Server and local event stream.

### Projection

A rebuildable view derived from canonical Origin Events, Beads, or other authoritative records.

### Capability Evidence

Versioned evidence that a Server, Runtime, Provider Session, or adapter supports a specific operation or semantic fact. Desired capability is not capability evidence.

### Attention Item

Durable actionable state requiring Operator awareness or action, such as an Approval, Question, failure, or blocked transition.

### Acknowledgement

A record that the Operator has seen or intentionally dismissed an Attention Item without necessarily resolving its cause.

### Resolution

The authoritative outcome that closes an Attention Item or its originating request.

### Notification

An ephemeral delivery mechanism pointing to an Attention Item. A Notification is never the authoritative actionable state.

## Canonical relationship and identity invariants

1. A Client may connect to many Servers; a Server may be observed by many Clients.
2. A Server runs on a Host through a Placement, but Server identity is independent of endpoint and placement details.
3. A Project may have several server-local Checkouts and Worktrees sharing one Repository Identity.
4. A Chat may exist without execution. When it executes, it contains one or more Execution Segments arranged as an Execution Segment DAG.
5. Every active Branch has at most one Execution Owner at one current Ownership Epoch.
6. A Transition records continuity and discontinuity explicitly; it never implies live process migration.
7. Origin Events retain their producing Server identity. Replicas and Projections remain non-authoritative.
8. An Approval or Question may be displayed by many Clients or Replicas, but only its originating Server validates and resolves it.
9. A Project has at most one current Task Authority. Task Replica / Interchange State may be stale or divergent and must be labelled accordingly.
10. Runtime Tasks, Agents, Execution Segments, and Beads are distinct. Relationships between them are explicit rather than assumed.
11. Terminal-derived observations are diagnostic unless a semantic runtime boundary promotes them with Capability Evidence.
12. Cached state is labelled stale or offline and is never presented as current live truth.

## Retired or qualified terms

- **Environment:** avoid as a standalone user-facing term. Use BleedingADE Server, Host, Placement, Runtime, or shell environment as appropriate. Existing `environmentId` may remain for T3 compatibility.
- **Session:** never use bare. Use Provider Session, Client authorization session, or Terminal Attachment.
- **Handover:** use the precise operation: reconnect, provider resume, relocate, takeover, debug fork, recovery fork, checkpoint restore, or restart. Live process migration is excluded initially.
- **History:** qualify it as Origin Event history, Chat Timeline, Terminal history, Bead audit history, Replica, search index, or memory.
