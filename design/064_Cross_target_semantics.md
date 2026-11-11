# Question

What must ordinary flang code mean consistently when evaluated during compilation, run as a PC executable, or lowered to DiamondFire?

Compile-time evaluation is performed by the compiler, while LLVM and Cranelift are possible PC code generators and DiamondFire is a different runtime environment. [Question 036](036_DiamondFire_native_types.md) already says ordinary flang types remain usable on DiamondFire, with separate native types where useful. This question asks whether *operations* on those types must be available and semantically identical everywhere. Syntax below is provisional.

# Research

- [LLVM Language Reference](https://llvm.org/docs/LangRef.html): LLVM IR is a low-level, target-aware representation; it does not define the source-language semantics that flang must share with a non-LLVM target.
- [Cranelift frontend documentation](https://docs.wasmtime.dev/api/cranelift_frontend/index.html): a language frontend builds Cranelift IR, again leaving source-language meaning to that frontend.
- [DiamondFire's Code Slots documentation](https://mcdiamondfire.com/docs/code-slots/): describes target-native value categories and action arguments, though the documented redesign is explicitly not yet usable; it is evidence of a distinct target interface, not a final flang mapping.
- [DiamondFire's Bucket Variables documentation](https://mcdiamondfire.com/docs/bucket-variables/overview/): documents a target-specific persistence mechanism rather than an ordinary PC heap.
- [Question 018](018_Comptime_if_branch_checking.md): a selected compile-time branch can contain target-specific names without type-checking the other branch.

# Answers

## A. A portable core with identical semantics on every runtime target

Every released core operation on ordinary flang types has the same observable result on PC and DiamondFire, even if DiamondFire needs emulation or a slower representation. Pure operations evaluated during compilation follow those same rules. Target-specific effects and `df` types live in separate APIs. This is easy for users to reason about but can make a DiamondFire backend expensive to implement.

## B. Shared semantics where supported, with target-qualified availability

An ordinary type can exist on every target, but some operations or capabilities may be unavailable on DiamondFire or at compile time. Available operations preserve their meaning; unavailable ones produce a target-specific compile error unless excluded by a compile-time branch. This makes a smaller prototype possible but means a library can be portable at the type level without all its functions being portable.

## C. Target-dependent meaning for ordinary operations

The same ordinary operation may have different numeric, layout, lifetime, or failure behavior depending on the target. This exposes native performance but weakens cross-target reasoning and may revise existing decisions about ordinary types and explicit native conversions.

# Selected answer

B, with qualification: ordinary flang operations retain their meaning wherever they are supported. A target or compile-time evaluator may reject an unavailable operation with a compile-time error; target-specific compile-time branches can select alternatives. Ordinary flang types remain usable on DiamondFire as previously decided, while DiamondFire-native primitives such as provisional `DfString`, `DfVector`, and `DfNum` are also available for native operations and representations. Their final names, precise semantics, and conversion APIs remain open in [Question 036](036_DiamondFire_native_types.md) and later target-specific questions.
