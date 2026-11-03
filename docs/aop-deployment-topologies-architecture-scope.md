# AOP Deployment Topologies (Architecture Scope)

Informative. This document fixes the three deployment topologies and the conditions under which each is the right shape. It deliberately does not mandate an estate shape: the architecture is a reference model, not a mandate to adopt a particular product, framework or topology.

## 1. Single-harness

One domain, one harness, the AOPs of that domain running inside it.

```
+------------------------------------------+
|            Domain Agent Harness          |
|  (the top-level AOP for the domain)      |
|                                          |
|   guardrail | credentials | escalation   |
|                                          |
|   AOP-1   AOP-2   AOP-3                  |
|     |       |       |                    |
|   skills and agents underneath           |
+------------------------------------------+
             |              ^
             v              |
      external systems   Context Graph of Work
```

Properties:

- The simplest topology and the first one an adopter should run. One trust boundary, one case record store, one escalation route.
- The harness boundary and the domain boundary coincide, which keeps the top-level-AOP obligation (SEC-11) auditable in one document.
- Fits a pilot and most single-procedure deployments.

ARC-T1: In every topology, the harness is the top-level AOP for its domain, and every AOP running inside it is bounded by it. A harness that permits what an inner AOP forbids is misconfigured; the inner AOP governs its procedure, the harness governs the domain.

## 2. Multi-harness

Several domains, each with its own harness, coordinated at the estate level.

```
+----------------+     +----------------+     +----------------+
| Harness: FM    |     | Harness: KYC   |     | Harness: CRM   |
| top-level AOP  |     | top-level AOP  |     | top-level AOP  |
+----------------+     +----------------+     +----------------+
        |                      |                      |
        +----------+-----------+---------------------+
                   |                       |
            estate registry          estate observation
        (versions, provenance,      (Context Graph of Work,
         discovery)                  cross-domain cases)
```

Properties:

- Each harness keeps its own trust boundary, credentials and escalation routes. Trust does not blur across domains: a credential scoped in one harness is not valid in another.
- Cross-domain procedures exist: the coordination dimension of the participating AOPs declares the handoff, and the handoff is itself governed (NM-CRD-1).
- The registry and the observation substrate are shared estate services; they carry no authority of their own.

ARC-T2: In the multi-harness topology, an AOP from one harness MUST NOT execute directly inside another. Cross-domain work crosses through declared handoffs, and the receiving harness re-evaluates authority for the actions it is asked to perform. Authority is never assumed to travel with the case.

## 3. Brownfield

An existing operational estate (SOP libraries, RPA, workflow engines, ticketing) gains AOP-governed agents around it rather than instead of it.

```
+---------------------------------------------------+
| existing estate                                   |
|   SOP library    RPA bots    workflow engines     |
+---------------------------------------------------+
        ^                ^               ^
        |                |               |
   +-------------------------------------------+
   |        AOP decision layer                 |
   |   (AOPs decide; existing systems do)      |
   +-------------------------------------------+
        |
   harness and guardrail around agent actions
```

Properties:

- Deterministic steps stay with the existing automation. The AOP becomes the decision layer above it: which variant applies, what the thresholds are, when to escalate.
- The SOP library remains the human-readable source of truth that AOPs are derived from; the workflow engine remains the execution substrate for the steps that are already automated.
- The observation surface extends to the legacy interactions that never appear in API logs, because those are exactly where the undocumented paths live.

ARC-T3: The brownfield topology adds governance; it does not retrofit authority onto systems that cannot enforce it. Where an existing system cannot fail closed, the AOP classifies its actions as `approval-required` at most, and the harness holds the boundary itself.

## 4. Choosing between them

| Condition | Topology |
|---|---|
| One domain, first deployment, pilot scope | Single-harness |
| Several domains with distinct risk profiles or regulators | Multi-harness |
| Significant existing automation and SOP estate | Brownfield, often combined with either of the others |

ARC-T4: A deployment MAY start single-harness and grow to multi-harness without changing the AOPs, because authority, structure and provenance are properties of the AOP documents, not of the topology. This is the practical test that the layering holds.
