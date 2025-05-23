+++
path = "2023/08/24/Rust-1.72.0"
title = "Announcing Rust 1.72.0"
authors = ["The Rust Release Team"]
aliases = [
    "2023/08/24/Rust-1.72.0.html",
    "releases/1.72.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.72.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.72.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.72.0](https://github.com/rust-lang/rust/releases/tag/1.72.0) on GitHub.

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.72.0 stable

### Rust reports potentially useful `cfg`-disabled items in errors

You can conditionally enable Rust code using `cfg`, such as to provide certain
functions only with certain crate features, or only on particular platforms.
Previously, items disabled in this way would be effectively invisible to the
compiler. Now, though, the compiler will remember the name and `cfg` conditions
of those items, so it can report (for example) if a function you tried to call
is unavailable because you need to enable a crate feature.

```
   Compiling my-project v0.1.0 (/tmp/my-project)
error[E0432]: unresolved import `rustix::io_uring`
   --> src/main.rs:1:5
    |
1   | use rustix::io_uring;
    |     ^^^^^^^^^^^^^^^^ no `io_uring` in the root
    |
note: found an item that was configured out
   --> /home/username/.cargo/registry/src/index.crates.io-6f17d22bba15001f/rustix-0.38.8/src/lib.rs:213:9
    |
213 | pub mod io_uring;
    |         ^^^^^^^^
    = note: the item is gated behind the `io_uring` feature

For more information about this error, try `rustc --explain E0432`.
error: could not compile `my-project` (bin "my-project") due to previous error
```

### Const evaluation time is now unlimited

To prevent user-provided const evaluation from getting into a compile-time
infinite loop or otherwise taking unbounded time at compile time, Rust
previously limited the maximum number of *statements* run as part of any given
constant evaluation. However, especially creative Rust code could hit these
limits and produce a compiler error. Worse, whether code hit the limit could
vary wildly based on libraries invoked by the user; if a library you invoked
split a statement into two within one of its functions, your code could then
fail to compile.

Now, you can do an unlimited amount of const evaluation at compile time. To
avoid having long compilations without feedback, the compiler will always emit
a message after your compile-time code has been running for a while, and repeat
that message after a period that doubles each time. By default, the compiler
will also emit a deny-by-default lint (`const_eval_long_running`) after a large
number of steps to catch infinite loops, but you can
`allow(const_eval_long_running)` to permit especially long const evaluation.

### Uplifted lints from Clippy

Several lints from Clippy have been pulled into `rustc`:

* [`clippy::undropped_manually_drops`](https://rust-lang.github.io/rust-clippy/rust-1.71.0/index.html#undropped_manually_drops) to [`undropped_manually_drops`](https://doc.rust-lang.org/1.72.0/rustc/lints/listing/deny-by-default.html#undropped-manually-drops) (deny)
  - `ManuallyDrop` does not drop its inner value, so calling `std::mem::drop` on it does nothing. Instead, the lint will suggest `ManuallyDrop::into_inner` first, or you may use the unsafe `ManuallyDrop::drop` to run the destructor in-place. This lint is denied by default.

* [`clippy::invalid_utf8_in_unchecked`](https://rust-lang.github.io/rust-clippy/rust-1.71.0/index.html#invalid_utf8_in_unchecked) to [`invalid_from_utf8_unchecked`](https://doc.rust-lang.org/1.72.0/rustc/lints/listing/deny-by-default.html#invalid-from-utf8-unchecked) (deny) and [`invalid_from_utf8`](https://doc.rust-lang.org/1.72.0/rustc/lints/listing/warn-by-default.html#invalid-from-utf8) (warn)
  - The first checks for calls to `std::str::from_utf8_unchecked` and `std::str::from_utf8_unchecked_mut` with an invalid UTF-8 literal, which violates their safety pre-conditions, resulting in undefined behavior. This lint is denied by default.
  - The second checks for calls to `std::str::from_utf8` and `std::str::from_utf8_mut` with an invalid UTF-8 literal, which will always return an error. This lint is a warning by default.

* [`clippy::cmp_nan`](https://rust-lang.github.io/rust-clippy/rust-1.71.0/index.html#cmp_nan) to [`invalid_nan_comparisons`](https://doc.rust-lang.org/1.72.0/rustc/lints/listing/warn-by-default.html#invalid-nan-comparisons) (warn)
  - This checks for comparisons with `f32::NAN` or `f64::NAN` as one of the operands. NaN does not compare meaningfully to anything – not even itself – so those comparisons are always false. This lint is a warning by default, and will suggest calling the `is_nan()` method instead.

* [`clippy::cast_ref_to_mut`](https://rust-lang.github.io/rust-clippy/rust-1.71.0/index.html#cast_ref_to_mut) to [`invalid_reference_casting`](https://doc.rust-lang.org/1.72.0/rustc/lints/listing/allowed-by-default.html#invalid-reference-casting) (allow)
  - This checks for casts of `&T` to `&mut T` without using interior mutability, which is immediate undefined behavior, even if the reference is unused. This lint is currently allowed by default due to potential false positives, but it is planned to be denied by default in 1.73 after implementation improvements.

### Stabilized APIs

- [`impl<T: Send> Sync for mpsc::Sender<T>`](https://doc.rust-lang.org/stable/std/sync/mpsc/struct.Sender.html#impl-Sync-for-Sender%3CT%3E)
- [`impl TryFrom<&OsStr> for &str`](https://doc.rust-lang.org/stable/std/primitive.str.html#impl-TryFrom%3C%26'a+OsStr%3E-for-%26'a+str)
- [`String::leak`](https://doc.rust-lang.org/stable/alloc/string/struct.String.html#method.leak)

These APIs are now stable in const contexts:

- [`CStr::from_bytes_with_nul`](https://doc.rust-lang.org/stable/std/ffi/struct.CStr.html#method.from_bytes_with_nul)
- [`CStr::to_bytes`](https://doc.rust-lang.org/stable/std/ffi/struct.CStr.html#method.to_bytes)
- [`CStr::to_bytes_with_nul`](https://doc.rust-lang.org/stable/std/ffi/struct.CStr.html#method.to_bytes_with_nul)
- [`CStr::to_str`](https://doc.rust-lang.org/stable/std/ffi/struct.CStr.html#method.to_str)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.72.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-172-2023-08-24), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-172).

### Future Windows compatibility

In a future release we're planning to increase the minimum supported Windows version to 10. The accepted proposal in compiler [MCP 651](https://github.com/rust-lang/compiler-team/issues/651) is that Rust 1.75 will be the last to officially support Windows 7, 8, and 8.1. When Rust 1.76 is released in February 2024, only Windows 10 and later will be supported as tier-1 targets. This change will apply both as a host compiler and as a compilation target.

**Update**: The planned increase to Windows' minimum support level has been delayed until Rust 1.78, due to be released in May 2024.

## Contributors to 1.72.0

Many people came together to create Rust 1.72.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.72.0/)
