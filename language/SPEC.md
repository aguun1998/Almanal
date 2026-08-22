# Λ0 0.25 — Direct-native AI-safe canonical surface

Status: current normative profile / Linux x86-64 / bounded logical authority

<!-- LANGUAGE_VERSION_MARKER=0.25 -->

This file specifies the language accepted by
`compiler/native/expr_elfgen.l0`. Historical transports and earlier profiles
are archived in `language/HISTORY.md`; known gaps are in `language/GAPS.md`.
Neither document is a normative dependency.

## Identity and trust boundary

Λ0 source is parsed, statically checked, and lowered directly to source-derived
x86-64 machine code in an ELF64 image. The canonical path does not emit foreign
source, invoke a C/C++/Python compiler, copy an existing ELF prefix, or reproduce
a capsule/payload.

The handwritten `bootstrap/l0i.S` evaluator is an explicit seed, not the
canonical compiler. Because the seed predates the 0.25 surface, bootstrap uses
the Λ-written `bootstrap/l4_decode.l0` bridge exactly once to obtain the seed
view of the checked-in canonical L4 compiler. Stage1 and every later compiler
consume L4 directly. Stage1 checks and rebuilds the same canonical source as
Stage2; Stage2 rebuilds Stage3. Byte equality is a reproducibility/fixed-point
guarantee, not an independent proof that the seed or compiler is correct.

## Canonical surface and evaluation rules

The **canonical 0.25 source surface is L4**, identified by the exact first line
`~L4`. L4 is a deterministic, lossless semantic recoding of the bootstrap-era
S-expression token stream. The language semantics, AST, typing, effects,
evaluation order, world/claim/proof authority, and native lowering are
unchanged by the surface migration. The verbose S-expression spelling remains
accepted as a compatibility and bootstrap projection, but it is not canonical.

L4 keeps parentheses as explicit structural checkpoints while reducing lexical
overhead with four mechanisms:

1. 45 high-frequency semantic atoms have stable one-byte spellings;
2. local repeated identifiers may bind a mnemonic alias as `name.x` and later
   refer to it as `.x`; aliases are scoped to one top-level form, the alias
   letter must occur in the original name, collisions are rejected, and the
   canonical encoder re-exposes the full name within 1024 encoded characters;
