# ALMANAL Lambda — Canonical Specification

Status: canonical release. Earlier H/R names were experimental checkpoints.

## Identity

ALMANAL Lambda integrates the AI-native Λ language with the ALMANAL synthesis, analysis, refactoring, and judgment system in one executable. Compilation and algorithm reasoning use the same representation, transformation, and verification foundation rather than separate products.

## AI entry point

Running the executable without arguments prints the minimum machine-readable discovery line:

```text
NAME=ALMANAL Lambda;START=check SOURCE;BUILD=build SOURCE OUTPUT;VERIFY=verify SOURCE BINARY;OPS=solve|almanal|analyze|refactor|judge|profile|check-types|check-effects|emit-ir
```

## Canonical Λ source

- Canonical extension: `.lh`
- Canonical Unicode normalization: NFC
- Whitespace outside quoted strings carries no syntax information and canonical source contains none.
- Implicit structures close only by encoded arity/structure rules.
- An explicit `(` structure closes only with its matching explicit `)`.
- `)` never closes an implicit structure.
- Explicit parentheses remain valid when an explicit boundary is useful.
- The symbol dictionary is embedded as `;...;;body`; no external map file is required.

## Extensibility

The language provides general computation, state/memory, byte processing, control, effects, and a host-native bridge instead of continuously adding feature-specific intrinsics.

```text
홰(op,args6_ptr)->i64
```

`홰` requires effect `외`. At the language level it denotes one host-native operation with six machine-word arguments. The current Linux x86-64 backend maps it to a raw system call; that mapping is a backend property, not target-independent Λ semantics.

New capabilities should normally be implemented in Λ. A new language primitive is justified only by a fundamental expressiveness counterexample that the existing basis cannot represent.

## Integrity state

- Top-level functions: 365
- Duplicate top-level definitions: 0
- Unreachable top-level functions from `굄`: 0
- Undefined direct function calls: 0
- ALMANAL analyze class-1 items: 0
- `check`: PASS
- `check-types`: PASS
- `check-effects`: PASS
- Canonical-source self-host build: byte-identical
- `verify`: PASS

Canonical executable SHA-256:

```text
0fbd978f4cefcf97b06d5e8c376a1e6e72c7a77dcb2ed23f6f603e40eb8f67e4
```

## Scope

The current implementation target is Linux x86-64. The language/compiler structure is considered closed until a concrete correctness, portability, expressiveness, or usage counterexample requires a change.
