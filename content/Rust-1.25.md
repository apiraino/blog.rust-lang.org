+++
path = "2018/03/29/Rust-1.25"
title = "Announcing Rust 1.25"
authors = ["The Rust Core Team"]
aliases = [
    "2018/03/29/Rust-1.25.html",
    "releases/1.25.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.25.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.25.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.25.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1250-2018-03-29

## What's in 1.25.0 stable

The last few releases have been relatively minor, but Rust 1.25 contains a
bunch of stuff! The first one is straightforward: we've [upgraded to LLVM 6]
from LLVM 4. This has a number of effects, a major one
being a step closer to AVR support.

A new way to write `use` statements has landed: [nested import groups]. If you've
ever written a set of imports like this:

```rust
use std::fs::File;
use std::io::Read;
use std::path::{Path, PathBuf};
```

You can now write this:

```rust
// on one line
use std::{fs::File, io::Read, path::{Path, PathBuf}};

// with some more breathing room
use std::{
    fs::File,
    io::Read,
    path::{
        Path,
        PathBuf
    }
};
```

This can reduce some repetition, and make things a bit more clear.

There are two big documentation changes in this release: first, [Rust By
Example is now included on doc.rust-lang.org]! We'll be redirecting the old
domain there shortly. We hope this will bring more attention to a great
resource, and you'll get a local copy with your local documentation.

Second, back in Rust 1.23, we talked about the change from Hoedown to
pulldown-cmark. In Rust 1.25, pulldown-cmark is now the default. We have
finally removed the last bit of C from rustdoc, and now properly follow the
CommonMark spec.

Finally, in [RFC 1358], `#[repr(align(x))]` was accepted. In Rust
1.25, [it is now stable]! This attribute lets you set the [alignment]
of your `struct`s:

```rust
struct Number(i32);

assert_eq!(std::mem::align_of::<Number>(), 4);
assert_eq!(std::mem::size_of::<Number>(), 4);

#[repr(align(16))]
struct Align16(i32);

assert_eq!(std::mem::align_of::<Align16>(), 16);
assert_eq!(std::mem::size_of::<Align16>(), 16);
```

If you're working with low-level stuff, control of these kinds of things
can be very important!

[upgraded to LLVM 6]: https://github.com/rust-lang/rust/pull/47828
[nested import groups]: https://github.com/rust-lang/rust/pull/47948
[Rust By Example is now included on doc.rust-lang.org]: https://doc.rust-lang.org/rust-by-example/
[RFC 1358]: https://github.com/rust-lang/rfcs/blob/master/text/1358-repr-align.md
[it is now stable]: https://github.com/rust-lang/rust/pull/47006
[alignment]: https://en.wikipedia.org/wiki/Data_structure_alignment

See the [detailed release notes][notes] for more.

### Library stabilizations

The biggest story in libraries this release is [`std::ptr::NonNull<T>`]. This type
is similar to `*mut T`, but is non-null and covariant. This blog post isn't the right
place to explain variance, but in a nutshell, `NonNull<T>`, well, guarantees that it
won't be null, which means that `Option<NonNull<T>>` has the same size as `*mut T`.
If you're building a data structure with unsafe code, `NonNull<T>` is often the right
type for you!

[`std::ptr::NonNull<T>`]: https://doc.rust-lang.org/std/ptr/struct.NonNull.html

`libcore` has [gained a `time` module](https://doc.rust-lang.org/core/time/),
containing the `Duration` type previously only available in `libstd`.

Additionally, the `from_secs`, and `from_millis` functions associated with
`Duration` were made `const fn`s, allowing them to be used to create a
`Duration` as a constant expression.

See the [detailed release notes][notes] for more.

### Cargo features

Cargo's CLI has one really important change this release: `cargo new` will
[now default](https://github.com/rust-lang/cargo/pull/5029) to generating a
binary, rather than a library. We try to keep Cargo's CLI quite stable, but
this change is important, and is unlikely to cause breakage.

For some background, `cargo new` accepts two flags: `--lib`, for creating libraries,
and `--bin`, for creating binaries, or executables. If you don't pass one of these
flags, in previous versions of Cargo, it would default to `--lib`. We made this
decision because each binary (often) depends on many libraries, and so the library
case is more common. However, this is incorrect; each library is *depended upon* by
many binaries. Furthermore, when getting started, what you often want is a program
you can run and play around with. It's not just new Rustaceans though; even very
long-time community members have said that they find this default surprising.
As such, we're changing it.

Similarly, `cargo new` previously would be a bit opinionated around the names
of packages it would create. Specifically, if your package began with `rust-`
or ended with `-rs`, Cargo would rename it. The intention was that well,
it's a Rust package, this information is redundant. However, people feel
quite strongly about naming, and when they bump into this, they're surprised
and often upset. As such, [we're not going to do that any
more](https://github.com/rust-lang/cargo/pull/5013).

Many users love `cargo doc`, a way to generate local documentation for their
Cargo projects. [It's getting a huge speed
boost](https://github.com/rust-lang/cargo/pull/4976) in this release, as now,
it uses `cargo check`, rather than a full `cargo build`, so some scenarios
will get faster.

Additionally, checkouts of git dependencies [should be a lot
faster](https://github.com/rust-lang/cargo/pull/4919), thanks to the use of
hard links when possible.

See the [detailed release notes][notes] for more.

## Contributors to 1.25.0

Many people came together to create Rust 1.25. We couldn't have done it
without all of you.

[Thanks!](https://thanks.rust-lang.org/rust/1.25.0)
