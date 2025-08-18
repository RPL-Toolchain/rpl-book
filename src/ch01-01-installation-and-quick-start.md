# Installation and Quick Start

1. Clone the repository and enter the directory: `git clone https://github.com/RPL-Toolchain/RPL.git && cd RPL`

2. Install RPL as a cargo subcommand: `cargo install --path .`

3. Run RPL analysis on your Rust project:

    - `RPL_PATS=/path/to/RPL/docs/patterns-pest cargo +nightly-2025-02-14 rpl` (using built-in RPL pattern definitions based on inline MIR)
    - `RUSTFLAGS="-Zinline-mir=false" RPL_PATS=/path/to/RPL/docs/patterns-pest cargo +nightly-2025-02-14 rpl` (using built-in RPL pattern definitions based on MIR)

> We have the following plans for easing the usage of RPL:
>
> -   Integration of standard patterns: Provide a set of commonly used patterns as part of RPL’s standard library.
> -   Configuration-based pattern management: Introduce a `rpl.toml` configuration file to manage RPL patterns in a structured and centralized manner.
