# Bitlang_preprocessed

Bitlang Preprocessed is an independent language used as the normalized output of the Bitlang preprocessor.

This repository is the canonical home for **Bitlang Preprocessed specifications, implementation, and tests**. It is managed separately from the human-writable Bitlang source language.

## Specifications

- [LANGUAGE_SPEC.ja.md](LANGUAGE_SPEC.ja.md) — language-wide rules, identifier normalization, nullability, and stage boundaries
- [PROPERTIES.ja.md](PROPERTIES.ja.md) — canonical explicit property model
- [BORROW_STATE.ja.md](BORROW_STATE.ja.md) — canonical borrow-state model

## Stage relationship

```text
Bitlang source
    -> Bitlang preprocessor
    -> Bitlang Preprocessed
    -> static analysis / compiler
    -> Bitlang Compiled
```

Bitlang source may omit or derive information for convenience. Bitlang Preprocessed must preserve the resolved semantic state explicitly so later stages do not need to reconstruct omitted source semantics.
