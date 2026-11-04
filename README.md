# AOP Architecture

`aop-architecture` is the single authority for the reference architecture of the Agentic Operating Procedure: the layered view of the system, the trust architecture and its boundaries, the observe-model-compile-execute pipeline, the deployment topologies, and the threat model.

| Field | Value |
|---|---|
| Authority | The reference architecture: layers, trust boundaries, pipeline, topologies, threat model |
| Tier | Architecture |
| Landing wave | 2 |
| Version | 0.1.1 |
| Status | Stable |
| Licence | MIT |

## What this repository owns

- The layered reference architecture: AOP, skill, harness, agent, run, and what each layer may depend on.
- The trust architecture: the four assurance mechanisms (prevent, verify, oversee, audit), the policy decision point and policy enforcement point outside the model context, and the recovery model.
- The compile-and-run pipeline: observe, model, compile, execute, and the feedback path from execution back into the substrate.
- The deployment topologies: single-harness, multi-harness, and brownfield.
- The threat model: a malicious or confused AOP, a compromised agent, a credential leak, and the prompt-injection path into the procedure.

## What this repository does not own

- The normative text that binds implementations. That is `aop-spec`. Where this repository and `aop-spec` disagree, `aop-spec` governs and this repository is corrected.
- The entity model of observed work, the stitcher, or drift detection. That is `aop-context-graph`.
- A working harness, guardrail, or evidence record. That is `aop-reference-harness`.

## Documents

| Document | Scope |
|---|---|
| [Layered reference architecture](./docs/aop-layered-reference-architecture-architecture-scope.md) | The layers, their responsibilities, and the dependency rule |
| [Trust architecture](./docs/aop-trust-architecture-architecture-scope.md) | Prevent, verify, oversee, audit; the enforcement point; recovery |
| [The compile-and-run pipeline](./docs/aop-compile-run-pipeline-architecture-scope.md) | Observe, model, compile, execute, and the closing of the loop |
| [Deployment topologies](./docs/aop-deployment-topologies-architecture-scope.md) | Single-harness, multi-harness, brownfield |
| [Threat model](./docs/aop-threat-model-architecture-scope.md) | Adversaries, attack paths, and the mechanisms that answer them |

## Interfaces fixed for later phases

Phase 3 closes when the interfaces that Phases 4 and 5 implement against are named. This landing names them:

| Consumer | Interface | Fixed by |
|---|---|---|
| `aop-toolkit` | Compiles an observed case corpus into an AOP document valid against the schema | `aop-spec`, schema |
| `aop-reference-harness` | Executes a run against one immutable AOP version and emits a case record | `aop-spec`, procedure structure and security clauses |
| `aop-registry` | Publishes and resolves AOP identifiers with their provenance chains | `aop-spec`, identifier and versioning |
| `aop-conformance` | Tests documents and harnesses against the conformance clauses | `aop-spec`, conformance clauses |
| `aop-evaluation` | Compares agent choices against the declared human baseline | `aop-spec`, Assurance dimension |
| `aop-context-graph` | Supplies the context snapshots referenced by the `cg://` scheme | `aop-spec`, identifier and versioning |
