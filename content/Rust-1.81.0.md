+++
path = "2024/09/05/Rust-1.81.0"
title = "Announcing Rust 1.81.0"
authors = ["The Rust Release Team"]
aliases = [
    "2024/09/05/Rust-1.81.0.html",
    "releases/1.81.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.81.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via `rustup`, you can get 1.81.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.81.0](https://doc.rust-lang.org/nightly/releases.html#version-1810-2024-09-05).

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.81.0 stable

### `core::error::Error`

1.81 stabilizes the `Error` trait in `core`, allowing usage of the trait in
`#![no_std]` libraries. This primarily enables the wider Rust ecosystem to
standardize on the same Error trait, regardless of what environments the
library targets.

### New sort implementations

Both the stable and unstable sort implementations in the standard library have
been updated to new algorithms, improving their runtime performance and
compilation time.

Additionally, both of the new sort algorithms try to detect incorrect
implementations of `Ord` that prevent them from being able to produce a
meaningfully sorted result, and will now panic on such cases rather than
returning effectively randomly arranged data.  Users encountering these panics
should audit their ordering implementations to ensure they satisfy the
requirements documented in [PartialOrd] and [Ord].

[PartialOrd]: https://doc.rust-lang.org/nightly/std/cmp/trait.PartialOrd.html
[Ord]: https://doc.rust-lang.org/nightly/std/cmp/trait.Ord.html

### `#[expect(lint)]`

1.81 stabilizes a new lint level, `expect`, which allows explicitly noting that
a particular lint *should* occur, and warning if it doesn't.  The intended use
case for this is temporarily silencing a lint, whether due to lint
implementation bugs or ongoing refactoring, while wanting to know when the lint
is no longer required.

For example, if you're moving a code base to comply with a new restriction
enforced via a Clippy lint like
[`undocumented_unsafe_blocks`](https://rust-lang.github.io/rust-clippy/stable/index.html#/undocumented_unsafe_blocks),
you can use `#[expect(clippy::undocumented_unsafe_blocks)]` as you transition,
ensuring that once all unsafe blocks are documented you can opt into denying
the lint to enforce it.

Clippy also has two lints to enforce the usage of this feature and help with
migrating existing attributes:

* [`clippy::allow_attributes`](https://rust-lang.github.io/rust-clippy/master/index.html#/allow_attributes) to restrict allow attributes in favor of `#[expect]` or to migrate `#[allow]` attributes to `#[expect]`
* [`clippy::allow_attributes_without_reason`](https://rust-lang.github.io/rust-clippy/master/index.html#/allow_attributes_without_reason) To require a reason for `#[allow]` attributes

### Lint reasons

Changing the lint level is often done for some particular reason. For example,
if code runs in an environment without floating point support, you could use
Clippy to lint on such usage with `#![deny(clippy::float_arithmetic)]`.
However, if a new developer to the project sees this lint fire, they need to
look for (hopefully) a comment on the deny explaining why it was added. With
Rust 1.81, they can be informed directly in the compiler message:

```
error: floating-point arithmetic detected
 --> src/lib.rs:4:5
  |
4 |     a + b
  |     ^^^^^
  |
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#float_arithmetic
  = note: no hardware float support
note: the lint level is defined here
 --> src/lib.rs:1:9
  |
1 | #![deny(clippy::float_arithmetic, reason = "no hardware float support")]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^
```

### Stabilized APIs

- [`core::error`](https://doc.rust-lang.org/stable/core/error/index.html)
- [`hint::assert_unchecked`](https://doc.rust-lang.org/stable/core/hint/fn.assert_unchecked.html)
- [`fs::exists`](https://doc.rust-lang.org/stable/std/fs/fn.exists.html)
- [`AtomicBool::fetch_not`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicBool.html#method.fetch_not)
- [`Duration::abs_diff`](https://doc.rust-lang.org/stable/core/time/struct.Duration.html#method.abs_diff)
- [`IoSlice::advance`](https://doc.rust-lang.org/stable/std/io/struct.IoSlice.html#method.advance)
- [`IoSlice::advance_slices`](https://doc.rust-lang.org/stable/std/io/struct.IoSlice.html#method.advance_slices)
- [`IoSliceMut::advance`](https://doc.rust-lang.org/stable/std/io/struct.IoSliceMut.html#method.advance)
- [`IoSliceMut::advance_slices`](https://doc.rust-lang.org/stable/std/io/struct.IoSliceMut.html#method.advance_slices)
- [`PanicHookInfo`](https://doc.rust-lang.org/stable/std/panic/struct.PanicHookInfo.html)
- [`PanicInfo::message`](https://doc.rust-lang.org/stable/core/panic/struct.PanicInfo.html#method.message)
- [`PanicMessage`](https://doc.rust-lang.org/stable/core/panic/struct.PanicMessage.html)

These APIs are now stable in const contexts:

- [`char::from_u32_unchecked`](https://doc.rust-lang.org/stable/core/char/fn.from_u32_unchecked.html) (function)
- [`char::from_u32_unchecked`](https://doc.rust-lang.org/stable/core/primitive.char.html#method.from_u32_unchecked) (method)
- [`CStr::count_bytes`](https://doc.rust-lang.org/stable/core/ffi/c_str/struct.CStr.html#method.count_bytes)
- [`CStr::from_ptr`](https://doc.rust-lang.org/stable/core/ffi/c_str/struct.CStr.html#method.from_ptr)

### Compatibility notes

#### Split panic hook and panic handler arguments

We have renamed [`std::panic::PanicInfo`] to [`std::panic::PanicHookInfo`]. The old
name will continue to work as an alias, but will result in a deprecation
warning starting in Rust 1.82.0.

`core::panic::PanicInfo` will remain unchanged, however, as this is now a
*different type*.

 The reason is that these types have different roles:
`std::panic::PanicHookInfo` is the argument to the [panic hook](https://doc.rust-lang.org/stable/std/panic/fn.set_hook.html) in std
context (where panics can have an arbitrary payload), while
`core::panic::PanicInfo` is the argument to the
[`#[panic_handler]`](https://doc.rust-lang.org/nomicon/panic-handler.html) in
`#![no_std]` context (where panics always carry a formatted *message*). Separating
these types allows us to add more useful methods to these types, such as
[`std::panic::PanicHookInfo::payload_as_str()`](https://doc.rust-lang.org/stable/std/panic/struct.PanicHookInfo.html#method.payload_as_str) and
[`core::panic::PanicInfo::message()`](https://doc.rust-lang.org/stable/core/panic/struct.PanicInfo.html#method.message).

[`std::panic::PanicInfo`]: https://doc.rust-lang.org/stable/std/panic/type.PanicInfo.html
[`std::panic::PanicHookInfo`]: https://doc.rust-lang.org/stable/std/panic/struct.PanicHookInfo.html

#### Abort on uncaught panics in `extern "C"` functions

This completes the transition started in [1.71](https://blog.rust-lang.org/2023/07/13/Rust-1.71.0.html#c-unwind-abi),
which added dedicated `"C-unwind"` (amongst other `-unwind` variants) ABIs for
when unwinding across the ABI boundary is expected. As of 1.81, the non-unwind
ABIs (e.g., `"C"`) will now abort on uncaught unwinds, closing the longstanding
soundness problem.

Programs relying on unwinding should transition to using `-unwind` suffixed ABI
variants.

#### WASI 0.1 target naming changed

Usage of the `wasm32-wasi` target (which targets WASI 0.1) will now issue a
compiler warning and request users switch to the `wasm32-wasip1` target
instead. Both targets are the same, `wasm32-wasi` is only being renamed, and
this [change to the WASI target](https://blog.rust-lang.org/2024/04/09/updates-to-rusts-wasi-targets.html)
is being done to enable removing `wasm32-wasi` in January 2025.

#### Fixes CVE-2024-43402

`std::process::Command` now correctly escapes arguments when invoking batch
files on Windows in the presence of trailing whitespace or periods (which are
ignored and stripped by Windows).

See more details in the previous [announcement of this change](https://blog.rust-lang.org/2024/09/04/cve-2024-43402.html).

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.81.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-181-2024-09-05), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-181).

## Contributors to 1.81.0

Many people came together to create Rust 1.81.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.81.0/)
