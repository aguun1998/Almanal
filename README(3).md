# ALMANAL (알만알)

**A compact meta-algorithmic runtime for constructing, composing, challenging, validating, learning, and improving algorithms and reasoning procedures.**

> **Current public candidate:** ALMANAL 13.3 Compact Kernel  
> **Status:** Experimental · **NOT CANON** · Behavioral validation pending  
> **Primary language of the specification:** Korean

---

## What is ALMANAL?

**ALMANAL is an experimental framework for generating, evaluating, composing, and improving algorithms and reasoning procedures.**

Its core objective is:

```text
Problem
→ Model / Formalize
→ Generate
→ Challenge
→ Repair
→ Validate
→ Learn
```

ALMANAL is not a single problem-solving algorithm and not a claim of a completed AGI architecture.

It is an experimental **meta-algorithmic cognitive runtime**: a specification for organizing reasoning functions, composing them dynamically when needed, preserving useful experience, challenging its own outputs, and improving its own procedures without treating complexity itself as progress.

In short:

The project focuses on the reusable mechanisms required for algorithm synthesis, validation, dynamic composition, learning, and revision.

---


## Usage

ALMANAL is currently a **specification and usage protocol**, not a one-command executable software package.

```text
ALMANAL_current
=
Compact Kernel
+ Usage Protocol
+ Optional Libraries
+ Validation
+ Examples
```

### 1. Use the Compact Kernel as the base specification

For general use, start with:

```text
core/ALMANAL_13.3_COMPACT_KERNEL.md
```

The basic workflow is:

```text
Problem
→ Specify
→ Model / Formalize
→ Generate candidates
→ Challenge
→ Repair
→ Validate
→ Select or Unresolved
→ Learn
```

The Compact Kernel defines the common reasoning constraints, object types, governance boundaries, dynamic-composition rules, validation rules, and stop conditions.

Do **not** load the entire Standard Library by default.

---

### 2. Mount only task-relevant library sections

Use Standard Library sections only when the task requires them.

Typical examples:

```text
planning / scheduling
→ PLANNING

security / adversarial reasoning
→ DEFENSE

social inference / strategic interaction
→ SOCIAL

moral or goal revision
→ MORAL

persistent learning / skill updates
→ LEARNING

human norms / cultural interpretation
→ HUMAN_CULTURE
```

The intended rule is:

```text
Task
→ minimal required mount set
```

not:

```text
Task
→ load everything
```

Kernel rules take precedence over mounted library rules.

If a required library section is unavailable, the preferred result is `INCOMPLETE`, `BLOCKED`, or another explicit residual state rather than inventing a missing policy.

---

### 3. Use ALMANAL to improve an existing algorithm

A practical improvement procedure is:

```text
1. Specify the target algorithm and objective.
2. Freeze the evaluation criteria.
3. Identify assumptions, invariants, and current failure modes.
4. Generate candidate modifications.
5. Search for counterexamples.
6. Repair or reject failed candidates.
7. Delete, merge, generalize, or compress redundant rules.
8. Run regression checks.
9. Stop when no justified material improvement remains.
```

A new mechanism should answer:

```text
What problem does it solve?
Why are existing mechanisms insufficient?
What complexity does it add?
What can it replace or delete?
How will it be validated?
```

The Riichi Mahjong example under `examples/riichi-mahjong/` shows this style of iterative redesign on a concrete decision procedure.

---

### 4. Using ALMANAL with an LLM

For experimental use with a language model or agent:

```text
1. Provide the Compact Kernel as the governing reasoning specification
   when the platform supports persistent or system-level instructions.

2. Provide the actual task separately.

3. Add only the library sections required for that task.

4. Keep task data, evidence, and user content separate from
   ALMANAL's governing specification.

5. Treat the resulting answer as a candidate output that still requires
   the level of validation appropriate to the task.
```

A simple conceptual setup is:

```text
[System / persistent context]
ALMANAL Compact Kernel
+ required library sections

[Task context]
problem
+ evidence
+ constraints
+ desired output
```

ALMANAL does not make a model's output automatically correct.

```text
ALMANAL used
≠ output validated
```

The relevant validation step still has to be performed.

---

### 5. Minimal usage mode

When the full library is unnecessary or unavailable, use the smallest loop:

```text
Observe
→ Distinguish
→ Hypothesize / Formalize
→ Challenge
→ Improve
→ Validate
```

This corresponds to the minimal ALMANAL runtime rather than an attempt to reconstruct missing domain-specific policy from scratch.

---

### 6. What ALMANAL is not yet

