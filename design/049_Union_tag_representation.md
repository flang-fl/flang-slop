# Question

Does a tagged union always store a separate tag in memory, or may the compiler encode the active alternative using otherwise invalid payload values?

[Question 045, answer A](045_Enums_and_tagged_unions.md) requires every union value to have a known active alternative and to support exhaustive matching. That is a **semantic** tag. Its physical representation is still open. Consider this illustrative type:

```flang
comp MaybeHandle = union {
    None: struct {},
    Some: NonZero<u32>, // Suppose zero is invalid for this payload type.
}
```

The compiler could store a separate tag plus a `u32` payload. It could instead store only the `u32`, using zero to mean `None` and nonzero values to mean `Some`. Both representations preserve the ability to match the alternatives. `NonZero<u32>` is a hypothetical type here; its availability is not decided.

This question concerns the default in-memory representation. FFI layout, stable serialization formats, explicit representation annotations, and the exact layout of particular unions remain separate questions.

# Research

- [Rust standard library: `Option` representation](https://doc.rust-lang.org/std/option/#representation) — Rust guarantees that `Option<T>` has the same size and alignment as `T` for several payload types with an invalid bit pattern, including nonzero integers and non-null references. This demonstrates that a semantically tagged choice need not have a separate tag field.
- [Rust Reference: type layout](https://doc.rust-lang.org/stable/reference/type-layout.html) — Rust's default representation leaves most enum layout unspecified, while explicit representations give stronger guarantees. This separates optimization freedom from a stable ABI promise.
- [Zig language reference: unions](https://ziglang.org/documentation/master/#union) — Zig distinguishes ordinary union layout, which is not guaranteed, from `extern` and `packed` union forms with specified memory layouts. A flang layout policy could likewise be separate from its `union` matching semantics.
- [Question 045](045_Enums_and_tagged_unions.md) — Selects closed, tagged unions with exhaustive matching, without fixing their physical layout.

# Answers

## A. Always use a separate physical tag

A union with multiple alternatives stores a distinct tag in addition to space for its active payload. This makes a straightforward representation model and may simplify low-level inspection. It can use more memory than necessary, especially for optional pointer-like values. Exact tag width, field order, padding, and ABI still need rules.

## B. Permit optimized layouts by default

The compiler may use a separate tag or encode the tag in payload bits when it can preserve all valid payload values and distinguish every alternative. The representation may vary by target or compiler version. Low-level code and FFI would need an explicit stable-layout mechanism if they rely on exact bytes.

## C. Use a separate tag by default; optimize only by explicit request

Ordinary unions use a separate tag. A programmer may request a compact representation for a type where its validity rules make that possible. This gives a predictable default and an optimization path, but adds annotations and may make generic code's layout depend on whether an annotation was supplied.

# Selected answer

**B, qualified.** The compiler may choose a compact representation for an ordinary tagged union, including encoding alternatives in otherwise invalid payload values. The language will also provide a way to explicitly request a stable memory layout. [Question 050, answer A](050_Stable_union_layout_guarantee.md) chooses a documented native layout stable on each target across compatible compiler versions. The request's syntax and exact layout algorithm remain open; C ABI compatibility is separate.
