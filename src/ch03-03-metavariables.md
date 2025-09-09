# Metavariables

A metavariable is a named placeholder prefixed with a dollar sign (e.g., `$T`), which can stand in for a concrete code element. For example, to model a pattern that captures any struct containing a public, constant raw pointer, developer could define the following pattern item (We will introduce the pattern item in the next section, now you can just treat it as a Rust item with a pattern name):

```rust
p [$T: type] = struct $S {
    pub $field: *const $T
}
```

This single pattern can match any struct that has this specific structure, regardless of the names used for the struct itself (`$S`), its field (`$field`), or the pointer's underlying type (`$T`).

Metavariables in RPL fall into two main categories based on whether they require an explicit declaration: implicitly declared and explicitly declared.

## Implicitly Declared Metavariables

The first category includes `$S` and `$field`, which are used to abstract away concrete names. They represent a form of lightweight, contextually-inferred binding. Because their roles as placeholders for a struct or field name are unambiguous from the syntax, they don't require an explicit declaration.

There are two kinds of implicit declaration:

1. abstracting the names of Rust items like structs, enums, functions.
2. abstracting a local name in MIR.

## Explicitly Declared Metavariables

The second category consists of metavariables that require explicit declaration in the brackets after the pattern item name (`[...]`). For instance, `$T` is a `type` metavariable used in this pattern to abstract the pointer's underlying type. The explicit declaration of these metavariables is necessary for the analysis process to understand their intended purpose and apply specific constraints.

There are three kinds of explicit declaration:

1. `type` metavariable: used to abstract a type.
2. `const` metavariable: used to abstract a constant value.
3. `place` metavariable: used to abstract a memory location.

## Special Wildcards and Placeholders

In addition to the named metavariables described above, RPL provides two special unnamed wildcards (`_` and `..`) to abstract away irrelevant details, enabling the creation of more general and focused patterns.

### The Single Item Wildcard: `_`

The underscore (`_`) acts as a wildcard that matches a single, arbitrary item. Its meaning is context-dependent, but it generally signifies that a value is present, but its specific content or origin is not relevant to the pattern.

Its most common and important use is on the right-hand side (r-value) of a mir statement:

```rust
let $local: $T = _;
```

This statement asserts that a local variable `$local` of type `$T` exists within the function body. It abstracts away the variable's specific origin, meaning it could be initialized from a function parameter, a newly computed value, or any other expression.

The `_` can also be used in other positions, such as in a function signature (`fn _`) to match a function with any name, or as a return type (`-> _`) to match a function with any return type.

### The Variadic Wildcard: `..`

The double-dot (`..`) is a variadic wildcard used to match **zero or more** items in a sequence. Its primary application is for abstracting the arguments of a function.

In a function signature, `..` can match any number of parameters of any type. For example, the following pattern will match any function, regardless of its arity (the number of arguments it takes):

```rust
fn $any_function (..) -> _
```

This is crucial for creating patterns that focus on a function's name or its internal logic without being constrained by its specific signature. Furthermore, it can be combined with concrete parameters to find a specific argument anywhere in the list. For instance, `fn _ (.., $target: String, ..)` matches a function that takes a `String` at any position in its argument list.
