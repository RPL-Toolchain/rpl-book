# File Structure and Code Blocks

The core syntax of the DSL is shown as follows:

```bnf
<rpl-pattern> ::= <rpl-header> <block-list>

<rpl-header> ::= `pattern' <identifier>

<block> ::= <patt-block>
          | <util-block>
          | <diag-block>

<block-list> ::= <block> <block-list> | <empty>

<patt-block> ::= `patt' `{' <use-path-list> <rpl-pattern-item-list> `}'

<util-block> ::= `util' `{' <use-path-list> <rpl-pattern-item-list> `}'

<diag-block> ::= `diag' `{' <diag-block-item-list> `}'

<rpl-pattern-item> ::= <attr-list> <identifier> (<meta-variable-decl-list> | <empty>) `=' <rust-items-or-pattern-operation>
```

We can see that:

1. a DSL file begins with a pattern declaration that assigns a name to the detection rule.
2. Following this declaration, the file is organized into three kinds of code blocks, each of which serves a specific and independent purpose.

## `patt` block

The `patt` block is the core of the rule definition, containing one or more named pattern items. Syntactically, each pattern item contains three parts: a unique name, a list of metavariable declarations, and a body. The body is used to define the specific content of the pattern and supports the following three forms:

1. modeling one or more abstracted Rust items;
2. customizing a single pattern;
3. performing logical operations on multiple patterns.

When a pattern item defines a single Rust item, it is typically a function. Alternatively, to model more complex scenarios, a pattern item can define multiple Rust items. A common example of this is pairing a `struct` or `enum` definition with its corresponding `impl` block to represent the relationship between an Abstract Data Type (ADT) and its corresponding methods.

## `util` block

The `util` block serves as a space for auxiliary definitions that support the patterns in the `patt` block. While the `patt` items are the public-facing targets for detection, the patterns defined within `util` blocks are private helpers.These can include reusable sub-patterns that would otherwise clutter the primary pattern definitions. This separation allows for more modular and readable rules by isolating the reusable utility logic from the specific code patterns that are being captured.

## `diag` block

The `diag` block is responsible for defining the diagnostic output that is presented to the user when a pattern from the `patt` block is successfully matched. It maps a pattern's name to a set of diagnostic information, which typically includes the following three components:

1. A severity level, such as `deny` or `warn`, to indicate the seriousness of the detected issue;
2. A primary descriptive message that explains the problem,
   which can be accompanied by specific labels that pinpoint locations in the code corresponding to metavariables or specific MIR statements;
3. supplemental information to aid the developer, such as a note to provide additional context, or a help message that suggests a potential fix or modification.

These fields can also include links to relevant documentation for further details. By separating the diagnostic messages from the pattern-matching logic, RPL allows for clear and maintainable rules where the detection and reporting concerns are independently managed.
