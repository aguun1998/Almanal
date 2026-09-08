from pathlib import Path

readme = """# ALMANAL_R v2.0

## Overview

ALMANAL_R is an experimental relational reasoning architecture for AI systems.

It represents goals, relations, definitions, transformations, proofs, alternatives, programs, language roles, self-reference, and realizations inside one relational world.

The current canonical artifact is:

`ALMANAL_R_v2.0`

## Core Principles

### 1. Relational representation

Relations are first-class forms.

A relation may participate in higher-order relations and may refer to itself or other relations.

### 2. Shared identity

Self-reference and cyclic structures reuse existing identities instead of recursively copying structure.

Examples:

- `X -> X` uses one identity.
- `A -> B -> A` uses two identities.
- A relation may refer to itself without infinite unfolding.

### 3. Need-scoped construction

Reasoning does not require global closure.

A goal induces dependencies. Only the reachable structure required by the current need is constructed.

Main concepts:

- `goal_dependency`
- `construction_frontier`
- `sufficient_closure`
- `need_generated`
- `need_scoped`
- `reachable_subgraph`
- `world_scoped`

### 4. Construction instead of post-hoc rejection

ALMANAL_R prefers admissible construction over:

`generate -> validate -> reject`

The intended architecture is:

`goal -> required relations -> admissible construction -> realization`

### 5. Contextual distinguishability

Alternatives are not split merely because they differ structurally.

A distinction becomes relevant when the current context and need produce a `distinguishability_witness`.

Absence of such a witness does not imply equality.

### 6. Partial composition

Composition is not assumed to be total.

Only jointly composable structures form a joint realization.

Execution order, parallelism, and scheduling are treated as projections of realization rather than core semantic categories.

### 7. Proof transport

If a transformation preserves the relevant structure, existing proof may be transported instead of recomputed.

This is represented through preserving transformations and proof transport relations.

### 8. Falsification

ALMANAL_R does not maintain a complete model of reality.

Reality checking is local and claim-scoped.

A reality counterexample may eliminate a candidate when it conflicts with the candidate's claim.

Failure to find a counterexample does not prove the claim.

Unresolved cases remain `OPEN`.

### 9. Creativity

Creative synthesis begins from an unresolved construction frontier.

A useful extension must:

- preserve the old paradigm through structural embedding,
- contain realizations outside the embedded old slice,
- provide a utility witness.

One possible operation is deprivileging:

`fixed constraint -> relation dimension`

However, creative synthesis is not restricted to this operation. Existing relations may also solve a frontier through new joint composition.

### 10. Finite representation of potentially unbounded realization

ALMANAL_R does not require explicit construction of an infinite object.

A finite generative representation may produce new successors only when required by the current need.

Typical structure:

`finite representation + generative transformation + need-scoped closure`

Cycles are represented through shared identity.

If no finite construction, proof transport, counterexample, or distinguishing witness is available, the result may remain `OPEN`.

## Definitions and Closure

Definitions are world-scoped relational constraints.

Definition, grammar, deduction, and realization use the same `sufficient_closure` architecture rather than separate global closure systems.

An exact definition corresponds to a single admissible equivalence class.

Ambiguous cases remain unresolved.

## Optimization

Optimization uses goal-relevant Pareto dominance.

A candidate is dominated when another candidate:

- preserves the same goal,
- is no worse on all relevant metrics,
- is strictly better on at least one relevant metric.

Incomparable candidates remain available.

No universal scalar score is required.

## Language and Representation

The textual form of ALMANAL_R is a serialization of the relational world.

Token spelling is not intended to be semantic authority.

Meaning is represented by relational structure.

The canonical representation is designed so that structurally equivalent forms normalize to the same canonical representation.

## Current Status

Version: `2.0`

Status: canonical experimental release

The current version includes:

- relational first-class forms
- shared identity
- self-reference
- finite cyclic graph representation
- need-scoped sufficient closure
- contextual distinguishability
- partial joint composition
- proof transport
- claim-scoped reality counterexamples
- creative synthesis
- structural paradigm extension
- generative representation for potentially unbounded realization
- Pareto dominance
- `OPEN / PROVEN / EXHAUSTED` search states

## Current Limitations

ALMANAL_R does not claim that:

- every problem is decidable,
- absence of a counterexample proves truth,
- every infinite process has a finite proof,
- every pair of candidates is comparable,
- every unresolved case can be closed,
- the current canonical artifact alone outperforms existing AI systems,
- a pretrained language model can already be replaced by ALMANAL_R alone.

Unresolved cases may remain `OPEN`.

## Intended Use with AI

The intended use is to place relational reasoning state between model perception and final realization.

Conceptually:

`tokens / observations -> relational world -> need-scoped construction -> realization -> tokens / actions`

A language model may provide:

- interpretation of natural language,
- candidate relation generation,
- external knowledge,
- final natural-language realization.

ALMANAL_R provides the structured relational state and reasoning process.

## Next Experiment: Qwen Integration

The next target is integration with a pretrained Qwen model.

Initial design:

1. Keep the pretrained Qwen model unchanged.
2. Use ALMANAL_R as external relational reasoning state.
3. Convert relevant model outputs into relational structures.
4. Perform need-scoped construction outside token-level reasoning.
5. Call the language model again only when new interpretation or language realization is required.

Evaluation should compare the modified system with the original model using:

- task accuracy
- generated token count
- model invocation count
- memory use
- execution time
- long-context coherence
- new failure cases
- problems solved only by the relational version

Later experiments may integrate relational state more directly into model execution.

## Basic Procedure

For a new task:

1. Bind the task as a goal and local world.
2. Identify goal dependencies.
3. Construct the unresolved frontier.
4. Build only the sufficient reachable closure.
5. Keep alternatives symbolic until they are distinguishable for the current need.
6. Reuse preserved proof where possible.
7. Use joint composition for compatible structures.
8. Expose claims to available counterexamples.
9. Keep unresolved results `OPEN`.
10. Record failures and bottlenecks for later revision.

## Files

- `ALMANAL_R_v2.0` — canonical relational artifact
- `README.md` — external technical description

## License

No license is specified in this README.
"""

path = Path("/mnt/data/README.md")
path.write_text(readme, encoding="utf-8")
print(f"Updated {path} ({path.stat().st_size} bytes)")
