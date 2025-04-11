+++
path = "2021/10/21/Rust-1.56.0"
title = "Announcing Rust 1.56.0 and Rust 2021"
authors = ["The Rust Release Team"]
aliases = ["2021/10/21/Rust-1.56.0.html"]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.56.0. This stabilizes the 2021 edition as well.
Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.56.0 is as easy as:

```console
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.56.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1560-2021-10-21

## What's in 1.56.0 stable

### Rust 2021

We wrote about plans for the Rust 2021 Edition [in May](https://blog.rust-lang.org/2021/05/11/edition-2021.html).
Editions are a mechanism for opt-in changes that may otherwise pose backwards compatibility risk. See [the edition guide](https://doc.rust-lang.org/stable/edition-guide/editions/index.html) for details on how this is achieved.
This is a smaller edition, especially compared to 2018, but there
are still some nice quality-of-life changes that require an edition opt-in to
avoid breaking some corner cases in existing code. See the new chapters of the
edition guide below for more details on each new feature and guidance for
migration.

* [Disjoint capture](https://doc.rust-lang.org/edition-guide/rust-2021/disjoint-capture-in-closures.html): closures now capture individual named fields rather than always capturing whole identifiers.
* [`IntoIterator` for arrays](https://doc.rust-lang.org/edition-guide/rust-2021/IntoIterator-for-arrays.html): `array.into_iter()` now iterates over items by value instead of by reference.
* [Or patterns in macro-rules](https://doc.rust-lang.org/edition-guide/rust-2021/or-patterns-macro-rules.html) now match top-level `A|B` in `:pat`.
* [Default Cargo feature resolver](https://doc.rust-lang.org/edition-guide/rust-2021/default-cargo-resolver.html) is now version 2.
* [Additions to the prelude](https://doc.rust-lang.org/edition-guide/rust-2021/prelude.html): `TryInto`, `TryFrom`, and `FromIterator` are now in scope by default.
* [Panic macros](https://doc.rust-lang.org/edition-guide/rust-2021/panic-macro-consistency.html) now always expect format strings, just like `println!()`.
* [Reserving syntax](https://doc.rust-lang.org/edition-guide/rust-2021/reserving-syntax.html) for `ident#`, `ident"..."`, and `ident'...'`.
* [Warnings promoted to errors](https://doc.rust-lang.org/edition-guide/rust-2021/warnings-promoted-to-error.html): `bare_trait_objects` and `ellipsis_inclusive_range_patterns`.

#### Disjoint capture in closures

Closures automatically capture values or references to identifiers that are
used in the body, but before 2021, they were always captured as a whole. The new
disjoint-capture feature will likely simplify the way you write closures, so
let's look at a quick example:

```rust
// 2015 or 2018 edition code
let a = SomeStruct::new();

// Move out of one field of the struct
drop(a.x);

// Ok: Still use another field of the struct
println!("{}", a.y);

// Error: Before 2021 edition, tries to capture all of `a`
let c = || println!("{}", a.y);
c();
```

To fix this, you would have had to extract something like `let y = &a.y;`
manually before the closure to limit its capture. Starting in Rust 2021,
closures will automatically capture only the fields that they use, so the
above example will compile fine!

This new behavior is only activated in the new edition, since it can change
the order in which fields are dropped. As for all edition changes, an
automatic migration is available, which will update your closures for which
this matters by inserting `let _ = &a;` inside the closure to force the
entire struct to be captured as before.

#### Migrating to 2021

The guide includes migration instructions for all new features, and in general
[transitioning an existing project to a new edition](https://doc.rust-lang.org/edition-guide/editions/transitioning-an-existing-project-to-a-new-edition.html).
In many cases `cargo fix` can automate the necessary changes. You may even
find that no changes in your code are needed at all for 2021!

However small this edition appears on the surface, it's still the product
of a lot of hard work from many contributors: see our dedicated
[celebration and thanks](https://github.com/rust-lang/rust/issues/88623) tracker!

### Cargo `rust-version`

`Cargo.toml` now supports a `[package]` [`rust-version`] field to specify
the minimum supported Rust version for a crate, and Cargo will exit with an
early error if that is not satisfied. This doesn't currently influence the
dependency resolver, but the idea is to catch compatibility problems before
they turn into cryptic compiler errors.

[`rust-version`]: https://doc.rust-lang.org/cargo/reference/manifest.html#the-rust-version-field

### New bindings in `binding @ pattern`

Rust pattern matching can be written with a single identifier that binds
the entire value, followed by `@` and a more refined structural pattern,
but this has not allowed additional bindings in that pattern -- until now!

```rust
struct Matrix {
    data: Vec<f64>,
    row_len: usize,
}

// Before, we need separate statements to bind
// the whole struct and also read its parts.
let matrix = get_matrix();
let row_len = matrix.row_len;
// or with a destructuring pattern:
let Matrix { row_len, .. } = matrix;

// Rust 1.56 now lets you bind both at once!
let matrix @ Matrix { row_len, .. } = get_matrix();
```

This actually was allowed in the days before Rust 1.0, but that was removed
due to known [unsoundness](https://github.com/rust-lang/rust/pull/16053) at
the time. With the evolution of the borrow checker since that time, and with
heavy testing, the compiler team determined that this was safe to finally
allow in stable Rust!

### Stabilized APIs

The following methods and trait implementations were stabilized.

- [`std::os::unix::fs::chroot`]
- [`UnsafeCell::raw_get`]
- [`BufWriter::into_parts`]
- [`core::panic::{UnwindSafe, RefUnwindSafe, AssertUnwindSafe}`]\
  \(previously only in `std`)
- [`Vec::shrink_to`]
- [`String::shrink_to`]
- [`OsString::shrink_to`]
- [`PathBuf::shrink_to`]
- [`BinaryHeap::shrink_to`]
- [`VecDeque::shrink_to`]
- [`HashMap::shrink_to`]
- [`HashSet::shrink_to`]

The following previously stable functions are now `const`.

- [`std::mem::transmute`]
- [`[T]::first`][`slice::first`]
- [`[T]::split_first`][`slice::split_first`]
- [`[T]::last`][`slice::last`]
- [`[T]::split_last`][`slice::split_last`]

[`std::os::unix::fs::chroot`]: https://doc.rust-lang.org/stable/std/os/unix/fs/fn.chroot.html
[`UnsafeCell::raw_get`]: https://doc.rust-lang.org/stable/std/cell/struct.UnsafeCell.html#method.raw_get
[`BufWriter::into_parts`]: https://doc.rust-lang.org/stable/std/io/struct.BufWriter.html#method.into_parts
[`core::panic::{UnwindSafe, RefUnwindSafe, AssertUnwindSafe}`]: https://github.com/rust-lang/rust/pull/84662
[`Vec::shrink_to`]: https://doc.rust-lang.org/stable/std/vec/struct.Vec.html#method.shrink_to
[`String::shrink_to`]: https://doc.rust-lang.org/stable/std/string/struct.String.html#method.shrink_to
[`OsString::shrink_to`]: https://doc.rust-lang.org/stable/std/ffi/struct.OsString.html#method.shrink_to
[`PathBuf::shrink_to`]: https://doc.rust-lang.org/stable/std/path/struct.PathBuf.html#method.shrink_to
[`BinaryHeap::shrink_to`]: https://doc.rust-lang.org/stable/std/collections/struct.BinaryHeap.html#method.shrink_to
[`VecDeque::shrink_to`]: https://doc.rust-lang.org/stable/std/collections/struct.VecDeque.html#method.shrink_to
[`HashMap::shrink_to`]: https://doc.rust-lang.org/stable/std/collections/hash_map/struct.HashMap.html#method.shrink_to
[`HashSet::shrink_to`]: https://doc.rust-lang.org/stable/std/collections/hash_set/struct.HashSet.html#method.shrink_to
[`std::mem::transmute`]: https://doc.rust-lang.org/stable/std/mem/fn.transmute.html
[`slice::first`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.first
[`slice::split_first`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.split_first
[`slice::last`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.last
[`slice::split_last`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.split_last

### Other changes

There are other changes in the Rust 1.56.0 release: check out what changed in
[Rust](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1560-2021-10-21),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-156-2021-10-21),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-156).

### Contributors to 1.56.0

Many people came together to create Rust 1.56.0 and the 2021 edition.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.56.0/)
