# Bitlang Explicit

Bitlang Explicit is the **fully explicit normalized form of Bitlang** produced by the Bitlang preprocessor.

It is not a separate semantic language from Bitlang source. Ordinary Bitlang source can explicitly write every canonical property used here; the difference is that source may omit or abbreviate them, while Bitlang Explicit requires the applicable final states to be explicit.

This repository is the canonical home for **the fully explicit Explicit form's specifications, implementation, and tests**. It is managed separately from the source-facing repository as a stage/responsibility boundary.

## Specifications

- [LANGUAGE_SPEC.ja.md](LANGUAGE_SPEC.ja.md) — language-wide rules, identifier normalization, nullability, and stage boundaries
- [PROPERTIES.ja.md](PROPERTIES.ja.md) — canonical explicit property model
- [BORROW_STATE.ja.md](BORROW_STATE.ja.md) — canonical borrow-state model

## Stage relationship

```text
Bitlang source
    -> Bitlang preprocessor
    -> Bitlang Explicit
    -> Bitlang Lowerer
    -> Bitlang Low
```

Bitlang source may omit or derive information for convenience. Bitlang Explicit must preserve the resolved semantic state explicitly so later stages do not need to reconstruct omitted source semantics.
