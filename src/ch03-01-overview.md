# Overview

This section provides a high-level overview of the RPL pattern modeling language. To illustrate its core concepts and syntax, we will use the following example, which models the memory safety vulnerability identified in `CVE-2020-25016`.

> The detailed information of `CVE-2020-25016` can be found [here](https://github.com/RPL-Toolchain/RPL/blob/master/docs/cve-and-ub-notes/memory-safety-related-cves/CVE-2020-25016.md).

```Rust
pattern CVE-2020-25016

patt {
    #[diag = "p_unsound_cast"]
    #[const(mutability_1 = "", mutability_2 = "")]
    p_unsound_cast_const[
        $T: type where is_all_safe_trait(self) && !is_primitive(self)
    ] = fn _ (..) -> _ {
        'cast_from:
        let $from_slice: &[$T] = _;
        let $from_raw: *const [$T] = &raw const *$from_slice;
        let $from_len: usize = PtrMetadata(copy $from_slice);
        let $ty_size: usize = SizeOf($T);
        let $to_ptr_t: *const $T = move $from_raw as *const $T (PtrToPtr);
        let $to_ptr: *const u8 = move $to_ptr_t as *const u8 (PtrToPtr);
        let $to_len: usize = Mul(move $from_len, move $ty_size);
        let $to_raw: *const [u8] = *const [u8] from (copy $to_ptr, copy $to_len);
        'cast_to:
        let $to_slice: &[u8] = &*$to_raw;
    }
}

diag {
    p_unsound_cast = {
        primary(cast_to) = "it is unsound to cast any slice `&{$mutability_1}[{$T}]` to a byte slice `&{$mutability_2}[u8]`",
        label(cast_to)   = "casted to a byte slice here",
        note(cast_from)  = "trying to cast from this value of `&{$mutability_1}[{$T}]` type",
        level            = "deny",
        name             = "unsound_slice_cast",
    }
}
```

While this example is intentionally simplified for clarity, it effectively showcases nearly all of the language's key syntactic features. The design of the language is guided by three fundamental principles, all of which are visible in this example:

## SoC-Based Modularization

The fundamental design principle of the language is modularization. The pattern is divided into distinct blocks: the `patt` block contains the core detection logic that describes the structure of the code to be found, while the `diag` block independently defines the reporting logic (the diagnostic message that will be shown to the user). This separation keeps the rule definition clean and maintainable.

## Metavariable-based Abstraction

The core mechanism for abstraction is the metavariable. In the example, `$T` is a metavariable that acts as a named placeholder prefixed with a dollar sign, which can stand in for any concrete Rust type. This allows the pattern to be abstract and general, capable of matching the vulnerability regardless of the specific slice type involved.

The abstraction provided by metavariables works in concert with Mid-Level Intermediate Representation(MIR)-based modeling
to capture the internal logic of functions at a semantic level. Using a syntax that mirrors MIR provides two key benefits:
First, it makes patterns inherently robust against superficial syntax noise. Different programming styles, such as using a single nested expression versus introducing multiple intermediate variables, often generate the same MIR sequence, allowing a single pattern to detect a logical flaw regardless of its specific source-level implementation. Second, it grants our fundamentally intra-procedural analysis certain inter-procedural capabilities. Compiling with the `-Zinline-mir` flag instructs the Rust compiler to inline the MIR of eligible called functions into their caller, based on internal heuristics like function size. This action merges the logic of these functions, allowing a single pattern to trace a sequence of operations that crosses the original function boundaries.

## Predicate-based Refinement

While the MIR-like body defines the structural shape of the pattern, the where clause applies additional semantic constraints. In this case, the predicates `is_all_safe_trait` and `!is_primitive` are used to filter the matches for the `$T` metavariable, ensuring that the rule only flags types where the cast is truly unsound.
