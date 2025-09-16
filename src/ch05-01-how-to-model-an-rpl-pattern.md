# How to Model an RPL Pattern

This section provides a detailed, step-by-step guide to modeling an RPL pattern, using CVE-2020-35881 as a practical example. We will cover all the necessary commands and processes.

> Note: All commands should be run from the root directory of the RPL project.

## 1. Obtain the MIR of the Relevant Code Snippet

First, create a new file that reproduces the vulnerable code. This file will also serve as the test case for your new pattern. For this example, create the file `tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs` and add the following code (this will also serve as the test case for the corresponding pattern):

```rust
use std::mem;

#[rpl::dump_mir(dump_cfg, dump_ddg)]
pub unsafe fn get_data<T: ?Sized>(val: *const T) -> *const () {
    unsafe { *mem::transmute::<*const *const T, *const *const ()>(&val) }
}

#[rpl::dump_mir(dump_cfg, dump_ddg)]
pub unsafe fn get_data_mut<T: ?Sized>(mut val: *mut T) -> *mut () {
    unsafe { *mem::transmute::<*mut *mut T, *mut *mut ()>(&mut val) }
}

fn main() {}
```

Note the `#[rpl::dump_mir]` attributes on lines 3 and 8, which instruct the compiler to dump the MIR for these functions.

Next, run the following command to process the file:

```bash
cargo uibless tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs
```

You may see the following output:

```bash
cargo uibless tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.19s
     Running tests/compile-test.rs (target/debug/deps/compile_test-251936bcca420e4b)
tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs ... FAILED

FAILED TEST: ...

error: there were 1 unmatched diagnostics
 --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:3:1
  |
3 | #[rpl::dump_mir(dump_cfg, dump_ddg)]
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Error: abort due to debugging
  |

error: there were 1 unmatched diagnostics
 --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:5:15
  |
5 |     unsafe { *mem::transmute::<*const *const T, *const *const ()>(&val) }
  |               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Error[rpl::wrong_assumption_of_fat_pointer_layout]: wrong assumption of fat pointer layout
  |

error: there were 1 unmatched diagnostics
 --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:8:1
  |
8 | #[rpl::dump_mir(dump_cfg, dump_ddg)]
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Error: abort due to debugging
  |

error: there were 1 unmatched diagnostics
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:15
   |
10 |     unsafe { *mem::transmute::<*mut *mut T, *mut *mut ()>(&mut val) }
   |               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Error[rpl::wrong_assumption_of_fat_pointer_layout]: wrong assumption of fat pointer layout
   |

error: expected error patterns, but found none

full stderr:
error: wrong assumption of fat pointer layout
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:5:15
   |
LL |     unsafe { *mem::transmute::<*const *const T, *const *const ()>(&val) }
   |              -^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |              ||
   |              |ptr transmute here
   |              try to get data ptr from first 8 bytes here
   |
   = help: the Rust Compiler does not expose the layout of fat pointers
   = note: `#[deny(rpl::wrong_assumption_of_fat_pointer_layout)]` on by default

error: wrong assumption of fat pointer layout
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:15
   |
LL |     unsafe { *mem::transmute::<*mut *mut T, *mut *mut ()>(&mut val) }
   |              -^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |              ||
   |              |ptr transmute here
   |              try to get data ptr from first 8 bytes here
   |
   = help: the Rust Compiler does not expose the layout of fat pointers

note: MIR of `get_data`
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:4:1
   |
LL |   #[rpl::dump_mir(dump_cfg, dump_ddg)]
   |   ------------------------------------ MIR dumped because of this attribute
LL | / pub unsafe fn get_data<T: ?Sized>(val: *const T) -> *const () {
LL | |     unsafe { *mem::transmute::<*const *const T, *const *const ()>(&val) }
LL | | }
   | |_^
   |
   = note: see `/Users/stuuupidcat/home/code/projects/RPL/mir_dump/cve_2020_35881_test.get_data.-------.dump_mir..mir` for dumped MIR
   = note: see `/Users/stuuupidcat/home/code/projects/RPL/mir_dump/cve_2020_35881_test.get_data.-------.dump_mir..mir.cfg.dot` for dumped control flow graph
   = note: see `/Users/stuuupidcat/home/code/projects/RPL/mir_dump/cve_2020_35881_test.get_data.-------.dump_mir..mir.ddg.dot` for dumped data dependency graph
