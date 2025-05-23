+++
path = "2020/10/08/Rust-1.47"
title = "Announcing Rust 1.47.0"
authors = ["The Rust Release Team"]
aliases = [
    "2020/10/08/Rust-1.47.html",
    "releases/1.47.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.47.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.47.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.47.0][notes] on GitHub.

[install]: https://www.rust-lang.org/tools/install
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1470-2020-10-08

## What's in 1.47.0 stable

This release contains no new language features, though it does add one
long-awaited standard library feature. It is mostly quality of life
improvements, library stabilizations and const-ifications, and toolchain
improvements. See the [detailed release notes][notes] to learn about other
changes not covered by this post.

#### Traits on larger arrays

Rust does not currently have a way to be generic over integer values. This
has long caused problems with arrays, because arrays have an integer as part
of their type; `[T; N]` is the type of an array of type `T` of `N` length.
Because there is no way to be generic over `N`, you have to manually implement
traits for arrays for every `N` you want to support. For the standard library,
it was decided to support up to `N` of 32.

We have been working on a feature called "const generics" that would allow
you to be generic over `N`. Fully explaining this feature is out of the scope
of this post, because we are not stabilizing const generics just yet.
However, the core of this feature has been implemented in the compiler, and
it has been decided that the feature is far enough along that we are okay
with [the standard library using it to implement traits on arrays of any
length](https://github.com/rust-lang/rust/pull/74060/). What this means in
practice is that if you try to do something like this on Rust 1.46:

```rust
fn main() {
    let xs = [0; 34];

    println!("{:?}", xs);
}
```

you'd get this error:

```
error[E0277]: arrays only have std trait implementations for lengths 0..=32
 --> src/main.rs:4:22
  |
4 |     println!("{:?}", xs);
  |                      ^^ the trait `std::array::LengthAtMost32` is not implemented for `[{integer}; 34]`
  |
  = note: required because of the requirements on the impl of `std::fmt::Debug` for `[{integer}; 34]`
  = note: required by `std::fmt::Debug::fmt`
  = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)
```

But with Rust 1.47, it will properly print out the array.

This should make arrays significantly more useful to folks, though it will
take until the const generics feature stabilizes for libraries to be able to do
this kind of implementation for their own traits. We do not have a current
estimated date for the stabilization of const generics.

#### Shorter backtraces

Back in Rust 1.18, we [made some changes to the backtraces `rustc` would
print on panic](https://github.com/rust-lang/rust/pull/38165). There are a
number of things in a backtrace that aren't useful the majority of the time.
However, at some point, [these
regressed](https://github.com/rust-lang/rust/issues/47429). In Rust 1.47.0,
the culprit was found, and [this has now been
fixed](https://github.com/rust-lang/rust/pull/75048). Since the regression,
this program:

```rust
fn main() {
    panic!();
}
```

would give you a backtrace that looks like this:

```
thread 'main' panicked at 'explicit panic', src/main.rs:2:5
stack backtrace:
   0: backtrace::backtrace::libunwind::trace
             at /cargo/registry/src/github.com-1ecc6299db9ec823/backtrace-0.3.46/src/backtrace/libunwind.rs:86
   1: backtrace::backtrace::trace_unsynchronized
             at /cargo/registry/src/github.com-1ecc6299db9ec823/backtrace-0.3.46/src/backtrace/mod.rs:66
   2: std::sys_common::backtrace::_print_fmt
             at src/libstd/sys_common/backtrace.rs:78
   3: <std::sys_common::backtrace::_print::DisplayBacktrace as core::fmt::Display>::fmt
             at src/libstd/sys_common/backtrace.rs:59
   4: core::fmt::write
             at src/libcore/fmt/mod.rs:1076
   5: std::io::Write::write_fmt
             at src/libstd/io/mod.rs:1537
   6: std::sys_common::backtrace::_print
             at src/libstd/sys_common/backtrace.rs:62
   7: std::sys_common::backtrace::print
             at src/libstd/sys_common/backtrace.rs:49
   8: std::panicking::default_hook::{{closure}}
             at src/libstd/panicking.rs:198
   9: std::panicking::default_hook
             at src/libstd/panicking.rs:217
  10: std::panicking::rust_panic_with_hook
             at src/libstd/panicking.rs:526
  11: std::panicking::begin_panic
             at /rustc/04488afe34512aa4c33566eb16d8c912a3ae04f9/src/libstd/panicking.rs:456
  12: playground::main
             at src/main.rs:2
  13: std::rt::lang_start::{{closure}}
             at /rustc/04488afe34512aa4c33566eb16d8c912a3ae04f9/src/libstd/rt.rs:67
  14: std::rt::lang_start_internal::{{closure}}
             at src/libstd/rt.rs:52
  15: std::panicking::try::do_call
             at src/libstd/panicking.rs:348
  16: std::panicking::try
             at src/libstd/panicking.rs:325
  17: std::panic::catch_unwind
             at src/libstd/panic.rs:394
  18: std::rt::lang_start_internal
             at src/libstd/rt.rs:51
  19: std::rt::lang_start
             at /rustc/04488afe34512aa4c33566eb16d8c912a3ae04f9/src/libstd/rt.rs:67
  20: main
  21: __libc_start_main
  22: _start
```

Now, in Rust 1.47.0, you'll see this instead:

```
thread 'main' panicked at 'explicit panic', src/main.rs:2:5
stack backtrace:
   0: std::panicking::begin_panic
             at /rustc/d6646f64790018719caebeafd352a92adfa1d75a/library/std/src/panicking.rs:497
   1: playground::main
             at ./src/main.rs:2
   2: core::ops::function::FnOnce::call_once
             at /rustc/d6646f64790018719caebeafd352a92adfa1d75a/library/core/src/ops/function.rs:227
```

This makes it much easier to see where the panic actually originated, and
you can still set `RUST_BACKTRACE=full` if you want to see everything.

#### LLVM 11

We have [upgraded to LLVM 11](https://github.com/rust-lang/rust/pull/73526/).
The compiler still supports being compiled with LLVM versions as old as 8,
but by default, 11 is what you'll be getting.

#### Control Flow Guard on Windows

`rustc` [now supports](https://github.com/rust-lang/rust/pull/73893/) `-C
control-flow-guard`, an option that will turn on [Control Flow
Guard](https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard)
on Windows. Other platforms ignore this flag.

### Library changes

Additionally, nine new APIs were stabilized this release:

- [`Ident::new_raw`]
- [`Range::is_empty`]
- [`RangeInclusive::is_empty`]
- [`Result::as_deref`]
- [`Result::as_deref_mut`]
- [`Vec::leak`]
- [`pointer::offset_from`]
- [`f32::TAU`]
- [`f64::TAU`]

The following previously stable APIs have now been made `const`:

- [The `new` method for all `NonZero` integers.][73858]
- [The `checked_add`, `checked_sub`, `checked_mul`, `checked_neg`, `checked_shl`,
  `checked_shr`, `saturating_add`, `saturating_sub`, and `saturating_mul`
  methods for all integers.][73858]
- [The `checked_abs`, `saturating_abs`, `saturating_neg`, and `signum`  for all
  signed integers.][73858]
- [The `is_ascii_alphabetic`, `is_ascii_uppercase`, `is_ascii_lowercase`,
  `is_ascii_alphanumeric`, `is_ascii_digit`, `is_ascii_hexdigit`,
  `is_ascii_punctuation`, `is_ascii_graphic`, `is_ascii_whitespace`, and
  `is_ascii_control` methods for `char` and `u8`.][73858]

[`Ident::new_raw`]:  https://doc.rust-lang.org/stable/proc_macro/struct.Ident.html#method.new_raw
[`Range::is_empty`]: https://doc.rust-lang.org/stable/std/ops/struct.Range.html#method.is_empty
[`RangeInclusive::is_empty`]: https://doc.rust-lang.org/stable/std/ops/struct.RangeInclusive.html#method.is_empty
[`Result::as_deref_mut`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.as_deref_mut
[`Result::as_deref`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.as_deref
[`TypeId::of`]: https://doc.rust-lang.org/stable/std/any/struct.TypeId.html#method.of
[`Vec::leak`]: https://doc.rust-lang.org/stable/std/vec/struct.Vec.html#method.leak
[`f32::TAU`]: https://doc.rust-lang.org/stable/std/f32/consts/constant.TAU.html
[`f64::TAU`]: https://doc.rust-lang.org/stable/std/f64/consts/constant.TAU.html
[`pointer::offset_from`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.offset_from
[73858]: https://github.com/rust-lang/rust/pull/73858/

See the [detailed release notes][notes] for more.

### Other changes

[Rustdoc has gained support for the Ayu theme](https://github.com/rust-lang/rust/pull/71237/).

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-147-2020-10-08
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-147

There are other changes in the Rust 1.47.0 release: check out what changed in
[Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.47.0

Many people came together to create Rust 1.47.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.47.0/)
