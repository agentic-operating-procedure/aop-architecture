# AOP Trust Architecture (Architecture Scope)

Informative. This document fixes the trust architecture: the four assurance mechanisms, the enforcement point, the credential model, and the recovery model. The normative obligations are the security clauses in `aop-spec` (SEC-1 to SEC-13); this document describes the shape implementations take to satisfy them.

## 1. The design stance

The governance question is not how much autonomy to give agents. It is what the real workflow justifies. Bounded execution is derived from the AOP's permitted steps, business rules, decision paths, exception patterns, policy constraints, and the risk level of each action, all of it measured from observed work.

Trust is earned mechanically, against the human baseline, not granted by configuration or by confidence in a model.

## 2. The four assurance mechanisms

The order matters: prevent is structural, verify is independent, oversee is human, audit is total.

### Prevent

The agent cannot exceed its authority because the check is performed by a component it cannot influence. The AOP and the guardrail act as a policy decision point and a policy enforcement point outside the model context. The agent proposes actions; it cannot decide whether they are permitted, and no prompt, tool result or agent-composed argument changes the outcome (SEC-1, SEC-2).

### Verify

Completed actions are independently checked against the intent the AOP recorded. Verification fails closed against observed paths: a known pattern within authority executes; a known pattern beyond authority requests approval; an unknown condition escalates. Uncertainty routes to a person, never to a guess (SEC-5).

### Oversee

Escalations arrive as evidence bundles carrying the proposed action, the triggering rule, that rule's observed provenance, the baseline comparison, and the case context. The approver rules on a case, not on a mystery (SEC-9). The human decides; the agent does not proceed on its own judgement while the escalation is open.

### Audit

Every action appends to an append-only, tamper-evident case record: inputs, context snapshot, rule applied, provenance, decision, approver where relevant, outcome (SEC-8). Every decision is replayable from the record alone (SEC-10). Routine sampling reviews provide assurance without a blanket throttle.

## 3. The guardrail decision flow

The guardrail is mechanical. At the enforcement point, per proposed action:

```
agent proposes action
        |
        v
action in the permitted set?  --no--> escalate to the named human decision
        | yes
        v
within authority thresholds?  --no--> request human approval (evidence bundle)
        | yes
        v
known exception pattern?      --yes-> execute per the validated path
        | no or uncertain
        v
escalate with the full trace
```

All branches log inputs, decision and rationale to the case record. There is no path that proceeds without a record (SEC-6).

## 4. The credential model

ARC-C1: Credentials are short-lived, scoped to one action class, and issued at the moment of execution (SEC-3). A credential scoped to a diagnostic action does not authorize a dispatch.

ARC-C2: High-risk actions receive credentials only after human approval, and the credential is issued after the approval.

ARC-C3: The credential broker sits beside the enforcement point, outside the agent's reach. The agent names the action class it needs; the broker checks the authority decision; the harness holds the credential for the duration of the action and revokes it at completion.

ARC-C4: Context access follows the same discipline: the snapshot is scoped to what the case and the authority level require (SEC-4). Context engineering doubles as data minimisation.

## 5. The recovery model

Failures and exceptions are normal operation, not edge cases. The recovery model, carried forward from the staged metamodel (AOP-MM stage 08), distinguishes:

| Concept | Meaning |
|---|---|
| Fault | The observed malfunction in the work |
| Error state | The state the run enters on a fault |
| Exception occurrence | The AOP-declared condition that fired |
| Operational deviation | Execution that left the matched variant |
| Recovery strategy | The specification-time answer, declared in the AOP |
| Recovery attempt | The runtime execution of that strategy, with its own lifecycle |
| Compensation execution | The undo path where the work must be rolled back |
| Safe state | The state the run must reach when recovery cannot proceed |

ARC-R1: A recovery attempt has its own lifecycle (requested, authorised, running, completed, and the terminal states) and its own record in the case record. Recovery is never outside governance.

ARC-R2: Verification outcomes are distinct: `verified`, `partially_verified`, `not_verified`, `inconclusive`, `verification_failed`. Not verified and verification failed are different outcomes with different routes.

ARC-R3: Where recovery fails, the run reaches the declared safe state and escalates. A run never ends silently in an unknown state.

## 6. The trust base

The guardrail, the credential broker and the case record are the root of trust, and their own protection is part of the architecture, not a deployment detail.

ARC-TB1: Policy bundles are signed and versioned, and rollback is a supported operation. A harness verifies the bundle digest of the AOP version it binds before the first run; a bundle that fails verification is not loaded, and the domain holds (SEC-14). The signing scheme and the key ceremony are deployment obligations with named interfaces; the reference harness records the bundle digest in the case record so every run is attributable to a verified bundle.

ARC-TB2: Key custody for the signing keys and external anchoring of the case record (periodic digest commitment to a store outside the harness estate) are named deployment obligations. The case record's hash chain detects tampering within the file (SEC-8); external anchoring detects replacement of the whole file.

ARC-TB3: Separation of duties: the AOP author, the ratifier and the harness operator are distinct roles. One person holding all three is the single point at which a confused AOP, a rubber-stamped ratification and a widened boundary become the same event. The document-level expression is the governance block's prepared_by and ratified_by (CC-23) and the accountable owner (NM-GOV-1).

ARC-TB4: The emergency stop (SEC-15) is part of the trust base: halting the domain is a governed action with its own record, and resumption requires the accountable owner. A stop that required no record and no owner would be a bypass wearing a safety hat.

## 7. What the trust architecture does not fix

The concrete guardrail implementation, the credential broker, the escalation dispatch and the evidence bundle writer belong to `aop-reference-harness`. The threat model that motivates each mechanism is in the threat model document. The normative clauses that bind every implementation are in `aop-spec`.
