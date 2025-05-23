+++
path = "2019/07/04/Rust-1.36.0"
title = "Announcing Rust 1.36.0"
authors = ["The Rust Release Team"]
aliases = [
    "2019/07/04/Rust-1.36.0.html",
    "releases/1.36.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.36.0.
Rust is a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.36.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the appropriate page on our website,
and check out the [detailed release notes for 1.36.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1360-2019-07-04

## What's in 1.36.0 stable

This release brings many changes, including the stabilization of the [`Future`] trait,
the [`alloc`][alloc-crate] crate, the [`MaybeUninit<T>`] type, [NLL for Rust 2015][felix-blog],
a new `HashMap<K, V>` implementation, and [`--offline`] support in Cargo.
Read on for a few highlights, or see the [detailed release notes][notes] for additional information.

### The [`Future`] is here!

[`Future`]: https://doc.rust-lang.org/std/future/trait.Future.html
[pr-future]: https://github.com/rust-lang/rust/pull/59739

In Rust 1.36.0 the long awaited [`Future`] trait has been [stabilized][pr-future]!

With this stabilization, we hope to give important crates, libraries,
and the ecosystem time to prepare for `async` / `.await`,
which we'll tell you more about in the future.

### The [`alloc`][alloc-crate] crate is stable

[alloc-crate]: https://doc.rust-lang.org/alloc/index.html

Before 1.36.0, the standard library consisted of the crates `std`, `core`, and `proc_macro`.
The `core` crate provided core functionality such as `Iterator` and `Copy`
and could be used in `#![no_std]` environments since it did not impose any requirements.
Meanwhile, the `std` crate provided types like `Box<T>` and OS functionality
but required a global allocator and other OS capabilities in return.

Starting with Rust 1.36.0, the parts of `std` that depend on a global allocator, e.g. `Vec<T>`,
are now available in the `alloc` crate. The `std` crate then re-exports these parts.
While `#![no_std]` *binaries* using `alloc` still require nightly Rust,
`#![no_std]` *library* crates can use the `alloc` crate in stable Rust.
Meanwhile, normal binaries, without `#![no_std]`, can depend on such library crates.
We hope this will facilitate the development of a `#![no_std]` compatible ecosystem of libraries
prior to stabilizing support for `#![no_std]` binaries using `alloc`.

If you are the maintainer of a library that only relies on some allocation primitives to function,
consider making your library `#[no_std]` compatible by using the following at the top of your `lib.rs` file:

```rust
#![no_std]

extern crate alloc;

use alloc::vec::Vec;
```

### [`MaybeUninit<T>`] instead of [`mem::uninitialized`]

[`MaybeUninit<T>`]: https://doc.rust-lang.org/std/mem/union.MaybeUninit.html
[`mem::uninitialized`]: https://doc.rust-lang.org/std/mem/fn.uninitialized.html
[gankro-blog]: https://gankro.github.io/blah/initialize-me-maybe/
[pr-60445]: https://github.com/rust-lang/rust/pull/60445

In previous releases of Rust, the [`mem::uninitialized`] function has allowed you to bypass Rust's
initialization checks by pretending that you've initialized a value at type `T` without doing anything.
One of the main uses of this function has been to lazily allocate arrays.

However, [`mem::uninitialized`] is an incredibly dangerous operation that essentially
cannot be used correctly as the Rust compiler assumes that values are properly initialized.
For example, calling `mem::uninitialized::<bool>()` causes *instantaneous __undefined behavior__*
as, from Rust's point of view, the uninitialized bits are neither `0` (for `false`)
nor `1` (for `true`) - the only two allowed bit patterns for `bool`.

To remedy this situation, in Rust 1.36.0, the type [`MaybeUninit<T>`] has been [stabilized][pr-60445].
The Rust compiler will understand that it should not assume that a [`MaybeUninit<T>`] is a properly initialized `T`.
Therefore, you can do gradual initialization more safely and eventually use `.assume_init()`
once you are certain that `maybe_t: MaybeUninit<T>` contains an initialized `T`.

As [`MaybeUninit<T>`] is the safer alternative, starting with Rust 1.39,
the function [`mem::uninitialized`] will be deprecated.

To find out more about uninitialized memory, [`mem::uninitialized`],
and [`MaybeUninit<T>`], read [Alexis Beingessner's blog post][gankro-blog].
The standard library also contains extensive documentation about [`MaybeUninit<T>`].

### NLL for Rust 2015

[nll-2018]: https://blog.rust-lang.org/2018/12/06/Rust-1.31-and-rust-2018.html#non-lexical-lifetimes
[soundness]: https://en.wikipedia.org/wiki/Soundness
[felix-blog]: http://blog.pnkfx.org/blog/2019/06/26/breaking-news-non-lexical-lifetimes-arrives-for-everyone/
[crater-nll]: https://github.com/rust-lang/rust/issues/60680#issuecomment-495089654

[In the announcement for Rust 1.31.0][nll-2018], we told you about NLL (Non-Lexical Lifetimes),
an improvement to the language that makes the borrow checker smarter and more user friendly.
For example, you may now write:

```rust
fn main() {
    let mut x = 5;
    let y = &x;
    let z = &mut x; // This was not allowed before 1.31.0.
}
```

In 1.31.0 NLL was stabilized only for Rust 2018,
with a promise that we would backport it to Rust 2015 as well.
With Rust 1.36.0, we are happy to announce that we have done so! NLL is now available for Rust 2015.

With NLL on both editions, we are closer to removing the old borrow checker.
However, the old borrow checker unfortunately accepted some [unsound][soundness] code it should not have.
As a result, NLL is currently in a "migration mode" wherein we will emit warnings instead
of errors if the NLL borrow checker rejects code the old AST borrow checker would accept.
Please see [this list][crater-nll] of public crates that are affected.

To find out more about NLL, MIR, the story around fixing soundness holes,
and what you can do about the warnings if you have them, read [Felix Klock's blog post][felix-blog].

### A new [`HashMap<K, V>`] implementation

[`hashbrown`]: https://crates.io/crates/hashbrown
[`HashMap<K, V>`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html
[pr-hashbrown]: https://github.com/rust-lang/rust/pull/58623
[SwissTable]: https://abseil.io/blog/20180927-swisstables
[pr-hashbrown-perf]: https://perf.rust-lang.org/compare.html?start=b57fe74a27590289fd657614b8ad1f3eac8a7ad2&end=abade53a649583e40ed07c26ee10652703f09b58&stat=wall-time

In Rust 1.36.0, the `HashMap<K, V>` implementation has been [replaced][pr-hashbrown]
with the one in the [`hashbrown`] crate which is based on the [SwissTable] design.
While the interface is the same, the `HashMap<K, V>` implementation is now
[faster on average][pr-hashbrown-perf] and has lower memory overhead.
Note that unlike the `hashbrown` crate,
the implementation in `std` still defaults to the SipHash 1-3 hashing algorithm.

### [`--offline`] support in Cargo

[`--offline`]: https://doc.rust-lang.org/cargo/commands/cargo-build.html#cargo_build_manifest_options
[`cargo fetch`]: https://doc.rust-lang.org/cargo/commands/cargo-fetch.html
[nrc-blog]: https://www.ncameron.org/blog/cargo-offline/
[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-136-2019-07-04

During most builds, Cargo doesn't interact with the network.
Sometimes, however, Cargo has to.
Such is the case when a dependency is added and the latest compatible version needs to be downloaded.
At times, network access is not an option though, for example on an airplane or in isolated build environments.

In Rust 1.36, a new Cargo flag has been stabilized: [`--offline`].
The flag alters Cargo's dependency resolution algorithm to only use locally cached dependencies.
When the required crates are not available offline, and a network access would be required,
Cargo will exit with an error.
To prepopulate the local cache in preparation for going offline,
use the [`cargo fetch`] command, which downloads all the required dependencies for a project.

To find out more about [`--offline`] and [`cargo fetch`], read [Nick Cameron's blog post][nrc-blog].

For information on other changes to Cargo, see the [detailed release notes][relnotes-cargo].

### Library changes

[`dbg!`]: https://doc.rust-lang.org/std/macro.dbg.html

The [`dbg!`] macro now supports multiple arguments.

Additionally, a number of APIs have been made `const`:

[`Layout::from_size_align_unchecked`]: https://doc.rust-lang.org/core/alloc/struct.Layout.html#method.from_size_align_unchecked
[`mem::needs_drop`]: https://doc.rust-lang.org/std/mem/fn.needs_drop.html
[`NonNull::dangling`]: https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.dangling
[`NonNull::cast`]: https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.cast

- [`Layout::from_size_align_unchecked`]
- [`mem::needs_drop`]
- [`NonNull::dangling`]
- [`NonNull::cast`]

New APIs have become stable, including:

[`Iterator::copied`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.copied
[`VecDeque::rotate_left`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.rotate_left
[`VecDeque::rotate_right`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.rotate_right
[`BorrowMut<str> for String`]: https://github.com/rust-lang/rust/pull/60404
[`str::as_mut_ptr`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_mut_ptr
[`pointer::align_offset`]: https://doc.rust-lang.org/std/primitive.pointer.html#method.align_offset
[`Read::read_vectored`]: https://doc.rust-lang.org/std/io/trait.Read.html#method.read_vectored
[`Write::write_vectored`]: https://doc.rust-lang.org/std/io/trait.Write.html#method.write_vectored
[`task::Waker`]: https://doc.rust-lang.org/std/task/struct.Waker.html
[`task::Poll`]: https://doc.rust-lang.org/std/task/enum.Poll.html

- [`task::Waker`] and [`task::Poll`]
- [`VecDeque::rotate_left`] and [`VecDeque::rotate_right`]
- [`Read::read_vectored`] and [`Write::write_vectored`]
- [`Iterator::copied`]
- [`BorrowMut<str> for String`]
- [`str::as_mut_ptr`]

Other library changes are available in the [detailed release notes][notes].

### Other changes

[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-136

Detailed 1.36.0 release notes are available for [Rust][notes],
[Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.36.0

Many people came together to create Rust 1.36.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.36.0/)
