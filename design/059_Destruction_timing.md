# Question

When an owned value reaches the end of its scope, which cleanup events have a specified time: user-defined destruction, destruction of fields, and physical release of storage?

[Question 054](054_Memory_and_aliasing.md) selects ownership and checked borrowing with automatic cleanup at scope exit. This question refines what programs may observe. Syntax below is provisional.

# Research

- [Rust Reference, *Destructors*](https://doc.rust-lang.org/reference/destructors.html): defines drop scopes and orders destructor calls when control flow leaves them; also notes that a type cannot rely on its destructor always running for soundness.
- [Rust `Drop` trait documentation](https://doc.rust-lang.org/stable/core/ops/trait.Drop.html): describes user-defined destructor behavior and the order of a value's destructor relative to its fields.
- [Rustonomicon, *Leaking*](https://doc.rust-lang.org/nomicon/leaking.html): shows why even an ownership language may permit leaked values and why unsafe abstractions cannot assume every destructor eventually runs.
- [Go garbage collector guide](https://go.dev/doc/gc-guide): offers a contrasting model: cleanup/finalizer execution may be delayed or never occur.

# Answers

## A. Scope exit fixes every physical deallocation

At scope exit, destructors run and backing storage is returned to the allocator immediately, even for values without custom cleanup. This gives strong timing guarantees but restricts allocation optimizations and exposes allocator behavior in the language contract.

## B. Scope exit fixes observable destruction; physical storage timing is unspecified

At scope exit, an owned value with `Drop` or fields needing `Drop` runs those destructors in a specified order, including on normal exit and early control flow. Other values logically cease to exist at scope exit. The compiler may optimize when underlying storage is returned or reused, provided safe observable behavior is unchanged. Explicit `close`/`drop` can release a resource earlier. The language must separately decide whether deliberate leaks or abrupt process termination can skip destructors.

```flang
{
    let file = open(path) // File implements Drop
    write(file, data)
} // File's Drop runs here; exact allocator bookkeeping is unspecified
```

## C. Scope exit ends access, but cleanup may be delayed

Even user-defined `Drop` behavior may happen later. Deterministic release of files, locks, and similar resources requires an explicit operation or scoped construct. This gives the implementation more scheduling freedom but changes the meaning of automatic cleanup.

# Selected answer

B. When an owner leaves scope, its `Drop` behavior and any required field destructors run in a defined order, including on normal exit and early control flow. Other owned values cease to exist at that point. The compiler may choose when and how to reclaim or reuse the backing storage, provided safe observable behavior is unchanged. Deliberate leaks and abrupt termination remain separate questions.
