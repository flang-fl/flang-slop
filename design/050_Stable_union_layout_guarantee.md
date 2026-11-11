# Question

What should an explicit stable-layout request for a tagged union guarantee?

[Question 049, answer B with qualification](049_Union_tag_representation.md) allows optimized default layouts and requires a way to request stable memory layout. “Stable” could mean either a documented native layout that remains unchanged across compatible compilers for one target, or identical bytes across different targets. These are different promises. Consider a provisional annotation:

```flang
comp Packet = union /* stable layout request */ {
    Number: u32,
    Text: FixedText,
}
```

The annotation spelling, `FixedText` layout, tag numbering, padding, and invalid-bit rules are not chosen. This question asks which **kind of guarantee** the stable-layout facility should provide. A C-compatible representation is a separate interop feature, not an answer to this question.

# Research

- [Rust Reference: type layout and representations](https://doc.rust-lang.org/reference/type-layout.html) — Rust distinguishes its flexible default representation from representations with specified tag and field layout. A representation may be predictable while still depending on the target's sizes and alignment rules.
- [Zig language reference: `extern` and `packed` layouts](https://ziglang.org/documentation/master/) — Zig's `extern` aggregate layout follows the target C ABI, while `packed` forms define a bit-level layout. These serve different uses despite both providing stronger guarantees than ordinary aggregate layout; flang can likewise treat C interop separately from its stable-layout facility.
- [LLVM Language Reference: data layout](https://llvm.org/docs/LangRef.html#data-layout) — LLVM records target-specific endianness, pointer size, and alignment in a module's data layout. This is why a target-native memory layout can be stable on each target yet differ across targets.
- [Question 049](049_Union_tag_representation.md) — Selects compiler-chosen ordinary union layouts plus an explicit stable-layout facility.

# Answers

## A. A language-defined native layout on each target

The stable request uses a documented flang layout algorithm, with specified tag representation, payload placement, padding, and alignment rules for each target. Recompiling for the same target with a newer compatible compiler preserves the layout. Different targets may lay out the same type differently. This serves shared libraries, foreign-language bindings written against flang's documented ABI, and low-level code that inspects memory. It can retain natural alignment and target-sized fields. Stable layout of nested payload types must also be established. It does not promise a portable serialized byte format.

## B. One portable byte layout across targets

The stable request fixes tag bytes, payload offsets, padding, and endianness independent of the target. This can be read the same way across platforms and is useful for mapped files, shared data, or a fixed binary protocol. It must reject or specially encode target-sized pointers and payload types without portable representations. Native access may require conversion or unaligned loads, and an in-memory layout promise alone does not handle schema evolution or validation of untrusted bytes.

## C. Offer separate native-stable and portable layout requests

Provide both A and B as distinct explicit requests with distinct names and constraints. This serves native ABI and portable-data uses directly, but enlarges the language's layout surface and requires precise rules for nested types, generics, and conversion between the two representations. C-compatible layout remains a third, separate interop question.

# Selected answer

**A.** An explicit stable-layout request guarantees a documented native layout across compatible compiler versions for the same target. The same type may have different layouts on different targets. Nested payload types must themselves have suitable layout guarantees. C ABI compatibility and a portable byte format are separate facilities. The request's syntax and exact layout algorithm remain open.
