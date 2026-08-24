# ALMANAL

ALMANAL is an algorithm synthesis, optimization, and verification system implemented on Λ, a compact programming language designed primarily for machine generation, inspection, and transformation.

## Current release

- **ALMANAL:** 24.2
- **Λ0:** 0.40
- **Target:** Linux x86-64

## Why Λ exists

Λ was created for a setting in which the primary programmer may be an AI rather than a human.

Conventional programming languages necessarily devote part of their syntax, naming practice, tooling model, and abstraction surface to human readability, familiarity, and manual development. Λ explores a different design point: retain the structure needed for deterministic parsing, static checking, transformation, and native execution, while reducing representation and conventions that are useful mainly for human convenience.

The practical design goals are:

- compact source representation;
- deterministic syntax and symbol identity;
- static type and effect checking;
- direct machine-oriented generation and transformation;
- reproducible verification paths;
- native Linux x86-64 execution.

Λ0 is the current compact core language and compiler implementation used by ALMANAL.

### Why Hangul identifiers are used

Λ0 0.40 uses modern Hangul syllables for many compact identifiers. This is an engineering choice for representation economy, not a claim that Hangul is intrinsically more efficient than ASCII.

A modern Hangul syllable occupies three bytes in UTF-8. When one syllable replaces an ASCII identifier longer than three bytes, the source representation can become smaller. More importantly, Hangul provides a large set of distinct single-syllable symbols, allowing many identifiers to remain one source symbol long instead of requiring multi-character mnemonic names.

For machine-generated and machine-transformed code, this provides a dense identifier space while retaining deterministic textual identity. Tokenizer cost remains model-dependent, so token savings are measured rather than assumed from character count alone.

## Why ALMANAL exists

ALMANAL was created to make algorithm construction and improvement a more explicit computational process.

Its role is to represent candidate algorithms, evaluate them against task and evidence constraints, search for improvements, test counterexamples, and compare alternatives while accounting for the burden required to obtain and maintain a result.

The implementation includes mechanisms for:

- algorithm candidate construction and comparison;
- explicit evidence and provenance handling;
- counterexample-oriented falsification;
- bounded search and verification procedures;
- Pareto-style comparison of time, memory, verification, maintenance, and related burden;
- reuse-first and necessary-action decisions intended to avoid work that is not required for the current objective.

ALMANAL uses Λ as its native implementation substrate. The release contains both the Λ source and the corresponding Linux x86-64 executable.

## Repository layout

```text
almanal/
  almanal            Native ALMANAL executable
  almanal.l0         ALMANAL Λ source

compiler/
  l0c                Native Λ compiler
  native/
    expr_elfgen.l0   Λ compiler source

README.md
CITATION.cff
LICENSE
```

## Basic use

The included executables target **Linux x86-64**.

### Λ compiler

Check a Λ source file:

```bash
./compiler/l0c check program.l0
```

Check types or effects separately:

```bash
./compiler/l0c check-types program.l0
./compiler/l0c check-effects program.l0
```

Compile Λ source to a native executable:

```bash
./compiler/l0c build program.l0 program
```

Run the result:

```bash
./program
```

Verify that a binary corresponds to a source file:

```bash
./compiler/l0c verify program.l0 program
```

The compiler also exposes `profile` and `emit-ir` operations.

### ALMANAL

ALMANAL uses a compact numeric stdin protocol intended primarily for programmatic use. Mode `0` performs the built-in self-check:

```bash
printf '0\n' | ./almanal/almanal
```

A successful run prints:

```text
ALMANAL-LAMBDA-SELFCHECK-PASS
```

Other modes accept mode-specific numeric inputs through stdin and expose the synthesis, verification, falsification, evidence, burden-comparison, and related operations implemented in `almanal/almanal.l0`.

## Citation

Citation metadata is provided in [`CITATION.cff`](CITATION.cff).

## License

See [`LICENSE`](LICENSE).
