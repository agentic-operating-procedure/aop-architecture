# Changelog

All notable changes to this repository are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning is semantic: the major version changes when a layer, a trust mechanism, a pipeline stage or a topology is added, removed or reordered, the minor version when substantive architectural content is added, and the patch version for corrections that do not change meaning.

## [Unreleased]

## [0.1.0] - 2026-11-03

### Reference architecture

- Published the layered reference architecture: the five layers, the dependency rule, the four runtime responsibilities, and the runtime entity vocabulary carried forward from the staged metamodel.
- Published the trust architecture: the four assurance mechanisms (prevent, verify, oversee, audit), the guardrail decision flow, the credential model, and the recovery model.
- Published the compile-and-run pipeline: observe, model, compile, execute, the shadow-mode bridge, and the feedback path that closes the loop.
- Published the three deployment topologies: single-harness, multi-harness, and brownfield, with the selection conditions.
- Published the threat model: the trust assumptions, the four primary threats (malicious or confused AOP, compromised agent, credential leak, prompt injection into the procedure) plus substrate poisoning, and the mechanism answering each.
- Named the interfaces that Phases 4 and 5 implement against, closing the Phase 3 interface obligation for this repository's tier.

[Unreleased]: https://github.com/agentic-operating-procedure/aop-architecture/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/agentic-operating-procedure/aop-architecture/releases/tag/v0.1.0