note: locals and scopes in this MIR
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:4:1
   |
LL |   pub unsafe fn get_data<T: ?Sized>(val: *const T) -> *const () {
   |   ^                                 ---               --------- _0: *const (); // scope[0]
   |   |                                 |
   |  _|                                 _1: *const T; // scope[0]
   | |
LL | |     unsafe { *mem::transmute::<*const *const T, *const *const ()>(&val) }
   | |               ---------------------------------------------------------
   | |               |                                                   |
   | |               |                                                   _3: *const *const T; // scope[0]
   | |               |                                                   _4: &*const T; // scope[0]
   | |               _2: *const *const (); // scope[0]
LL | | }
   | |_^ scope[0]
note: bb0: {
          _4 = &_1; // scope[0]
          _3 = &raw const (*_4); // scope[0]
          _2 = move _3 as *const *const () (Transmute); // scope[0]
          _0 = copy (*_2); // scope[0]
          return; // scope[0]
      }
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:6:2
   |
LL |     unsafe { *mem::transmute::<*const *const T, *const *const ()>(&val) }
   |              ----------------------------------------------------------
   |              ||                                                   |
   |              ||                                                   _4 = &_1; // scope[0]
   |              ||                                                   _3 = &raw const (*_4); // scope[0]
   |              |_2 = move _3 as *const *const () (Transmute); // scope[0]
   |              _0 = copy (*_2); // scope[0]
LL | }
   |  ^ return; // scope[0]

note: MIR of `get_data_mut`
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:9:1
   |
LL |   #[rpl::dump_mir(dump_cfg, dump_ddg)]
   |   ------------------------------------ MIR dumped because of this attribute
LL | / pub unsafe fn get_data_mut<T: ?Sized>(mut val: *mut T) -> *mut () {
LL | |     unsafe { *mem::transmute::<*mut *mut T, *mut *mut ()>(&mut val) }
LL | | }
   | |_^
   |
   = note: see `/Users/stuuupidcat/home/code/projects/RPL/mir_dump/cve_2020_35881_test.get_data_mut.-------.dump_mir..mir` for dumped MIR
   = note: see `/Users/stuuupidcat/home/code/projects/RPL/mir_dump/cve_2020_35881_test.get_data_mut.-------.dump_mir..mir.cfg.dot` for dumped control flow graph
   = note: see `/Users/stuuupidcat/home/code/projects/RPL/mir_dump/cve_2020_35881_test.get_data_mut.-------.dump_mir..mir.ddg.dot` for dumped data dependency graph
note: locals and scopes in this MIR
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:9:1
   |
LL |   pub unsafe fn get_data_mut<T: ?Sized>(mut val: *mut T) -> *mut () {
   |   ^                                     -------             ------- _0: *mut (); // scope[0]
   |   |                                     |
   |  _|                                     _1: *mut T; // scope[0]
   | |
LL | |     unsafe { *mem::transmute::<*mut *mut T, *mut *mut ()>(&mut val) }
   | |               -----------------------------------------------------
   | |               |                                           |
   | |               |                                           _3: *mut *mut T; // scope[0]
   | |               |                                           _4: &mut *mut T; // scope[0]
   | |               _2: *mut *mut (); // scope[0]
LL | | }
   | |_^ scope[0]
note: bb0: {
          _4 = &mut _1; // scope[0]
          _3 = &raw mut (*_4); // scope[0]
          _2 = move _3 as *mut *mut () (Transmute); // scope[0]
          _0 = copy (*_2); // scope[0]
          return; // scope[0]
      }
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:11:2
   |
LL |     unsafe { *mem::transmute::<*mut *mut T, *mut *mut ()>(&mut val) }
   |              ------------------------------------------------------
   |              ||                                           |
   |              ||                                           _4 = &mut _1; // scope[0]
   |              ||                                           _3 = &raw mut (*_4); // scope[0]
   |              |_2 = move _3 as *mut *mut () (Transmute); // scope[0]
   |              _0 = copy (*_2); // scope[0]
LL | }
   |  ^ return; // scope[0]

error: abort due to debugging
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:3:1
   |
LL | #[rpl::dump_mir(dump_cfg, dump_ddg)]
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: remove this attribute
   |
   = note: `#[rpl::dump_hir]`, `#[rpl::print_hir]` and `#[rpl::dump_mir]` are only used for debugging
   = note: this error is to remind you removing these attributes

