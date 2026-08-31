# ALMANAL Lambda Grammar & AI Quick Reference

Status: AI-facing grammar reference for the canonical **ALMANAL Lambda** release.

This file describes source structure, not the full implementation. The canonical source extension is `.lh` and canonical Unicode normalization is NFC.

## 1. Canonical source

Canonical source has **zero whitespace outside quoted strings**.
Whitespace may be accepted in noncanonical input at valid expression boundaries, but it carries no syntax information.

```text
Source      := Dictionary Body
Dictionary  := ';' Symbol (';' Symbol)* ';;'
Body        := Expr*
```

`;;` terminates the self-contained symbol dictionary. No external map file is required.

## 2. Expression classes

```text
Expr := FixedForm
      | StructuralSyllable
      | PagedSymbol
      | ExtendedForm
      | Atom
      | Integer
      | String
      | ExplicitForm
```

The grammar is self-delimiting: implicit forms end by encoded arity; explicit forms end by explicit parentheses.

## 3. Implicit structure

A precomposed Hangul syllable encodes symbol identity and local structure.

Let the syllable decomposition be `(L,V,T)`:

- `(L,V)` selects a base symbol slot.
- `T = 0`: atomic symbol.
- `T = 1..17`: ordinary form with arity `T-1`.
- `T = 18..27`: function-call form with arity `T-18`; decoding is conceptually `(가 f child...)`.
- reserved base `히`: anonymous list/tuple space.

An implicit form consumes exactly its encoded number of child expressions and then closes automatically.

```text
F G a b H c d
```

If `F`, `G`, and `H` each encode arity 2, `G` closes after `a,b`, `H` after `c,d`, and `F` after both children. No `)))` suffix exists.

## 4. Fixed structural vowel opcodes

These standalone compatibility vowels encode **head + fixed arity + structure start** in one codepoint.

| opcode | head | arity |
|---|---|---:|
| `ㅏ` | `마` | 4 |
| `ㅐ` | `=` | 2 |
| `ㅑ` | `사` | 1 |
| `ㅒ` | `차` | 2 |
| `ㅓ` | `아` | 1 |
| `ㅔ` | `카` | 2 |
| `ㅕ` | `하` | 1 |
| `ㅖ` | `or` | 2 |
| `ㅗ` | `저` | 2 |
| `ㅘ` | `고` | 2 |
| `ㅙ` | `fn` | 5 |
| `ㅚ` | `거` | 1 |
| `ㅛ` | `파` | 2 |
| `ㅜ` | `+` | 2 |
| `ㅝ` | `<` | 2 |
| `ㅞ` | `서` | 0 |
| `ㅟ` | `커` | 1 |
| `ㅠ` | `퍼` | 1 |
| `ㅡ` | `>` | 2 |
| `ㅢ` | `>=` | 2 |
| `ㅣ` | `오` | 1 |

Do not infer a different arity for these opcodes.

## 5. Standalone consonants and extended forms

Standalone consonants `ㄱ ㄴ ㄷ ㄹ ㅁ ㅂ ㅅ ㅇ ㅈ ㅊ ㅋ ㅌ ㅍ` provide symbol-table pages beyond the first precomposed-syllable page.

`ㅎ` is the extended-structure prefix for uncommon large arities. The bootstrap-compatible forms are:

```text
ㅎfN:
ㅎcN:
```

where `N` is the explicit arity encoded by the extension form.

## 6. Explicit parentheses

Parentheses are valid but optional.

```text
ExplicitForm := '(' ExplicitContent ')'
```

Rules are strict:

- explicit `(` closes **only** with its matching explicit `)`;
- `)` never closes an implicit structure;
- implicit structures close **only** by their encoded arity;
- explicit regions may nest;
- parentheses inside quoted strings are data.

Use explicit parentheses only when the boundary itself is useful or when an implicit structure is inappropriate.

Example hybrid subtree:

```text
F G a b (* c d)
```

The outer compact forms close by arity. `(* c d)` closes only at its matching `)`.

## 7. Literals and atoms

- one decimal digit may appear directly as ASCII;
- other integer literals use `#<decimal>:`;
- quoted UTF-8 strings retain their existing string syntax and escapes;
- whitespace and parentheses inside strings are data;
- one-letter ASCII local atoms may remain raw where that representation is cheaper.

Examples:

```text
7
#137:
"text (data)"
```

## 8. Whitespace and Unicode

Canonical `.lh`:

- NFC Unicode;
- zero space/tab/LF/CR outside strings.

Noncanonical input may contain ignorable whitespace where accepted by the parser. Canonical emission removes it.

Do not substitute visually similar decomposed Jamo sequences for canonical precomposed syllables.

## 9. Effects and host extension

The generic host bridge is:

```text
홰(op,args6_ptr)->i64
```

It requires effect `외`.

Language meaning: perform one host-native operation with six machine-word arguments stored at `args6_ptr`.

Current `linux-x86_64` backend mapping only:

- `op` = Linux x86-64 syscall number;
- result = raw `rax`;
- this backend mapping is not target-independent Λ semantics.

Prefer implementing new capabilities in Λ over adding new compiler intrinsics.

## 10. AI validation loop

First discovery:

```text
./ALMANAL_Lambda
```

Recommended generation loop:

```text
check SOURCE
check-types SOURCE
check-effects SOURCE
build SOURCE OUTPUT
verify SOURCE OUTPUT
```

Other available operations are discoverable from the no-argument output.

Compiler diagnostics such as `E_TYPE_CALL_ARG`, `E_EFFECT_UNDECLARED`, `CHECK-PASS`, and `VERIFY-PASS` should be treated as machine feedback. Generate exact codepoints, run checks, and repair from diagnostics rather than guessing after an error.

## 11. AI-specific notes

- Source-byte density and LLM-token density are not identical; tokenizer behavior is model-dependent.
- Do not rely on visual similarity of rare Hangul syllables. Copy exact Unicode codepoints when modifying existing source.
- Fixed structural opcodes and encoded arities are semantic anchors. They should be preserved exactly.
- If a compact edit becomes uncertain, use an explicit parenthesized subtree, validate it, then compact only when useful.

## 12. Minimal invariants

A canonical program must satisfy all of the following:

```text
NFC
no non-string whitespace
self-contained ;...;; dictionary
implicit close = arity only
explicit close = matching parenthesis only
no cross-closing between implicit/explicit structures
all required effects declared
check/check-types/check-effects pass
```
