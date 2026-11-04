# AOP Threat Model (Architecture Scope)

Informative. This document fixes the threat model for AOP-governed execution: the adversaries, the attack paths, and the architectural mechanism that answers each. The normative obligations these mechanisms satisfy are the security clauses in `aop-spec`.

## 1. Trust assumptions

| Assumption | Consequence if false |
|---|---|
| The enforcement point is outside the agent's context and out of its influence | The entire model collapses to prompt-level safety; no bounded-execution claim survives |
| The AOP version bound to a run is immutable and its provenance is intact | Rules could be swapped under a running case |
| The case record is append-only and tamper-evident | Audit and replay claims are void |
| The observation substrate is not writable by the agents it observes | Agents could shape the procedures compiled from their own behaviour |
| Credentials are brokered outside the agent | The agent becomes its own authority at run time |

The first assumption is the one the whole programme rests on. Everything in this document is secondary to keeping the policy decision point and the policy enforcement point where the agent cannot reach them.

## 2. Threats and answers

### T1: The malicious or confused AOP

Attack: the procedure itself is the adversary. An AOP author (or a compromised compile) encodes rules that exfiltrate data, widen authority, or prohibit escalation. A confused AOP is the accidental version: an over-broad permitted set, a threshold copied from a stale document, a variant frequency wrong enough to route the wrong path.

Path: AOP document -> compiler -> registry -> harness -> agent.

Answer:

- Authority coverage is closed by default (NM-AUT-3): anything not explicitly ruled is prohibited or default-classed.
- The decision-rights ladder binds rule-level authority to the declared risk-axis assessment (NM-AUT-6); a rule cannot be more autonomous than its axes justify.
- Every rule carries observed provenance (NM-PRV-3); an authority-bearing rule without observation is a conformance failure (CC-6), which makes silent rule fabrication expensive.
- Conformance profiles and deliberately broken fixtures (`aop-conformance`) test exactly these failure shapes before publication.

Residual risk: a malicious AOP whose rules are internally consistent and provenance-backed. The answer is the governance layer: approval, release and operational authorisation are distinct events with evidence records (NM-VER-5), so a suspicious AOP can be suspended per scope without touching the estate.

### T2: The compromised agent

Attack: the agent (model-driven) is manipulated, by instruction injection, by a crafted context, or by its own failure modes, into proposing actions outside its authority.

Path: agent context -> agent -> proposed action -> guardrail.

Answer:

- The proposal is checked by the guardrail outside the model context (SEC-1). A compromised agent can propose anything; it cannot execute anything the guardrail did not approve.
- Fail-closed enforcement (SEC-5): unmatched variants, unknown conditions and enforcement errors escalate rather than proceed.
- Credentials are short-lived, action-class scoped, and issued by the broker (SEC-3, ARC-C1 to ARC-C3). A compromised agent holds almost nothing worth stealing at any moment.
- Context minimisation (SEC-4) bounds what a successful injection can even see.

Residual risk: denial-of-service by escalation flooding. The answer is rate and pattern monitoring at the harness, recorded per case; escalation precision is an evaluation metric (`aop-evaluation`), not an afterthought.

### T3: The credential leak

Attack: credentials harvested from the agent context, from logs, or from a long-lived token store are replayed outside the harness.

Answer:

- Short-lived, action-class-scoped credentials (SEC-3): the window and the blast radius are both bounded.
- High-risk credentials are issued only after human approval, after it, not before (ARC-C2).
- The case record ties every credential issuance to a run, a rule and a decision, so replay is detectable by correlation.

Residual risk: a compromised broker. That is a platform security problem outside this architecture's scope; the harness profile (`aop-conformance`) requires the broker to sit outside the agent-reachable surface, and the deployment is expected to apply ordinary platform hardening to it.

### T4: The prompt-injection path into the procedure

Attack: untrusted content in the case context (a customer message, a document, a web page) carries instructions aimed at the agent: skip the check, widen the scope, call this tool.

Path: untrusted content -> context snapshot -> agent.

Answer:

- The context snapshot is immutable for the run (NM-CTX-2) and scoped (SEC-4); injected instructions can reach the agent but not the guardrail, the broker, or the AOP.
- The agent cannot act on injected instructions that exceed authority, because the guardrail decides (T2).
- Where injected content resembles a known exception pattern, the exception edge fires; where it is unknown, the run fails closed and escalates with the full trace.
- The provenance chain keeps untrusted sources attributable: the context element model requires a source per element (NM-CTX-1), so post-incident review can identify which input carried the injection.

Residual risk: social engineering of the human approver. The evidence bundle (SEC-9) is the mitigation: the approver sees the triggering rule, its observed provenance, and the baseline comparison, so an out-of-pattern request is visible as out-of-pattern.

### T5: Substrate poisoning

Attack: the observation stream is manipulated (synthetic events, suppressed events) so that the next compile encodes the attacker's preferred procedure.

Path: event source -> normalisation -> Context Graph -> compile.

Answer:

- The observation posture is declared: the capture surface is a declared subset of the work surface (ARC-P1), so sources are enumerated.
- Variants carry observed frequencies and case provenance; a synthetic variant with no case history is visible in the provenance chain (NM-PRV-1 to NM-PRV-4).
- Compilation is versioned and governed: a new AOP version goes through approval and operational authorisation (NM-VER-5), giving a human gate before poisoned output becomes effective.

Residual risk: slow, low-and-slow manipulation of frequencies. The drift-detection and periodic-review obligations (NM-CTL-4, NM-VER-5) exist for exactly this; recalibration is itself a reviewed event.

### T6: The guardrail or policy bundle compromised

Attack: the enforcement point itself is the adversary, through a modified policy bundle, a tampered harness deploy, or a signing key compromise. Every check above assumes the checker is honest; this threat removes the assumption.

Path: bundle source -> signing -> harness load -> enforcement.

Answer: signed versioned bundles with rollback (ARC-TB1); digest verification at load and per-run attribution in the case record; separation of duties among author, ratifier and operator (ARC-TB3), so no single role can both write and approve a bundle; external anchoring (ARC-TB2) makes wholesale log replacement visible; emergency stop (SEC-15) lets a named role halt the domain without operator tools.

Residual risk: compromise of the signing key itself. Key custody is a named deployment obligation (ARC-TB2); the architecture keeps the blast radius visible by recording which bundle version authorised every run.

### T7: Case-record truncation or wholesale tampering

Attack: the append-only property is defeated not by editing an entry but by replacing or truncating the record store, or by selective deletion between anchoring points.

Path: record store -> harness operator or platform compromise -> audit void.

Answer: the hash chain detects intra-file edits (SEC-8, HAR-6 breaks on a single edit); external anchoring of the chain head (ARC-TB2) detects truncation and wholesale replacement; retention obligations (CG-O6) and audited access (CG-O7) bound who can touch the store at all. Replay (SEC-10) against an independently held copy of any case's bundle makes selective deletion evident.

Residual risk: the anchoring store itself. It sits outside the harness estate precisely so that compromising both requires separate failures.

## 3. What this model does not cover

- Platform security of the host estate (identity, network, key management beyond the broker boundary). The architecture assumes it is present and hardens the AOP-specific paths on top of it.
- Model-level safety of the agent itself. The architecture does not trust the model to be safe; that is the design. Model behaviour is an evaluation concern (`aop-evaluation`), not a governance mechanism.
- Legal and regulatory risk classification. The Authority dimension's risk axes require the adopting organisation to classify its own operations; the architecture carries the classification, it does not produce it.
