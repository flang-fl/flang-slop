# Question

May two names in an `enum T` hold the same compile-time `T` value? If so, are they distinct enum options?

[Question 045, answer A](045_Enums_and_tagged_unions.md) makes an enum value a compact identifier for a named option, rather than a stored `T` value. This leaves a useful distinction between the **identity of an option** and its associated **compile-time value**:

```flang
comp Signals = enum Color {
    STOP = Color.new(255, 0, 0);
    DANGER = Color.new(255, 0, 0);
    GO = Color.new(0, 255, 0);
}

let signal: Signals = Signals.DANGER;
// Is signal distinct from Signals.STOP despite the equal Color values?
```

The syntax is illustrative. This question does not choose how to retrieve the associated `Color` from a `Signals` value, or whether reverse lookup from `Color` to `Signals` exists.

# Research

- [Swift language reference: enumeration declarations](https://docs.swift.org/latest/documentation/the-swift-programming-language/declarations/) — Swift requires each raw-value case to have a unique raw value and provides a fallible conversion back from a raw value to its case. Its raw values are restricted to certain literal-compatible types with equality, so this rule cannot automatically extend to flang's arbitrary compile-time `T` values.
- [Swift language guide: enumerations](https://docs.swift.org/latest/documentation/the-swift-programming-language/guidedtour/) — Swift explicitly treats cases as values distinct from their associated raw values. This supports separating case identity from associated data even if flang chooses a different duplicate rule.
- [Rust Reference: enumeration discriminants](https://doc.rust-lang.org/reference/items/enumerations.html#discriminants) — Rust rejects equal discriminants for different variants because the discriminant identifies the variant. Flang's proposed compact option identifiers can remain unique even if the associated `T` values are equal; equal `T` values would therefore be a different issue from duplicate discriminants.
- [Question 045](045_Enums_and_tagged_unions.md) — Selects compact, exhaustively matchable enum options associated with compile-time values of a separate type.

# Answers

## A. Equal values may belong to distinct options

`STOP` and `DANGER` are separate `Signals` values with separate option identifiers. A match can distinguish them. Their associated `Color` values happen to be equal. This does not require a compile-time equality operation on every possible `T`, but a reverse lookup from a `Color` value could be ambiguous and would need a separate policy.

## B. Reject equal associated values

The compiler rejects `Signals` if two associated `Color` values are equal. Each value can then identify at most one option, which makes reverse lookup conceptually straightforward. This requires a well-defined compile-time equality relation for every admissible `T`, or restrictions on which types can be used in `enum T`.

## C. Equal values create aliases for one option

`STOP` and `DANGER` name the same `Signals` value and share an option identifier. This permits alternate names without a distinct runtime case. Exhaustive matching needs a rule for aliases: allowing both arms would make one unreachable, while requiring both would pretend there are two runtime possibilities. As in B, alias detection requires equality for the associated values.

# Selected answer

**A.** Equal associated compile-time values may belong to distinct enum options. Each option has its own identifier and remains distinguishable in an exhaustive match. A reverse lookup from an associated value, if supported, needs a separate rule.
