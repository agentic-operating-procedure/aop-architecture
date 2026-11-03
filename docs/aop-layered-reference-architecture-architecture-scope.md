# AOP Layered Reference Architecture (Architecture Scope)

Informative. This document fixes the layered view of the system. Where a layering question arises, this document answers it; where a normative obligation is stated, `aop-spec` governs.

## 1. The layers

Five layers, descending in authority and ascending in replaceability.

| # | Layer | Responsibility | Replaceability |
|---|---|---|---|
| 1 | AOP | State the operating intent, the authority, the constraints and the assurance obligations for a procedure | Authoritative; replaced only by a new version with provenance |
| 2 | Agent Skill | Provide one reusable, atomic, independently testable capability | Highly replaceable; a skill never carries authority |
| 3 | Domain Agent Harness | Control the execution environment: tools, credentials, escalation, verification | Reference-quality; domains implement their own against the conformance profile |
| 4 | Agent | Consume an AOP, a skill and a context snapshot; produce actions and evidence | The most replaceable layer; a model-driven agent and a rules-based executor are interchangeable from the architecture's point of view |
| 5 | Run | One instance of execution against one immutable AOP version | Ephemeral; produces the case record and the outcome |

## 2. The dependency rule

ARC-1: Dependency flows upward only. A consumer of a lower layer does not depend on the consumer of a higher one.

Concretely:

- The AOP does not know which agent executes it.
- The harness does not know who evaluates it.
- The skill does not know which AOP invokes it.
- The run does not know which registry catalogues the AOP it executed.

ARC-2: Authority flows downward only. An agent holds no authority; a skill holds no authority; the run inherits the authority the AOP states; the harness bounds everything inside it as the top-level AOP for its domain (SEC-11).

ARC-3: A layer may be replaced without changes above it, provided the replacement satisfies the same normative clauses. This is the test of the layering: if replacing the agent requires editing the AOP, the authority model has leaked downward and the deployment is not conformant.

## 3. Runtime responsibilities

At run time four distinct responsibilities cooperate, and the separation between them is where the trust boundaries sit:

| Responsibility | Held by | May not |
|---|---|---|
| Specification | The AOP version bound to the run | Change during the run |
| Enforcement | The harness guardrail, outside the model context | Take its decision from anything the agent controls |
| Orchestration | The harness runtime loop | Execute an action the enforcement point did not approve |
| Execution | The agent | Decide its own authority; hold standing credentials; see context beyond its case and authority level |

ARC-4: The enforcement point is a policy decision point plus a policy enforcement point, and both sit outside the model context. The agent proposes; the guardrail disposes.

## 4. Runtime entities

The runtime model uses the entity vocabulary carried forward from the staged metamodel (AOP-MM stage 07): AOP Run, Execution Context, Agent Session, Activity Instance, Coordination Arrangement, Runtime Event, Execution State, Synchronisation Point, Human Interaction Request, Runtime Handoff, Runtime Assurance Record, Runtime Failure, Runtime Recovery, Runtime Evidence Record, Run Outcome.

ARC-5: A run references exactly one immutable AOP version (NM-STR-11). Every runtime event is attributable to the run, and through the run to the AOP version, the variant, and the decision rule that authorised it.

ARC-6: A run terminates in exactly one outcome disposition: `achieved`, `partially_achieved`, `not_achieved`, `accepted_with_deviation`, `inconclusive` (NM-STR-14). The disposition is recorded in the case record and emitted to the substrate.

## 5. What crosses a layer boundary

The artefacts that move between layers, and nothing else:

| Boundary | Crosses |
|---|---|
| AOP to agent | The AOP document, the context snapshot, the authority decisions |
| Agent to skill | The declared inputs; the declared reports return |
| Harness to agent | Tool access, scoped credentials, escalation requests, verification verdicts |
| Run to substrate | The case record, the outcome, the execution evidence that becomes new observation |

Anything else crossing a boundary is an architecture violation and is logged as a finding against `aop-governance`.
