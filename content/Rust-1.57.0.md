+++
layout = "post"
date = 2021-12-02
title = "Announcing Rust 1.57.0"
author = "The Rust Release Team"
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.57.0.
Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.57.0 is as easy as:

```console
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.57.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1570-2021-12-02

## What's in 1.57.0 stable

Rust 1.57 brings `panic!` to const contexts, adds support for custom profiles to Cargo, and stabilizes fallible reservation APIs.

### `panic!` in const contexts

With previous versions of Rust, the `panic!` macro was not usable in `const fn` and other compile-time contexts. Now, this has been stabilized. Together with the stabilization of `panic!`, several other standard library APIs are now usable in const, such as `assert!`.

This stabilization does not yet include the full formatting infrastructure, so the `panic!` macro must be called with either a static string (`panic!("...")`), or with a single `&str` interpolated value (`panic!("{}", a)`) which must be used with `{}` (no format specifiers or other traits).

It is expected that in the future this support will expand, but this minimal stabilization already enables straightforward compile-time assertions, for example to verify the size of a type:

```rust
const _: () = assert!(std::mem::size_of::<u64>() == 8);
const _: () = assert!(std::mem::size_of::<u8>() == 1);
```

### Cargo support for custom profiles

Cargo has long supported four profiles: `dev`, `release`, `test`, and `bench`. With Rust 1.57, support has been added for arbitrarily named profiles.

For example, if you want to enable link time optimizations ([LTO]) only when making the final production build, adding the following snippet to Cargo.toml enables the `lto` flag when this profile is selected, but avoids enabling it for regular release builds.

```toml
[profile.production]
inherits = "release"
lto = true
```

Note that custom profiles must specify a profile from which they inherit default settings. Once the profile has been defined, Cargo commands which build code can be asked to use it with `--profile production`. Currently, this will build artifacts in a separate directory (`target/production` in this case), which means that artifacts are not shared between directories.

[LTO]: https://doc.rust-lang.org/nightly/cargo/reference/profiles.html#lto

### Fallible allocation

Rust 1.57 stabilizes `try_reserve` for `Vec`, `String`, `HashMap`, `HashSet`, and `VecDeque`. This API enables callers to fallibly allocate the backing storage for these types.

Rust will usually abort the process if the global allocator fails, which is not always desirable. This API provides a method for avoiding that abort when working with the standard library collections. However, Rust does not guarantee that the returned memory is actually allocated by the kernel: for example, if overcommit is enabled on Linux, the memory may not be available when its use is attempted.

### Stabilized APIs

The following methods and trait implementations were stabilized.

- [`[T; N]::as_mut_slice`][`array::as_mut_slice`]
- [`[T; N]::as_slice`][`array::as_slice`]
- [`collections::TryReserveError`]
- [`HashMap::try_reserve`]
- [`HashSet::try_reserve`]
- [`String::try_reserve`]
- [`String::try_reserve_exact`]
- [`Vec::try_reserve`]
- [`Vec::try_reserve_exact`]
- [`VecDeque::try_reserve`]
- [`VecDeque::try_reserve_exact`]
- [`Iterator::map_while`]
- [`iter::MapWhile`]
- [`proc_macro::is_available`]
- [`Command::get_program`]
- [`Command::get_args`]
- [`Command::get_envs`]
- [`Command::get_current_dir`]
- [`CommandArgs`]
- [`CommandEnvs`]

The following previously stable functions are now `const`.

- [`hint::unreachable_unchecked`]

### Other changes

There are other changes in the Rust 1.57.0 release: check out what changed in
[Rust](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1570-2021-12-02),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-157-2021-12-02),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-157).

### Contributors to 1.57.0

Many people came together to create Rust 1.57.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.57.0/)

[`array::as_mut_slice`]: https://doc.rust-lang.org/std/primitive.array.html#method.as_mut_slice
[`array::as_slice`]: https://doc.rust-lang.org/std/primitive.array.html#method.as_slice
[`collections::TryReserveError`]: https://doc.rust-lang.org/std/collections/struct.TryReserveError.html
[`HashMap::try_reserve`]: https://doc.rust-lang.org/std/collections/hash_map/struct.HashMap.html#method.try_reserve
[`HashSet::try_reserve`]: https://doc.rust-lang.org/std/collections/hash_set/struct.HashSet.html#method.try_reserve
[`String::try_reserve`]: https://doc.rust-lang.org/alloc/string/struct.String.html#method.try_reserve
[`String::try_reserve_exact`]: https://doc.rust-lang.org/alloc/string/struct.String.html#method.try_reserve_exact
[`Vec::try_reserve`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.try_reserve
[`Vec::try_reserve_exact`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.try_reserve_exact
[`VecDeque::try_reserve`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.try_reserve
[`VecDeque::try_reserve_exact`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.try_reserve_exact
[`Iterator::map_while`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.map_while
[`iter::MapWhile`]: https://doc.rust-lang.org/std/iter/struct.MapWhile.html
[`proc_macro::is_available`]: https://doc.rust-lang.org/proc_macro/fn.is_available.html
[`Command::get_program`]: https://doc.rust-lang.org/std/process/struct.Command.html#method.get_program
[`Command::get_args`]: https://doc.rust-lang.org/std/process/struct.Command.html#method.get_args
[`Command::get_envs`]: https://doc.rust-lang.org/std/process/struct.Command.html#method.get_envs
[`Command::get_current_dir`]: https://doc.rust-lang.org/std/process/struct.Command.html#method.get_current_dir
[`CommandArgs`]: https://doc.rust-lang.org/std/process/struct.CommandArgs.html
[`CommandEnvs`]: https://doc.rust-lang.org/std/process/struct.CommandEnvs.html
[`hint::unreachable_unchecked`]: https://doc.rust-lang.org/std/hint/fn.unreachable_unchecked.html
