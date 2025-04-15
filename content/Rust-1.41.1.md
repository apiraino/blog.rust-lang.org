+++
path = "2020/02/27/Rust-1.41.1"
title = "Announcing Rust 1.41.1"
authors = ["The Rust Release Team"]
aliases = [
    "2020/02/27/Rust-1.41.1.html",
    "releases/1.41.1",
]

[extra]
release = true
+++

The Rust team has published a new point release of Rust, 1.41.1.
Rust is a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.41.1 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the appropriate page on our website.

[install]: https://www.rust-lang.org/tools/install

## What's in 1.41.1 stable

Rust 1.41.1 addresses two critical regressions introduced in Rust 1.41.0:
a soundness hole related to static lifetimes, and a miscompilation causing segfaults.
These regressions do not affect earlier releases of Rust,
and we recommend users of Rust 1.41.0 to upgrade as soon as possible.
Another issue related to interactions between `'static` and `Copy` implementations,
dating back to Rust 1.0, was also addressed by this release.

### A soundness hole in checking `static` items

In Rust 1.41.0, due to some changes in the internal representation of `static` values,
the borrow checker accidentally allowed some unsound programs.
Specifically, the borrow checker would not check that `static` items had the correct type.
This in turn would allow the assignment of a temporary,
with a lifetime less than `'static`, to a `static` variable:

```rust
static mut MY_STATIC: &'static u8 = &0;

fn main() {
    let my_temporary = 42;
    unsafe {
        // Erroneously allowed in 1.41.0:
        MY_STATIC = &my_temporary;
    }
}
```

This was addressed in 1.41.1, with the program failing to compile:
```
error[E0597]: `my_temporary` does not live long enough
 --> src/main.rs:6:21
  |
6 |         MY_STATIC = &my_temporary;
  |         ------------^^^^^^^^^^^^^
  |         |           |
  |         |           borrowed value does not live long enough
  |         assignment requires that `my_temporary` is borrowed for `'static`
7 |     }
8 | }
  | - `my_temporary` dropped here while still borrowed

```

You can learn more about this bug in [issue #69114][69114] and the [PR that fixed it][pr_69145].

[69114]: https://github.com/rust-lang/rust/issues/69114
[pr_69145]: https://github.com/rust-lang/rust/pull/69145

### Respecting a `'static` lifetime in a `Copy` implementation

[1.40.0_post]: https://blog.rust-lang.org/2019/12/19/Rust-1.40.0.html#borrow-check-migration-warnings-are-hard-errors-in-rust-2015

Ever since Rust 1.0, the following erroneous program has been compiling:

```rust
#[derive(Clone)]
struct Foo<'a>(&'a u32);
impl Copy for Foo<'static> {}

fn main() {
    let temporary = 2;
    let foo = (Foo(&temporary),);
    drop(foo.0); // Accessing a part of `foo` is necessary.
    drop(foo.0); // Indexing an array would also work.
}
```

In Rust 1.41.1, this issue was fixed [by the same PR as the one above][pr_69145].
Compiling the program now produces the following error:

```rust
error[E0597]: `temporary` does not live long enough
  --> src/main.rs:7:20
   |
7  |     let foo = (Foo(&temporary),);
   |                    ^^^^^^^^^^ borrowed value does not live long enough
8  |     drop(foo.0);
   |          ----- copying this value requires that
   |                `temporary` is borrowed for `'static`
9  |     drop(foo.0);
10 | }
   | - `temporary` dropped here while still borrowed
```

This error occurs because `Foo<'a>`, for some `'a`, only implements `Copy` when `'a: 'static`.
However, the `temporary` variable,
with some lifetime `'0` does not outlive `'static` and hence `Foo<'0>` is not `Copy`,
so using `drop` the second time around should be an error.

### Miscompiled bound checks leading to segfaults

In a few cases, programs compiled with Rust 1.41.0 were omitting bound checks in the memory allocation code.
This caused segfaults if out of bound values were provided.
The root cause of the miscompilation was a change in a LLVM optimization pass,
introduced in LLVM 9 and reverted in LLVM 10.

Rust 1.41.0 uses a snapshot of LLVM 9, so we cherry-picked the revert into Rust 1.41.1,
addressing the miscompilation. [You can learn more about this bug in issue #69225][69225].

[69225]: https://github.com/rust-lang/rust/issues/69225

## Contributors to 1.41.1

Many people came together to create Rust 1.41.1.
We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.41.1/)