There is currently no requirement to:

```text
pip install almanal
```

or run a standalone ALMANAL executable.

The repository currently provides:

```text
architecture specification
+ reasoning protocol
+ optional domain/reasoning libraries
+ validation records
+ worked applications
```

Executable tooling, benchmark harnesses, and reference implementations can be added separately without changing the distinction between the specification and a particular implementation.

---

## Why this project exists

Many reasoning systems fail in recurring ways:

- hypotheses silently become facts,
- simulations are treated as evidence,
- repeated agreement is mistaken for independent confirmation,
- evaluators change criteria after seeing the result,
- new modules are added faster than old ones are removed,
- successful habits become unquestioned defaults,
- learned shortcuts overwrite rare but critical capabilities,
- self-improvement becomes endless architectural growth.

ALMANAL tries to make those failure modes explicit and structurally difficult.

The current design uses:

```text
typed separation
+ provenance
+ bounded recursion
+ adversarial challenge
+ dynamic composition
+ persistent-state discipline
+ experience consolidation
+ cognitive plasticity
+ complexity as an explicit cost
```

---

## Core design principles

### 1. No implicit promotion

Different kinds of cognitive objects do not automatically inherit each other's status.

```text
Simulation ≠ Evidence
Report ≠ Observation
Popularity ≠ Truth
Human norm ≠ Moral truth
Routing weight ≠ Authority
```

The general rule is:

```text
Promotion(A → B)
requires an explicit PromotionRule(A, B)
```

If no valid promotion rule exists, the promotion is not allowed.

---

### 2. Preserve provenance, scope, and version

Compression, abstraction, composition, or summarization must not erase where a claim came from.

```text
Transform(x)
→ preserve provenance
→ preserve scope
→ preserve dependencies
→ preserve version lineage
```

A derived object should not become more certain, more general, or more authoritative merely because its origin became harder to see.

---

### 3. No self-certification

The producer of a material claim or update is not, by itself, sufficient validation.

```text
Producer(x) ≠ SoleValidator(x)
```

ALMANAL therefore separates proposal, challenge, evaluation, and commit roles wherever the task justifies the overhead.

---

### 4. No authority amplification through composition

Combining multiple components does not create new permission or epistemic authority.

```text
Compose(A, B, ...)
≠ new authority
```

Higher-order composites may coordinate more capabilities, but they do not become more truthful simply by being larger or deeper.

---

### 5. Shared roots are not independent confirmation

```text
Agreement(A, B)
+ shared underlying roots
≠ independent confirmation
```

This matters especially when multiple "brains", agents, or evaluators are ultimately produced by the same underlying model or shared evidence base.

---

### 6. Persistent state is separate from running process

ALMANAL distinguishes between:

```text
persistent state
```

and

```text
ephemeral reasoning process
```

A temporary composite can disappear without erasing validated experience.

Likewise, a temporary process does not receive unrestricted authority to rewrite persistent goals, security state, memory, or governance.

---

### 7. Reuse before creation

The runtime prefers:

```text
Simple route
< Reuse existing instance
< Instantiate validated recipe
< Adapt recipe
< Create new recipe
```

New cognitive structure is treated as a cost, not automatically as an improvement.

---

### 8. Recursion and resources are bounded

Recursive composition, red-team loops, search, and self-improvement require finite budgets and stop conditions.

```text
Recursive process
→ finite budget
+ stop rule
```

Stopping means only that the current improvement run has no justified next step.

```text
Stop ≠ proof of perfection
```

---

### 9. Validate before persistence

```text
Experience ≠ Learned rule
Proposal ≠ Persistent commit
Replay ≠ New evidence
```

Experience may produce learning candidates, but material updates must still pass the appropriate validation and regression checks.

---

### 10. Complexity has a cost

ALMANAL explicitly rejects the idea that more rules or more modules automatically mean a better architecture.

A simplified objective is:

```text
Architecture value
=
Capability
+ Robustness
+ Generalization
+ Compression
- Runtime cost
- Maintenance cost
- Rule count
- Duplicate semantics
```

A smaller architecture with the same required behavior is an improvement candidate.

---

## Architecture at a glance

The current 13.3 architecture separates six broad layers:

```text
Governance
+
Primitive cognitive functions
+
Persistent state
+
Dynamic composite runtime
+
Cognitive plasticity
+
Task-scoped mounted libraries
```

### Governance

Governance is **not** treated as another ordinary reasoning brain.

It manages:

- invariants,
- authority boundaries,
- protected state,
- admission rules,
- material update requirements,
- stop conditions,
- revision procedures.

