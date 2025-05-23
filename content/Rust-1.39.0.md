+++
path = "2019/11/07/Rust-1.39.0"
title = "Announcing Rust 1.39.0"
authors = ["The Rust Release Team"]
aliases = [
    "2019/11/07/Rust-1.39.0.html",
    "releases/1.39.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.39.0. Rust is a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.39.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the appropriate page on our website, and check out the [detailed release notes for 1.39.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1390-2019-11-07

## What's in 1.39.0 stable

The highlights of Rust 1.39.0 include `async`/`.await`, shared references to by-move bindings in `match` guards, and attributes on function parameters. Also, see the [detailed release notes][notes] for additional information.

### The `.await` is over, `async fn`s are here

[rel-1360]: https://blog.rust-lang.org/2019/07/04/Rust-1.36.0.html#the-future-is-here
[`Future`]: https://doc.rust-lang.org/nightly/std/future/trait.Future.html
[niko-post-async]: https://blog.rust-lang.org/2019/11/07/Async-await-stable.html

Previously in Rust 1.36.0, [we announced][rel-1360] that the [`Future`] trait is here. Back then, we noted that:

> With this stabilization, we hope to give important crates, libraries, and the ecosystem time to prepare for `async` / `.await`, which we'll tell you more about in the future.

A promise made is a promise kept. So in Rust 1.39.0, we are pleased to announce that `async` / `.await` is stabilized! Concretely, this means that you can define `async` functions and blocks and `.await` them.

An `async` function, which you can introduce by writing `async fn` instead of `fn`, does nothing other than to return a `Future` when called. This `Future` is a suspended computation which you can drive to completion by `.await`ing it. Besides `async fn`, `async { ... }` and `async move { ... }` blocks, which act like closures, can be used to define "async literals".

For more on the release of `async` / `.await`, read [Niko Matsakis's blog post][niko-post-async].

### References to by-move bindings in `match` guards

[pr-bind-by-move]: https://github.com/rust-lang/rust/pull/63118/#issuecomment-522823925

When pattern matching in Rust, a variable, also known as a "binding", can be bound in the following ways:

- by-reference, either immutably or mutably. This can be achieved explicitly e.g. through `ref my_var` or `ref mut my_var` respectively. Most of the time though, the binding mode will be inferred automatically.

- by-value -- either by-copy, when the bound variable's type implements `Copy`, or otherwise **_by-move_**.

Previously, Rust would forbid taking shared references to **_by-move_** bindings in the `if` guards of `match` expressions. This meant that the following code would be rejected:

```rust
fn main() {
    let array: Box<[u8; 4]> = Box::new([1, 2, 3, 4]);

    match array {
        nums
//      ---- `nums` is bound by move.
            if nums.iter().sum::<u8>() == 10
//                 ^------ `.iter()` implicitly takes a reference to `nums`.
        => {
            drop(nums);
//          ----------- `nums` was bound by move and so we have ownership.
        }
        _ => unreachable!(),
    }
}
```

[With Rust 1.39.0][pr-bind-by-move], the snippet above is now accepted by the compiler. We hope that this will give a smoother and more consistent experience with `match` expressions overall.

### Attributes on function parameters

[pr-attr]: https://github.com/rust-lang/rust/pull/64010/

With Rust 1.39.0, attributes are now allowed on parameters of functions, closures, and function pointers. Whereas before, you might have written:

```rust
#[cfg(windows)]
fn len(slice: &[u16]) -> usize {
    slice.len()
}
#[cfg(not(windows))] 
fn len(slice: &[u8]) -> usize {
    slice.len()
}
```

...[you can now][pr-attr], more succinctly, write:

```rust
fn len(
    #[cfg(windows)] slice: &[u16], // This parameter is used on Windows.
    #[cfg(not(windows))] slice: &[u8], // Elsewhere, this one is used.
) -> usize {
    slice.len()
}
```

The attributes you can use in this position include:

1. Conditional compilation: `cfg` and `cfg_attr`

2. Controlling lints: `allow`, `warn`, `deny`, and `forbid`

3. Helper attributes used by procedural macro attributes applied to items.

   Our hope is that this will be used to provide more readable and ergonomic macro-based DSLs throughout the ecosystem.

### Borrow check migration warnings are hard errors in Rust 2018

[rel-1360-nll]: https://blog.rust-lang.org/2019/07/04/Rust-1.36.0.html#nll-for-rust-2015
[rel-1310]: https://blog.rust-lang.org/2018/12/06/Rust-1.31-and-rust-2018.html#non-lexical-lifetimes
[err-2018]: https://github.com/rust-lang/rust/pull/63565
[err-2015]: https://github.com/rust-lang/rust/pull/64221
[rip-ast-borrowck]: https://github.com/rust-lang/rust/pull/64790
[niko-blog-nll]: https://blog.rust-lang.org/2019/11/01/nll-hard-errors.html

In the 1.36.0 release, [we announced][rel-1360-nll] that NLL had come to Rust 2015 after first being released for Rust 2018 in [1.31][rel-1310].

As noted in the 1.36.0 release, the old borrow checker had some bugs which would allow memory unsafety. These bugs were fixed by the NLL borrow checker. As these fixes broke some stable code, we decided to gradually phase in the errors by checking if the old borrow checker would accept the program and the NLL checker would reject it. If so, the errors would instead become warnings.

With Rust 1.39.0, these warnings are now [errors in Rust 2018][err-2018].
In the next release, Rust 1.40.0, [this will also apply to Rust 2015][err-2015], which will finally allow us to [remove the old borrow checker][rip-ast-borrowck], and keep the compiler clean.

If you are affected, or want to hear more, read [Niko Matsakis's blog post][niko-blog-nll].

### More `const fn`s in the standard library

[`Vec::new`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.new
[`String::new`]: https://doc.rust-lang.org/std/string/struct.String.html#method.new
[`LinkedList::new`]: https://doc.rust-lang.org/std/collections/linked_list/struct.LinkedList.html#method.new
[`str::len`]: https://doc.rust-lang.org/std/primitive.str.html#method.len
[`slice::len`]: https://doc.rust-lang.org/std/primitive.slice.html#method.len
[`str::as_bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`abs`]: https://doc.rust-lang.org/std/primitive.i8.html#method.abs
[`wrapping_abs`]: https://doc.rust-lang.org/std/primitive.i8.html#method.wrapping_abs
[`overflowing_abs`]: https://doc.rust-lang.org/std/primitive.i8.html#method.overflowing_abs

With Rust 1.39.0, the following functions became `const fn`:

- [`Vec::new`], [`String::new`], and [`LinkedList::new`]
- [`str::len`], [`[T]::len`][`slice::len`], and [`str::as_bytes`]
- [`abs`], [`wrapping_abs`], and [`overflowing_abs`]

### Additions to the standard library 

[`Pin::into_inner`]: https://doc.rust-lang.org/std/pin/struct.Pin.html#method.into_inner
[`Instant::checked_duration_since`]: https://doc.rust-lang.org/std/time/struct.Instant.html#method.checked_duration_since
[`Instant::saturating_duration_since`]: https://doc.rust-lang.org/std/time/struct.Instant.html#method.saturating_duration_since

In Rust 1.39.0 the following functions were stabilized:

- [`Pin::into_inner`]
- [`Instant::checked_duration_since`] and [`Instant::saturating_duration_since`]

### Other changes

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-139-2019-11-07
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-139
[compat-notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#compatibility-notes

There are other changes in the Rust 1.39.0 release: check out what changed in [Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

Please also see the [compatibility notes][compat-notes] to check if you're affected by those changes.

## Contributors to 1.39.0

Many people came together to create Rust 1.39.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.39.0/)
