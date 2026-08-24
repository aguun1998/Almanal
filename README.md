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

ALMANAL was subsequently moved onto Λ as its native implementation substrate. The current repository therefore contains both the ALMANAL system and the Λ compiler path that grew out of ALMANAL's own optimization process.

## Purpose of ALMANAL

ALMANAL is intended to be supplied to a sufficiently capable AI model as an algorithm-generation and algorithm-improvement framework.

Typical use:

1. Provide the ALMANAL source/context to the AI model.
2. Provide the problem, constraints, available evidence, and objective.
3. Ask the model to generate an algorithm or improve an existing one using ALMANAL.
4. Let the model use ALMANAL's internal procedures for construction, comparison, falsification, verification, reuse, and burden reduction as needed.
5. Request the resulting algorithm, implementation, proof/evidence summary, or other required output.

The verification machinery exists to support this generation/improvement task: it is used to reject bad candidates, narrow claims, find counterexamples, and decide whether a proposed change is justified.

## ALMANAL architecture

ALMANAL separates the object being improved from the evidence and context used to judge it. A useful high-level representation is:

```text
Candidate = (
    TaskBinding T,
    ContextGraph C,
    AlgorithmCore A,
    EvidenceGraph E
)
```

The four parts have different roles:

- **TaskBinding** — the objective, constraints, success conditions, and task-specific contract.
- **ContextGraph** — assumptions, facts, dependencies, reusable knowledge, and the context required by the current task.
- **AlgorithmCore** — the candidate algorithm or transformation currently being constructed or improved.
- **EvidenceGraph** — proofs, checks, observations, counterexamples, provenance, and other evidence relevant to claims about the candidate.

Keeping these concerns separate is deliberate. Changing an algorithm does not require treating all context as new, and adding evidence does not require rewriting the algorithm. Improvement can therefore be expressed as localized changes to context, algorithm, or evidence rather than as an unconditional full restart.

At a high level, an ALMANAL-guided improvement cycle is:

```text
Problem / objective
        |
        v
Bind task and relevant context
        |
        v
Reuse what is already sufficient
        |
        v
Generate or modify candidate algorithm
        |
        v
Attempt falsification / counterexample search
        |
        v
Update evidence and claim scope
        |
        v
Compare benefit against burden
        |
        v
Accept, reject, narrow, or leave unresolved
```

The process is intentionally selective. If a result can be closed deductively, unnecessary experiment is avoided. If existing structure can be reused, regeneration is avoided. If a proposed improvement adds more time, memory, verification, or maintenance burden than its benefit justifies, it can be rejected even when it is locally interesting.

ALMANAL therefore treats several costs as first-class design quantities rather than optimizing runtime alone. Depending on the task, relevant burden may include execution time, memory, search effort, verification work, maintenance, external dependency, and the number of actions required to reach a justified result.

Falsification is asymmetric by design: finding one valid counterexample may be enough to refute a universal claim, while failing to find one is not automatically promoted into proof. Claims remain scoped to the evidence and contract that support them.

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

## Λ0 architecture

Λ0 uses a compact, explicitly structured source representation. The syntax is designed so that machine-generated code has clear structural boundaries and a deterministic interpretation without depending on formatting conventions intended primarily for human readers.

The native compiler path can be summarized as:

```text
Λ source
   |
   v
Parse / structural decoding
   |
   v
Type and effect checks
   |
   v
Intermediate representation
   |
   v
Native x86-64 / ELF emission
   |
   v
Linux executable
```

The compiler exposes these stages through operations such as `check`, `check-types`, `check-effects`, `emit-ir`, `build`, `verify`, and `profile`.

The canonical native path emits Linux x86-64 ELF output directly rather than using C or C++ as a required source-to-source backend. The compiler source is itself maintained in Λ (`compiler/native/expr_elfgen.l0`), while `compiler/l0c` is the corresponding native executable distributed in the repository.

This architecture keeps the language implementation close to the representation it is intended to process: the same compact language used by ALMANAL is also used to express the compiler source.

### Why Hangul identifiers are used

Λ0 0.40 uses modern Hangul syllables for many compact identifiers. This is an engineering choice for representation economy, not a claim that Hangul is intrinsically more efficient than ASCII.

A modern Hangul syllable occupies three bytes in UTF-8. When one syllable replaces an ASCII identifier longer than three bytes, source size can decrease. More importantly, Hangul provides a large set of distinct single-syllable symbols, so many identifiers can remain one textual symbol long instead of requiring multi-character mnemonic names.

For machine-generated and machine-transformed code, that gives Λ a dense identifier space while retaining deterministic textual identity. It also makes it possible to assign many internal functions and symbols compact one-character names without exhausting a small alphabet.

Tokenizer cost is model-dependent, so token savings are measured rather than inferred from character count alone.

## Relationship between ALMANAL and Λ

ALMANAL and Λ operate at different levels.

```text
AI model
   |
   | uses ALMANAL as a generation/improvement framework
   v
Algorithm design, falsification, verification, comparison
   |
   | may be represented and implemented in Λ
   v
Λ source
   |
   v
Native executable
```

ALMANAL defines the procedure for producing and improving algorithms. Λ provides a compact implementation substrate that was created later, as a consequence of optimizing ALMANAL itself.

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