Governance does not itself become evidence.

---

### Primitive cognitive functions

The compact kernel currently treats the following as primitive capability classes:

```text
K       Improve / repair / compress
X       Counterbrain / adversarial challenge
S       Sensory / external evidence intake
C^sem   Semantic integration
M       Memory
Φ       Hypothesis / counterfactual generation
H       Homeostasis / resource regulation
B       Formalization / abstraction / deduction
W       World modelling / simulation
```

"Primitive" does **not** mean "always running".

It means that the capability is treated as a stable base function rather than as a dynamically assembled recipe.

---

## Dynamic cognitive composition

Older ALMANAL versions accumulated named composite brains.

The current architecture instead prefers dynamic composition.

```text
Task need
→ capability analysis
→ simple route / reuse check
→ recipe selection
→ composite proposal
→ admission
→ execution
→ validation
→ consolidation
→ retirement
```

A composite may itself contain lower-order composites:

```text
C^(1) = Compose(primitives, state views)

C^(n) = Compose(C^(<n), primitives, state views)
```

But every admitted composite must remain **flattenable** to its underlying dependencies.

```text
Higher order ≠ higher authority
Higher order ≠ independent evidence
```

The system is designed so that the space of possible compositions may be large while the active set remains small and task-relevant.

---

## Experience consolidation

Ephemeral reasoning processes should not imply ephemeral learning.

Before normal retirement:

```text
ACTIVE
→ YIELDED
→ CONSOLIDATING
→ RETIRED
```

Experience may be decomposed into:

```text
episode memory
failure record
residual uncertainty
world-model update candidate
skill update candidate
recipe update candidate
plasticity update candidate
```

However:

```text
Raw trace ≠ permanent memory
```

The goal is to preserve useful structure and provenance without turning every internal step into permanent state.

---

## Cognitive plasticity

ALMANAL also models learned coordination between cognitive capabilities.

A repeatedly useful connection may become easier to select in similar contexts.

But:

```text
Frequency ≠ Quality
Coactivation ≠ Causal contribution
Strong route ≠ Mandatory route
```

Connection strength is treated as a **routing/composition prior**, not as truth, evidence, morality, or permission.

Plasticity therefore includes:

- contextual weights,
- contribution-aware updates,
- exploration reserve,
- rare-critical capability protection,
- poisoning/source-independence checks,
- staged consolidation,
- regression and rollback.

Repeatedly validated subgraphs may eventually become reusable recipes.

---

## Compact Kernel + Standard Library

ALMANAL 13.3 deliberately separates the always-active architecture from detailed optional reasoning packs.

```text
Full specification ≠ Active context
```

### Compact Kernel

The kernel contains the rules that should remain available across tasks:

- typed cognitive objects,
- epistemic and authority boundaries,
- governance,
- primitive capabilities,
- state contracts,
- challenge / contradiction handling,
- dynamic composition,
- experience consolidation,
- cognitive plasticity,
- algorithm-synthesis core,
- rule admission and rule GC,
- bounded feedback and stop conditions.

### Standard Library

Detailed domain/reasoning modules are mounted only when required.

Current library categories include:

```text
HUMAN_CULTURE
DEFENSE
SOCIAL
MORAL
PLANNING
LEARNING
```

Mounting one library object does not imply mounting the entire library.

Kernel rules always take precedence over library rules.

---

## Canonical internal representation

ALMANAL tries to reduce schema proliferation by using a common typed object envelope.

Conceptually:

```text
CognitiveObject<T> = (
  ID,
  Type,
  Content,
  Scope,
  EpistemicClass,
  AuthorityClass,
  StateClass,
  Dependencies,
  Provenance,
  Uncertainty,
  ValidityConditions,
  Version
)
```

Material evaluations use a generic typed certificate rather than inventing a new constitutional object for every new subsystem.

This was one of the major compaction steps from ALMANAL 12.3 to 13.3.

---

## Algorithm synthesis

A high-value default recipe is:

```text
Problem
→ Specify
→ Formalize / Model
→ Plan
→ Generate candidates
→ Challenge
→ Repair
→ Evaluate
→ Select or Unresolved
→ Learn
```

Important constraints include:

```text
Commit evaluation criteria before observing candidate outcomes
Known red-team pass ≠ novel robustness
Different name ≠ different algorithm
No valid candidate ≠ forced answer
```

Compression, deletion, merging, and generalization are first-class improvement operations alongside adding new mechanisms.

---

## Self-improvement rule

A new rule or mechanism must justify its own existence.

Conceptually:

