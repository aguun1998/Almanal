# ALMANAL Lambda

ALMANAL Lambda is an experimental AI-native language, compiler, and algorithm synthesis/reasoning system packaged as one self-contained executable.

The canonical source format is designed for compact machine use: Hangul codepoints encode symbols and structural arity, canonical source contains no non-string whitespace, and explicit parentheses are available only when an explicit boundary is useful.

The compiler emits static Linux x86-64 ELF binaries directly and includes deterministic checks for syntax, types, effects, and binary verification.

## Quick start

```sh
chmod +x ALMANAL_Lambda
./ALMANAL_Lambda
./ALMANAL_Lambda check ALMANAL_Lambda.lh
./ALMANAL_Lambda check-types ALMANAL_Lambda.lh
./ALMANAL_Lambda check-effects ALMANAL_Lambda.lh
./ALMANAL_Lambda build ALMANAL_Lambda.lh output
./ALMANAL_Lambda verify ALMANAL_Lambda.lh output
```

For the source format, see `GRAMMAR.md`. For canonical implementation properties and scope, see `SPEC.md`.
