# ALMANAL

ALMANAL is an algorithm synthesis, optimization, and verification system implemented on Λ, a compact programming language designed primarily for machine generation, inspection, and transformation.

## Current release

- **ALMANAL:** 24.2
- **Λ0:** 0.40
- **Target:** Linux x86-64

## Λ0

Λ0 is a statically checked language with a compact textual representation and a native compiler targeting Linux x86-64 ELF executables.

The current package includes the compiler executable and its Λ source. The compiler provides `build`, `profile`, `check`, `check-types`, `check-effects`, `emit-ir`, and `verify` operations.

### Hangul identifiers and representation economy

Λ0 0.40 uses modern Hangul syllables for many compact identifiers. This is an engineering choice for representation economy, not a claim that Hangul is intrinsically more efficient than ASCII.

A modern Hangul syllable occupies three bytes in UTF-8. When one syllable replaces an ASCII identifier longer than three bytes, the source representation can become smaller. More generally, Hangul provides a large set of distinct single-syllable symbols, allowing many identifiers to remain one source symbol long instead of requiring multi-character mnemonic names. For machine-generated and machine-transformed code, this can reduce source size and identifier-handling overhead while preserving deterministic symbol identity.

Tokenizer cost is model-dependent, so token savings are not assumed from character count alone; they must be measured for the tokenizer in use.

## ALMANAL

ALMANAL uses Λ as its implementation substrate. The current implementation combines algorithm construction and evaluation with explicit evidence handling, counterexample-oriented falsification, and burden-aware improvement procedures.

The release contains both the Λ source and the corresponding native executable.

## Repository layout

```text
almanal/
  almanal        Native ALMANAL executable
  almanal.l0     ALMANAL Λ source

compiler/
  l0c            Native Λ compiler
  native/
    expr_elfgen.l0   Λ compiler source

README.md
CITATION.cff
LICENSE
```

## Basic compiler use

Check a Λ source file:

```bash
./compiler/l0c check program.l0
```

Compile it:

```bash
./compiler/l0c build program.l0 program
```

Run the resulting executable:

```bash
./program
```

## Citation

Citation metadata is provided in [`CITATION.cff`](CITATION.cff).

## License

See [`LICENSE`](LICENSE).
