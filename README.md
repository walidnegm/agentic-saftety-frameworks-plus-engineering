# Agentic Safety Plus: Engineering

*Agent-Native Engineering Practices for High-Velocity Software Factories Built on Layered Agent Safety.*

## What this is

A living specification of how software gets produced when autonomous LLM-driven agents are first-class participants in the build process, not tools a human directs. Agents draft plans, adversarially review each other's work, generate code in parallel pods, verify artifacts at scale, surface drift, and orchestrate deploys. The human is the functional architect and thread coordinator — strategic direction and architectural judgment, not implementation.

## Why it exists

In the current 2026 landscape, frontier models routinely produce thousands of lines of production-grade code per day, deliver dozens of features per week, and sustain hourly deployment cadence. Raw generative capability is no longer the bottleneck — coordination is. Traditional Agile and Waterfall methods were built for human iteration cycles and sequential hand-offs; under agentic acceleration they degrade rapidly. Epics proliferate, backlogs explode, architectural integrity erodes through accumulated drift, and trust in generated artifacts collapses under context loss, sycophancy, and hallucination.

This framework operationalizes adversarial verification, multi-LLM separation of concerns, runtime governance, and continuous auditing into a cohesive, high-velocity software factory engineered for massive parallelism while preserving deterministic control.

## The shape of the framework

**Primitives** (the four concepts to internalize):

- **PPRE** — Plan → Pushback → Revisit → Execute. A four-step gating cycle for every non-trivial workstream. Each letter prevents a named failure mode: single-author planning, sycophantic review, hesitation in revisit, serial execution.
- **Assume-compromise** — system-wide posture in which every artifact is treated as potentially flawed until it has survived independent adversarial verification at four layers (proposal, implementation, runtime, post-merge).
- **Behavior pinning** — every change ships with a regression test that pins the *shape of the contract*, not solely the absence of the immediate defect.
- **Symptom-to-root with codebase-wide invariant audit** — a reported symptom is the entry point to a root invariant; every fix sweeps the codebase for peer violations and pins the invariant globally, not just at the symptom site.

**Supporting mechanisms**:

- **Eighteen agents in six lifecycle buckets** — Planning & Specification, Execution, Verification, Coherence, Runtime Stabilization, Operational Plumbing. Each bucket is a coordination boundary; agents enforce the primitives at specific layers and stages.
- **Central Factory Supervisor** — orchestrator above all agents that owns a Postgres-backed work registry, enforces global invariants as state-transition predicates, synchronizes context across LLM A (planning) and LLM B (execution), and escalates only when judgment is required.
- **CDID, not CI/CD** — Concurrent Development, Integration, and Deployment. Many independent changes coexist in different stages — design, implementation, verification, deploy — at once, without losing coherence.
- **Continuous code dosing with change-type routing** — once releases are continuous, deployments stop being events and become an infusion. But not every change ships the same way: features still need flag-ramp + acceptance, architectural refactors still need full PPRE + human review, hygiene runs continuously only when external behavior is preserved. The framework's value is in routing each change to the rollout discipline its blast radius warrants.
- **Refactor discipline** — agentic velocity lowers the cost of refactoring, but does not eliminate the need for refactoring discipline. Mechanical cleanup runs continuously; architectural refactors require full PPRE; speculative "cleaner-but-no-measurable-benefit" refactors are avoided.

## The core thesis

Velocity and rigor are not adversaries. The factory engineers them as the same property: speed is a function of how aggressively verification machinery can run in parallel with development, and safety is a function of how many independent verification streams a change has survived. The framework is also independent of context-window size — even infinite-context LLMs need to be agentified to be accountable.

## Worked examples in the spec

The full spec includes concrete walkthroughs:

- **PPRE in action** — a diagnosis-loop case where the initial plan was wrong and pushback corrected it before any code was written.
- **Symptom-to-root** — a 500 traced through API → gRPC → engine code, root invariant identified, codebase-wide audit (4 sites, 1 unguarded), and a global behavior pin written.
- **Supervisor work-registry record** — a concrete JSON shape for an Epic in flight, with status transitions encoded as predicates the daemon evaluates.

## Who this is for

Engineering leads, founders, and architects building agentic systems at sustained velocity who need a vocabulary for the disciplines, agents, and orchestration that make autonomous code production trustworthy at scale.

→ **Read the full framework:** [agentic-safety-plus_engineering.md](agentic-safety-plus_engineering.md)

---

*Living spec, work in progress. Companion to the layered agent safety framework at [walidnegm/agent-frameworks-safety-plus](https://github.com/walidnegm/agent-frameworks-safety-plus).*
