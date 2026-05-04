# Agentic Safety Plus: Engineering

## Agent-Native Engineering Practices for High-Velocity Software Factories Built on Layered Agent Safety

> *Living specification, work in progress. This document describes generic agent-native engineering practices — vocabulary, disciplines, agent roles, invariants — independent of any specific agent SDK or platform. Implementation details for a specific stack are deferred to companion documents.*

---

## Table of Contents

1. [Scope: what this document is, and what it is not](#1-scope-what-this-document-is-and-what-it-is-not)
2. [Executive summary](#2-executive-summary)
3. [Agentic Software Factory at a Glance](#3-agentic-software-factory-at-a-glance)
4. [Flow diagram](#4-flow-diagram)
5. [LLM A vs LLM B: planning–execution separation](#5-llm-a-vs-llm-b-planningexecution-separation)
6. [What changes when building software automatically](#6-what-changes-when-building-software-automatically)
7. [The headline shift: from CI/CD to CDID](#7-the-headline-shift-from-cicd-to-cdid)
8. [Continuous code dosing: how releases change](#8-continuous-code-dosing-how-releases-change)
9. [The four disciplines](#9-the-four-disciplines)
10. [Agents by lifecycle bucket](#10-agents-by-lifecycle-bucket)
11. [The Central Factory Supervisor](#11-the-central-factory-supervisor)
12. [Integration with the layered safety framework](#12-integration-with-the-layered-safety-framework)
13. [Radical differences from traditional methods](#13-radical-differences-from-traditional-methods)
14. [Gap analysis vs. incumbent agent frameworks](#14-gap-analysis-vs-incumbent-agent-frameworks)
15. [Why the framework persists even with infinite context](#15-why-the-framework-persists-even-with-infinite-context)
16. [Workload attribution: where the LLMs run](#16-workload-attribution-where-the-llms-run)
17. [Our Role (Us the People)](#17-our-role-us-the-people)
18. [Failure modes the framework explicitly handles](#18-failure-modes-the-framework-explicitly-handles)
19. [Why this matters](#19-why-this-matters)

---

## 1. Scope: what this document is, and what it is not

This document describes the engineering-framework layer — **how software gets produced when autonomous LLM-driven agents are first-class participants in the build process, not tools a human directs.** Agents draft plans, adversarially review each other's work, generate code in parallel pods, verify artifacts, run scenario corpora at scale, surface drift, and orchestrate deploys. The human is functional architect and thread coordinator, not implementer. The framework is the discipline that makes this autonomous production process trustworthy at sustained velocity — concerned with how plans are formed, how proposals are reviewed, how code is generated, how artifacts are verified, how deploys are orchestrated, how the backlog is groomed, and how the human stays in the loop without becoming the bottleneck.

It is **not** the architecture of the software being built. The software produced by this framework is itself frequently agentic — the apps shipped contain LLM-driven agents, workflow runtimes, embeddings, retrieval, tool use, and runtime governance. Those are *application-architecture* concerns — concerns about the **substrate** the produced app runs on. They include:

- The **LLM registry / factory** that the produced app routes through (local vs. remote, OpenAI vs. Anthropic vs. open-weights, embeddings vs. completions, audit logging, per-service config, fallback policy).
- The **container runtime** (Docker, ECS Fargate, Lambda, on-prem) the produced app runs inside.
- The **persistence layer** (Postgres, vector store, Redis, blob storage) the produced app stores state in.
- The **agent runtime** the produced app's own agents use to talk to tools, hold sessions, and coordinate.
- The **workflow definition language** the produced app uses to express user-defined flows.
- The **runtime governance surface** (feature flags, prompt libraries, guardrails, parameterization) the produced app exposes for safe behavior tuning.
- The **observability stack** the produced app emits to.

Conflating the two layers produces two kinds of mistakes. If the engineering framework leaks application-architecture decisions ("the factory MUST route through OpenAI") it locks every app built with it into one substrate. If the application architecture leaks engineering-framework decisions ("the app's own agents MUST follow PPRE") it confuses runtime safety with build-time discipline.

**This document covers the engineering framework only.** Application-architecture (substrate) concerns are out of scope and may live in a separate document.

---

## 2. Executive summary

In the current 2026 landscape of AI-assisted software development, frontier models routinely produce thousands of lines of production-grade code per day, deliver dozens of features per week, and sustain hourly deployment cadence. Raw generative capability is no longer the bottleneck.

The new bottleneck is **coordination**. Traditional Agile and Waterfall methodologies were built for human iteration cycles and sequential hand-offs. Under agentic acceleration they degrade rapidly: epics proliferate, backlogs explode, architectural integrity erodes through accumulated drift, and trust in generated artifacts collapses under the weight of context loss, sycophancy, and hallucination.

**Agentic Safety Plus: Engineering** is a principled extension of the layered agent safety framework. It operationalizes adversarial verification, multi-LLM separation of concerns, runtime governance, and continuous auditing into a cohesive, high-velocity software factory. The system is engineered for massive parallelism while preserving deterministic control, architectural soundness, and long-term maintainability. The human founder retains the role of functional architect and thread coordinator, intervening only at high-judgment boundaries.

---

## 3. Agentic Software Factory at a Glance

The framework rests on a small number of premises and yields a small number of operating commitments. The remainder of this document elaborates each.

**Premises.** Traditional software development assumes deterministic code, sequential hand-offs, human-paced iteration, and implicit trust in produced artifacts. Agent-native engineering inherits none of these. The artifact is non-deterministic in two ways — stochastic LLM outputs and emergent flow — the production process is concurrent across many specialized agents, the cadence is machine-paced, and trust is engineered rather than assumed.

**Four disciplines** convert raw agent parallelism into trustworthy throughput:

- **PPRE** — Plan → Pushback → Revisit → Execute. A four-step gating cycle for every non-trivial workstream, terminating in execution only after the plan has survived independent critique.
- **Assume-compromise** — system-wide posture in which every artifact is treated as potentially flawed until it has survived independent adversarial verification at *multiple* layers: proposal (PPRE pushback), implementation (behavior pinning), runtime (Reality-Anchor), and post-merge (Reflection Loop). PPRE's pushback step is one instance of this posture; the posture is system-wide, not gate-specific.
- **Behavior pinning** — every change ships with a regression test that pins the shape of the contract, not solely the absence of the immediate defect.
- **Symptom-to-root** — a reported symptom is treated as the entry point to a root architectural finding, not as a target for patching.

The disciplines do not enforce themselves. They are activated through a cast of specialized agents — eighteen in total, grouped into **six buckets** by lifecycle stage. Each bucket is a coordination boundary; the disciplines flow through agents within and across buckets. A Central Factory Supervisor sits above all agents, owns the work registry, enforces global invariants, and routes events between agents. The human is the functional architect — strategic direction and architectural inflection — not the implementer.

> **Primitives vs. supporting mechanisms.** The four disciplines above are the **primitives** the reader should internalize — they recur throughout the document and shape every agent's behavior. The agents (Reality-Anchor, Reflection Loop, Hygiene, Conflict Prediction, Semantic Merge, and the other thirteen) and the Supervisor itself are the **supporting mechanisms** that enforce the primitives at specific layers and lifecycle stages. Hold the four primitives in working memory; the mechanisms can be looked up.


**Six agent buckets** organized by lifecycle stage:

| Bucket | Lifecycle stage | Constituent agents |
|---|---|---|
| Planning & Specification | Pre-implementation | Backlog, Epic Designer, Architecture Review, Module Governance |
| Execution | Implementation | Coding Pod Factory |
| Verification | Pre-merge gate | Testing Orchestrator, Safety & Security Audit, Costing |
| Coherence | Active during multi-pod parallelism | Conflict Prediction, Epic Decomposer, Semantic Merge |
| Runtime Stabilization | Continuous, during execution and on the codebase | Reality-Anchor (Watcher), Live Debugging, Hygiene |
| Operational Plumbing | Pipelines, telemetry, runtime configuration, post-merge audit | CI/CD, Observability, Feature Flag, Adversarial Reflection Loop |


**Headline shift: from CI/CD to CDID** — Concurrent Development, Integration, and Deployment. CI/CD optimizes how fast *one* change reaches production; CDID optimizes how many *independent* changes coexist in different stages — design, implementation, verification, deployment — at once, without losing coherence. Concurrency here is concrete: multiple coding pods generating against isolated worktrees in parallel, multiple plans under independent adversarial review, and multiple deploys in different rollout phases simultaneously. §7 unpacks the shift, and §8 covers how each individual change enters production once rollout itself becomes continuous (continuous code dosing).

---

## 4. Flow Diagram

```
                    ┌──────────────────────────────────────────────┐
                    │       CENTRAL FACTORY SUPERVISOR             │
                    │    (Orchestrator + Multi-LLM Router +        │
                    │     Work Registry + Invariant Enforcer)      │
                    │   Tracks all agent activity, routes tasks,   │
                    │   manages shared context, handles exceptions,│
                    │   prevents lost or misrouted work            │
                    └─────────────────┬────────────────────────────┘
                                      │
                                      ▼
                          Idea / User Request / Anomaly / Audit Finding
                                      │
                                      ▼
       ┌──────────────────────────────────────────────────────────┐
       │                Planning & Specification                   │
       │    Backlog → Epic Designer → Architecture Review (LLM A) │
       │                       ↓                                   │
       │              Module Governance (boundaries)               │
       └─────────────────────────┬────────────────────────────────┘
                                 │ approved Epic
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │                       Verification                        │
       │     Costing  ←  PPRE pushback   →  Safety & Security      │
       └─────────────────────────┬────────────────────────────────┘
                                 │ plan survives review
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │                        Coherence                          │
       │   Conflict Prediction → Epic Decomposer → Semantic Merge │
       │       (work-registry consult, surface declaration)       │
       └─────────────────────────┬────────────────────────────────┘
                                 │ surface claim registered
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │                  Execution (LLM B, ×N pods)               │
       │             Parallel coding pods on isolated worktrees    │
       │     Reality-Anchor (Watcher) monitors for drift           │
       └─────────────────────────┬────────────────────────────────┘
                                 │ branch + behavior pin + tests
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │           Testing & Scenario Verification                 │
       │   Functional/Scenario Modeler → Adversarial Generator →   │
       │   Distribution-shift verification (N-run pinning)         │
       └─────────────────────────┬────────────────────────────────┘
                                 │ pinned distribution holds
                                 ▼
       ┌──────────────────────────────────────────────────────────┐
       │                  Operational Plumbing                     │
       │      CI/CD → Feature Flag rollout → Observability →       │
       │      Adversarial Reflection Loop (post-merge audit)       │
       └─────────────────────────┬────────────────────────────────┘
                                 │ continuous code dosing
                                 ▼
                       Production (hourly cadence)

       ↑──────────────────────────────────────────────────────────┐
       │   Runtime Stabilization (continuous, all stages)          │
       │   Reality-Anchor + Live Debugging + Hygiene Agent         │
       └──────────────────────────────────────────────────────────┘

           Findings / anomalies / drift loop back into the Backlog
```

The Supervisor is the central nervous system; every other component reports state into it and consumes routing decisions from it.

---

## 5. LLM A vs LLM B: Planning–Execution Separation

Adversarial isolation depends on the planning layer and the execution layer being genuinely independent — different model instances with different system prompts, different memory, different tool surfaces. The separation is enforced by the Supervisor as a routing rule rather than treated as soft convention.

| Property | LLM A — Planning Layer | LLM B — Execution Layer |
|---|---|---|
| **Role** | Architect, reviewer, critic | Implementer, code generator |
| **Tool allowlist** | Read-only against the codebase | Edit / Write / Bash / test runners |
| **System prompt** | Adversarial: identify what is wrong with this proposal | Constructive: implement this approved plan |
| **Memory** | Long-horizon (architecture history, prior decisions, drift map) | Per-task (re-injected; stateless across Epics) |
| **Visibility into the other side** | Sees execution outputs (diffs, test results) | Does not see planning-layer reasoning until LLM A has produced its independent assessment — adversarial isolation suppresses sycophancy |
| **Authority** | Blocking on plan advancement | Blocking on individual task completion only; downstream verification still gates the merge |
| **Concurrency** | Singletons per workstream (one architect at a time) | N parallel pods (one per approved Epic, isolated worktrees) |
| **Failure mode prevented** | Sycophancy (agreement with the smarter-sounding plan) | Context collapse (loss of prior task state) |

The two layers may be different model classes — a frontier model for LLM A's reasoning combined with a faster model for LLM B's code generation — or the same model class with different system prompts and tool surfaces. The separation is in role and routing, not necessarily in model weights.

> **NB: Why two layers, not three or more.** The A/B split is the minimal viable separation that captures most of the benefit of multi-LLM orchestration. Adding additional top-level LLMs (one for testing, one for security, one for cost, etc.) produces diminishing returns and steep orchestration tax for three reasons:
>
> - **Coordination complexity explodes.** Every additional top-level LLM requires shared-context handoff, conflict resolution, and independent monitoring. The Supervisor's routing surface grows combinatorially with the number of distinct LLM "brains."
> - **Cost, latency, and debuggability degrade together.** More LLMs mean more API calls, more tokens, and longer end-to-end latency on every Epic. When a failure occurs, tracing responsibility across four or more independent reasoners becomes materially harder than across two.
> - **Specialization belongs inside the layers, not above them.** The recommended evolution is *more sub-roles within A and B*, not more peers. Inside LLM A, distinct sub-agents handle architecture vs. security vs. cost review — each operating against the same long-horizon memory and adversarial stance, but with focused system prompts. The Supervisor handles dynamic routing (e.g., a security-heavy plan goes to the security sub-role of A; a routine refactor goes to the architecture sub-role). Local-LLM workloads (dev/test scenario runs, per §16) live as routing variants within the existing A/B contract — they are not a third top-level layer.

---

## 6. What changes when building software automatically

Agentic applications fail differently than deterministic ones, and the engineering discipline has to follow. Two properties of the produced artifact change at once, and they compound:

- **The code is stochastic.** Every LLM call produces a sample from a distribution. The same input does not produce the same output. Behavior is shaped by prompt, model, temperature, context, and prior turns — not by a deterministic function of inputs.
- **The flow is emergent.** Agentic applications do not execute a pre-defined state machine. The next step is decided at runtime by the agent's reasoning over the user's input, conversation history, the tool surface, and the agent's policy. The flow exists in the moment of execution; it cannot be enumerated in advance.

Both properties hold simultaneously, and they compound. A traditional bug is a deterministic deviation from a deterministic spec. An agentic bug is a *distribution shift* — the same code, run a thousand times, produces a population of behaviors, and the bug is that some non-trivial fraction of that population is wrong. Such a bug cannot be detected with a unit test; it surfaces only when scenarios are run at scale and the failure rate is measured against a target.

### The implications for engineering discipline

Once the artifact is non-deterministic in this way, the engineering discipline rewrites itself:

- **Simulation is not optional.** Shipping by reading the code and approving the diff is unsound, because the diff does not determine the behavioral distribution. The factory must run a scenario corpus at scale, measure failure rates against benchmarks, and ship only when the rates clear the bar. Self-driving teams already operate this way (millions of simulated miles before any code reaches production); the agent factory adopts the same discipline.
- **The operational design domain is explicit.** Every Epic declares what kinds of inputs the agent is supposed to handle, what kinds it explicitly is not, and how it fails when out of domain. This is a design-time contract enforced at the proposal stage, not a runtime concern alone.
- **Adversarial scenario generation is mandatory.** An adversarial agent generates inputs specifically designed to surface failure modes the original Epic author missed. Without this, the corpus only ever exercises scenarios the author thought of — and the unknown unsafe stays unknown.
- **Behavior pinning replaces unit testing.** A deterministic app pins code paths. An agentic app pins behavioral distributions: "in scenario X, the agent reaches outcome Y with probability ≥ Z." Regressions are detected as distribution drift, not as line-by-line code change.
- **Incident analysis is structured.** Every observed failure becomes a pinned scenario in the corpus — full input context, decision trace, environmental state — so the failure mode cannot return silently.
- **Coherence under parallelism is engineered, not assumed.** Traditional dev teams hit conflicts rarely because their concurrency rate is low; CDID multiplies the rate by agent count and per-agent throughput. Worktree isolation handles independent work but defers overlapping work to merge time, where text-based git merge cannot catch semantic conflicts (a rename + a new caller text-merge cleanly and break at runtime). Coherence is held at three layers: **plan-stage conflict prevention** (every Epic declares its affected surface — files, tables, services, prompt keys — and the Supervisor refuses to parallelize Epics whose surfaces intersect without explicit agent-to-agent negotiation), **commit-time intent metadata** (every commit carries structured tags — function added, function renamed, schema migrated — so semantic merge compares concepts, not lines), and **stale-plan invalidation** (every plan hashes the codebase state it was reasoned against; if the foundation moves underneath the plan, the plan reopens to PPRE rather than executing against assumptions that no longer hold). This is infrastructure work that lives in the Central Factory Supervisor (§11), not a property of any agent prompt — the Supervisor maintains a live work-registry that is the source of truth for what surface is being touched by whom.
- **The substrate must be deterministic where the policy is not.** The LLM registry, prompt library, feature-flag surface, and persistence layer are the *deterministic plinth* on which the stochastic policy and emergent flow rest. Their behavior must be exact and predictable, because everything above them — the model output, the agent's decisions, the user's flow — is not. Without a deterministic substrate, the stochasticity compounds and becomes unmanageable.

### Functional testing: behaviors, not fixed functional activity

The biggest implication for how tests are written: **behaviors are tested, not functional activities.** Traditional functional testing assumes a deterministic spec — given input X, expect output Y, pass/fail per case. That works for `is_even(4) == True`. It does not work for an agent that classifies a customer message into a workflow path. The same message with the same context can resolve to different paths; the right answer is fuzzy; the failure mode is "confidently wrong" rather than "crashed."

A concrete example. An agent renders a project-health widget on a dashboard. The spec says: "display health (red/yellow/green) with a short explanation." The agent honors that contract — but improvises in places the spec didn't lock down. On one render the widget is left-aligned, on the next center-aligned. On one the explanation sits below the indicator, on the next beside it. On one the bullets in the explanation are ordered by recency, on the next by severity, on the third by the order the LLM happened to emit them. The traditional functional test (`widget.color in {red,yellow,green}`, explanation field is non-empty) passes every time. The behavioral test asserts something the unit test cannot express: across 100 renders of the same project state under realistic prompt variants, *layout is stable (same alignment, same vertical placement of indicator vs. explanation), bullet ordering follows a fixed rule, and no explanation is truncated mid-sentence*. The agent operates inside a latitude the spec did not constrain; the behavioral test is the only mechanism that surfaces drift in that latitude before users notice the inconsistency.

Test cases are designed for behavioral distributions: ranges, convergence, absences ("no run produces X"), reference-set consistency — not single-value equality. The Functional / Scenario Modeler (§10.3) is the agent that makes this kind of test tractable to author at velocity. The formal name for the underlying mindset shift in safety-engineering literature is SOTIF (ISO 21448 — Safety of the Intended Functionality), developed by the autonomous-vehicle community to verify intended functionality under stochastic policies. The framework adopts the mindset.

### Why traditional methods break

The failure of traditional methods is direct:

- **Implementation-as-bottleneck assumption breaks.** With agents, implementation is fast. What takes time is *deciding what is worth implementing* (which scenarios matter, which failure modes are acceptable) and *verifying that what was implemented behaves correctly under realistic distributions* (the simulation pass). Methods that optimize the implementation stage produce faster wrong-direction work.
- **Backlog-as-manageable-artifact assumption breaks.** Agents produce plausible scenarios, edge cases, and findings at machine scale. The backlog grows exponentially with the simulation corpus. It must be groomed by machinery, not weekly meetings.
- **Trust-as-implicit assumption breaks.** No artifact in an agentic system is trustworthy by default. Trust is engineered by surviving adversarial review at proposal time, scenario simulation at integration time, and behavior pinning at production time.

The factory is built on the inverses: implementation is cheap, the backlog is a chaotic living entity managed continuously, and no artifact is trusted until it has driven the scenario corpus and held its behavioral distribution.

---

## 7. The headline shift: from CI/CD to CDID

Traditional pipelines describe themselves as Continuous Integration / Continuous Deployment. CI/CD is a *within-change* property: it automates the path from a single commit to production. The agent factory operates a different property: **Concurrent Development, Integration, and Deployment (CDID)**.

CDID means at any given moment:

- Multiple unrelated workstreams are being designed, debated, and replanned in parallel.
- Multiple unrelated implementation pods are generating code in parallel against isolated branches.
- Multiple unrelated artifacts are being independently audited, tested, and benchmarked in parallel.
- Multiple unrelated deployments are being orchestrated and verified in parallel.

CI/CD asks: *how fast can one change reach production?* CDID asks: *how many independent changes can be in different stages of dev, integ, and deploy simultaneously without losing coherence?* The first scales with pipeline speed; the second scales with orchestration quality. The agent factory is built for the second.

---

## 8. Continuous code dosing: how releases change

CDID describes how *many* changes can be in flight at once. **Continuous code dosing** is the parallel concept at the deploy boundary: how each individual change *enters* production. Once releases happen continuously rather than as discrete events, deployments stop being a single shipped artifact and start looking like a continuous infusion of code — many micro-features and small updates flow simultaneously, and "the release" stops being a noun. This reshapes every aspect of how features reach users — rollout pacing, feature-flag discipline, monitoring, attribution, rollback, and incident response — not just one of them.

Five things change:

- **Rollout shifts toward continuous, not exclusively staged.** The traditional release plan (cut a branch, stabilize, RC, GA) still exists for changes whose blast radius warrants it — schema migrations, breaking API contracts, large user-visible reworks. What changes is that the *median* change no longer travels that path: small, low-blast-radius changes ship dark behind a feature flag, ramp to a fraction of users, and expand or roll back based on signal. Feature flags become a *targeted blast-radius mitigation*, not a wholesale replacement for environment promotion. The trap to avoid is flag sprawl — where every change gets a flag, flag debt accumulates, and code paths that should have been deleted live forever. The Feature Flag Agent owns flag lifecycle (creation, ramp, full-on, removal) precisely so dosing does not collapse into permanent indecision.
- **Quick adjustment.** Each dose is small enough that correction or tuning happens on the timescale of minutes, not days. A bad release is recoverable by stopping the drip on a single feature flag, tuning the dose down, and redeploying — without rolling back unrelated work that shipped in the same window.
- **Continuous monitoring is mandatory.** With a single big release, metrics are watched for an hour after deploy and the deploy is then declared done. With code dosing the monitoring *is* the deployment — there is no "after"; metrics are watched always, because there is always a recent dose.
- **Dose-response attribution is harder.** When metric X drifts up 2% over a day, was it the prompt change at T-3h, the new feature flag at T-7h, the bug fix at T-14h, or the migration at T-20h? Single-deploy incident response assumes one suspect; code-dosing incident response means the hypothesis space is the last N changes weighted by blast radius.
- **Side effects accumulate.** Three code doses can each be safe individually but produce a regression in combination. "What changed?" stops being a single-shot question and becomes a layered audit across a sliding window.

Three agent roles carry the operational load: the Observability Agent (§10.6) attributes anomalies to specific recent doses with confidence intervals, the Live Debugging Agent (§10.5) reads code, runtime traces, and change history together at the dose's timescale, and the Feature Flag Agent (§10.6) provides the per-feature dose-tuning surface that makes "stop the drip on this one flag" a one-command operation. Incident response is no longer an event triggered by a release — it is a continuous practice woven into the deployment itself.

> **Aspirational vs. operationally practiced.** None of the five properties above are operationally practiced in most current platforms, even though their prerequisites typically exist. Feature flags exist but the discipline to "stop the drip" on a single flag in minutes does not; the runbook is missing. Observability exists but not auto-attribution of anomalies to recent doses with confidence intervals. Change history exists (git) but is not consumed as a dose-weighted hypothesis-space ranking when an incident fires. The framework's value lies precisely in formalizing and automating these properties — not in asserting they are already met. Implementation-state tracking is deferred to platform-specific companion documents.

---

## 9. The four disciplines

Concurrency without discipline is chaos. Four named disciplines convert raw parallelism into trustworthy throughput. They are mandatory, not aspirational, and they are enforced as global invariants by the Central Factory Supervisor.

### 9.1 Plan → Pushback → Revisit → Execute (PPRE)

PPRE is the factory's central provocation. The four words look procedural, but each names a *failure mode* the discipline exists to prevent. Every non-trivial workstream passes through the cycle before any code is written:

- **Plan — distributed, not authored.** In a human team, "plan" is a single architect's document. In the factory, planning is a distributed act: Backlog, Epic Designer, Architecture Review, and Costing contribute parallel inputs to a structured proposal — diagnosis, proposed change, affected surface, verification strategy, blast radius, out-of-scope list, surface claim for the work registry. The plan is the consensus artifact of distributed planning, not one agent's draft. The failure mode it prevents: *single-author planning*, where one model's framing locks in unexamined assumptions before pushback can land.
- **Pushback — adversarial isolation, not collegial review.** An independent reviewer (LLM A planning layer, Safety Audit, an external pass, or the human) critiques the plan in a fresh session *without* seeing the author's reasoning trace. Adversarial isolation is what prevents the reviewer from agreeing with whichever framing reads as smarter. Pushback surfaces hidden assumptions, mis-scoped lifts, missing edge cases, and architectural ripple effects while changing direction is still cheap. The failure mode it prevents: *sycophantic review*, where the reviewer agrees with a plausible-sounding plan instead of evaluating it.
- **Revisit — converge, do not hesitate.** Revisit is the discipline of taking pushback seriously and *moving*: revise the plan, or defend the original with new evidence, then advance. The failure mode it prevents is the inverse of pushback's: *hesitation* — accepting all pushback unconditionally (sycophancy at the author layer), looping on the same critique, or freezing the workstream entirely. The Supervisor monitors revisit cycles and forces escalation if the plan does not converge after a bounded number of rounds. Revisit ≠ hesitate; revisit ends.
- **Execute — parallel across stages, not only coding.** "Execute" is the end-to-end realization of the approved plan, not the act of typing code. It spans four stages — *implementation* (multiple coding pods on isolated worktrees generating against the spec), *verification* (scenario-corpus runs, behavior-pin synthesis, security-audit replay), *deployment* (dose ramp, feature-flag tuning, rollout-phase gates), and *post-merge audit* (Reflection Loop). Each stage runs in parallel where dependencies allow, and the Supervisor reconciles results across stages (semantic merge, pin verification, deploy gates) before the plan is declared complete. Every commit, test outcome, audit finding, and deploy event traces back to the approved plan; mid-flight deviations at any stage reopen PPRE rather than being silently absorbed. The failure mode it prevents: *serial execution at any stage* — coding pods running one at a time, the scenario corpus blocked behind a single runner, deploys waiting for the previous one to finish, post-merge audit run only weekly — any of which collapses CDID's parallelism at the corresponding stage. (CDID describes cross-Epic concurrency; Execute describes within-Epic stage parallelism. Both are real and both are required.)

PPRE catches errors at the cheapest layer (the proposal) instead of the most expensive (production). It is the inverse of "fail fast through working code"; it is "fail at the proposal so working code is never wrong in the first place." The four named failure modes — single-author planning, sycophantic review, hesitation in revisit, serial execution — are each a distinct way the cycle collapses without the named guardrail.

**Worked example.** A user-visible bug surfaces — repeated failed save attempts on an option-picker UI, with the system re-prompting the user with the identical option list after each selection, no progress, no error message that points at the cause. The cycle:

- **Plan.** A diagnostic agent inspects the conversation log, observes the user repeatedly selecting an option from the same list, observes the system rejecting each selection with the identical re-prompt, and proposes: *the user's tenant has zero records in the option lookup table; the picker is reading from a global catalog while the validator scopes its check to the tenant.* Plan: seed the tenant's lookup table or relax the validator's scope.
- **Pushback.** An independent reviewer rejects the plan: *"the picker is supposed to read from the platform catalog and the validator is supposed to accept anything from that catalog — the cross-tenant scope is the design, not the bug. Something else is broken."* The plan does not advance.
- **Revisit.** The original diagnoser re-reads the validator code in light of the pushback and finds the gate: `if effective_id == 'PLACEHOLDER' or effective_id == '': reject`. The placeholder sentinel happens to be the *real id* of one legitimate option in the catalog — so any user picking that option is rejected as if they had not picked anything. The cycle is forced by id collision, not by tenant scope.
- **Execute.** A targeted gate change plus a tracking flag for explicit user resolution; backwards-compatible with in-flight state from the old code.

Without the Pushback step, the original (wrong) diagnosis would have shipped. The fix would still have stopped the user-visible loop — but the *stated reason* would have hidden the actual root cause and left the placeholder collision unaddressed in any structurally similar gate elsewhere.

> **Implementation:** a `/ppre-plan` skill that orchestrates distributed planning across reviewer subagents in parallel, invokes adversarial reviewers in fresh sessions (no parent context), enforces a bounded-revisit convergence policy, and parallelizes execution across pod-isolated worktrees. A PreToolUse hook on Edit/Write refuses code-modifying operations unless an approved plan file with a survived-pushback marker exists for the workstream.

### 9.2 Assume-compromise (system-wide posture)

Where PPRE is a gate, assume-compromise is a stance — applied wherever an artifact is produced or evaluated, not only at plan time. The posture operates at four layers:

- **Proposal layer** — the Architecture Review Agent (LLM A) and Safety & Security Audit Agent adversarially review every plan. Findings are blocking; a plan with unresolved findings does not advance.
- **Implementation layer** — every change ships with a behavior pin (§9.3). The merge gate verifies the pin exists and passes. The pin codifies *why* the fix is correct, not just that it is.
- **Runtime layer** — the Reality-Anchor (Watcher) monitors active agent sessions for drift, hallucinated artifacts, and sycophantic agreement against pinned facts; intervenes mid-execution.
- **Post-merge layer** — Reflection Loop agents re-audit merged work daily, looking for sycophancy and context-collapse failures that escaped earlier gates.

PPRE's Pushback step is one instance the posture mandates; the runtime Watcher, the merge gate, and the post-merge audit are the others.

> **Implementation:** read-only reviewer subagents (`architect-reviewer`, `safety-auditor`) invoked in fresh sessions without the author's reasoning trace — adversarial isolation prevents sycophancy. Their findings are appended to the plan file. The Supervisor enforces the four-layer posture by refusing to advance state when any layer's verification has not run.

### 9.3 Behavior pinning via regression tests

Every change ships with regression tests that pin the *shape of the contract*, not just the absence of the immediate bug. A bug fix that ships without a test pinning *why* the fix is correct is incomplete and the Supervisor blocks the merge.

This produces a living, ever-growing corpus of behavioral pins — the system's institutional memory expressed as executable assertions. Each pin documents:

- The original failure mode, in prose, at the top of the test
- The architectural assumption the fix encoded
- The trigger conditions that would surface the regression

The corpus serves as both a test suite and a design-review record. Future agents reading the corpus reconstruct *why* a path was chosen, not just *that* it exists.

> **Implementation:** a `/behavior-pin` skill drafts a regression test for a given fix description before the fix lands. A PostToolUse hook on Edit/Write that touches service-layer code requires a corresponding test in the regression suite in the same commit; merge is blocked otherwise.

### 9.4 Symptom-to-root, with codebase-wide invariant audit

A small symptom (a UI element rendering wrong, a single failed health-check, a one-row data anomaly) is treated as the entry point to a potential architectural finding, not as a target for patching. The first response to any reported symptom is to trace it through the layers — UI → API → service → data model → schema — until the actual invariant being violated is identified.

A patch that addresses the symptom without identifying the root invariant is rejected. The artifact that ships is the root fix; the symptom disappears as a consequence. The corpus of past symptom-to-root traces becomes input to the Architecture Review Agent's continuous audit.

**Codebase-wide invariant audit (mandatory).** Once the root invariant is identified, the discipline mandates a codebase-wide search for *other* sites that violate the same invariant. Stochastic code generation produces partial fixes by default — the model patches the site it was shown and confidently leaves identical violations elsewhere untouched. Examples observed in practice: a missing `tenant_id` filter is fixed on one query while three peer queries on adjacent tables still leak; a deprecated helper is replaced in one router and left in eight others; a sanitization step is added to one input path and not to a structurally identical path one module over. Without an explicit audit-all-sites step, the framework accepts partial fixes and ships systemic inconsistency.

The discipline therefore requires four steps, not two:

1. **Trace** the symptom to the root invariant being violated.
2. **Find all sites** that could violate the same invariant (codebase grep + structural search + agent-driven semantic search).
3. **Fix all sites in the same workstream** (or open structured backlog items for sites whose remediation is out of scope, with the original Epic linking them so they cannot be lost).
4. **Pin the invariant globally** with a regression test or static check that detects future violations regardless of which file introduces them — turning a one-time fix into a structural guard.

Step 4 is what converts "fix the bug" into "make the bug class impossible." The pin lives in the regression corpus alongside the per-fix pins from §9.3, but its scope is the invariant rather than the original symptom site. The Hygiene Agent (§10.5) consumes the resulting global-invariant set for proactive scans on its nightly sweep — finding violations of the invariant before any user reports a symptom.

**Worked example.** A user reports a 500 on a results-rendering page. The upstream API surfaces a generic "engine error" with no detail. Walking the four steps:

1. **Trace.** API logs show a gRPC INTERNAL response from a downstream numeric-processing service. The downstream service's logs surface the actual exception: `AttributeError: 'numpy.float64' object has no attribute 'isna'` at `service.py:78`. The line: `per_unit = pd.to_numeric(table.get('cost'), errors='coerce'); fallback_mask = per_unit.isna() | (per_unit <= 0)`.
2. **Root invariant.** When the column is absent, `table.get('cost')` returns `None`; `pd.to_numeric(None, errors='coerce')` returns a *scalar* `numpy.float64` nan, not a Series. The scalar has no `.isna()` method. The invariant violated: *every conditional-column read in the service must defend against absence — either by guarding with `if col in table.columns` or by passing an explicit `pd.Series(default, index=table.index)` as the `.get()` default. The `.isna()` step assumes a Series; the code path that produces a scalar is not handled.*
3. **Find all sites.** Codebase-wide search across the service for `pd.to_numeric(<frame>.get(`. Four call sites total. Three already use the explicit-Series-default pattern. One does not — the unguarded site is the bug.
4. **Pin globally.** A regression test that synthesizes an input frame *without* the optional column and asserts every public callable in the service handles it without `AttributeError`. The pin's scope is the invariant ("conditional-column reads must not assume Series type"), not the original symptom site. A future call site that introduces the unguarded pattern fails the pin even if the original site was never touched.

Step 3 is what the framework adds beyond a traditional symptom→fix loop: in stochastic code generation, *partial fixes* — one site patched, identical violations elsewhere left untouched — are the default failure mode rather than the exception, and an explicit audit-all-sites step is the only thing that closes them. Step 4 is what makes the fix structural rather than local.

> **Implementation:** a `/symptom-to-root` skill that, given a reported symptom, (1) traces it through the codebase layer-by-layer to a root-cause finding, (2) issues a codebase-wide search for structurally similar violation sites, and (3) emits a list of all sites plus a proposed global pin into the PPRE plan. The plan cannot advance until the human or Architecture Review Agent has either confirmed all sites are in scope or explicitly deferred specific sites with linked backlog items. No silent partial fixes.

---

## 10. Agents by lifecycle bucket

Eighteen agent roles compose the factory. Bucketing them by lifecycle stage — when in the production process they operate — makes the cast tractable to reason about. Each agent is independently instantiated; each bucket is a coordination boundary.

### 10.1 Planning & Specification (pre-implementation)

- **Backlog & Prioritization Agent** — continuously ingests, ranks, and grooms the exponentially growing backlog using hybrid LLM access (planning model for ranking, execution model for refinement). Outputs a strategically ordered, deduplicated, scored work queue. Deletes obsolete items. Surfaces patterns to the Architecture Review Agent (e.g., "five backlog items from this month all touch the same architectural seam").
- **Epic Designer Agent** — transforms prioritized backlog items into precise, verifiable specifications: scope statement, success criteria, affected surface, test plan, rollback plan, blast-radius assessment. The Epic spec is the input to PPRE. An Epic that cannot be specified to verifiable form is returned to the Backlog Agent for reformulation.
- **Architecture Review Agent (LLM A — Planning Layer)** — performs independent, continuous architectural validation. Serves as the primary adversarial reviewer in PPRE. Detects drift between intent (the plan) and direction (where the codebase is heading). Maintains the architectural map and flags proposals that would violate established invariants. Authority is blocking on plan advancement.
- **Feature & Module Governance Agent** — enforces tiered conformance rules and modular boundaries. Decides whether a change belongs in a service, a router, or a library. Owns the dependency graph and prevents cross-cutting violations. Pairs with Architecture Review on plan-stage decisions; runs continuous audits at implementation time to catch boundary violations as they form.

### 10.2 Execution (implementation)

- **Automatic Coding Agent Factory (LLM B — Execution Layer)** — operates N parallel coding pods. Each pod implements one approved Epic on an isolated branch against the test plan defined in the Epic. Pods are stateless across Epics; persistent context is managed by the Supervisor and re-injected per task. Multiple pods may work concurrently on unrelated Epics. Each pod generates code, runs the local test suite, and surfaces the diff for downstream review.

### 10.3 Verification (pre-merge gate)

- **Testing Strategy & Execution Orchestrator** — coordinates four sub-systems: the **Functional / Scenario Modeler** (extracts live user flows from product and PR diffs, organizes them into logical execution sequences — state, control points, maneuvers, transitions — and synthesizes enriched route-graph scenarios with UI state, business state, waits, nondeterminism, execution profiles, and settle conditions; scenarios are directly mappable to browser-driver harnesses or executed natively); the **unit / regression suite** (owns the behavior-pinning corpus and the merge gate that requires every fix to ship with its pin); the **end-to-end harness** (full-stack scenario execution against staging); and **adversarial test generation** (uses an LLM to generate edge cases the original author missed).

  > The Functional / Scenario Modeler is the load-bearing piece for distribution-shift testing — it converts the SOTIF-style requirement of "exercise the operational design domain at coverage X% with residual unknown-unsafe estimated at Y" into concrete, replayable scenarios that the verification machinery can run at scale.

- **Safety & Security Audit Agent** — conducts relentless adversarial auditing across all artifacts: static analysis, dynamic analysis, prompt integrity checks, schema integrity checks, secret-leak detection, dependency CVE monitoring. Engaged from the first moment of every change, not as a downstream gate. Authority is blocking on production deployment.
- **Costing Analysis Agent** — provides real-time economic and risk assessment of proposed work. For every Epic: estimates token cost, infrastructure cost, deploy risk, blast radius, opportunity cost. Surfaces "this Epic is larger in scope than the originating request suggests" before PPRE advances to execution. Authority is advisory; the human or Supervisor decides whether the cost is acceptable. Consumes the workload-attribution split (§16) to estimate per-environment LLM spend.

### 10.4 Coherence (active during multi-pod parallelism)

- **Conflict Prediction Agent** — reads in-flight Epic specs from the Supervisor's work registry and predicts surface intersections before they materialize. Fires at PPRE-approve time; if a new Epic's affected surface intersects with active claims, the Supervisor consults this agent's output before deciding sequence, split, or negotiate.
- **Epic Decomposer Agent** — invoked when the Supervisor decides an intersecting Epic should be split. Reads the Epic spec, identifies surface-disjoint sub-Epics, and emits PPRE-ready specs for each. Each sub-Epic enters PPRE independently with its own surface claim.
- **Semantic Merge Agent** — reads common ancestor, branch A, branch B, and each branch's intent metadata, then produces a merged result that preserves both intents. Catches semantic conflicts — rename plus new caller, schema migration plus new query — that text-based merge misses. Escalates to human only when the intents genuinely contradict.

### 10.5 Runtime Stabilization (continuous; during execution and on the codebase)

- **Reality-Anchor Agent (Watcher)** — runtime stabilizer that monitors active agent sessions and intervenes when the agent drifts. Detection signals: repetition loops, contradiction with prior turns in the same session, hallucinated artifacts (claimed file, function, or flag does not exist), drift from the task spec, sycophantic agreement that contradicts pinned facts. Authority: interjects with grounding prompts ("re-read the spec"; "verify this file exists"; "you stated X earlier; reconcile"), pauses execution, escalates to the human if the drift does not resolve. Distinct from reviewer agents (plan stage) and reflection-loop agents (post-completion audit); the Watcher operates *during* execution.
- **Live Debugging Agent** — reads precise code blocks, runtime traces, recent diffs, and observed behavior together to produce a structured root-cause hypothesis when something breaks in production or in dev. Triggered by error reports, observability anomalies, or user-submitted failure signals. Output is either a backlog item with full context or a direct PPRE plan input. Synthesizes across code, runtime, and history at once where traditional debuggers show one slice at a time.
- **Hygiene Agent** — continuously sweeps the codebase for *architectural slop*: dead code, unused exports, near-duplicate helpers, inconsistent naming across similar modules, accreted complexity that works but is not load-bearing. Also runs the proactive form of the §9.4 codebase-wide invariant audit: nightly, scans every site against the global-invariant set produced by past symptom-to-root traces (e.g., "every database query filters on tenant_id," "every LLM call routes through the registry"). Detects violations introduced after the original audit and opens backlog items so they reach a coding pod before a user surfaces the symptom. Authority is advisory; opens backlog items rather than blocking, since most slop is not urgent and bundling it carelessly can produce worse churn than the slop itself. Addresses two failure modes: the last-mile architectural slop in agent-generated code, and the partial-fix problem in which a stochastic patch covered one site and missed peers that violate the same invariant.

### 10.6 Operational Plumbing (pipelines, telemetry, runtime config, post-merge audit)

- **CI/CD Automation Agent** — owns pipeline definitions; ensures every change has a build, migration, and deploy path. Pairs with Safety & Security Audit on the deploy gate.
- **Observability Agent** — real-time production monitoring with feedback loops back into the Backlog Agent. Surfaces anomalies as new backlog items, anchored to traces. Attributes anomalies to specific recent code doses (§8) with confidence intervals.
- **Feature Flag & Runtime Configuration Agent** — manages bundled rollouts, tiered access, prompt libraries, guardrails, and parameterization. First-class versioned configuration surfaces. Provides the per-feature dose-tuning surface that makes "stop the drip on this one flag" a one-command operation.
- **Adversarial / Reflection Loop Agents** — cross-cutting agents that run LLM A ↔ LLM B double-checks on completed artifacts before they reach production. Catches sycophancy and context-collapse failures that escaped the proposal-stage review.

### Cross-cutting protocol: agent-to-agent negotiation

The Coherence layer's resolution paths (sequence, split, negotiate) require agents to communicate with one another at a structured-protocol level rather than as free-form chat. Implementation pattern: a small set of typed tools — `query_active_claims`, `propose_intent`, `respond_to_intersection`, `request_handoff` — invoked by agents against one another with the Supervisor as message broker. The Supervisor enforces message ordering, timeouts, and escalation. This converts negotiation from informal coordination into a verifiable protocol with audit trail.

---

## 11. The Central Factory Supervisor

The Supervisor is the central nervous system. It is not a coding agent and never writes implementation code. Its responsibilities:

- **Real-time orchestration** — routes tasks to the right agent at the right time, manages the dependency graph between agents, ensures parallelism without conflicts.
- **Multi-LLM context synchronization** — maintains shared context across LLM A (planning) and LLM B (execution) instances, prevents context drift between them.
- **Exception handling** — when an agent fails, the Supervisor decides whether to retry, escalate to a different agent, or escalate to the human.
- **Global invariant enforcement** — encodes the four disciplines as enforceable rules. A plan without survived pushback cannot advance to execution. A fix without a behavior pin cannot merge. A symptom without a root trace cannot ship as a patch.
- **Coherence enforcement (the work registry)** — the Supervisor owns a Postgres-backed *work registry* that is the source of truth for which Epic is touching which surface. Every plan declares its affected surface at PPRE-approve time and is registered as an active claim. New Epics consult the registry before advancing; surface intersections trigger one of three resolution paths: **sequence** (one Epic waits), **split** (the Supervisor invokes a decomposer agent that breaks an Epic into surface-disjoint sub-Epics), or **negotiate** (the two agents exchange intent and converge on a shared resolution under a hard timeout). Stale-plan invalidation runs against the registry too: every plan carries a base SHA, and if the registry shows the surface has moved underneath, the plan reopens to PPRE rather than executing against stale assumptions.
- **Assurance** — guarantees no artifact or decision is lost in the parallel execution graph. Every plan, pushback, revision, commit, test result, audit finding, and deploy event is durable and traceable.
- **Human escalation** — the Supervisor recognizes when a decision exceeds its authority (architectural inflection, strategic priority, ambiguous intent, unresolved coherence negotiation) and escalates to the human coordinator with a complete context package.

**The work-registry record (concrete shape).** The Supervisor's authority rests on the work registry — every Epic in flight is one row, persisted, queryable, and the source of truth for surface claims, status transitions, and base-SHA staleness. A registry record looks like this:

```json
{
  "epic_id": "epic_2026_05_04_a3f9",
  "title": "Refactor session lifecycle for multi-tenant isolation",
  "status": "ppre_approved",
  "base_sha": "a8c1f3d2e9b4...",
  "created_at": "2026-05-04T13:08:22Z",
  "ppre": {
    "plan_path": ".plans/epic_2026_05_04_a3f9.approved.md",
    "pushback_findings": [],
    "pushback_resolved_at": "2026-05-04T14:23:11Z",
    "revisit_rounds": 2
  },
  "surface_claim": {
    "files":   ["api/services/session.py", "api/routers/auth.py"],
    "tables":  ["sessions", "session_events"],
    "services":["api"],
    "prompt_keys": []
  },
  "blast_radius": "medium",
  "owners": ["pod_3"],
  "execute": {
    "pin_required": true,
    "pin_path": "regression_suite/test_session_lifecycle.py",
    "pin_status": "drafted",
    "pods_active": ["pod_3"],
    "deploy_phase": null
  },
  "next_action": "execute"
}
```

Status transitions are predicates the daemon evaluates: `draft → ppre_approved` requires `pushback_findings = []` and `revisit_rounds ≤ MAX_ROUNDS`; `ppre_approved → executing` requires no surface intersection with another active claim (or a documented sequence/split/negotiate resolution); `executing → mergeable` requires `pin_status = "passing"`; `merged → closed` requires `deploy_phase = "completed"` and an Observability Agent attribution window with no unattributed anomalies. Stale-plan invalidation: if `base_sha` is no longer reachable from `main` because another Epic merged a touching commit, the record reverts to `draft` and PPRE reopens.

The schema is intentionally small. Most coordination work the Supervisor does is consulting and updating this record — the rest of its complexity is event routing, not data model.

> **Implementation:** the Supervisor is a long-running daemon (not a chat session) built on top of an agent SDK. It receives events from a queue (Postgres `LISTEN/NOTIFY` or a thin webhook receiver), routes them to subagents via the SDK's programmatic API, and persists every decision to a structured audit table. Global invariants are predicates the daemon evaluates before transitioning state. The daemon is the one piece of infrastructure most agent SDKs do not provide out of the box; it has to be built.

---

## 12. Integration with the layered safety framework

The factory is an operational extension of the layered agent safety architecture. Specifically:

- **Planning vs. execution separation** — LLM A and LLM B are different model instances with different system prompts, different memory, and adversarial visibility into each other's output. This is not a soft convention; it is enforced by the Supervisor's routing.
- **Persistent sessions where appropriate** — agents that require long-horizon context (Architecture Review, Backlog) maintain persistent state. Agents that benefit from a clean slate (coding pods) are stateless per-task with context re-injected.
- **Adversarial isolation** — reviewers cannot see the author's reasoning trace until they have produced their own independent assessment. This mitigates sycophancy and "agreeing with the smarter-sounding plan."
- **Continuous backlog and architecture auditing** — not periodic, not on demand. The respective agents run continuously and surface findings as they form.
- **Phased findings-driven feedback loops** — findings from any layer (production observability, safety audit, architecture drift) loop back into the Backlog Agent as new prioritized items.
- **Runtime-configurable guardrails** — the safety surface itself is a versioned, runtime-adjustable configuration, owned by the Feature Flag & Runtime Configuration Agent.

The human founder remains in the central coordination role.

---

## 13. Radical differences from traditional methods

The framework's commitments differ from traditional Agile and Waterfall practice along sixteen distinct dimensions. The contrasts are not incremental refinements; in most rows the traditional and agentic approaches answer entirely different questions, because the underlying production model has changed — stochastic artifacts, emergent flow, machine-paced cadence, and engineered (rather than implicit) trust each rewrite a different assumption that traditional methods rest on.

The table below is a reference card. Each row names the dimension, the traditional default, the framework's commitment, and the enforcing mechanism that makes the commitment operative. The rightmost column is the load-bearing one: without an enforcing mechanism, every commitment is aspirational. §14 follows with a different and more useful comparison for an audience already working in agentic tools — against existing open-source agent frameworks (LangGraph, CrewAI, AutoGen, MetaGPT).

| # | Dimension | Traditional Agile / Waterfall | Agentic Safety Plus: Engineering | Enforcing mechanism |
|---|---|---|---|---|
| 1 | **Cadence** | Human-scale iteration cycles | Machine-scale continuous flow with hourly production releases | Supervisor scheduling + CDID parallelism |
| 2 | **Execution Model** | Strictly sequential hand-offs | Architecture review, security audit, scenario modeling, code generation, CI/CD all execute concurrently | Specialized LLM roles + parallel coding pods + Supervisor-managed shared context |
| 3 | **Planning Discipline** | Plan once, then build; mid-flight changes are friction | PPRE cycle is mandatory before any non-trivial code | Adversarial reviewer agents block advancement of unresolved plans |
| 4 | **Adversarial Review** | Code review *after* implementation | Most aggressive review at the *proposal* stage; post-implementation review is a backstop | LLM A planning-layer review + Safety Audit Agent + external-review relay |
| 5 | **Trust & Verification** | Implicit trust until a gate intervenes | Engineered lack of trust at every layer; multi-source adversarial verification, behavior pinning, post-deploy assertions | Multi-source verification chain |
| 6 | **Behavior Pinning** | Tests verify code paths; institutional knowledge is tribal | Every change ships with a regression test pinning the shape of the contract | Supervisor-enforced merge gate |
| 7 | **Symptom-to-Root** | Patches address the immediate symptom | A symptom is the entry point to a root finding; symptom-only patches rejected | Architecture Review traces symptoms layer-by-layer |
| 8 | **Backlog Dynamics** | Periodic grooming of a manageable artifact | Exponentially-growing living entity; continuous machine-scale grooming | Dedicated Backlog Agent with hybrid LLM access |
| 9 | **Architecture Governance** | Periodic reviews; drift accumulates between them | Continuous architectural validation; drift is detected as it forms | Architecture Review Agent running continuously |
| 10 | **Functional Specification** | Manual test authoring; brittle scripts | AI-native simulation: live user flows extracted and synthesized into route-graph scenarios | Functional / Scenario Modeler subagent |
| 11 | **Behavior Configuration** | Static, compiled logic | Highly parameterized, runtime-governed: feature flags, prompt libraries, guardrails, configuration are first-class | Dedicated Feature Flag & Runtime Configuration Agent |
| 12 | **Security & Pipeline** | Downstream gates; security as an add-on | Upfront, always-on, continuous static + dynamic analysis | Safety Audit Agent + CI/CD Agent engaged from the first commit |
| 13 | **Orchestration** | Ad-hoc, single-threaded coordination | Central Factory Supervisor enforces global invariants and shared context | Supervisor sits above all agents |
| 14 | **Concurrency Property** | CI/CD: optimize one change's path to production | CDID: many independent changes coexist in different stages of dev, integ, and deploy in parallel | Supervisor + isolation primitives + adversarial verification |
| 15 | **Human Role** | Primary implementer and reviewer | Functional architect and thread coordinator; intervenes at critical decision boundaries | Built-in assurance handles routine; human handles judgment |
| 16 | **Safety Foundation** | Ad-hoc or external | Built directly on the layered agent safety framework | Safety is structural, not a wrapper |

---

## 14. Gap analysis vs. incumbent agent frameworks

The §13 table contrasts the framework with traditional methods, but the more useful comparison for an audience already living in agentic tools is against the existing open-source agent frameworks: MetaGPT, AgentFactory, Gas Town, LangGraph / CrewAI / AutoGen. Most of those tools optimize for *demonstrating multi-agent coordination*. They do not optimize for *engineering trustworthy software at sustained velocity*.

The fourth column is the practical question: would an implementation built on a modern agent SDK + a tool-protocol layer (e.g., MCP) close the gap? Most yes, two require building thin infrastructure on top of the SDK.

| Aspect | Traditional Agile / Waterfall | Existing open-source agent tools (MetaGPT, AgentFactory, LangGraph/CrewAI, AutoGen) | Agentic Safety Plus: Engineering | Closeable with a modern agent SDK + tool-protocol layer? |
|---|---|---|---|---|
| **Execution Model** | Strictly sequential stages | Mostly linear or limited parallelism; graph-driven but typically single-thread | Massively parallel, continuous flow with dedicated orchestration | **Yes** — backgrounded subagents + worktree isolation |
| **Velocity / Cadence** | Human-paced (weekly sprints) | Moderate acceleration; demo-friendly, not production-throughput | Engineered for hourly production releases | **Yes** — but cadence is an emergent property of the rest |
| **Trust & Verification** | Implicit trust + manual review | Basic self-critique or limited cross-checking; no formal adversarial isolation | Relentless adversarial auditing + multi-LLM cross-checking + behavior pinning | **Yes** — adversarial isolation by spawning reviewer subagents in fresh sessions without parent context |
| **Backlog Management** | Periodic human grooming | Basic pipeline or queue management; no continuous re-ranking | Dedicated continuous intelligent grooming for explosive growth | **Yes** — cron + memory + a long-running backlog subagent |
| **Architecture Governance** | Periodic reviews | Basic or absent | Dedicated, continuous, independent Architecture Review Agent | **Yes** — cron-driven background subagent with adversarial system prompt |
| **Security Discipline** | Downstream gates | Basic or add-on | Upfront + continuous Safety & Security Audit Agent | **Yes** — `safety-auditor` subagent + tool-server adapters for static/dynamic analysis tools |
| **Runtime Behavior Control** | Static code + manual feature flags | None native — agent behavior is hardcoded in graph structure | Dedicated Feature Flag & Runtime Configuration Agent (prompts, guardrails, parameterization) | **Yes via substrate** — the SDK does not ship feature flags, but the produced app's substrate provides them; the framework just reads them |
| **Orchestration & Assurance** | Human-driven coordination | Graph or crew-based (no central assurance layer; no global invariants) | Central Factory Supervisor + full observability | **Partial** — the Supervisor must be built as a custom daemon driving the SDK; SDK gives the primitives but not the daemon |
| **Cross-LLM Context Coherence** | N/A | Limited; usually single-model or shallow handoff | Persistent shared context across LLM A and LLM B with conflict resolution | **Partial** — memory + SDK subagents handle some of it; true cross-session shared context needs a Postgres-backed shared store the Supervisor manages |
| **Sycophancy Mitigation** | N/A | Reviewer sees author's reasoning; agreement is the easy path | Adversarial isolation: reviewers do not see author's reasoning trace until they have produced their own | **Yes** — enforced by spawning reviewer subagents in fresh sessions without parent context |
| **Distribution-Shift Testing** | N/A | Run flow once, check output; no scenario-corpus replay | Behavior pinning at distribution level ("scenario X reaches outcome Y with probability ≥ Z") | **Yes** — subagent runs the same scenario N times, accumulates outcome distribution, reports against a pinned threshold |
| **Webhook-Driven Orchestration** | Manual triggers | Manual triggers; some have CLI or HTTP front-ends | Webhook-driven (CI events, observability alerts, deploy events) | **No, requires extra infra** — a thin webhook receiver translating events into Supervisor invocations is needed |
| **Human Role** | Primary builder / reviewer | Mostly supervisory or absent | Functional architect + thread coordinator with explicit visibility | **Yes** — modern SDKs preserve human-in-the-loop via approval gates by default |
| **Safety Foundation** | Ad-hoc or external | Limited; no layered model | Built directly on the layered agent safety framework with adversarial loops | **Yes** — SDK + the platform's existing safety layer (LLM Registry, prompt guarding, rate limiting) |

The two "Partial / No" rows — Supervisor daemon and webhook orchestration — are the only items that require building infrastructure beyond the SDK. Both are tractable: the Supervisor is ~2-3 weeks for an MVP; the webhook receiver is a thin service. Nothing requires a fundamentally new SDK capability.

---

## 15. Why the framework persists even with infinite context

A reasonable reaction to all this: "won't most of this become unnecessary when one LLM can hold the whole codebase in context?" Some mechanisms simplify — context-shuffling between agents, cross-agent context synchronization, the Supervisor's role as context-router. But the disciplines themselves do not depend on context-window constraints; they depend on properties of the policy and properties of accountability:

- **Stochasticity remains.** Infinite context does not fix sycophancy, hallucination, or distribution drift. The policy still samples from a distribution; the verification still has to characterize the distribution.
- **Parallelism still beats serial work.** One giant LLM working serially through the backlog does not match many specialized agents working in parallel against isolated worktrees, no matter how big its context.
- **Adversarial isolation is independent of context size.** A single LLM critiquing its own work has the same sycophancy problem regardless of how much it can read. Independent reviewer instances are still required.
- **Verified execution is still required.** The LLM saying it did X is not the same as X being done. Behavior pins, post-deploy assertions, and the Supervisor's audit trail exist to verify, not to remember.
- **Human-interpretable accountability is independent of the model.** Release date, quality, blast radius, residual risk — these KPIs require a structured production process regardless of how capable the underlying model becomes.

Even an infinite-context LLM has to be **agentified** to be accountable: tools, gates, KPIs, audit. The framework is about engineering discipline for stochastic policies under human-interpretable accountability — not about working around context limits. As models grow more capable, the framework simplifies in places (less plumbing) but does not become unnecessary; it becomes easier to implement.

---

## 16. Workload attribution: where the LLMs run

A consequence of the framework operating on apps that themselves use LLMs: there are at least three distinct LLM workloads in any given system, and each can be attributed independently to a different provider, model, and inference placement. Treating them as one obscures real cost, reliability, and compliance decisions.

| Workload | What it is | Today | Decisions worth making |
|---|---|---|---|
| **Build-time** | The framework's own agents calling LLMs (Architect Reviewer, Coding Pod, Safety Auditor, etc.) | Cloud frontier model | Quality matters more than cost; reasoning agents need top-tier models. Local LLMs are typically not strong enough for the planning-layer roles. |
| **Runtime** | The produced app's user-facing LLM calls in production | Cloud, mediated by the substrate's LLM Registry | Per-service-key placement: data residency, latency budget, regulatory scope. May be cloud, on-prem, or hybrid. Edge unlikely for server-side apps. |
| **Dev/test** | The produced app exercised in dev or by the simulation/scenario corpus | Today: hits the same public LLM as production | Local LLM (Ollama, vLLM) for cost-controlled scenario runs and reproducible regression suites. The scenario simulator — finally cheap enough to run thousands of scenarios. |

The third row is the most underused decoupling. Per the §6 thesis, behavior pinning at scale requires running the scenario corpus *often* — every PR, every nightly. Hitting the production model class for every run produces cost that does not scale and rate limits that throttle the corpus. A local LLM in dev (sized to match the production model class as closely as the hardware allows) lets the scenario corpus run continuously without burning tokens or touching production capacity. The behavioral distributions captured against the local model will not be byte-identical to production, but they are a useful regression signal — drift in either model surfaces immediately.

Workload attribution is a Costing Agent (§10.3) input. Every Epic's plan should declare which workloads it touches and which provider/placement combination is acceptable for each. The decisions tie back to the substrate's LLM Registry: the Registry abstracts these choices away from call sites, but they have to be made deliberately at the architecture level.

> **Implementation:** the LLM Registry already accepts a service-key per call. Adding an `environment` axis (production / staging / dev / test) lets a single service-key resolve to different providers per environment. The Costing Agent reads from the LLM Registry's audit log, attributing every token spend to its workload bucket.

---

## 17. Our Role (Us the People)

The factory's assurance machinery handles routine; humans handle judgment, relationships, and the irreducibly human work that no agent can substitute for. Implementation work is delegated; the rest is not. The roles below are not an exhaustive list — they begin to surface what matters, and the list is expected to grow as the framework is exercised in practice.

### Strategic and architectural authority

- **Strategic direction.** What the product should be, what the priorities are, what is worth building, when to add or retire agent roles.
- **Functional architect.** System design of the factory itself: how agents interact, what invariants the Supervisor enforces, where the gates sit, how new disciplines are added.
- **Architectural inflection points.** When an agent surfaces a finding that the proposed work is materially larger than the originating request implied, the human decides whether to take the larger path.
- **Coding leadership.** The human owns the technical direction even when not writing the code. Pattern decisions, idiom selection, "this is how we do it here" guidance flow from the human to the agents through invariants, system prompts, and architectural pins.
- **Final judgment on adversarial conflicts.** When LLM A and LLM B disagree at a depth the Supervisor cannot resolve, the human is the tiebreaker.

### Relationship and product work (irreducibly human)

- **Customer relationships.** Direct relationships with customers — sales conversations, escalations, partnership negotiations, roadmap commitments, contract terms — are not delegated. The framework can produce code at machine pace; trust between people cannot be produced at machine pace.
- **Real empathy.** Reading what a customer actually needs (vs. what was asked), sensing when a frustration is foundational vs. transient, recognizing when a feature request is the wrong solution to a real problem. These are human judgments grounded in understanding the customer as a person, not as a ticket queue.
- **Product management.** Vision, prioritization tradeoffs, GTM positioning, pricing, segmentation. The Backlog Agent ranks; the human decides what entries belong in the backlog at all.
- **Product ownership.** Acceptance criteria for the human-impactful work, sign-off on user-facing changes, judgment calls on what "done" means for features that affect customer trust.
- **Project management.** Scope-vs-time tradeoffs, dependency negotiation across teams, communicating progress to stakeholders, managing expectations. The Supervisor coordinates agents; the human coordinates people.
- **Customer service.** When a customer interaction crosses a threshold — escalation, complaint, retention conversation — the human takes the conversation directly. The Live Debugging Agent might draft a remediation; the human delivers the response.

### Multi-layer supervision

A consequence of agent-mediated production: customer-feedback flows are now agent-intermediated. A user complaint about broken behavior is routed by the Observability Agent to the Live Debugging Agent, which produces a structured root-cause hypothesis, which becomes a backlog item or a direct PPRE plan input, which is reviewed by the Architecture Review Agent and Safety & Security Audit Agent, which advances to a coding pod, which produces a behavior pin, which deploys via the CI/CD Agent. None of these steps require human intervention by default — and that is the point. The framework is designed to handle volume.

But human supervision is not removed from the path; it is *positioned* at the layers where judgment matters:

- **Proposal-stage supervision.** The human reviews PPRE plan outputs whose blast radius exceeds a configured threshold (schema migrations, security-surface changes, customer-facing copy changes). Below the threshold, the agent path runs through.
- **Customer-impact supervision.** Any backlog item generated from a customer complaint is human-acknowledged before it advances. The framework does not autonomously close customer feedback loops.
- **Adversarial-conflict supervision.** When reviewer agents and execution agents disagree past the Supervisor's resolution depth, the human is paged with a complete context package.
- **Drift-escalation supervision.** When the Reality-Anchor (Watcher) cannot stabilize a confused session within its configured retry budget, the human is paged.
- **Strategic supervision.** The human owns priorities, ROI calls, and when to add or retire agent roles. The factory does not autonomously decide what to build.

### Intervention

Intervention is its own category — the human's authority to stop, redirect, or override the factory at any point. The Watcher Agent intervenes mid-session for technical drift; humans intervene for strategic, ethical, customer-relationship, or judgment-call drift. This authority is unconditional: a plan can be in flight, a coding pod can be mid-execution, a deploy can be queued — the human can pause, redirect, or kill it. The factory is built for autonomous operation but explicitly preserves the human's right of intervention everywhere.

The factory delegates implementation; oversight, relationships, and judgment remain distributed across the loop and concentrated in the human at the points where they are irreducible.

---

## 18. Failure modes the framework explicitly handles

| Failure mode | How the framework catches it |
|---|---|
| **Sycophancy** — reviewer agrees with author's reasoning rather than evaluating independently | Adversarial isolation: reviewers do not see author's reasoning trace until they have produced their own assessment |
| **Context collapse** — agent loses track of prior decisions across long sessions | Supervisor maintains durable context; agents re-inject context per task rather than relying on session memory |
| **Hallucination of fact** — agent claims a function/file/flag exists when it does not | Behavior-pin corpus + verify-before-recommend discipline; runtime checks of claimed artifacts |
| **Drift between plan and implementation** — implementation silently deviates from approved plan | Every commit traces back to the approved plan; deviations reopen PPRE |
| **Architectural drift** — small local changes accumulate into structural decay | Architecture Review Agent runs continuously, not periodically |
| **Backlog overload** — exponential growth makes priorities meaningless | Continuous machine-scale grooming with deletion authority |
| **Trust erosion** — past decisions get re-litigated because their rationale was lost | Behavior pins document why each decision was made; future agents read the rationale |
| **Symptom-only patching** — fix lands on the surface, root cause persists | Symptom-to-root discipline rejects symptom-only patches |
| **Partial / stochastic fixes** — fix patches one violation site of an invariant, leaves identical violations elsewhere untouched (e.g., one query gets the missing tenant_id filter; three peer queries do not) | Symptom-to-root §9.4 mandates codebase-wide invariant audit + global pin after every root identification; Hygiene Agent (§10.5) re-runs the global-invariant set nightly |
| **Silent verification gaps** — fix lands without a regression test | Supervisor-enforced merge gate requires a behavior pin |

---

## 19. Why this matters

The factory is not "Agile, faster" or "Waterfall, with AI." It is a categorically different production model — closer to a continuously-running, multi-line manufacturing plant than to a development team. Three properties define it:

- **Concurrency over sequencing.** Multiple workstreams in different stages, coordinated by orchestration rather than synchronization.
- **Verification over trust.** Adversarial review at the proposal layer, behavior pinning at the implementation layer, post-deploy assertions at the production layer.
- **Discipline over heroism.** PPRE, symptom-to-root, behavior pinning, and assume-compromise verification are mandatory, enforced by the Supervisor as global invariants.

This produces a system in which **velocity and rigor compound rather than trade off**. Traditional engineering treats them as adversaries — speed at the cost of safety, safety at the cost of speed. The agent factory engineers them as the same property: speed is a function of how aggressively the verification machinery can run in parallel with development, and safety is a function of how many independent verification streams a change has survived.

The human founder remains central, but their role shifts from carrying the coordination load to designing the machinery that carries it. They are the architect of the factory and the final judge of strategic direction; everything else runs underneath.

---

*This document is intended as a living specification. It evolves alongside the layered agent safety framework as the system matures. Public companion repository: <https://github.com/walidnegm/agent-frameworks-safety-plus>*