error: abort due to debugging
  --> tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:8:1
   |
LL | #[rpl::dump_mir(dump_cfg, dump_ddg)]
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ help: remove this attribute
   |
   = note: `#[rpl::dump_hir]`, `#[rpl::print_hir]` and `#[rpl::dump_mir]` are only used for debugging
   = note: this error is to remind you removing these attributes

error: aborting due to 4 previous errors


full stdout:


FAILURES:
    tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs

test result: FAIL. 1 failed; 98 filtered out


thread 'main' panicked at tests/compile-test.rs:212:6:
called `Result::unwrap()` on an `Err` value: tests failed

Location:
    /Users/stuuupidcat/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/ui_test-0.29.2/src/lib.rs:369:13
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: test failed, to rerun pass `--test compile-test`
```

You may see a series of errors. This is expected and can be disregarded. The output in this case indicates two things:

1. You have successfully dumped the MIR. The process ends with an `Error: abort due to debugging` message because #[rpl::dump_mir] is a debug-only attribute.
2. You may see an error: `there were 1 unmatched diagnostics` message. This occurs because the pattern for this CVE already exists in our library, and it has correctly detected the vulnerability in your new test file. However, you haven't added/labeled the possible error output to the test file, so the 'diagnostics is unmatched'.

After running the command, you will find a new `mir_dump` directory in the project root:

```bash
➜  RPL git:(dev) ✗ la mir_dump
total 48
-rw-r--r--@ 1 stuuupidcat  staff   1.3K  9 15 17:30 cve_2020_35881_test.get_data_mut.-------.dump_mir..mir
-rw-r--r--@ 1 stuuupidcat  staff   365B  9 15 17:30 cve_2020_35881_test.get_data_mut.-------.dump_mir..mir.cfg.dot
-rw-r--r--@ 1 stuuupidcat  staff   1.7K  9 15 17:30 cve_2020_35881_test.get_data_mut.-------.dump_mir..mir.ddg.dot
-rw-r--r--@ 1 stuuupidcat  staff   1.3K  9 15 17:30 cve_2020_35881_test.get_data.-------.dump_mir..mir
-rw-r--r--@ 1 stuuupidcat  staff   367B  9 15 17:30 cve_2020_35881_test.get_data.-------.dump_mir..mir.cfg.dot
-rw-r--r--@ 1 stuuupidcat  staff   1.7K  9 15 17:30 cve_2020_35881_test.get_data.-------.dump_mir..mir.ddg.dot
```

This directory contains the raw MIR (`.mir`), Control-Flow Graph (`.cfg.dot`), and Data-Dependence Graph (`.ddg.dot`) for the target functions. The `.dot` files can be visualized using tools like Graphviz.

Let's examine the MIR for `get_data_mut` by opening the corresponding `cve_2020_35881_test.get_data_mut.-------.dump_mir..mir` file. You will see the following content:

```rust
fn get_data_mut(_1: *mut T) -> *mut () {
    debug val => _1;                     // in scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:9:39: 9:46
    let mut _0: *mut ();                 // return place in scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:9:59: 9:66
    let mut _2: *mut *mut ();            // in scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:15: 10:68
    let mut _3: *mut *mut T;             // in scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:59: 10:67
    let mut _4: &mut *mut T;             // in scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:59: 10:67

    bb0: {
        _4 = &mut _1;                    // scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:59: 10:67
        _3 = &raw mut (*_4);             // scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:59: 10:67
        _2 = move _3 as *mut *mut () (Transmute); // scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:15: 10:68
        _0 = copy (*_2);                 // scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:10:14: 10:68
        return;                          // scope 0 at tests/ui/cve/cve_2020_35881_test/cve_2020_35881_test.rs:11:2: 11:2
    }
}
```

This is the raw material for our pattern. In this case, all of the MIR statements are relevant and will be modeled.

## 2. Abstract the MIR to Form a Pattern

Now that we have the MIR, the next step is to convert it into a generic RPL pattern. Create a new file at `docs/patterns-pest/cve/cve_2020_35881.rpl` and follow these steps.

First, give your pattern a name:

```rust
pattern CVE-2020-35881
```

Next, add a patt block and begin translating the MIR statements. The process involves "hollowing out" the concrete MIR by replacing specific locals and types with abstract metavariables.

```rust
patt {
    const_const[
        $T: type,
    ] = {
        fn _(..) -> _ {
            let $ptr: *const $T = _;
            // _4 = &_1;
            let $ref_to_ptr: &*const $T = &$ptr;
            // _3 = &raw const (*_4);
            let $ptr_to_ptr_t: *const *const $T = &raw const (*$ref_to_ptr);
            // _2 = move _3 as *const *const () (Transmute);
            'ptr_transmute:
            let $ptr_to_ptr: *const *const() = move $ptr_to_ptr_t as *const *const () (Transmute);
            // _0 = copy (*_2);
            'data_ptr_get:
            let $data_ptr: *const () = copy (*$ptr_to_ptr);
        }
    }
}
```

In the `const_const` pattern item above (which models the `get_data` function), the first statement `let $ptr: *const $T = _;` indicates that we are looking for a pointer of type `*const $T` that comes from an arbitrary source (`_`). The subsequent statements directly correspond to the MIR from the `get_data` function.

Note the key differences between the raw MIR and the RPL pattern:

1.  Concrete local (`_1`, `_2`, etc.) have been replaced with abstract metavariables with descriptive names (e.g., `$ptr`, `$ref_to_ptr`).

2.  The `let` keyword has been added to form a valid RPL statement declaration.

3.  The `'ptr_transmute:` and `'data_ptr_get:` labels have been added to key statements. These labels allow us to reference these specific points in the code when creating diagnostic messages.

## 3. Add Diagnostic Information

```rust
diag {
    const_const = {
        primary(ptr_transmute) = "wrong assumption of fat pointer layout",
        label(ptr_transmute)   = "ptr transmute here",
        label(data_ptr_get)    = "try to get data ptr from first 8 bytes here",
        help                   = "the Rust Compiler does not expose the layout of fat pointers",
        name                   = "wrong_assumption_of_fat_pointer_layout",
        level                  = "deny",
    }
}
```

The block maps a diagnostic name (here, `const_const`) to a set of properties that build the final compiler output.

## 4. Complete the Rule with All Variants

Finally, we complete the rule by adding the pattern for the mutable case (`mut_mut`) and reusing the diagnostic message from the `const_const` pattern. The complete file looks like this:

```rust
pattern CVE-2020-35881

