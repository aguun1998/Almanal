# Λ0 0.25 L4 Canonical Surface

Status: **CANONICAL / AI-SAFE / semantics unchanged**

L4 is the canonical textual surface of Λ0 0.25. It replaces the bootstrap-era
verbose S-expression spelling as the checked-in source representation while
preserving the same semantic token stream and AST.

The design objective is not maximum compression at any cost. It is the Pareto
point between source/context cost and deterministic local decoding:

`compactness + semantic visibility + local recoverability + hard failure on ambiguity`

## 1. Marker and structural checkpoints

Every canonical file begins exactly with:

```text
~L4
```

Each top-level form occupies one physical line. Parentheses remain explicit
structural checkpoints. L4 does not rely on implicit fixed arity to infer tree
shape.

A run of closing parentheses may be represented by `..N`, where `N` is the
exact positive number of closes. The decoder tracks current structural depth
and rejects a marker whose count exceeds that depth. The line must finish at
exactly depth zero. Thus `..N` is checked run-length encoding, not guessed
repair.

## 2. Stable one-byte semantic atoms

The following 45 spellings are canonical aliases:

| Verbose | L4 | Verbose | L4 | Verbose | L4 |
|---|---:|---|---:|---|---:|
| `fn` | `F` | `call` | `C` | `effects` | `E` |
| `i64` | `I` | `u64` | `U` | `bool` | `B` |
| `unit` | `V` | `str` | `S` | `tuple` | `T` |
| `array` | `A` | `let` | `L` | `if` | `?` |
| `set` | `:` | `seq` | `;` | `while` | `W` |
| `for` | `O` | `assert` | `!` | `and` | `&` |
| `or` | `|` | `not` | `~` | `true` | `Y` |
| `false` | `N` | `array_get` | `G` | `array_push` | `P` |
| `array_set` | `H` | `array_pop` | `D` | `array_clone` | `K` |
| `len` | `M` | `byte_at` | `@` | `slice` | `X` |
| `cat` | `Q` | `print` | `J` | `print_i64` | `Z` |
| `read_i64` | `R` | `show_i64` | `$` | `read_text` | `{` |
| `write_text` | `}` | `fs_read` | `[` | `fs_write` | `]` |
| `unsafe_proc` | `^` | `proc` | `_` | `run` | `` ` `` |
| `first` | `,` | `second` | `'` | `>=` | `#` |

Existing arithmetic/comparison operator atoms such as `+`, `-`, `*`, `/`,
`%`, `=`, `!=`, `<`, `<=`, and `>` keep their existing spellings.

`_` is intentionally not treated as a self-delimiting byte inside ordinary
identifiers because underscores are common in semantic names. `!` is
special-cased so `!=` remains one ordinary operator atom.

## 3. Local mnemonic symbol aliases

Repeated identifiers may be compressed without deleting their semantic name.
Within one top-level form:

```text
semantic_name.s
```

binds local alias `s` to `semantic_name`, and later:

```text
.s
```

means exactly that same identifier.

Rules:

- aliases are one lowercase ASCII letter;
- the letter must occur in the original identifier (mnemonic constraint);
- scope ends at the top-level line boundary;
- rebinding an alias to a different name is an error;
- repeating the same `name.x` binding is a legal semantic refresh;
- the canonical encoder refreshes the full name before more than 1024 encoded
  characters pass without it.

This deliberately rejects opaque global numbering such as `v173` as the
canonical AI-facing representation. The semantic name remains locally visible
while repeated lexical cost is removed.

## 4. Escapes

An ordinary identifier that would collide with a one-byte semantic alias, begin
with `\\`, begin with `.`, contain `.`, or otherwise be ambiguous in L4 is
prefixed with one `\\` in the surface representation. The decoder removes
exactly that surface escape.

Quoted strings are not rewritten by semantic aliasing or local-symbol aliasing.
Literal newlines inside canonical string tokens are forbidden; escaped newline
bytes remain ordinary string content according to the underlying Λ string
syntax.

## 5. Deterministic rejection

The native decoder must reject rather than infer or repair at least:

- missing `~L4` when L4 parsing is requested;
- unknown `.x` references;
- non-mnemonic `name.x` bindings;
- alias collision inside a top-level form;
- invalid or over-deep `..N` close markers;
- extra closing parentheses;
- nonzero depth at the end of a top-level line;
- unterminated quoted strings.

The compiler may also accept the old verbose projection for bootstrap and
compatibility, but checked-in canonical compiler and ALMANAL sources use L4.

## 6. Trust path

`compiler/native/expr_elfgen.l0` contains the native L4 decoder and consumes
canonical L4 directly at Stage1 and later. `tools/l0-surface.py` is only an
authoring/reference encoder-decoder and is not a compiler dependency.

The handwritten Stage0 evaluator predates L4. Bootstrap therefore executes the
Λ-written `bootstrap/l4_decode.l0` bridge to obtain a verbose seed view of the
canonical compiler. Because Stage0 deliberately uses a bounded non-reclaiming
256 MiB bump heap, `bootstrap/l4-decode-file.sh` feeds the decoder chunks of 24
complete L4 top-level lines and resets Stage0 between chunks. L4 makes this an
exact semantic boundary: one physical line is exactly one top-level form and
one alias scope. Concatenating the decoded forms is byte-identical to decoding
the same canonical file as one logical unit. This bridge does not change
language semantics and is outside Stage1+ parsing. Embedding a duplicate L4
parser into handwritten assembly, or enlarging the seed heap just for surface
decode, was rejected by SCA.

## 7. Canonicality and equivalence

For every valid canonical source `L`:

```text
DecodeL4(L) = verbose semantic token stream
AST(DecodeL4(L)) = AST(L)
```

The migration gates are:

1. deterministic L4 decode;
2. native Stage1+ direct L4 parsing;
3. Stage1 = Stage2 = Stage3;
4. compiler-source mutation changes generated machine code;
5. semantic authority and public-bypass regressions remain green;
6. ALMANAL compiles and selfchecks from canonical L4;
7. malformed L4 is rejected;
8. source/context savings are material without an AI-readability regression.

## 8. Deliberately rejected denser surfaces

The following are not canonical even if smaller:

- implicit fixed-arity prefix parsing that removes structural checkpoints;
- opaque global symbol IDs replacing semantic names;
- context-dependent glyph meanings;
- silent close/alias repair;
- packed binary/base-N source presented directly to an AI as code.

Such encodings may exist as transport/serialization formats, but they are not
the Λ AI-facing source language.
