+++
path = "2025/01/09/Rust-1.84.0"
title = "Announcing Rust 1.84.0"
authors = ["The Rust Release Team"]
aliases = [
    "2025/01/09/Rust-1.84.0.html",
    "releases/1.84.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.84.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via `rustup`, you can get 1.84.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.84.0](https://doc.rust-lang.org/stable/releases.html#version-1840-2025-01-09).

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.84.0 stable

### Cargo considers Rust versions for dependency version selection 

1.84.0 stabilizes the minimum supported Rust version (MSRV) aware resolver,
which prefers dependency versions compatible with the project's declared
[MSRV](https://doc.rust-lang.org/cargo/reference/rust-version.html).
With MSRV-aware version selection, the toil is reduced for maintainers to
support older toolchains by not needing to manually select older versions for
each dependency.

You can opt-in to the MSRV-aware resolver via [`.cargo/config.toml`](https://doc.rust-lang.org/cargo/reference/config.html#resolverincompatible-rust-versions):

```toml
[resolver]
incompatible-rust-versions = "fallback"
```

Then when adding a dependency:

```
$ cargo add clap
    Updating crates.io index
warning: ignoring clap@4.5.23 (which requires rustc 1.74) to maintain demo's rust-version of 1.60
      Adding clap v4.0.32 to dependencies
    Updating crates.io index
     Locking 33 packages to latest Rust 1.60 compatible versions
      Adding clap v4.0.32 (available: v4.5.23, requires Rust 1.74)
```

When [verifying the latest dependencies in CI](https://doc.rust-lang.org/cargo/guide/continuous-integration.html#verifying-latest-dependencies), you can override this:

```
$ CARGO_RESOLVER_INCOMPATIBLE_RUST_VERSIONS=allow cargo update
    Updating crates.io index
     Locking 12 packages to latest compatible versions
    Updating clap v4.0.32 -> v4.5.23
```

You can also opt-in by setting [`package.resolver = "3"`](https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions) in the Cargo.toml manifest file though that will require raising your MSRV to 1.84.  The new resolver will be enabled by default for projects using the 2024 edition
(which will stabilize in 1.85).

This gives library authors more flexibility when deciding
their policy on adopting new Rust toolchain features. Previously, a library
adopting features from a new Rust toolchain would force downstream users of
that library who have an older Rust version to either upgrade their toolchain
or manually select an old version of the library compatible with their
toolchain (and avoid running `cargo update`). Now, those users will be able to
automatically use older library versions compatible with their older toolchain.

See the [documentation](https://doc.rust-lang.org/cargo/reference/rust-version.html#setting-and-updating-rust-version) for more considerations when deciding on an MSRV policy.

### Migration to the new trait solver begins

The Rust compiler is in the process of moving to a new implementation for the
trait solver. The next-generation trait solver is a reimplementation of a core
component of Rust's type system. It is not only responsible for checking
whether trait-bounds - e.g. `Vec<T>: Clone` - hold, but is also used by many
other parts of the type system, such as normalization - figuring out the
underlying type of `<Vec<T> as IntoIterator>::Item` - and equating types
(checking whether `T` and `U` are the same).

In 1.84, the new solver is used for checking coherence of trait impls. At a
high level, coherence is responsible for ensuring that there is at most one
implementation of a trait for a given type while considering not yet written
or visible code from other crates.

This stabilization fixes a few mostly theoretical correctness issues of the
old implementation, resulting in potential "conflicting implementations of trait ..."
errors that were not previously reported. We expect the affected patterns to be
very rare based on evaluation of available code through [Crater]. The stabilization
also improves our ability to prove that impls do *not* overlap, allowing more code
to be written in some cases.

For more details, see a [previous blog post](https://blog.rust-lang.org/inside-rust/2024/12/04/trait-system-refactor-initiative.html)
and the [stabilization report](https://github.com/rust-lang/rust/pull/130654).

[Crater]: https://github.com/rust-lang/crater/

### Strict provenance APIs

In Rust, [pointers are not simply an "integer" or
"address"](https://rust-lang.github.io/rfcs/3559-rust-has-provenance.html). For
instance, a "use after free" is undefined behavior even if you "get lucky" and the freed memory gets
reallocated before your read/write. As another example, writing
through a pointer derived from an `&i32` reference is undefined behavior, even
if writing to the same address via a different pointer is legal. The underlying
pattern here is that *the way a pointer is computed matters*, not just the
address that results from this computation. For this reason, we say that
pointers have **provenance**: to fully characterize pointer-related undefined
behavior in Rust, we have to know not only the address the pointer points to,
but also track which other pointer(s) it is "derived from".

Most of the time, programmers do not need to worry much about provenance, and
it is very clear how a pointer got derived. However, when casting pointers to
integers and back, the provenance of the resulting pointer is underspecified.
With this release, Rust is adding a set of APIs that can in many cases replace
the use of integer-pointer-casts, and therefore avoid the ambiguities inherent
to such casts. In particular, the pattern of using the lowest bits of an
aligned pointer to store extra information can now be implemented without ever
casting a pointer to an integer or back. This makes the code easier to reason
about, easier to analyze for the compiler, and also benefits tools like
[Miri](https://github.com/rust-lang/miri) and architectures like
[CHERI](https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/) that aim to
detect and diagnose pointer misuse.

For more details, see the standard library [documentation on provenance](https://doc.rust-lang.org/std/ptr/index.html#provenance).

### Stabilized APIs

- [`Ipv6Addr::is_unique_local`](https://doc.rust-lang.org/stable/core/net/struct.Ipv6Addr.html#method.is_unique_local)
- [`Ipv6Addr::is_unicast_link_local`](https://doc.rust-lang.org/stable/core/net/struct.Ipv6Addr.html#method.is_unicast_link_local)
- [`core::ptr::with_exposed_provenance`](https://doc.rust-lang.org/stable/core/ptr/fn.with_exposed_provenance.html)
- [`core::ptr::with_exposed_provenance_mut`](https://doc.rust-lang.org/stable/core/ptr/fn.with_exposed_provenance_mut.html)
- [`<ptr>::addr`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.addr)
- [`<ptr>::expose_provenance`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.expose_provenance)
- [`<ptr>::with_addr`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.with_addr)
- [`<ptr>::map_addr`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.map_addr)
- [`<int>::isqrt`](https://doc.rust-lang.org/stable/core/primitive.i32.html#method.isqrt)
- [`<int>::checked_isqrt`](https://doc.rust-lang.org/stable/core/primitive.i32.html#method.checked_isqrt)
- [`<uint>::isqrt`](https://doc.rust-lang.org/stable/core/primitive.u32.html#method.isqrt)
- [`NonZero::isqrt`](https://doc.rust-lang.org/stable/core/num/struct.NonZero.html#impl-NonZero%3Cu128%3E/method.isqrt)
- [`core::ptr::without_provenance`](https://doc.rust-lang.org/stable/core/ptr/fn.without_provenance.html)
- [`core::ptr::without_provenance_mut`](https://doc.rust-lang.org/stable/core/ptr/fn.without_provenance_mut.html)
- [`core::ptr::dangling`](https://doc.rust-lang.org/stable/core/ptr/fn.dangling.html)
- [`core::ptr::dangling_mut`](https://doc.rust-lang.org/stable/core/ptr/fn.dangling_mut.html)
- [`Pin::as_deref_mut`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.as_deref_mut)

These APIs are now stable in const contexts

- [`AtomicBool::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicBool.html#method.from_ptr)
- [`AtomicPtr::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicPtr.html#method.from_ptr)
- [`AtomicU8::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicU8.html#method.from_ptr)
- [`AtomicU16::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicU16.html#method.from_ptr)
- [`AtomicU32::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicU32.html#method.from_ptr)
- [`AtomicU64::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicU64.html#method.from_ptr)
- [`AtomicUsize::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicUsize.html#method.from_ptr)
- [`AtomicI8::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicI8.html#method.from_ptr)
- [`AtomicI16::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicI16.html#method.from_ptr)
- [`AtomicI32::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicI32.html#method.from_ptr)
- [`AtomicI64::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicI64.html#method.from_ptr)
- [`AtomicIsize::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicIsize.html#method.from_ptr)
- [`<ptr>::is_null`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.is_null-1)
- [`<ptr>::as_ref`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.as_ref-1)
- [`<ptr>::as_mut`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.as_mut)
- [`Pin::new`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.new)
- [`Pin::new_unchecked`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.new_unchecked)
- [`Pin::get_ref`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.get_ref)
- [`Pin::into_ref`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.into_ref)
- [`Pin::get_mut`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.get_mut)
- [`Pin::get_unchecked_mut`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.get_unchecked_mut)
- [`Pin::static_ref`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.static_ref)
- [`Pin::static_mut`](https://doc.rust-lang.org/stable/core/pin/struct.Pin.html#method.static_mut)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.84.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-184-2025-01-09), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-184).

## Contributors to 1.84.0

Many people came together to create Rust 1.84.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.84.0/)
