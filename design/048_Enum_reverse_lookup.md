# Question

Should `enum T` provide a built-in way to find options from an associated `T` value?

[Question 046, answer A](046_Duplicate_enum_values.md) allows distinct options to share the same associated value. Thus converting `T` back to one enum option may be ambiguous:

```flang
comp Signals = enum Color {
    STOP = Color.new(255, 0, 0);
    DANGER = Color.new(255, 0, 0);
    GO = Color.new(0, 255, 0);
}

let red = Color.new(255, 0, 0);
// A reverse query for red could find both Signals.STOP and Signals.DANGER.
```

`T` might also have no defined equality operation. This question concerns conversion from an **associated value**, not conversion from the enum's compact integer option identifier. Any syntax below is illustrative.

# Research

- [Swift language reference: raw-value enumerations](https://docs.swift.org/latest/documentation/the-swift-programming-language/declarations/) — Swift provides a fallible initializer from a raw value to one enum case. It can do so unambiguously because Swift requires each raw value to be unique. Flang has deliberately allowed duplicate associated values, so the same single-case API needs an ambiguity policy.
- [Rust standard library: `TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html) — Rust models conversions that can fail with a result carrying an error. A flang one-case reverse conversion could similarly distinguish no match from multiple matches, if the language supplies such a conversion.
- [Zig language reference: `@fromBackingInt`](https://ziglang.org/documentation/master/) — Zig converts from an integer backing tag to an enum value. That is a different operation from searching arbitrary associated `T` values, and it does not resolve duplicate-value ambiguity.
- [Question 047, answer A](047_Enum_associated_value_access.md) — Retrieving an associated value from an option is allowed at runtime when its type has a runtime representation. That forward operation remains unambiguous even with duplicates.

# Answers

## A. No built-in reverse lookup from associated values

The language does not automatically convert `T` to `enum T` or supply a special reverse-lookup operation. Programs can write a search over enum options when enumeration and equality facilities are available. This keeps enum declarations valid for `T` without equality and avoids choosing one duplicate. Whether enumeration and reflection facilities exist remains separate.

## B. Built-in query returns every matching option

When `T` has a suitable equality operation, a built-in query returns all options whose associated values equal the input. `red` would produce both `STOP` and `DANGER`; an unmatched value produces an empty result. The result could be a collection or iterator, with its exact type and allocation behavior decided later. For `T` without equality, the query is unavailable.

## C. Built-in single-option conversion reports ambiguity

When `T` has suitable equality, an explicit conversion returns one option on a unique match and reports no match or ambiguity otherwise. `red` would report ambiguity. This serves callers expecting one case but requires a result/error shape and may encourage treating associated values as unique despite Question 046.

# Selected answer

**A, qualified.** There is no built-in reverse lookup or automatic conversion from an associated `T` value to `enum T`. A programmer may write a search over the options. A standard-library trait for `enum<T>` when `T` implements `Equality` is a possible convenience, but has not been selected; its API, result shape, and behavior for duplicate values remain open.
