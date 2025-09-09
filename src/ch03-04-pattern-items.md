# Pattern Items

## `use` statements

The DSL supports the `use` statements within both `patt` and `util` blocks. Their primary purpose is to import types, such as structs and enums, from external crates and modules into the local scope of the pattern definition. This feature is particularly important for the modeling of function logic, since patterns often need to reference complex types defined in Rust standard libraries. By importing these types, developers can refer to them by their simple names (e.g., `Vec` instead of `alloc::vec::Vec`) within the pattern body.
