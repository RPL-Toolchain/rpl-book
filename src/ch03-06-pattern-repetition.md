# Pattern Repetition (TBC)

A motivating example is described as follows:

## Clippy lint: `cast-slice-different-sizes`

Checks for `as` casts between raw pointers to slices with differently sized elements.

The produced raw pointer to a slice **does not update its length metadata**. Producing a slice reference from the raw pointer will either create a slice with less data (which can be surprising) or create a slice with more data and cause Undefined Behavior.

## Examples

```rust
// missing data
let a = [1_i32, 2, 3, 4]; // 4
let p = &a as *const [i32] as *const [u8]; // 16
unsafe {
    println!("{:?}", &*p);
}
```

```rust
// Undefined Behavior (note: also potential alignment issues)
let a = [1_u8, 2, 3, 4]; // 4
let p = &a as *const [u8] as *const [u32]; // 1
unsafe {
    println!("{:?}", &*p); // 4个u32
}
```

## A RPL pattern would cause a false positive

```rust
pattern cast-slice-different-sizes

patt {
    p[
        $T: type, // element type before the cast
        $U: type, // element type after the cast
    ] = fn _ (..) -> _ {
        'src:
        let $p: *const [$T] = _;
        'cast:
        let $q: *const [$U] = move $p as *const [$U] (PtrToPtr);
    } where {
        !compatible_layout($T, $U)
    }
}
```

The ideal pattern should not match the following code, but the current pattern will match it:

```rust
let x: [i32; 3] = [1, 2, 3];
let r_x = &x;

let long_chain_restore =
    r_x as *const [i32]
		as *const [u32]
		as *const [u16]
		as *const [i8]
		as *const [u8]
		as *const [u32];
```

## A solution inspired by the declarative macros in the Rust language

```rust
pattern cast-slice-different-sizes

patt {
    p[
        $T: type,
        $($U: type)+,
        $($W: type)+,
        $($p_in: local)+,
        $($p_out: local)+,
    ] = fn _ (..) -> _ {
        'src:
        let $src: *const [$T] = _;
        'casts: ${
            let $p_in: *const [$U];
            let $p_out: *const [$W] = move $p_in as *const [$W] (PtrToPtr);
        }+
    } where {
        !compatible_layout($T, tail($($W)+)),
        head($($p_in)+) == $src,
        // list comprehension
        all(
            [nth($($p_out)+, i) == nth($($p_in)+, i+1) | i in [0..len($($p_out)+) - 1]]
        )
    }
}
```

## Another solution inspired by the Semgrep

```rust
pattern cast-slice-different-sizes

patt {
    p[
        $T: type,
        $N: usize,
        $..Us: [type; $N],
        $..Ws: [type; $N],
    ] = fn _ (..) -> _ {
        'src:
        let $src: *const [$T] = _;
        'casts: {
            let $..p_ins: *const [$..Us];
            let $..p_outs: *const [$..Ws] = move $..p_ins as *const [$..Ws] (PtrToPtr);
        }+
        // Some problem here.
        // What does $..Us and $..Ws mean? (one or multiple?)
    } where {
        ...
    }
}
```
