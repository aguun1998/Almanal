# ALMANAL

**ALMANAL is an AI-oriented algorithm synthesis, optimization, and verification system built on Λ, a programming language designed primarily for AI rather than human authors.**

Λ intentionally deprioritizes many conventions that exist mainly for human programming convenience. Instead, it prioritizes:

* compact representation
* deterministic structure
* static verification
* low ambiguity
* direct native execution
* efficient AI generation, inspection, and transformation

ALMANAL uses Λ as its native substrate for deriving, synthesizing, validating, and improving algorithms.

## Current Release

* **ALMANAL:** 20.1
* **Λ:** 0.25
* **Canonical surface:** L4
* **Target:** Linux x86-64

## Λ

Λ is a statically checked, direct-native programming language designed for AI systems.

Its canonical compiler can compile Λ source directly into Linux x86-64 ELF executables without translating through C, C++, Python, or another source language.

Current language features include:

* `i64`, `u64`, `bool`, `unit`, and strings
* arrays and tuples
* tagged sums
* functions and recursion
* lexical bindings and mutable state
* conditionals and loops
* file and process operations
* explicit effects
* compile-time worlds, claims, and bounded proofs
* direct ELF64 machine-code generation
* L4 compact canonical syntax

The compiler is self-hosted: the Λ compiler can compile its own canonical source and reaches a reproducible native fixed point.

## L4

L4 is the canonical textual representation of Λ0 0.25.

It reduces source and context size while preserving deterministic decoding and semantic structure.

L4 uses:

* compact semantic atoms
* local mnemonic identifier aliases
* explicit structural checkpoints
* checked closing-run encoding
* hard rejection of ambiguous or malformed input

L4 is not intended to maximize human readability.

Its goal is:

> **semantic compression without semantic blindness**

## ALMANAL

ALMANAL is designed to make algorithm discovery more structured than unconstrained brute-force program search.

Its current synthesis architecture can be summarized as:

```text
World / Rules / Goal / Prior
        ↓
Deductive Baseline
        ↓
Structural Synthesis
        ↓
Constraint / CEGIS Guided Search
        ↓
Semantic Compression
        ↓
Validation
        ↓
Verified Policy Improvement
```

The system separates:

* theoretical reachability
* practical discoverability
* correctness and evidence

A complete search lane remains available as a backstop, while practical synthesis focuses on structured and guided search.

## Repository Structure

```text
language/
  SPEC.md
  COMPACT_NOTATION.md

compiler/
  native/
    expr_elfgen.l0

almanal/
  almanal.l0

LAMBDA_TUTORIAL.md
```

## Documentation

For a practical introduction to Λ:

**[`LAMBDA_TUTORIAL.md`](LAMBDA_TUTORIAL.md)**

Normative language specification:

**[`language/SPEC.md`](language/SPEC.md)**

Canonical L4 notation:

**[`language/COMPACT_NOTATION.md`](language/COMPACT_NOTATION.md)**

## Quick Start

The current native binaries target **Linux x86-64**.

Check Λ source:

```bash
./compiler/l0c check program.l0
```

Compile Λ source:

```bash
./compiler/l0c build program.l0 program
```

Run the resulting executable:

```bash
./program
```

## Project Status

The core Λ compiler, native self-host path, L4 canonical surface, and current ALMANAL architecture are implemented and regression-tested.

Current research work focuses more on **external validation** than on adding language features, especially:

* blind algorithm rediscovery
* broader synthesis domains
* raw problem → WorldSpec extraction
* stronger semantic-equivalence verification
* novel algorithm discovery

This project does **not** currently claim that Λ's optimizer is more mature than GCC or LLVM, nor that ALMANAL can efficiently discover every computable algorithm.

The purpose of the project is to explore a different software-development stack:

> **What should programming languages and algorithm synthesis systems look like when the primary programmer is an AI rather than a human?**

## License

See [`LICENSE`](LICENSE).
