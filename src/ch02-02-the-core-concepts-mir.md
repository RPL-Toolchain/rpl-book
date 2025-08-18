# The Core Concepts of MIR

## Control-Flow Graph

The foundational structure of MIR is the Control-Flow Graph. The CFG represents a program as a directed graph where nodes are basic blocks and edges represent the flow of control between them.

### BasicBlock

A BasicBlock is a node in the CFG. It is a straight-line sequence of code with a single entry point and a single exit point, where no branches or jumps occur within the block. In Rust MIR, a basic block is defined as a sequence of zero or more Statements followed by exactly one Terminator. This structure guarantees that control flow can only enter at the beginning of the block and can only exit at the end, via the terminator. There are no branches or jumps in the middle of a basic block.

In current version of rustc (1.91.0-nightly), the data structure of a basic block is defined as follows:

```rust
#[non_exhaustive]
pub struct BasicBlockData<'tcx> {
    pub statements: Vec<Statement<'tcx>>,
    pub terminator: Option<Terminator<'tcx>>,
    pub is_cleanup: bool, // Whether this block is a cleanup block in an unwind path
}
```

### Statement

A Statement represents an action that occurs within a basic block. Crucially, statements have a single, implicit successor: the next statement in the block or, if it is the last one, the block's terminator.

In current version of rustc (1.91.0-nightly), there are 14 kinds of statements.

```rust
pub enum StatementKind<'tcx> {
    Assign(Box<(Place<'tcx>, Rvalue<'tcx>)>),
    FakeRead(Box<(FakeReadCause, Place<'tcx>)>),
    SetDiscriminant {
        place: Box<Place<'tcx>>,
        variant_index: VariantIdx,
    },
    Deinit(Box<Place<'tcx>>),
    StorageLive(Local),
    StorageDead(Local),
    Retag(RetagKind, Box<Place<'tcx>>),
    PlaceMention(Box<Place<'tcx>>),
    AscribeUserType(Box<(Place<'tcx>, UserTypeProjection)>, Variance),
    Coverage(CoverageKind),
    Intrinsic(Box<NonDivergingIntrinsic<'tcx>>),
    ConstEvalCounter,
    Nop,
    BackwardIncompatibleDropHint {
        place: Box<Place<'tcx>>,
        reason: BackwardIncompatibleDropReason,
    },
}
```

> In most cases, we only use Assign statements when modeling the logic of a function body.

### Terminator

A Terminator is the final instruction in every basic block and is the sole mechanism for directing control flow between blocks. It explicitly defines the successor(s) to the current block. This is where all branching, function calls, returns, and panics are represented. A terminator can have zero successors (e.g., return), one successor (e.g., goto), or multiple successors (e.g., switchInt for an if or match, or Call which has a success path and a potential unwind path).

In current version of rustc (1.91.0-nightly), there are 10 kinds of terminators.

```rust
pub enum TerminatorKind<'tcx> {
    Goto {
        target: BasicBlock,
    },
    SwitchInt {
        discr: Operand<'tcx>,
        targets: SwitchTargets,
    },
    UnwindResume,
    UnwindTerminate(UnwindTerminateReason),
    Return,
    Unreachable,
    Drop {
        place: Place<'tcx>,
        target: BasicBlock,
        unwind: UnwindAction,
        replace: bool,
        drop: Option<BasicBlock>,
        async_fut: Option<Local>,
    },
    Call {
        func: Operand<'tcx>,
        args: Box<[Spanned<Operand<'tcx>>]>,
        destination: Place<'tcx>,
        target: Option<BasicBlock>,
        unwind: UnwindAction,
        call_source: CallSource,
        fn_span: Span,
    },
    TailCall {
        func: Operand<'tcx>,
        args: Box<[Spanned<Operand<'tcx>>]>,
        fn_span: Span,
    },
    Assert {
        cond: Operand<'tcx>,
        expected: bool,
        msg: Box<AssertMessage<'tcx>>,
        target: BasicBlock,
        unwind: UnwindAction,
    },
    Yield {
        value: Operand<'tcx>,
        resume: BasicBlock,
        resume_arg: Place<'tcx>,
        drop: Option<BasicBlock>,
    },
    CoroutineDrop,
    FalseEdge {
        real_target: BasicBlock,
        imaginary_target: BasicBlock,
    },
    FalseUnwind {
        real_target: BasicBlock,
        unwind: UnwindAction,
    },
    InlineAsm {
        asm_macro: InlineAsmMacro,
        template: &'tcx [InlineAsmTemplatePiece],
        operands: Box<[InlineAsmOperand<'tcx>]>,
        options: InlineAsmOptions,
        line_spans: &'tcx [Span],
        targets: Box<[BasicBlock]>,
        unwind: UnwindAction,
    },
}
```

