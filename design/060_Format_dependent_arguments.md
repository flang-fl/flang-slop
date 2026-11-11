# Question

Can a compile-time-known value determine the number or types of a function's runtime arguments, as in a type-checked `println`? What happens when the format string is known only at runtime?

This follows the proposed distinction between inferred compile-time parameters in `<>` and explicit `comp` parameters in `()` in [Question 055](055_Generic_type_model.md). Syntax below is provisional.

# Research

- [Rust `format_args!` documentation](https://doc.rust-lang.org/std/macro.format_args.html): the macro receives a format string literal, checks the formatting arguments, and builds a `fmt::Arguments` value for runtime formatting.
- [Rust `format!` documentation](https://doc.rust-lang.org/std/macro.format%21.html): explicitly requires the first argument to be a string literal.
- [Zig language reference, *Compile-Time Parameters*](https://ziglang.org/documentation/master/#Compile-Time-Parameters): shows compile-time parameters and a userland formatting example where the format may be any compile-time-known string; a runtime-known value passed to a compile-time parameter is a compile error.
- [Abadi et al., *Dynamic Typing in a Statically Typed Language* (1991)](https://research.google/pubs/dynamic-typing-in-a-statically-typed-language-2/): an explicit dynamic value is one possible way to carry heterogeneous arguments through a runtime formatting API.
- [Xi and Pfenning, *Dependent Types in Practical Programming* (1999)](https://www.cs.cmu.edu/~fp/papers/popl99.pdf): shows a practical restricted form of types depending on static values, while warning through its design that unrestricted inference is difficult.

# Answers

## A. Fixed argument shape; compile-time validation of a tuple

A format function has one runtime tuple argument. Its compile-time body parses the format and checks the tuple's arity and each element's formatting trait. Generic tuple parameters are inferred from the supplied tuple. This needs compile-time reflection over tuple types but no dependent source-level parameter list.

```flang
fn println<#Args: type>(comp fmt: str, args: Args) {
    comp assert_format_compatible(fmt, Args)
    // format args at runtime
}
println("x = {}", (x,))
```

## B. Format-dependent variadic runtime parameters

The format determines a runtime parameter pack, allowing `println("x = {}", x)` directly. This needs variadic function types or a compiler-supported way for compile-time computation to define a call's runtime argument shape. Placeholder `{}` may impose a trait constraint rather than a single concrete type.

```flang
fn println<#Args: type...>(comp fmt: str, args: Args...) {
    comp assert_format_compatible(fmt, Args)
}
println("x = {}", x)
```

## C. Formatting-specific macro or intrinsic

The compiler or a macro system parses the format at the call site and expands to ordinary typed code. The core function model stays simpler, but formatting does not demonstrate a general ability to derive runtime call shapes from compile-time values.

## D. A compile-time signature block generates runtime parameters

After the caller supplies `fmt`, a compile-time declaration block computes parameter specifications from its placeholders. The compiler then checks the remaining call arguments against that generated signature and specializes the function body. The syntax and `parameter(...)` operation below illustrate the idea; neither is selected.

```flang
fn println(comp fmt: str)
    comp params {
        for hole in parse_format(fmt) {
            parameter(hole.name, impl hole.format_trait)
        }
    }
{
    // The generated runtime arguments are available as a pack.
}
```

This is a general form of compile-time signature construction, not merely a formatting feature. It needs a staged type-checking order: evaluate the explicit compile-time arguments, build the runtime parameter list, then check runtime arguments. A placeholder such as `{}` constrains its argument to implement a trait; it does not name one concrete type. Each call may therefore still infer and specialize generic argument types. It also needs a rule for taking the function as a value before or after specialization.

### Comparing B and D

Both can support the same `println("x = {}", x)` call and compile-time rejection of an incompatible format. B infers a heterogeneous pack from the supplied arguments and then checks `format_compatible(fmt, Args)`; D evaluates `fmt`, constructs parameter specifications, and then checks the supplied arguments. For a generic `{}` placeholder, both still infer the concrete argument type and require a formatting trait.

D is more useful when a compile-time value fixes a *concrete expected type*. For example, if a schema says the next field is `u64`, the generated signature can give an unsuffixed numeric literal or an empty container that expected type while checking the call. Under a simple B design, those expressions are typed before `format_compatible` runs and may default to the wrong type or lack enough context. B could recover that contextual typing with a more elaborate constraint solver, reducing its simplicity advantage.

B has a smaller core language surface: generic packs and a compile-time predicate suffice. D adds a general way for compile-time code to create named or unnamed parameter specifications, plus staged signature construction. That extra power can be valuable for schema-driven APIs, but it makes function type identity, overload resolution, diagnostics, and first-class function values more involved. Both approaches need a rule for using a generic function as a value before specialization.

### Provisional worked sketch for D

This assumes a compiler-provided compile-time `ParamList` API. `generic(bound: Trait)` creates a fresh inferred type parameter with a trait constraint; it does not create a trait object. `parse_format` recognizes only text, `{}` for `Display`, and `{:x}` for `LowerHex` in this sketch. Every placeholder consumes one runtime argument.

```flang
comp fn format_params(fmt: str) -> ParamList {
    let mut result = ParamList()

    for part in parse_format(fmt).parts {
        comp match part {
            Text(_) => {}
            Hole(DisplayStyle, _) => result.push(generic(bound: Display))
            Hole(HexStyle, _) => result.push(generic(bound: LowerHex))
        }
    }

    result
}

fn println(comp fmt: str,
           ...args: comp { format_params(fmt) }) {
    comp plan = parse_format(fmt)
    comp for part in plan.parts {
        comp match part {
            Text(text) => write_or_stop(stdout, text)
            Hole(DisplayStyle, index) => write_display_or_stop(stdout, args[index])
            Hole(HexStyle, index) => write_hex_or_stop(stdout, args[index])
        }
    }
    write_or_stop(stdout, "\n")
}
```

The `comp for` emits runtime write operations in source order; it does not print during compilation. `write_or_stop` stands in for an I/O error policy, which is undecided. For `println("n={} hex={:x}", name, number)`, the generated call shape is conceptually `(name: T0 where T0: Display, number: T1 where T1: LowerHex)`. `fmt`, `T0`, and `T1` are compile-time specialization inputs. The actual runtime arguments remain `name` and `number`.

In all four cases, a string known only at runtime cannot set a statically checked number and type of arguments at that call. A separate runtime API could parse it and accept a fixed array or list of dynamically formattable values, returning a formatting error when the string and arguments disagree. A runtime branch between known formats can instead call the compile-time-checked function separately in each branch.

# Selected answer

D, provisionally. A compile-time-known argument may feed a compile-time signature block that generates the remaining runtime parameter specifications before those arguments are type-checked. Generated parameters may be fresh inferred types constrained by traits, not only fixed concrete types. The exact syntax, `ParamList` API, format grammar, and rules for unspecialized first-class function values remain open.