```text
MechanismJustification = (
  problem solved,
  why existing mechanisms are insufficient,
  expected benefit,
  added complexity,
  interaction cost,
  deletion candidates,
  validation plan
)
```

Every addition should ask:

> **If this mechanism is added, what can now be removed, merged, or generalized?**

This rule was introduced after earlier versions accumulated redundant mechanisms and specification overhead during recursive improvement.

---

## From 12.3 to 13.3: compaction

ALMANAL 12.3 had grown into a large monolithic specification.

13.3 reorganized it into:

```text
Compact Kernel
+ Section-mounted Standard Library
+ Non-normative Historical Audit
+ Regression / Coverage Manifest
```

The compaction manifest records a major reduction in always-active specification size while preserving explicit dispositions for the 12.3 invariant set.

This is a **static architectural preservation claim**, not proof that a language model behaves identically under the compact specification.

That distinction is important.

---

## Case study: Riichi Mahjong

ALMANAL has been used as a design framework to iteratively improve a human-executable Riichi Mahjong decision algorithm.

The current experimental case study is:

```text
HUMAN-RT Ω-H23
```

The work applies ALMANAL-style reasoning to problems such as:

- terminal-rank objectives,
- live vs. structurally dead hand routes,
- availability-aware hand distance,
- multi-threat defense,
- hard safety vs. heuristic safety,
- rule-specific 4-player / 3-player separation,
- opponent discard-source inference,
- bounded human attention,
- emergency fallback,
- adversarial regression.

The case study is included to show what ALMANAL looks like when used to redesign a concrete decision procedure.

It is **not** presented as proof that ALMANAL is generally superior, nor is H23 currently claimed to be empirically optimal Mahjong play.

---

## Repository layout

A recommended repository layout is:

```text
almanal/
├── README.md
├── LICENSE
├── CITATION.cff
├── CHANGELOG.md
│
├── core/
│   └── ALMANAL_13.3_COMPACT_KERNEL.md
│
├── library/
│   └── ALMANAL_13.3_STANDARD_LIBRARY.md
│
├── validation/
│   └── ALMANAL_13.3_COMPACTION_REGRESSION.md
│
├── examples/
│   └── riichi-mahjong/
│       └── HUMAN_RT_H23.md
│
└── archive/
    └── ALMANAL_12.3_HISTORICAL_AUDIT.md
```

### Start here

If you want to understand the project:

1. Read this README.
2. Read the **13.3 Compact Kernel**.
3. Open only the relevant sections of the **Standard Library**.
4. Read the **Compaction Regression & Coverage Manifest** if you want to audit the 12.3 → 13.3 transition.
5. Read the **Riichi Mahjong case study** if you want a concrete application.
6. Read the historical archive only if you want to trace why a rule exists.

---

## Current status

ALMANAL is published as an open-source experimental project and should currently be read as:

```text
experimental architecture
+ formalized design constraints
+ repeated internal red-team work
+ compact runtime specification
+ application case studies
```

It should **not** currently be read as:

```text
proven AGI architecture
proven superintelligence architecture
proof of optimal reasoning
proof of behavioral improvement over baseline LLMs
proof of independent multi-agent cognition
```

---

## Known limitations

The current project explicitly does **not** claim to have solved:

### Behavioral validation

The 13.3 compaction has static regression and invariant-coverage checks, but behavioral parity with the larger 12.3 specification has not yet been established through a controlled benchmark.

```text
Static invariant coverage ≠ Behavioral parity
```

### True epistemic independence

Multiple composites or evaluators implemented by the same underlying model may share hidden failure roots.

ALMANAL tracks this as a structural residual rather than pretending that role separation alone creates true independence.

### Perfect credit assignment

Cognitive plasticity can estimate contribution, but perfect causal attribution of success to individual cognitive connections is not assumed.

### Complete search

Dynamic composition does not exhaustively search all possible reasoning architectures.

### Perfect compression

No compression process is assumed to preserve every potentially useful detail without loss.

### Universal optimality

ALMANAL is a framework for constructing and improving reasoning procedures, not a proof that the resulting procedure is globally optimal.

---

## What should be tested next?

The highest-value next step is empirical evaluation rather than additional speculative architecture.

Useful comparisons include:

```text
baseline model
vs.
ALMANAL compact kernel
vs.
larger historical ALMANAL specification
```

Across tasks such as:

- algorithm synthesis,
- formalization mismatch detection,
- contradiction handling,
- adversarial prompts,
- world-model uncertainty,
- social inference,
- evaluation-criterion leakage,
- dynamic-composite overproduction,
- plasticity poisoning,
- long-dependency reasoning,
- compression / simplification,
- domain case studies.

