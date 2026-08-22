# Wayfinder dependency graph

GitHub issues are the canonical tickets. This file is a readable mirror of their intended dependency structure.

```mermaid
flowchart TD
  define_job["Define the job BleedingADE must be hired to do"]
  product_promises["Reconcile the product promises and set hard non-goals"]
  verify_ecosystem[["Verify the candidate ecosystem and its integration surfaces"]]
  map_t3_seams[["Map T3 Code extension seams and upstream maintenance cost"]]
  trust_deployment["Define deployment, trust, privacy, and ownership assumptions"]
  domain_language["Define the canonical BleedingADE domain language"]
  execution_topology["Choose the T3 Code, AgentBox, and Herdr execution topology"]
  session_continuity["Choose session identity, continuity, and handover semantics"]
  event_fabric["Choose the semantic event fabric and capability contract"]
  prove_runtime_adapters(["Prove the runtime adapter strategy for T3 providers, OMP, Prime Agent, and Claude workflows"])
  beads_role["Define Beads data ownership and native graph intelligence"]
  prototype_ia(["Prototype the project-first web and mobile information architecture"])
  attention_semantics["Define global attention inbox semantics and interruption policy"]
  flywheel_scope["Choose Agent Flywheel integrations and phase boundaries"]
  recovery_invariants["Define failure, recovery, concurrency, and security invariants"]
  minimum_release["Define the minimum coherent release and its acceptance proof"]
  repo_upstream_strategy["Choose repository, fork, and upstream strategy"]
  approve_handoff["Approve the implementation handoff boundary"]
  define_job --> domain_language
  product_promises --> domain_language
  verify_ecosystem --> domain_language
  map_t3_seams --> domain_language
  verify_ecosystem --> execution_topology
  map_t3_seams --> execution_topology
  trust_deployment --> execution_topology
  domain_language --> execution_topology
  trust_deployment --> session_continuity
  domain_language --> session_continuity
  execution_topology --> session_continuity
  verify_ecosystem --> event_fabric
  map_t3_seams --> event_fabric
  domain_language --> event_fabric
  execution_topology --> event_fabric
  verify_ecosystem --> prove_runtime_adapters
  execution_topology --> prove_runtime_adapters
  event_fabric --> prove_runtime_adapters
  define_job --> beads_role
  verify_ecosystem --> beads_role
  map_t3_seams --> beads_role
  domain_language --> beads_role
  define_job --> prototype_ia
  product_promises --> prototype_ia
  domain_language --> prototype_ia
  beads_role --> prototype_ia
  event_fabric --> prototype_ia
  define_job --> attention_semantics
  domain_language --> attention_semantics
  event_fabric --> attention_semantics
  prototype_ia --> attention_semantics
  define_job --> flywheel_scope
  verify_ecosystem --> flywheel_scope
  domain_language --> flywheel_scope
  event_fabric --> flywheel_scope
  beads_role --> flywheel_scope
  trust_deployment --> recovery_invariants
  execution_topology --> recovery_invariants
  session_continuity --> recovery_invariants
  event_fabric --> recovery_invariants
  prove_runtime_adapters --> recovery_invariants
  product_promises --> minimum_release
  session_continuity --> minimum_release
  prove_runtime_adapters --> minimum_release
  beads_role --> minimum_release
  prototype_ia --> minimum_release
  attention_semantics --> minimum_release
  flywheel_scope --> minimum_release
  recovery_invariants --> minimum_release
  verify_ecosystem --> repo_upstream_strategy
  map_t3_seams --> repo_upstream_strategy
  minimum_release --> repo_upstream_strategy
  minimum_release --> approve_handoff
  repo_upstream_strategy --> approve_handoff
```

## Initial frontier

- **Define the job BleedingADE must be hired to do** — `grilling`, `HITL`
- **Reconcile the product promises and set hard non-goals** — `grilling`, `HITL`
- **Verify the candidate ecosystem and its integration surfaces** — `research`, `AFK`
- **Map T3 Code extension seams and upstream maintenance cost** — `research`, `AFK`
- **Define deployment, trust, privacy, and ownership assumptions** — `grilling`, `HITL`

Research tickets may be worked in parallel. HITL tickets require Petr and must not be self-resolved.
