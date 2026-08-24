# ALMANAL

ALMANAL is a machine-oriented framework for guiding a capable AI model through algorithm generation and iterative algorithm improvement.

The intended use is simple: provide ALMANAL to the AI as working context, provide the problem and objective, and ask the model to generate or improve an algorithm under ALMANAL's procedures. Verification, falsification, evidence handling, comparison, and burden analysis are parts of that generation/improvement process rather than separate end goals.

## Current release

- **ALMANAL:** 24.2
- **Λ0:** 0.40
- **Native target:** Linux x86-64

## Project origin and chronology

ALMANAL came first.

It began as an AI-oriented framework for producing and improving algorithms. As ALMANAL itself was repeatedly improved, the implementation language and representation became part of the optimization problem: conventional programming languages retain syntax, naming conventions, abstractions, and tooling assumptions that are useful for human programmers but are not always necessary when the primary author and transformer is an AI.

**Λ was created during that ALMANAL improvement process.** It was not the original starting point of the project and was not introduced as an unrelated parallel language project. It emerged as a way to reduce representation and implementation burden while preserving the structure needed for deterministic parsing, static checking, transformation, verification, and native execution.

ALMANAL was subsequently moved onto Λ as its native implementation substrate. The current repository therefore contains both the ALMANAL system and the Λ compiler/runtime path that grew out of ALMANAL's own optimization process.

## Purpose of ALMANAL

ALMANAL is intended to be supplied to a sufficiently capable AI model as an algorithm-generation and algorithm-improvement framework.

Typical use:

1. Provide the ALMANAL source/context to the AI model.
2. Provide the problem, constraints, available evidence, and objective.
3. Ask the model to generate an algorithm or improve an existing one using ALMANAL.
4. Let the model use ALMANAL's internal procedures for construction, comparison, falsification, verification, reuse, and burden reduction as needed.
5. Request the resulting algorithm, implementation, proof/evidence summary, or other required output.

ALMANAL's internal mechanisms include:

- candidate construction and comparison;
- explicit evidence and provenance handling;
- counterexample-oriented falsification;
- bounded search and verification procedures;
- comparison of time, memory, verification, maintenance, and related burden;
- reuse-first decisions;
- necessary-action minimization, so work not required for the current objective can be omitted.

These mechanisms support the main task: producing or improving algorithms with the AI model.

## Why Λ exists

Λ is the language that emerged while optimizing ALMANAL itself.

Its design assumes that source may primarily be generated, inspected, transformed, and maintained by machines rather than manually authored by humans. This changes which costs are worth optimizing.

The practical design goals include:

- compact source representation;
- deterministic syntax and symbol identity;
- static type and effect checking;
- low-cost machine generation and transformation;
- reproducible verification paths;
- direct native execution on Linux x86-64.

Λ0 is the current compact core language and compiler implementation used by ALMANAL.

### Why Hangul identifiers are used

Λ0 0.40 uses modern Hangul syllables for many compact identifiers. This is an engineering choice for representation economy, not a claim that Hangul is intrinsically more efficient than ASCII.

A modern Hangul syllable occupies three bytes in UTF-8. When one syllable replaces an ASCII identifier longer than three bytes, source size can decrease. More importantly, Hangul provides a large set of distinct single-syllable symbols, so many identifiers can remain one textual symbol long instead of requiring multi-character mnemonic names.

For machine-generated and machine-transformed code, that gives Λ a dense identifier space while retaining deterministic textual identity. Tokenizer cost is model-dependent, so token savings are measured rather than inferred from character count alone.

## Using ALMANAL with an AI

ALMANAL is primarily intended to be used as AI context rather than as a human-facing interactive application.

A minimal instruction pattern is:

```text
Use the supplied ALMANAL framework.
Generate an algorithm for the following problem, or improve the supplied algorithm.
Apply ALMANAL's verification and falsification procedures as part of the work.
Return the final algorithm and the requested implementation/output.
```

For difficult tasks, use a model with enough context capacity and reasoning capability to retain both ALMANAL and the target problem during the same work session.

## Λ compiler

The included native compiler targets **Linux x86-64**.

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

## Native ALMANAL executable

The repository also contains the native ALMANAL executable and its Λ source. These provide the machine-level implementation and self-check path; they are not the primary human-facing usage model described above.

Mode `0` performs the built-in self-check:

```bash
printf '0\n' | ./almanal/almanal
```

A successful run prints:

```text
ALMANAL-LAMBDA-SELFCHECK-PASS
```

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

## Citation

Citation metadata is provided in [`CITATION.cff`](CITATION.cff).

## License

See [`LICENSE`](LICENSE).
