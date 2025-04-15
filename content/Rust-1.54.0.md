+++
path = "2021/07/29/Rust-1.54.0"
title = "Announcing Rust 1.54.0"
authors = ["The Rust Release Team"]
aliases = [
    "2021/07/29/Rust-1.54.0.html",
    "releases/1.54.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.54.0. Rust is a programming language empowering everyone
to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.54.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.54.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1540-2021-07-29

## What's in 1.54.0 stable

### Attributes can invoke function-like macros

Rust 1.54 supports invoking function-like macros inside attributes. Function-like macros can be either `macro_rules!` based or procedural macros which are invoked like `macro!(...)`. One notable use case for this is including documentation from other files into Rust doc comments. For example, if your project's README represents a good documentation comment, you can use `include_str!` to directly incorporate the contents. Previously, various workarounds allowed similar functionality, but from 1.54 this is much more ergonomic.


```rust
#![doc = include_str!("README.md")]
```

Macros can be nested inside the attribute as well.
For example, the `concat!` macro can be used to construct a doc comment from within a macro that uses `stringify!` to include substitutions:

```rust
macro_rules! make_function {
    ($name:ident, $value:expr) => {
        #[doc = concat!("The `", stringify!($name), "` example.")]
        ///
        /// # Example
        ///
        /// ```
        #[doc = concat!(
            "assert_eq!(", module_path!(), "::", stringify!($name), "(), ",
            stringify!($value), ");")
        ]
        /// ```
        pub fn $name() -> i32 {
            $value
        }
    };
}

make_function! {func_name, 123}
```

Read [here](https://github.com/rust-lang/rust/pull/83366) for more details.

### wasm32 intrinsics stabilized

A number of intrinsics for the wasm32 platform have been stabilized, which gives access to the SIMD instructions in WebAssembly.

Notably, unlike the previously stabilized `x86` and `x86_64` intrinsics, these do not have a safety requirement to only be called when the appropriate target feature is enabled. This is because WebAssembly was written from the start to validate code safely before executing it, so instructions are guaranteed to be decoded correctly (or not at all).

This means that we can expose some of the intrinsics as entirely safe functions, for example [`v128_bitselect`](https://doc.rust-lang.org/beta/core/arch/wasm32/fn.v128_bitselect.html). However, there are still some intrinsics which are unsafe because they use raw pointers, such as [`v128_load`](https://doc.rust-lang.org/beta/core/arch/wasm32/fn.v128_load.html).

### Incremental Compilation is re-enabled by default

Incremental compilation has been re-enabled by default in this release, after it being disabled by default in 1.52.1.

In Rust 1.52, additional validation was added when loading incremental compilation data from the on-disk cache.
This resulted in a number of pre-existing potential soundness issues being uncovered as the validation changed these silent bugs into internal compiler errors (ICEs).
In response, the Compiler Team decided to disable incremental compilation in the 1.52.1 patch, allowing users to avoid encountering the ICEs and the underlying unsoundness, at the expense of longer compile times. [^1]

Since then, we've conducted a [series of retrospectives][retros] and contributors have been hard at work resolving the reported issues, with some fixes landing in 1.53 and the majority landing in this release. [^2]

There are currently still two known issues which can result in an ICE.
Due to the lack of automated crash reporting, we can't be certain of the full extent of impact of the outstanding issues. However, based on the feedback we received from users affected by the 1.52 release, we believe the remaining issues to be rare in practice.

Therefore, incremental compilation has been re-enabled in this release!

[^1]: The [1.52.1 release notes] contain a more detailed description of these events.
[^2]: The tracking issue for the issues is [#84970].

[#84970]: https://github.com/rust-lang/rust/issues/84970
[1.52.1 release notes]: https://blog.rust-lang.org/2021/05/10/Rust-1.52.1.html
[retros]: https://github.com/rust-lang/compiler-team/issues/435

### Stabilized APIs

The following methods and trait implementations were stabilized.

- [`BTreeMap::into_keys`]
- [`BTreeMap::into_values`]
- [`HashMap::into_keys`]
- [`HashMap::into_values`]
- [`arch::wasm32`]
- [`VecDeque::binary_search`]
- [`VecDeque::binary_search_by`]
- [`VecDeque::binary_search_by_key`]
- [`VecDeque::partition_point`]

[`BTreeMap::into_keys`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.into_keys
[`BTreeMap::into_values`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.into_values
[`HashMap::into_keys`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.into_keys
[`HashMap::into_values`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.into_values
[`arch::wasm32`]: https://doc.rust-lang.org/core/arch/wasm32/index.html
[`VecDeque::binary_search`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.binary_search
[`VecDeque::binary_search_by`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.binary_search_by
[`VecDeque::binary_search_by_key`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.binary_search_by_key
[`VecDeque::partition_point`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.partition_point

### Other changes

There are other changes in the Rust 1.54.0 release:
check out what changed in [Rust](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1540-2021-07-29), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-154-2021-07-29), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-154).

rustfmt has also been fixed in the 1.54.0 release to properly format nested
out-of-line modules. This may cause changes in formatting to files that were
being ignored by the 1.53.0 rustfmt. See details [here](https://github.com/rust-lang/rust/pull/86424).

### Contributors to 1.54.0

Many people came together to create Rust 1.54.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.54.0/)
