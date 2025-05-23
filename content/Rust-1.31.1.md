+++
path = "2018/12/20/Rust-1.31.1"
title = "Announcing Rust 1.31.1"
authors = ["The Rust Release Team"]
aliases = [
    "2018/12/20/Rust-1.31.1.html",
    "releases/1.31.1",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.31.1. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.31.1 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.31.1][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1311-2018-12-20

## What's in 1.31.1 stable

This patch release fixes a build failure on `powerpc-unknown-netbsd` by
way of [an update to the `libc`
crate](https://github.com/rust-lang/rust/pull/56562) used by the compiler.

Additionally, the Rust Language Server was updated to fix two critical bugs.
First, [hovering over the type with documentation above single-line
attributes led to 100% CPU
usage:](https://github.com/rust-lang/rls/pull/1170)

```rust
/// Some documentation
#[derive(Debug)] // Multiple, single-line
#[allow(missing_docs)] // attributes
pub struct MyStruct { /* ... */ }
```

[Go to definition was fixed for std types](https://github.com/rust-lang/rls/pull/1171):
Before, using the RLS on `HashMap`, for example, tried to open this file

```
~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/libstd/collections/hash/map.rs
```

and now RLS goes to the correct location (for Rust 1.31, note the extra `src`):

```
~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/src/libstd/collections/hash/map.rs
```
