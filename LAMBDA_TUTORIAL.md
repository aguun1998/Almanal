# Λ0 0.25 / L4 Tutorial

> **Λ is an AI-oriented programming language. Human convenience is intentionally not a primary design goal.**
>
> Λ0 removes or de-emphasizes many conventions that exist mainly to make programming languages comfortable for human authors: long keywords, verbose repeated identifiers, formatting freedom, implicit coercions, and ambiguity-tolerant syntax. The canonical L4 surface instead prioritizes low context cost, deterministic decoding, explicit structure, static verifiability, and direct native execution. Humans can read and write Λ, but the language is designed first for AI systems that generate, inspect, transform, and verify code.

Current profile: **Λ0 0.25 / L4 canonical surface**  
Target: **Linux x86-64**  
Canonical compiler: `compiler/native/expr_elfgen.l0`

---

## 1. What Λ is

Λ0 is a statically checked, direct-native language. Canonical source is parsed and lowered directly to x86-64 machine code inside an ELF64 executable.

The canonical path does **not** translate Λ into C, C++, Python, or another source language before compilation.

Its current core includes:

- `i64`, `u64`, `bool`, `unit`, `str`
- arrays, tuples, and binary tagged sums
- functions and recursion
- lexical `let`, mutable `set`, `if`, `seq`, `while`, and counted `for`
- file and process operations
- explicit effect declarations
- compile-time worlds, claims, and bounded P0 proofs
- direct Linux x86-64 ELF emission
- a canonical compact source representation called **L4**

Evaluation order is left-to-right. `and` and `or` short-circuit. Type mismatches, invalid bounds, malformed input, arithmetic faults, failed assertions, and semantic-check failures terminate with an error rather than being silently repaired.

---

## 2. Two textual views: verbose projection and canonical L4

The easiest way to learn Λ semantics is to read the verbose S-expression projection first.

Example:

```lisp
(fn main () i64
  (effects io)
  (seq
    (print "Hello from Lambda")
    0))
```

Λ0 0.25 still accepts this verbose spelling for compatibility, but it is **not canonical**.

Canonical files use **L4** and begin with the exact first line:

```text
~L4
```

L4 preserves the same semantic token stream and AST while reducing lexical/context cost. For example, common semantic words have stable one-byte atoms:

```text
fn        -> F
call      -> C
effects   -> E
i64       -> I
u64       -> U
bool      -> B
unit      -> V
str       -> S
let       -> L
if        -> ?
set       -> :
seq       -> ;
while     -> W
for       -> O
assert    -> !
and       -> &
or        -> |
not       -> ~
array     -> A
array_get -> G
array_push-> P
len       -> M
print     -> J
```

Parentheses are deliberately retained as structural checkpoints. L4 is compact, but it is not an implicit-arity bytecode notation.

---

## 3. Program structure

A program is built from top-level forms. Exactly one `main` function is required.

Verbose grammar:

```lisp
(fn NAME ((PARAM TYPE)...) RETURN (effects EFFECT...) BODY)
```

Minimal integer-returning program:

```lisp
(fn main () i64
  (effects)
  0)
```

A function call is:

```lisp
(call FUNCTION ARG...)
```

Example:

```lisp
(fn add ((a i64) (b i64)) i64
  (effects)
  (+ a b))

(fn main () i64
  (effects)
  (call add 20 22))
```

Functions may recurse.

---

## 4. Types

Scalar types:

```text
i64
u64
bool
unit
str
```

Structural types:

```lisp
(array T)
(tuple A B)
(sum A B)
```

Λ has no implicit numeric conversions or overload-based coercions. Function arguments, returns, assignments, array elements, tuple components, and sum arms must match structurally.

Conditions used by `if`, `while`, `and`, `or`, and `not` must be `bool`.

### Integer notes

- `i64` uses two's-complement 64-bit runtime arithmetic.
- division by zero traps.
- canonical `u64` decimal literals use the suffix `u`.

Example:

```text
42
42u
```

---

## 5. Local values and mutation

Lexical binding:

```lisp
(let NAME TYPE INIT BODY)
```

Example:

```lisp
(let x i64 10
  (+ x 5))
```

