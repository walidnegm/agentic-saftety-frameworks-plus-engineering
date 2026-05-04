# Agentic Safety Plus: Engineering

*Agent-Native Engineering Practices for High-Velocity Software Factories Built on Layered Agent Safety.*

## What this is

A living specification of how software gets produced when autonomous LLM-driven agents are first-class participants in the build process, not tools a human directs. Agents draft plans, adversarially review each other's work, generate code in parallel pods, verify artifacts at scale, surface drift, and orchestrate deploys. The human is the functional architect and thread coordinator — strategic direction and architectural judgment, not implementation.

## Why it exists

In the current 2026 landscape, frontier models routinely produce thousands of lines of production-grade code per day, deliver dozens of features per week, and sustain hourly deployment cadence. Raw generative capability is no longer the bottleneck — coordination is. Traditional Agile and Waterfall methods were built for human iteration cycles and sequential hand-offs; under agentic acceleration they degrade rapidly. Epics proliferate, backlogs explode, architectural integrity erodes through accumulated drift, and trust in generated artifacts collapses under context loss, sycophancy, and hallucination.

This framework operationalizes adversarial verification, multi-LLM separation of concerns, runtime governance, and continuous auditing into a cohesive, high-velocity software factory engineered for massive parallelism while preserving deterministic control.

## The shape of the framework

- **Four disciplines** — PPRE (Plan → Pushback → Revisit → Execute), assume-compromise verification, behavior pinning via regression tests, symptom-to-root with codebase-wide invariant audit. The disciplines do not enforce themselves; they are activated through agents.
- **Eighteen agents in six lifecycle buckets** — Planning & Specification, Execution, Verification, Coherence, Runtime Stabilization, Operational Plumbing. Each bucket is a coordination boundary; agents within and across buckets carry the disciplines.
- **Central Factory Supervisor** — orchestrator above all agents that owns the work registry, enforces global invariants, routes events, synchronizes context across LLM A (planning) and LLM B (execution), and escalates only when judgment is required.
- **CDID, not CI/CD** — Concurrent Development, Integration, and Deployment. Many independent changes coexist in different stages — design, implementation, verification, deploy — at once, without losing coherence.
- **Continuous code dosing** — once releases are continuous, deployments stop being events and become an infusion. Rollout pacing, monitoring, dose-response attribution, and incident response all change.

## The core thesis

Velocity and rigor are not adversaries. The factory engineers them as the same property: speed is a function of how aggressively verification machinery can run in parallel with development, and safety is a function of how many independent verification streams a change has survived. The framework is also independent of context-window size — even infinite-context LLMs need to be agentified to be accountable.

## Who this is for

Engineering leads, founders, and architects building agentic systems at sustained velocity who need a vocabulary for the disciplines, agents, and orchestration that make autonomous code production trustworthy at scale.

→ **Read the full framework:** [agentic-safety-plus_engineering.md](agentic-safety-plus_engineering.md)

---

*Living spec, work in progress. Companion to the layered agent safety framework at [walidnegm/agent-frameworks-safety-plus](https://github.com/walidnegm/agent-frameworks-safety-plus).*
