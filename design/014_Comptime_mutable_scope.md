# Question

Can a `comp mut` binding inside a function drive specialization while nearby operations remain in the generated runtime function? How long does that mutable compile-time state live?

The language is intended to support rich compile-time computation. The proposed binding forms are `let`, `let mut`, `comp`, and `comp mut` ([Question 013](013_Local_binding_mutability.md)). The user's function example below points toward partial evaluation within one body, rather than only evaluating an entire function at compile time.

# Research

- [Zig language reference: local and compile-time variables](https://ziglang.org/documentation/master/#Compile-Time-Variables) — A `comptime` local has loads and stores performed during compilation. Combined with inline loops, Zig shows a function partly evaluated at compile time and partly at runtime; its example generates different runtime functions for different compile-time arguments.
- [Zig 0.12 release notes: compile-time memory](https://ziglang.org/download/0.12.0/release-notes.html#comptime-memory-changes) — Pointers to mutable compile-time variables cannot escape into runtime values. This illustrates an important boundary if `comp mut` can create mutable data.
- [Rust Reference: constant evaluation](https://doc.rust-lang.org/reference/const_eval.html) — Compile-time evaluation permits some mutable local work but disallows writes to static items. This is a precedent for allowing temporary compile-time mutation without shared compile-time global state.
- [Rust Reference: constant items](https://doc.rust-lang.org/reference/items/constant-items.html) — A constant is a value rather than a mutable global storage location. This offers a model for immutable top-level `comp` bindings.

## Confirmed runtime-array example

```flang
fn evil() {
    comp mut x = 0;
    somearray[x] = 5;
    x++;
    somearray[x] = 6;
}
```

The user confirmed that `somearray` is a runtime array. `x` starts at 0 for a specialization of `evil`, is incremented during compilation, and has no runtime storage. The generated function would behave approximately like:

```flang
fn evil() {
    somearray[0] = 5;
    somearray[1] = 6;
}
```

The user also wants a compile-time array to be possible when scoped to the file. The lifetime and visibility of such an array, and whether mutable `comp` bindings may exist at file scope, remain open. The rule for modifying `comp mut x` under runtime-dependent control flow is [Question 015](015_Runtime_control_of_comptime_state.md).

# Answers

## A. Only fully compile-time executions can use `comp mut`

```flang
comp table = build_table();

fn build_table() -> Table {
    comp mut builder = TableBuilder();
    builder.finish()
}
```

The entire call is evaluated during compilation. A runtime write such as `somearray[x] = 5` cannot be interleaved into this execution. This is simpler to stage but does not directly express a mixed-stage version of `evil`.

## B. Function-local `comp mut` drives mixed-stage specialization

```flang
fn evil() {
    comp mut x = 0;
    somearray[x] = 5;
    x++;
    somearray[x] = 6;
}
```

The compiler updates `x` while constructing a specialization and emits the runtime operations with known indices. The compile-time state is local to that specialization and does not change on each runtime call. Whether `comp mut` may also appear at module level remains a separate question.

## C. Mixed-stage specialization plus shared module-level compile-time mutation

```flang
comp mut registry = Registry();

comp register_type(name: String) {
    registry.add(name);
}
```

This includes B and additionally lets different specializations share and mutate `registry`. It can support registries or code generation pipelines, but compilation results may depend on evaluation order, and incremental compilation would need precise rules for replaying or invalidating that state. These costs are design inferences, not claims about a specific language.

# Selected answer

**B**, for function-local `comp mut` in a mixed-stage function. In the confirmed example, changes to `x` happen during compilation, while writes to the runtime array remain in generated code. This does not decide whether file-scoped mutable compile-time state is allowed. A compile-time array scoped to the file is desired, but its exact binding and mutation rules are open.