Mutation:

```lisp
(set NAME VALUE)
```

`set` returns `unit`.

---

## 6. Control flow

Conditional:

```lisp
(if CONDITION YES NO)
```

Sequence:

```lisp
(seq EXPR...)
```

`seq` evaluates expressions left-to-right and returns the last expression.

Loop:

```lisp
(while CONDITION BODY)
```

Counted loop:

```lisp
(for NAME START END BODY)
```

`for` evaluates `START` and `END` once and iterates over the half-open interval `[START, END)`.

`while`, `for`, and `set` return `unit`.

---

## 7. Operators

Arithmetic and comparisons:

```text
+  -  *  /  %
=  !=
<  <=  >  >=
u<=
```

Boolean operators:

```text
and  or  not
```

Bit operations currently include:

```text
bxor
rotl
```

Arithmetic operands must use one identical numeric type. `< <= > >=` operate on `i64`; `u<=` is the explicit full-width unsigned comparison.

---

## 8. Tuples and tagged sums

Tuple construction:

```lisp
(tuple A B)
```

Projection:

```lisp
(first PAIR)
(second PAIR)
```

Binary tagged sums:

```lisp
(in0 A B VALUE)
(in1 A B VALUE)
```

Exhaustive case analysis:

```lisp
(case SUM LEFT_NAME LEFT_EXPR RIGHT_NAME RIGHT_EXPR)
```

Both `case` arms must return the same type.

---

## 9. Arrays

Array type:

```lisp
(array T)
```

Operations:

```lisp
(array_get A I)
(array_push A V)
(array_set A I V)
(array_pop A)
(array_clone A)
```

Indices are `i64`. Invalid indices and popping an empty array trap.

---

## 10. Strings, bytes, memory, and output

String/byte operations:

```lisp
(len X)
(cat A B)
(byte_at S I)
(slice S START COUNT)
(str_from_bytes A)
(show_i64 N)
```

Low-level memory operations:

```lisp
(alloc N)
(data S)
(load8 ADDRESS OFFSET)
(store8 ADDRESS OFFSET VALUE)
```

Arguments and output:

```lisp
(arg_count)
(arg I)
(print S)
(eprint S)
(print_i64 N)
(read_i64)
```

Assertions and explicit traps:

```lisp
(assert BOOL)
(assert BOOL EXIT_CODE)
(trap EXIT_CODE)
```

A failed one-argument `assert` exits with code `122` by default.

---

## 11. Files and processes

File operations:

```lisp
(read_text PATH)
(write_text PATH DATA)
(file_read PATH ADDRESS COUNT)
(file_write PATH ADDRESS COUNT)
```

Process execution:

```lisp
(run COMMAND)
```

These are implemented with native Linux x86-64 system-call machinery. `run` is a runtime operation, not a compiler backend.

---

## 12. Effects

Functions explicitly declare operational authority.

Available effects:

```text
io
fs_read
fs_write
proc
unsafe_proc
args
```

Examples of requirements:

```text
print / eprint / print_i64 / read_i64 -> io
arg / arg_count                        -> args
read_text / file_read                  -> fs_read
write_text / file_write                -> fs_write
run                                    -> unsafe_proc
```

A caller must declare every effect declared by each callee. Unknown or duplicate effects are rejected.

Effects grant runtime authority only; they are not logical proof authority.

---

## 13. Worlds, claims, and bounded P0 proofs

Λ0 also contains compile-time logical metadata:

```lisp
(world ID OBSERVER MODEL DOMAIN COST PROOF_SYSTEM)
(claim ID WORLD STATUS PAYLOAD)
(proof ID CLOSED_P0_BOOL)
```

Claim statuses:

```text
proven
refuted
unknown
both
```

Important rule:

```text
UNKNOWN != FALSE
```

`proven` and `refuted` require a `p0` world and a closed P0 boolean whose checked result matches the status. `unknown` and `both` carry opaque string payloads and grant no proof authority.

P0 is intentionally bounded. It contains only `i64`, `bool`, checked arithmetic, comparisons, boolean operators, and `if`. Overflow, division by zero, unsupported forms, or open terms reject compilation.

