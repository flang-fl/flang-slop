# Question

Where does authority to use external resources such as `stdin`, `stdout`, the filesystem, the clock, and the network originate, and how does an ordinary function obtain it?

Should `main` or another entry point receive capabilities and delegate them, or can imported code obtain process resources directly? Can `println` be called without an explicit output argument? [Question 056](056_Effects_and_failure.md) selects explicit result values for recoverable failures and no general effect annotation, but does not answer authority or dependency visibility. The code below uses provisional syntax.

# Research

- [Miller, Yee, and Shapiro, *Capability Myths Demolished* (2003)](https://papers.agoric.com/assets/pdf/papers/capability-myths-demolished.pdf): distinguishes authority conveyed by references from ambient authority; explicit capabilities make delegation and confinement possible.
- [Lewis, Shields, Meijer, and Launchbury, *Implicit Parameters: Dynamic Scoping with Static Types* (2000)](https://www.cs.tufts.edu/comp/150FP/archive/jeffrey-lewis/implicit-parms.pdf): shows how a dependency can be supplied implicitly while appearing in a static type.
- [Haskell 2010 Report, *System.IO* and program execution](https://www.haskell.org/definition/haskell2010.pdf): I/O actions are ultimately executed from `main`, while `stdin` and `stdout` are standard handles available from a library module. This separates the execution root from argument-passing style.
- [Koka, *Programming with Row Polymorphic Effect Types* (2014)](https://arxiv.org/abs/1406.2061): illustrates a stronger alternative where effects appear in function signatures; flang's [Question 056](056_Effects_and_failure.md) currently does not choose that model.

# Answers

## A. Ambient standard resources

Any permitted function can call `println` or access a standard stream through a global namespace. This is convenient and requires no parameter plumbing. It gives imported code ambient authority and makes redirection or isolation depend on process-level facilities or mutable globals.

This is strongest for small command-line programs and logging from deep call stacks. A stable process-wide stream is easy to explain. The same convenience makes it harder to run two tests with different outputs concurrently, or to keep an imported library from writing to the host's console. A target with no standard stream needs a target-specific meaning or an unavailable operation.

```flang
fn report() { println("ready") }
```

## B. Explicit capability arguments from the entry point

The entry point receives or constructs resource handles and passes them to functions that need them. A function's explicit arguments show its authority, and tests can pass substitute handles. The cost is repetitive parameters through call chains.

This is strongest for reusable libraries, sandboxed components, and target portability: a caller can pass a DiamondFire chat sink where another target passes a terminal writer. It also works well with ownership because the handle's lifetime and sharing policy are ordinary type questions. Passing the same handle through many otherwise unrelated functions is the main ergonomic cost. The origin need not literally be a function named `main`; tests and embedding hosts can supply capabilities too.

The cost can be reduced without hiding authority: bundle related capabilities in an application context and give methods one `self` receiver; construct a service object or closure that captures a narrower writer once; return text or structured data from pure helpers and print at the boundary. The next option, C, is a stronger convenience mechanism because it makes argument passing implicit. A narrowly scoped build-only `debug_print` exception is considered separately in [Question 063](063_Debug_output_capability.md).

```flang
fn report(out: &mut Writer) { out.println("ready") }
fn main(io: IO) { report(&mut io.stdout) }
```

## C. Typed implicit capability parameters

The entry point supplies capabilities, and functions declare a requirement such as `using Console` without including it in the ordinary argument list. Calls supply it implicitly from a lexical or call context, with a compile-time error if none is available. `println` stays concise while its authority remains visible in an extended function type. Resolution and propagation rules need design.

This is strongest when many functions need a common service and both concise calls and static authority checks matter. A test can provide a substitute `Console` without replacing process globals. The dependency still propagates through function types or inference, so library APIs need a policy for revealing it. Multiple consoles, first-class functions, and whether a spawned task inherits the context require rules. A typed capability requirement is narrower than a general effect annotation, but it is still part of the function's contract.

```flang
fn report() using Console { println("ready") }
fn main(io: IO) {
    with Console = io.stdout { report() }
}
```

## D. Dynamically scoped output context

`println` uses the current task's output context, provided by the runtime or overridden in a scope. Ordinary function types do not disclose this dependency. Redirection is convenient, but tasks, threads, nesting, and calls into foreign code need precise context-propagation rules.

This is strongest for temporary redirection, structured logging, and task-local output without changing signatures. Unlike A, different tasks or scopes can use different writers simultaneously. Unlike C, the type checker cannot tell whether a call has output access; a missing context becomes a runtime question. A library called inside a context can use its authority whether the caller intended that or not.

```flang
fn report() { println("ready") }
with_stdout(buffer) { report() }
```

Hybrids are possible, such as an ambient convenience `println` over a capability-based lower-level I/O API. A selection should say which access route ordinary code relies on and whether libraries can obtain authority without a value originating at an entry point.

The origin of a resource and its passing style are separate choices. A host can provide `stdout` at startup under any option. The key difference is whether an arbitrary callee can reach that authority without the caller passing or declaring it.

# Selected answer

B, with qualification: ordinary production I/O uses explicit capabilities passed from an entry point or embedding host through functions that need them. A narrowly scoped debug-output operation may print without an explicit output parameter; see [Question 063](063_Debug_output_capability.md). The exact shape of entry-point capabilities and whether additional convenience wrappers exist remain open.
