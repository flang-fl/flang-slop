# flang design seed

**Status:** exploratory. The name and all syntax examples are provisional.

## Starting constraints

- A statically typed programming language.
- Ahead of time compilation to LLVM, at least initially.
- Runtime type information and an `Any`-like type are open questions.

## Decisions

- A named block may produce a value through an explicit `break` targeting its name. The spelling of labels and `break` remains provisional. [Question 002, answer C](002_Blocks_as_values.md)
- `while` and `for` loops may produce a value through `break`; a loop's `else` expression supplies its value when the loop finishes normally. Label and `break` spelling remains provisional. [Question 003, answer A](003_Loop_values.md)
- An unnamed `break` or `continue` targets the innermost loop. A `break` may carry a value there; exiting a named block or outer loop requires its name. [Question 004, answer A](004_Unnamed_breaks.md)
- Every block uses the same result rule, including blocks in `if` branches: without a break targeting it, the final expression is its value, or unit if there is no final expression. If a block has a break targeting it, every path that completes that block must break to it. [Question 007, answer B](007_If_branch_values.md); this revises [Question 005](005_Block_fallthrough.md).
- `if` can produce a value, and parentheses around its condition are optional. Its braced branches follow the ordinary block result rule. [Question 006, answer A](006_If_values.md); [Question 007, answer B](007_If_branch_values.md)
- An `if` without `else` is allowed only when its true branch produces unit; a value-producing `if` requires `else`. [Question 008, answer A with qualification](008_If_without_else.md)
- A non-unit expression result must be used or explicitly discarded; merely placing the expression in statement position is not enough. [Question 009, answer B](009_Discarding_values.md)
- A function returns the value of its body block when that block completes normally. A non-unit return type requires a compatible body value. [Question 010, answer A](010_Function_body_results.md)
- `return` exits the current function early. If a function contains a `return` targeting it, every path that completes that function must use `return`; a final expression alone is then insufficient. [Question 011, answer A with qualification](011_Early_function_exit.md)
- A named function must declare a non-unit result type; omitting the result type means unit. [Question 012, answer A](012_Function_result_types.md)
- Local `let` bindings are immutable by default; `let mut` permits both reassignment and mutation of data through that binding. Aliasing rules remain open. [Question 013, answer A with qualification](013_Local_binding_mutability.md)
- Compile-time computation is a rich part of the language, with planned binding forms `let`, `let mut`, `comp`, and `comp mut`. Function-local `comp mut` can change during compilation while nearby runtime operations remain in the generated function. Its state belongs to that compilation or specialization, not to each runtime call. File-scoped mutable compile-time state remains open. [Question 014, answer B](014_Comptime_mutable_scope.md)
- Runtime-dependent control flow cannot update `comp mut` state. A compile-time `if` can update it when its condition is known during compilation. [Question 015, answer A](015_Runtime_control_of_comptime_state.md)
- Parse both branches of a compile-time conditional, but resolve names and type-check only the selected branch. The inactive branch must be syntactically valid. [Question 018, answer A](018_Comptime_if_branch_checking.md)
- A public module can select a separate implementation and re-export its public members under one stable namespace. The exact `import`/`load` and `reexport` spellings remain open. [Question 022, answer C](022_Conditional_module_files.md)
- A source file is an importable unit exposing a struct-like namespace to permitted callers. A library developer may explicitly write a root file that imports and re-exports other files, including by scanning a directory; sibling files are not aggregated by default. Compiler cache and code-generation granularity remain open. [Question 023, answer E](023_Compilation_unit_size.md)
- A root file can own a logical library scope. Implementation files may restrict direct imports to that library, while the root can re-export selected names to outside callers. The exact directives and individual-declaration visibility remain open. [Question 024, answer E](024_File_import_visibility.md)

## Question index

- [002: Should a block produce a value?](002_Blocks_as_values.md)
- [003: Can loops produce values?](003_Loop_values.md)
- [004: May a value-carrying break omit its target?](004_Unnamed_breaks.md)
- [005: What happens when a named block reaches its end?](005_Block_fallthrough.md)
- [006: Can an `if` produce a value?](006_If_values.md)
- [007: How do braced `if` branches produce values?](007_If_branch_values.md)
- [008: What does `if` without `else` mean?](008_If_without_else.md)
- [009: How are unwanted values discarded?](009_Discarding_values.md)
- [010: Does a function return its body's value?](010_Function_body_results.md)
- [011: How can a function exit early?](011_Early_function_exit.md)
- [012: Must a function declare its result type?](012_Function_result_types.md)
- [013: Can local names be reassigned?](013_Local_binding_mutability.md)
- [014: Where may `comp mut` state live?](014_Comptime_mutable_scope.md)
- [015: What if runtime control flow affects `comp mut`?](015_Runtime_control_of_comptime_state.md)
- [016: How can compile-time code inspect and interact with the compiler?](016_Compiler_introspection_and_messages.md)
- [017: How is a compile-time `if` written?](017_Comptime_if_syntax.md)
- [018: How is an unchosen compile-time branch checked?](018_Comptime_if_branch_checking.md)
- [019: Which marker, if any, identifies compiler built-ins?](019_Builtin_sigil.md)
- [020: Can a compile-time conditional select file-level declarations?](020_Comptime_if_file_scope.md)
- [021: Does a compile-time conditional yield its selected branch's value?](021_Comptime_if_values.md)
- [022: How can a module choose a platform-specific implementation file?](022_Conditional_module_files.md)
- [023: How large is an importable compilation unit?](023_Compilation_unit_size.md)
- [024: Who may import a file, and can re-export broaden access?](024_File_import_visibility.md)
- [025: How do platform implementations satisfy one public interface?](025_Platform_interface_conformance.md)

Decisions made in the question files will be reflected here as the language takes shape.