Worlds, claims, and proofs are checked at compile time and erased before runtime code emission.

---

## 14. L4 local identifier aliases

L4 can compress repeated identifiers without replacing semantic names with opaque global numbers.

Inside one top-level form:

```text
semantic_name.s
```

binds alias `s` to that identifier. Later:

```text
.s
```

means the same identifier.

Rules:

- alias is one lowercase ASCII letter;
- that letter must occur in the original identifier;
- scope ends at the current top-level line;
- rebinding the alias to another name is an error;
- repeating the same binding is allowed as a semantic refresh;
- canonical encoding re-exposes the full semantic name within 1024 encoded characters.

This keeps source compact without making identifiers permanently opaque to an AI reader.

---

## 15. L4 exact close-run encoding

A run of closing parentheses can be written as:

```text
..N
```

where `N` is the exact positive number of closes.

The decoder tracks structural depth. If `N` exceeds the current depth, compilation fails. Every canonical top-level line must finish at depth zero.

Therefore `..N` is checked run-length encoding, not automatic syntax repair.

---

## 16. L4 rejection policy

The canonical surface rejects ambiguity instead of guessing intent.

Examples that must fail include:

- unknown `.x` aliases;
- non-mnemonic alias bindings;
- alias collisions;
- invalid or over-deep `..N` markers;
- extra closing parentheses;
- nonzero depth at the end of a top-level line;
- unterminated strings.

This is a central Λ design rule: **smaller syntax is acceptable only while deterministic decoding and semantic visibility remain intact.**

---

## 17. Compiler commands

The native compiler interface is:

```text
expr_elfgen SOURCE_TEXT [OUTPUT]
expr_elfgen build SOURCE_FILE OUTPUT
expr_elfgen profile SOURCE_FILE OUTPUT
expr_elfgen check SOURCE_FILE
expr_elfgen check-types SOURCE_FILE
expr_elfgen check-effects SOURCE_FILE
expr_elfgen emit-ir SOURCE_FILE
```

Typical workflow:

```bash
./compiler/l0c check program.l0
./compiler/l0c build program.l0 program
./program
```

Useful inspection commands:

```bash
./compiler/l0c check-types program.l0
./compiler/l0c check-effects program.l0
./compiler/l0c emit-ir program.l0
./compiler/l0c profile program.l0 program
```

`check` performs full static validation without emitting an ELF. `emit-ir` prints the deterministic typed IR boundary after validation. `profile` performs a normal checked build and reports compiler phase counters.

---

## 18. A small worked example

Verbose projection:

```lisp
(fn square ((x i64)) i64
  (effects)
  (* x x))

(fn main () i64
  (effects io)
  (let answer i64 (call square 7)
    (seq
      (print_i64 answer)
      0)))
```

Semantics:

1. `square` receives one `i64` and returns one `i64`.
2. `main` declares `io` because it calls `print_i64`.
3. `answer` is statically typed as `i64`.
4. evaluation is left-to-right.
5. the program prints `49` and returns `0`.

In canonical checked-in source, the same semantic structure is represented with the L4 marker, one-byte semantic atoms, optional mnemonic aliases, and checked close runs.

---

## 19. What Λ intentionally does not optimize for

Λ0 0.25 is not trying to maximize:

- human familiarity;
- English-like syntax;
- beginner ergonomics;
- permissive parsing;
- implicit conversions;
- decorative syntax;
- compatibility with conventional compiler pipelines.

Its design target is instead:

```text
AI generation and transformation
+ low source/context cost
+ explicit semantic structure
+ deterministic parsing
+ static rejection of invalid programs
+ reproducible native compilation
```

That trade-off is deliberate. Human readability is useful only where it also helps semantic reliability; it is not an independent governing constraint.

---

## 20. Current boundary

Λ0 0.25 is a bounded current profile, not a claim of universal language completeness. Its canonical compiler targets Linux x86-64, its P0 proof language is intentionally restricted, and its native self-host fixed point is a reproducibility property rather than an independent proof of compiler correctness.

For exact normative behavior, refer to:

```text
language/SPEC.md
language/COMPACT_NOTATION.md
```

Those files override this tutorial if any wording here is ambiguous.