> In most cases, we only use Goto, SwitchInt, Return, Drop, and Call statements when modeling the logic of a function body.

## Desugared Data Representation

In parallel with simplifying control flow, MIR also simplifies the representation of data and computations. Complex, nested expressions are broken down into a sequence of simple operations on a small set of data-related concepts.

### Local

Memory locations allocated on the stack (conceptually, at least).

This includes function arguments, user-declared variables, and compiler-generated temporary variables used to hold intermediate results. In MIR, locals are not identified by name but by a simple index, such as `\_0`, `\_1`, `\_2`, and so on. The local `\_0` is always reserved for the return value.

### Place

Expressions that identify a location in memory. It is the MIR equivalent of an "l-value" in C.

-   The simplest Place is just a Local (e.g., \_1).
-   More complex Places are formed by starting with a base Local and applying a sequence of **projections**, such as field access, array indexing, or pointer dereferencing. For example, \_1.f represents the field f of the struct stored in local \_1, and \*\_2 represents the memory location pointed to by the local \_2.

In current version of rustc (1.91.0-nightly), the data structure of a place is defined as follows:

```rust
pub struct Place<'tcx> {
    pub local: Local,
    pub projection: &'tcx List<PlaceElem<'tcx>>,
}

pub enum PlaceElem<'tcx> {
    Deref,
    Field(FieldIdx, Ty<'tcx>),
    Index(Local),
    ConstantIndex {
        offset: u64,
        min_length: u64,
        from_end: bool,
    },
    Subslice {
        from: u64,
        to: u64,
        from_end: bool,
    },
    Downcast(Option<Symbol>, VariantIdx),
    OpaqueCast(Ty<'tcx>),
    UnwrapUnsafeBinder(Ty<'tcx>),
    Subtype(Ty<'tcx>),
}
```

### Rvalue

Expressions that produce a value.

The "R" stands for the fact that these are the "right-hand side" of an assignment.

In current version of rustc (1.91.0-nightly), the data structure of an rvalue is defined as follows:

```rust
pub enum Rvalue<'tcx> {
    Use(Operand<'tcx>),
    Repeat(Operand<'tcx>, Const<'tcx>),
    Ref(Region<'tcx>, BorrowKind, Place<'tcx>),
    ThreadLocalRef(DefId),
    RawPtr(RawPtrKind, Place<'tcx>),
    Len(Place<'tcx>),
    Cast(CastKind, Operand<'tcx>, Ty<'tcx>),
    BinaryOp(BinOp, Box<(Operand<'tcx>, Operand<'tcx>)>),
    NullaryOp(NullOp<'tcx>, Ty<'tcx>),
    UnaryOp(UnOp, Operand<'tcx>),
    Discriminant(Place<'tcx>),
    Aggregate(Box<AggregateKind<'tcx>>, IndexVec<FieldIdx, Operand<'tcx>>),
    ShallowInitBox(Operand<'tcx>, Ty<'tcx>),
    CopyForDeref(Place<'tcx>),
    WrapUnsafeBinder(Operand<'tcx>, Ty<'tcx>),
}
```

### Operand

The arguments to an Rvalue.

In current version of rustc (1.91.0-nightly), the data structure of an operand is defined as follows:

```rust
pub enum Operand<'tcx> {
    Copy(Place<'tcx>),
    Move(Place<'tcx>),
    Constant(Box<ConstOperand<'tcx>>),
}
```
