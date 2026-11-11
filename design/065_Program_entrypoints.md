# Question

How is flang code entered by a PC process, the compile-time evaluator, and DiamondFire's host environment?

An executable can have `main`, but DiamondFire code may be invoked by host actions or events. Compile-time evaluation happens while compiling rather than through a runtime `main`. [Question 062](062_External_capabilities.md) says ordinary external authority comes from an entry point or embedding host; this question identifies those boundaries. All syntax below is provisional.

# Research

- [Clang, *Assembling a Complete Toolchain*](https://clang.llvm.org/docs/Toolchain.html): distinguishes source lowering, code generation, assembly, and linking for native executables.
- [DiamondFire official tutorials](https://mcdiamondfire.com/tutorial/): present programs as game logic built from code blocks and actions, unlike a PC executable launched by an operating system.
- [DiamondFire Bucket Variables](https://mcdiamondfire.com/docs/bucket-variables/overview/): persistent state is attached to the host's plot and storage facilities, so an entry boundary must identify its lifetime and authority.
- [Haskell 2010 Report, *System.IO*](https://www.haskell.org/definition/haskell2010.pdf): an example of rooting runtime I/O execution in `main`, while functions are still reusable below that root.
- [Zig Build System](https://ziglang.org/learn/build-system/): a `build` function receives a build context and declares target-specific executable artifacts; flang's proposed function would additionally register runtime entries inside those artifacts.
- [Cargo Build Scripts](https://doc.rust-lang.org/cargo/reference/build-scripts.html): build-time code can configure later compilation, but runs as a separate host executable and communicates through build instructions rather than as flang compile-time code.

# Answers

## A. Target-specific entry declarations with shared library functions

PC programs define `main`; DiamondFire programs declare handlers for host-defined invocations; compile-time expressions or functions run when the compiler requires their values. Shared functions are ordinary flang code called from these entries. This matches each host but requires separate entry syntax and adapters.

```flang
fn main(io: PC_IO) { /* PC */ }
// DiamondFire event declaration spelling remains open.
```

## B. One portable application interface

An application implements a common trait or module contract. PC and DiamondFire adapters invoke it, possibly presenting PC startup as an event. This makes whole applications more portable but may force incompatible host lifecycles into one abstraction.

## C. Host-declared exports without a built-in `main`

Source files define functions; a build or embedding host names which ones become PC startup code, DiamondFire handlers, or compile-time jobs. This is flexible for libraries and embedding, but pushes more behavior into configuration and makes a standalone source file less self-describing.

## D. One compile-time build entry registers target runtime entries

A designated root file has a compile-time entry function, provisionally named `build`. The compiler invokes it with a build context containing the selected target and configuration authority. Its calls register functions as PC runtime entry points or DiamondFire event handlers. The registered functions are referenced as values at compile time; they run later on the target when invoked. Imported libraries do not run their own build entries merely because they are imported.

```flang
comp fn build(ctx: BuildContext) {
    comp if ctx.target == .pc {
        ctx.runtime_entry(pc_main)
    } else if ctx.target == .diamondfire {
        ctx.on(.player_join, on_player_join)
    }
}

fn pc_main(io: PC_IO) { /* runtime PC code */ }
fn on_player_join(event: PlayerJoin, game: DfGame) { /* DF runtime code */ }
```

The names, signatures, configuration API, number of artifacts per invocation, and event schema remain open. The compiler must distinguish executing `build` from invoking a registered runtime function, and track external inputs consumed by `build` as in [Question 066](066_Comptime_external_inputs.md). Registration can make a call graph root for target code generation; generic entries may need specialization before registration.

# Selected answer

D, with qualification: a designated compile-time entry function is invoked by the compiler with a build context. It can configure target-specific build behavior and register runtime entry functions, including PC process startup and DiamondFire event handlers. The name `build`, root-file convention, exact context API, registration syntax, and artifact multiplicity remain open. A registered runtime function is not executed during registration.