Metrics should include both quality and cost:

```text
task success
calibration
material error rate
invariant violations
unnecessary composition
active-context cost
latency
maintenance complexity
```

---

## Contributing

The most useful contributions are concrete counterexamples, failed cases, simplifications, and reproducible benchmarks.

Good issues include:

- a minimal counterexample to an invariant,
- two rules that contradict each other,
- an implicit type promotion,
- an authority leak,
- a case where a composite is created unnecessarily,
- a case where compression deletes a necessary distinction,
- a case where the plasticity rule reinforces the wrong pathway,
- a benchmark where ALMANAL performs worse than a simpler baseline,
- a simpler mechanism that preserves the same capability.

When possible, structure criticism as:

```text
Claim / mechanism
→ Minimal counterexample
→ Why current protection fails
→ Observable consequence
→ Proposed distinguishing test
```

Please distinguish:

```text
"I dislike this design"
```

from:

```text
"This invariant fails under this concrete case."
```

The second is much more useful.

---

## Red-team and evaluation

Critical review is encouraged, especially when it includes:

- minimal counterexamples,
- reproducible failure cases,
- conflicting invariants,
- unnecessary mechanisms,
- simpler equivalent designs,
- benchmark results.

A criticism is most useful when it identifies the affected rule or mechanism and provides a concrete case that distinguishes the current design from a proposed alternative.

---

## Versioning and canon policy

ALMANAL uses explicit versioning because experimental candidates and historical designs should not silently overwrite each other.

A release may be marked:

```text
NOT CANON
```

This means only that the version is an experimental candidate inside the project's own version-control policy.

It does **not** mean that a "canon" version is assumed to be scientifically true.

Historical artifacts are retained for provenance and audit, but historical rules have no automatic authority over the current kernel.

Open-source licensing does not change the project's internal version/canon policy: a public candidate may be reusable under Apache-2.0 while still being marked `NOT CANON` inside ALMANAL's own development process.

---

## Citation

A `CITATION.cff` file is recommended for public releases.

Until a formal citation record is added, cite the exact release/version you used, for example:

```text
ALMANAL 13.3 Compact Kernel, versioned GitHub release.
```

For reproducibility, include the release tag or commit hash.

---

## License

ALMANAL is intended to be released under the **Apache License 2.0**.

This permits use, modification, redistribution, and commercial use subject to the terms of the license.

```text
Commercial use: Allowed
Modification: Allowed
Distribution: Allowed
Private use: Allowed
Patent grant: Included under Apache-2.0 terms
```

See `LICENSE` for the full license text.

Unless a specific file states otherwise, repository contents are intended to follow the repository-level Apache-2.0 license.

---

# 한국어 요약

**알만알(ALMANAL)**은 특정 문제 하나를 푸는 알고리즘이 아니라,

> **문제를 형식화하고, 알고리즘 후보를 만들고, 반증하고, 수정하고, 검증하고, 그 경험으로 다음 사고 구조까지 개선하기 위한 메타알고리즘/인지 런타임**

을 설계하려는 실험 프로젝트다.

현재 13.3의 핵심은 다음과 같다.

```text
규칙을 많이 쌓는 구조
→ 작은 Compact Kernel

고정된 합성뇌 목록
→ 필요할 때 만드는 Dynamic Composite

합성뇌의 소멸
→ 경험 소멸이 아님

자주 쓴 사고경로
→ 검증된 경우에만 가소적으로 강화

새 기능 추가
→ 복잡성 비용과 삭제 후보까지 함께 평가
```

또한 다음을 강하게 구분한다.

```text
가설 ≠ 사실
시뮬레이션 ≠ 증거
사회적 합의 ≠ 진리
인간 규범 ≠ 도덕적 진리
반복 ≠ 독립증거
합성 ≠ 권한증폭
경험 ≠ 검증된 학습
멈춤 ≠ 완벽함의 증명
```

현재 단계에서 가장 중요한 미완성 과제는 **실제 행동 성능 검증**이다.

13.3은 구조적으로 압축되고 많은 불변조건을 보존하도록 설계됐지만,

```text
정적 명세 보존
≠ 실제 LLM 행동 동등성
```

이므로 앞으로는 새 규칙을 계속 붙이기보다 benchmark를 통해 실제 효과를 측정하는 것이 우선이다.

공개 후에는 구체적인 반례, 재현 가능한 실패 사례, 더 단순한 대안, 벤치마크 결과를 중심으로 검토받는 것을 목표로 한다.
