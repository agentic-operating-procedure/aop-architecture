# AOP Compile-and-Run Pipeline (Architecture Scope)

Informative. This document fixes the pipeline topology: the four stages, their inputs and outputs, and the feedback loop that closes the system. The normative requirements on the compiled artefact are in `aop-spec`; the substrate semantics are owned by `aop-context-graph`.

## 1. The loop

```
   observe --> model --> compile --> execute
      ^                               |
      |                               v
      +--------- observed cases <--- evidence
```

The pipeline is a closed loop, not a relay. Execution evidence becomes new observation; the AOP stays true to the work because it is continuously re-derived from the work.

## 2. Stage 1: Observe

Input: task-level work context across the operational estate, including cross-application work and legacy interactions that never appear in API logs.

Output: normalised events.

Constraints:

- ARC-P1: The capture surface is a declared subset of the work surface. Personal content classes, message bodies and keystrokes, are excluded by design. The work is modelled, not the person.
- ARC-P2: Observation is continuous, not a periodic batch export. A batch-built graph cannot detect drift at the rate work changes.

## 3. Stage 2: Model

Input: normalised events.

Output: the Context Graph of Work.

Sub-components: normalisation, then case stitching, then graph construction. The graph carries entities for cases, tasks, steps, actors, applications, decisions, rules, exceptions, escalations and outcomes, with frequencies and evidence attached to every edge. Every node carries which cases traversed it, how often, at what cost, and with what outcome.

The distinction that matters: a knowledge graph answers what is true. A Context Graph of Work answers what is happening and why this path: sequence, systems, decisions, dependencies, exceptions, and the reasons behind the paths taken.

Drift detection runs here: when the work changes (a new workaround appears, an exception rate rises), the twin detects the drift and raises the recalibration obligation declared in the AOP's Control dimension (NM-CTL-4).

## 4. Stage 3: Compile

Input: the Context Graph of Work, plus the source SOP where one exists.

Output: a versioned AOP document, valid against the schema, with provenance.

Sub-components, in order:

1. Variant mining: enumerate the variants with their observed frequencies.
2. Decision-rule extraction: extract the rules with their thresholds. Where the SOP says a word such as "severe", the compiler finds the threshold at which humans actually escalate.
3. Exception and escalation pattern extraction: the conditions, the handling paths, the target roles, the bounds.
4. Governance bounds: the permitted set, the prohibited set, the risk-axis assessment, the authority classes.
5. Provenance attachment: every rule carries the case count and path frequency it was derived from (NM-PRV-3).
6. Versioning: the output is a new AOP version with its provenance block, never an edit to an existing version.

ARC-P3: The compiler emits nothing that the schema would reject. A compile that cannot produce a conformant document fails, and the failure is a finding, not a relaxed schema.

ARC-P4: SOP intake is translation, not conversion. The SOP is parsed into a lightweight intent hypothesis, which observation then tests. Matches, variants, exceptions, workarounds and undocumented paths are classified against it. The SOP remains the human-readable source of truth it was derived from; it is not the compiled artefact.

## 5. Stage 4: Execute

Input: a case, the bound AOP version, the context snapshot.

Output: actions under authority, and a complete case record.

The runtime sequence (NM-STR-13): snapshot the case context, match the variant, check authority, execute or escalate, record, emit. Matching selects an observed path, not a happy path; an unmatched variant fails closed (SEC-5).

Three properties the runtime must hold:

- Determinism where it matters. Routine cases run the same path with the same rules every time; there is no reasoning roulette on the cases the AOP already covers.
- Traceability by construction. Every action is attributable to the variant, the rule, and the rule's provenance, because the runtime loop is built that way rather than reconstructed afterwards.
- A clean human interface. Escalations arrive with the evidence bundle attached.

Shadow mode is the bridge into execution: the compiled AOP runs in observation-only mode against live cases, the agent's choices are compared with the human choices on the same cases, discrepancies are reviewed, and promotion to bounded execution is a decision made against that comparison, not against confidence.

## 6. The feedback path

ARC-P5: The case record is emitted back to the substrate as new observation. The loop closes: execution feeds the next compile, drift is detected, recalibration produces a new version, and the version's provenance chain reaches the cases it was learned from, including the cases the runtime itself produced.

ARC-P6: The feedback path never mutates a bound AOP version mid-run, and never mutates a released version at all (NM-ASR-4, SEC-8). Change arrives only as a new version through the governance of `aop-governance`.