patt {
    #[diag = "fat_pointer"]
    const_const[
        $T: type,
    ] = {
        fn _ (..) -> _ {
            let $ptr: *const $T = _;
            let $ref_to_ptr: &*const $T = &$ptr;
            let $ptr_to_ptr_t: *const *const $T = &raw const (*$ref_to_ptr);
            'ptr_transmute:
            let $ptr_to_ptr: *const *const() = move $ptr_to_ptr_t as *const *const () (Transmute);
            'data_ptr_get:
            let $data_ptr: *const () = copy (*$ptr_to_ptr);
        }
    }
    #[diag = "fat_pointer"]
    mut_mut[
        $T: type,
    ] = {
        fn _ (..) -> _ {
            let $ptr: *mut $T = _;
            let $ref_to_ptr: &mut *mut $T = &mut $ptr;
            let $ptr_to_ptr_t: *mut *mut $T = &raw mut (*$ref_to_ptr);
            'ptr_transmute:
            let $ptr_to_ptr: *mut *mut() = move $ptr_to_ptr_t as *mut *mut () (Transmute);
            'data_ptr_get:
            let $data_ptr: *mut () = copy (*$ptr_to_ptr);
        }
    }
}

diag {
    fat_pointer = {
        primary(ptr_transmute) = "wrong assumption of fat pointer layout",
        label(ptr_transmute)   = "ptr transmute here",
        label(data_ptr_get)    = "try to get data ptr from first 8 bytes here",
        help                   = "the Rust Compiler does not expose the layout of fat pointers",
        name                   = "wrong_assumption_of_fat_pointer_layout",
        level                  = "deny",
    }
}
```

The final rule contains two pattern items, `const_const` and `mut_mut`, to handle both immutable and mutable pointer variations. The `#[diag = "fat_pointer"]` attribute links both patterns to the same diagnostic message defined in the diag block.

> One might also consider the `const_mut` and `mut_const` variants. We have intentionally omitted a mechanism for metavariables to abstract over mutability due to a design trade-off. Although such a feature would allow the four mutability variants to be consolidated into one concise pattern, we believe this would sacrifice the clarity and readability of the rule.

In the next chapter, we will introduce how to ensure the correctness of the pattern through unit tests.
