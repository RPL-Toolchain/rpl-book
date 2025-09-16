# Contributing to RPL

We have outlined our primary project goals and the key tasks we are focused on to achieve them.

## Our Goals

Our long-term vision for RPL is centered on the following objectives:

1. **A Richer Pattern/Knowledge Base**: To create a large-scale, high-quality library of RPL patterns.

2. **Proven Real-World Effectiveness**: To detect and report vulnerabilities in real-world Rust repositories.

3. **A More User-Friendly Experience**: To improve the usability of the DSL syntax and the supporting toolchain.

## Key Tasks

To achieve these goals, we are focused on the following key tasks:

1. **Expanding the Pattern Library**: We are seeking contributions to add new patterns from a variety of sources, including:

    - Rewriting existing lints from Clippy.

    - Modeling common misuses of unsafe APIs in the Rust standard library.

    - Creating patterns based on published CVEs and security advisories.

    - Rewriting the patterns from other research projects like Rudra and SafeDrop into RPL patterns.

2. **Expanding the Predicate Library**: As we add more patterns, we must also enhance their precision and recall. A crucial part of this is building a richer library of predicates.

3. **Improving the Core Toolchain**: This includes:

    - Optimizing the DSL syntax,

    - Developing a configuration system (like `rpl.toml`) similar to Clippy's, to allow users to customize rules.

    - Refactor the graph matching algorithm logic to support inter-procedural pattern matching (instead of simply using the inline version of MIR).

    - Updating the nightly rustc version that the toolchain depends on periodically.

    - Fixing bugs and updating documentation.
