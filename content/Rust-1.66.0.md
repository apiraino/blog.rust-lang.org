+++
path = "2022/12/15/Rust-1.66.0"
title = "Announcing Rust 1.66.0"
authors = ["The Rust Release Team"]
aliases = [
    "2022/12/15/Rust-1.66.0.html",
    "releases/1.66.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.66.0. Rust is a
programming language empowering everyone to build reliable and efficient
software.

If you have a previous version of Rust installed via rustup, you can get 1.66.0
with:

```
$ rustup update stable
```

If you don't have it already, you can [get
`rustup`](https://www.rust-lang.org/install.html) from the appropriate page on
our website, and check out the [detailed release notes for
1.66.0](https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1660-2022-12-15)
on GitHub.

If you'd like to help us out by testing future releases, you might consider
updating locally to use the beta channel (`rustup default beta`) or the nightly
channel (`rustup default nightly`). Please
[report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you
might come across!

## What's in 1.66.0 stable

### Explicit discriminants on enums with fields

Enums with integer representations can now use explicit discriminants, even when they have fields.

```rust
#[repr(u8)]
enum Foo {
    A(u8),
    B(i8),
    C(bool) = 42,
}
```

Previously, you could use explicit discriminants on enums with representations, but only if none of their variants had fields. Explicit discriminants are useful when passing values across language boundaries where the representation of the enum needs to match in both languages. For example,

```rust
#[repr(u8)]
enum Bar {
    A,
    B,
    C = 42,
    D,
}
```

Here the `Bar` enum is guaranteed to have the same layout as `u8`. In addition, the `Bar::C` variant is guaranteed to have a discriminant of 42. Variants without explicitly-specified values will have discriminants that are automatically assigned according to their order in the source code, so `Bar::A` will have a discriminant of 0, `Bar::B` will have a discriminant of 1, and `Bar::D` will have a discriminant of 43. Without this feature, the only way to set the explicit value of `Bar::C` would be to add 41 unnecessary variants before it!

Note: whereas for field-less enums it is possible to inspect a discriminant via `as` casting (e.g. `Bar::C as u8`), Rust provides no language-level way to access the raw discriminant of an enum with fields. Instead, currently unsafe code must be used to inspect the discriminant of an enum with fields. Since this feature is intended for use with cross-language FFI where unsafe code is already necessary, this should hopefully not be too much of an extra burden. In the meantime, if all you need is an opaque handle to the discriminant, please see the `std::mem::discriminant` function.

### `core::hint::black_box`

When benchmarking or examining the machine code produced by a compiler, it's often useful to prevent optimizations from occurring in certain places. In the following example, the function `push_cap` executes `Vec::push` 4 times in a loop:

```rust
fn push_cap(v: &mut Vec<i32>) {
    for i in 0..4 {
        v.push(i);
    }
}

pub fn bench_push() -> Duration { 
    let mut v = Vec::with_capacity(4);
    let now = Instant::now();
    push_cap(&mut v);
    now.elapsed()
}
```

If you inspect the optimized output of the compiler on x86_64, you'll notice that it looks rather short:

```asm
example::bench_push:
  sub rsp, 24
  call qword ptr [rip + std::time::Instant::now@GOTPCREL]
  lea rdi, [rsp + 8]
  mov qword ptr [rsp + 8], rax
  mov dword ptr [rsp + 16], edx
  call qword ptr [rip + std::time::Instant::elapsed@GOTPCREL]
  add rsp, 24
  ret
```

In fact, the entire function `push_cap` we wanted to benchmark has been optimized away!

We can work around this using the newly stabilized `black_box` function. Functionally, `black_box` is not very interesting: it takes the value you pass it and passes it right back. Internally, however, the compiler treats `black_box` as a function that could do anything with its input and return any value (as its name implies).

This is very useful for disabling optimizations like the one we see above. For example, we can hint to the compiler that the vector will actually be used for something after every iteration of the for loop.

```rust
use std::hint::black_box;

fn push_cap(v: &mut Vec<i32>) {
    for i in 0..4 {
        v.push(i);
        black_box(v.as_ptr());
    }
}
```

Now we can find the unrolled for loop in our [optimized assembly output](https://rust.godbolt.org/z/Ws1GGbY6Y):

```asm
  mov dword ptr [rbx], 0
  mov qword ptr [rsp + 8], rbx
  mov dword ptr [rbx + 4], 1
  mov qword ptr [rsp + 8], rbx
  mov dword ptr [rbx + 8], 2
  mov qword ptr [rsp + 8], rbx
  mov dword ptr [rbx + 12], 3
  mov qword ptr [rsp + 8], rbx
```

You can also see a side effect of calling `black_box` in this assembly output. The instruction `mov qword ptr [rsp + 8], rbx` is uselessly repeated after every iteration. This instruction writes the address `v.as_ptr()` as the first argument of the function, which is never actually called.

Notice that the generated code is not at all concerned with the possibility of allocations introduced by the `push` call. This is because the compiler is still using the fact that we called `Vec::with_capacity(4)` in the `bench_push` function. You can play around with the placement of `black_box`, or try using it in multiple places, to see its effects on compiler optimizations.

### cargo remove

In Rust 1.62.0 we introduced `cargo add`, a command line utility to add dependencies to your project. Now you can use `cargo remove` to remove dependencies.

### Stabilized APIs

- [`proc_macro::Span::source_text`](https://doc.rust-lang.org/stable/proc_macro/struct.Span.html#method.source_text)
- [`u*::{checked_add_signed, overflowing_add_signed, saturating_add_signed, wrapping_add_signed}`](https://doc.rust-lang.org/stable/std/primitive.u8.html#method.checked_add_signed)
- [`i*::{checked_add_unsigned, overflowing_add_unsigned, saturating_add_unsigned, wrapping_add_unsigned}`](https://doc.rust-lang.org/stable/std/primitive.i8.html#method.checked_add_unsigned)
- [`i*::{checked_sub_unsigned, overflowing_sub_unsigned, saturating_sub_unsigned, wrapping_sub_unsigned}`](https://doc.rust-lang.org/stable/std/primitive.i8.html#method.checked_sub_unsigned)
- [`BTreeSet::{first, last, pop_first, pop_last}`](https://doc.rust-lang.org/stable/std/collections/struct.BTreeSet.html#method.first)
- [`BTreeMap::{first_key_value, last_key_value, first_entry, last_entry, pop_first, pop_last}`](https://doc.rust-lang.org/stable/std/collections/struct.BTreeMap.html#method.first_key_value)
- [Add `AsFd` implementations for stdio lock types on WASI.](https://github.com/rust-lang/rust/pull/101768/)
- [`impl TryFrom<Vec<T>> for Box<[T; N]>`](https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#impl-TryFrom%3CVec%3CT%2C%20Global%3E%3E-for-Box%3C%5BT%3B%20N%5D%2C%20Global%3E)
- [`core::hint::black_box`](https://doc.rust-lang.org/stable/std/hint/fn.black_box.html)
- [`Duration::try_from_secs_{f32,f64}`](https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.try_from_secs_f32)
- [`Option::unzip`](https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.unzip)
- [`std::os::fd`](https://doc.rust-lang.org/stable/std/os/fd/index.html)

### Other changes

There are other changes in the Rust 1.66 release, including:

- You can now use `..=X` ranges in patterns.
- Linux builds now optimize the rustc frontend and LLVM backend with LTO and BOLT, respectively, improving both runtime performance and memory usage.

Check out everything that changed in
[Rust](https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1660-2022-12-15),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-166-2022-12-15),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-166).

### Contributors to 1.66.0

Many people came together to create Rust 1.66.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.66.0/)