3. alias-conflicting ordinary identifiers use a leading `\` escape;
4. an exact close run may be encoded as `..N`; the decoder accepts it only when
   `N > 0` and `N` does not exceed the current structural depth. Every canonical
   top-level form occupies one line and must end at depth zero.

Strings are never alias-rewritten. Unknown local aliases, non-mnemonic alias
bindings, alias collisions, malformed close counts, unterminated strings, and
unbalanced structure fail deterministically rather than being guessed or
repaired. `language/COMPACT_NOTATION.md` is the normative surface mapping.

Evaluation order is left-to-right. `and` and `or` short-circuit. Runtime
assertion, bounds, malformed input, arithmetic fault, or semantic-check failure
terminates nonzero.

`i64` arithmetic is two's-complement 64-bit runtime arithmetic. Division by
zero traps. `u64` literals use the canonical decimal suffix `u`. Strings are
byte strings; `(len S)` in the verbose projection, equivalently `(MS)` in L4,
counts bytes. Arrays are mutable; tuples and sums are structural values.

## Types

Scalar types are `i64`, `u64`, `bool`, `unit`, and `str`. Structural types
recursively contain types:

```text
(array T)
(tuple A B)
(sum A B)
```

There are no implicit conversions or overload-based coercions. Function
arguments and returns, assignments, array elements, tuple projections, and sum
arms must match structurally. Conditions of `if`, `while`, `and`, `or`, and
`not` must be `bool`.

## Top-level forms

### Functions

```lisp
(fn NAME ((PARAM TYPE)...) RETURN (effects EFFECT...) BODY)
```

Exactly one `(fn main () i64 ... BODY)` is required. Calls are
`(call NAME ARG...)`. Functions may recurse. Parameters use the native stack
frame; lexical `let` values use frame-local slots.

### Worlds, claims, and P0 proofs

```lisp
(world ID OBSERVER MODEL DOMAIN COST PROOF_SYSTEM)
(claim ID WORLD STATUS PAYLOAD)
(proof ID CLOSED_P0_BOOL)
```

All top-level IDs share one namespace and must be unique. A claim must reference
a declared world. The four statuses are `proven`, `refuted`, `unknown`, and
`both`.

- `proven` requires a `p0` world and a closed P0 boolean that evaluates true.
- `refuted` requires a `p0` world and a closed P0 boolean that evaluates false.
- `unknown` and `both` require an opaque string payload and grant no proof
  authority. In particular, `UNKNOWN != FALSE`.
- `(proof ID E)` requires closed P0 expression `E` to evaluate true.

P0 contains only `i64`, `bool`, checked `+ - * / %`, comparisons, boolean
operators, and `if`. Overflow, division by zero, an open term, or an unsupported
form rejects compilation. Worlds/claims/proofs are checked and erased before
runtime code emission. World descriptors are identifiers, not executable
axioms; self-improvement never self-authorizes.

## Expressions

```lisp
(let NAME TYPE INIT BODY)       (set NAME VALUE)
(if COND YES NO)                (seq EXPR...)
(while COND BODY)               (for NAME START END BODY)
(call NAME ARG...)
```

`for` evaluates `START` and `END` once and iterates over the half-open interval
`[START, END)`. `while`, `for`, and `set` return `unit`. `seq` returns its last
expression.

Arithmetic and comparison operators are `+ - * / % = != < <= > >= u<=`, `and
or not`, `bxor`, and `rotl`. Arithmetic operands must have one identical numeric
type; `%` requires an integer type. `bxor` takes two `u64`; `rotl` takes
`u64, i64`. Equality requires identical non-unit types. Ordered comparison
with `< <= > >=` requires `i64`; `u<=` is the explicit full-width unsigned
less-than-or-equal operation.

Structural data:

```lisp
(tuple A B)                     (first PAIR) (second PAIR)
(in0 A B VALUE)                 (in1 A B VALUE)
(case SUM LEFT_NAME LEFT_EXPR RIGHT_NAME RIGHT_EXPR)
(array T)                       (array_get A I)
(array_push A V)                (array_set A I V)
(array_pop A)                   (array_clone A)
```

Binary `case` is exhaustive by construction and both arms must return the same
type. Array indices are `i64`; invalid indices and empty pop trap.

Strings, memory, process arguments, and output:

```lisp
(len X)                         (cat A B)
(byte_at S I)                   (slice S START COUNT)
(str_from_bytes A)              (show_i64 N)
(alloc N)                       (data S)
(load8 ADDRESS OFFSET)          (store8 ADDRESS OFFSET VALUE)
(arg_count)                     (arg I)
(print S)                       (eprint S)
(print_i64 N)
(read_i64)
(assert BOOL)                   (assert BOOL EXIT_CODE)
(trap EXIT_CODE)
```

`assert` returns `unit` when true and exits with code 122 by default when
false; its optional `i64` code classifies the failure. `trap` unconditionally
exits with its `i64` code. `eprint` writes a string plus newline to standard
error. These exits and writes are emitted as native Linux syscalls.

File/process operations:

```lisp
(read_text PATH)                (write_text PATH DATA)
(file_read PATH ADDRESS COUNT)  (file_write PATH ADDRESS COUNT)
(run COMMAND)
```

The native runtime implements these operations with Linux x86-64 syscalls;
`run` uses direct process creation/execution/wait machinery and is not a
compiler backend.

## Effects

Allowed effects are `io`, `fs_read`, `fs_write`, `proc`, `unsafe_proc`, and
`args`. Declarations reject unknown or duplicate effects. A caller must declare
every effect declared by each callee.

- `print`, `eprint`, `print_i64`, `read_i64` require `io`.
- `arg`, `arg_count` require `args`.
- `read_text`, `file_read` require `fs_read`.
- `write_text`, `file_write` require `fs_write`.
- `run` requires `unsafe_proc`.

Effects are operational authority only. They are not logical evidence, proof
authority, or permission for a candidate to promote itself.

## Compiler interface and guarantees

```text
expr_elfgen SOURCE_TEXT [OUTPUT]
expr_elfgen build SOURCE_FILE OUTPUT
expr_elfgen profile SOURCE_FILE OUTPUT
expr_elfgen check SOURCE_FILE
expr_elfgen check-types SOURCE_FILE
expr_elfgen check-effects SOURCE_FILE
expr_elfgen emit-ir SOURCE_FILE
```

Both ordinary entry paths perform the same static semantic validation before
emitting ELF. There is no public semantic-bypass option. The Stage0 evaluator
has one internal, source-identity-bound exception for compiling the exact
checked-in compiler source; generated Stage1 and later compilers do not contain
that exception.

The three `check` commands do not emit an ELF: `check` runs full validation,
while the two narrower commands isolate type and effect failures. Diagnostics
use stable error codes and report file, byte span, expected/actual values, and
context where available. `emit-ir` first performs full validation and then
prints the deterministic `L0-TYPED-IR-V1` semantic/lowering boundary.

`profile` performs a normal fully checked native build and writes compact
debug counters to standard error. `parse_us` measures a complete structural
syntax walk, `semantic_us` full type/effect validation, `lower_us` function
metadata lowering, and `emit_us` x86/ELF emission. `ast_nodes` and
`types_checked` count the corresponding visited source nodes; `alloc_bytes`
is the compiler's `brk`-arena growth during the measured pipeline (large mmap
objects are excluded). `emit_bytes`, `lowered_functions`, and `candidates` are
also reported. The ordinary `build` path does not collect these counters.

The compiler emits one ELF64 `PT_LOAD` image with source-derived machine-code
bytes and no copied native prefix. Changing executable source semantics must
change emitted code. Native self-host completion requires Stage2 and Stage3
byte equality plus executable regression evidence; it does not imply
independent diverse double compilation.
