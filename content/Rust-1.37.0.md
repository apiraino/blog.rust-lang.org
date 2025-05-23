+++
path = "2019/08/15/Rust-1.37.0"
title = "Announcing Rust 1.37.0"
authors = ["The Rust Release Team"]
aliases = [
    "2019/08/15/Rust-1.37.0.html",
    "releases/1.37.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.37.0. Rust is a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.37.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the appropriate page on our website, and check out the [detailed release notes for 1.37.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1370-2019-08-15

## What's in 1.37.0 stable

The highlights of Rust 1.37.0 include referring to `enum` variants through `type` aliases, built-in `cargo vendor`, unnamed `const` items, profile-guided optimization, a `default-run` key in Cargo, and `#[repr(align(N))]` on `enum`s. Read on for a few highlights, or see the [detailed release notes][notes] for additional information.

### Referring to `enum` variants through `type` aliases

With Rust 1.37.0, you can now refer to `enum` variants through type aliases. For example:

```rust
type ByteOption = Option<u8>;

fn increment_or_zero(x: ByteOption) -> u8 {
    match x {
        ByteOption::Some(y) => y + 1,
        ByteOption::None => 0,
    }
}
```

In implementations, `Self` acts like a type alias. So in Rust 1.37.0, you can also refer to `enum` variants with `Self::Variant`:

```rust
impl Coin {
    fn value_in_cents(&self) -> u8 {
        match self {
            Self::Penny => 1,
            Self::Nickel => 5,
            Self::Dime => 10,
            Self::Quarter => 25,
        }
    }
}
```

[type_rel_report]: https://github.com/rust-lang/rust/pull/61682/#issuecomment-502472847

To be more exact, Rust now allows you to refer to `enum` variants through *"type-relative resolution"*, `<MyType<..>>::Variant`. More details are available in [the stabilization report][type_rel_report].

### Built-in Cargo support for vendored dependencies

[vendor-crate]: https://crates.io/crates/cargo-vendor

After being available [as a separate crate][vendor-crate] for years, the `cargo vendor` command is now integrated directly into Cargo. The command fetches all your project's dependencies unpacking them into the `vendor/` directory, and shows the configuration snippet required to use the vendored code during builds.

There are multiple cases where `cargo vendor` is already used in production: the Rust compiler `rustc` uses it to ship all its dependencies in release tarballs, and projects with monorepos use it to commit the dependencies' code in source control.

### Using unnamed `const` items for macros

[unnamed_const_pr]: https://github.com/rust-lang/rust/pull/61347/

You can now create [unnamed `const` items][unnamed_const_pr]. Instead of giving your constant an explicit name, simply name it `_` instead. For example, in the `rustc` compiler we find:

```rust
/// Type size assertion where the first parameter
/// is a type and the second is the expected size.
#[macro_export]
macro_rules! static_assert_size {
    ($ty:ty, $size:expr) => {
        const _: [(); $size] = [(); ::std::mem::size_of::<$ty>()];
        //    ^ Note the underscore here.
    }
}

static_assert_size!(Option<Box<String>>, 8); // 1.
static_assert_size!(usize, 8); // 2.
```

Notice the second `static_assert_size!(..)`: thanks to the use of unnamed constants, you can define new items without naming conflicts. Previously you would have needed to write `static_assert_size!(MY_DUMMY_IDENTIFIER, usize, 8);`. Instead, with Rust 1.37.0, it now becomes easier to create ergonomic and reusable declarative and procedural macros for static analysis purposes.

### Profile-guided optimization

[rustc_book_pgo]: https://doc.rust-lang.org/rustc/profile-guided-optimization.html
[pgo_pr]: https://github.com/rust-lang/rust/pull/61268/
[pgo_wiki]: https://en.wikipedia.org/wiki/Profile-guided_optimization

The `rustc` compiler now comes with [support for Profile-Guided Optimization (PGO)][pgo_pr] via the `-C profile-generate` and `-C profile-use` flags.

[Profile-Guided Optimization][pgo_wiki] allows the compiler to optimize code based on feedback from real workloads. It works by compiling the program to optimize in two steps:

1. First, the program is built with instrumentation inserted by the compiler. This is done by passing the `-C profile-generate` flag to `rustc`. The instrumented program then needs to be run on sample data and will write the profiling data to a file.
2. Then, the program is built *again*, this time feeding the collected profiling data back into `rustc` by using the `-C profile-use` flag. This build will make use of the collected data to allow the compiler to make better decisions about code placement, inlining, and other optimizations.

For more in-depth information on Profile-Guided Optimization, please refer to the corresponding [chapter in the rustc book][rustc_book_pgo].

### Choosing a default binary in Cargo projects

[`default-run`]: https://doc.rust-lang.org/cargo/reference/manifest.html#the-default-run-field
[`cargo run`]: https://doc.rust-lang.org/cargo/commands/cargo-run.html

[`cargo run`] is great for quickly testing CLI applications. When multiple binaries are present in the same package, you have to explicitly declare the name of the binary you want to run with the `--bin` flag. This makes `cargo run` not as ergonomic as we'd like, especially when a binary is called more often than the others.

Rust 1.37.0 addresses the issue by adding [`default-run`], a new key in `Cargo.toml`. When the key is declared in the `[package]` section, `cargo run` will default to the chosen binary if the `--bin` flag is not passed.

### `#[repr(align(N))]` on `enum`s

[enum_align_pr]: https://github.com/rust-lang/rust/pull/61229
[ref_align_mod]: https://doc.rust-lang.org/reference/type-layout.html#the-alignment-modifiers
[ref_align_explain]: https://doc.rust-lang.org/reference/type-layout.html#size-and-alignment

[The `#[repr(align(N))]` attribute][ref_align_mod] can be used to raise the [alignment][ref_align_explain] of a type definition. Previously, the attribute was only allowed on `struct`s and `union`s. With Rust 1.37.0, the attribute can now also be used [on `enum` definitions][enum_align_pr]. For example, the following type `Align16` would, as expected, report `16` as the alignment whereas the natural alignment without `#[repr(align(16))]` would be `4`:

```rust
#[repr(align(16))]
enum Align16 {
    Foo { foo: u32 },
    Bar { bar: u32 },
}
```

The semantics of using `#[repr(align(N))` on an `enum` is the same as defining a wrapper struct `AlignN<T>` with that alignment and then using `AlignN<MyEnum>`:

```rust
#[repr(align(N))]
struct AlignN<T>(T);
```

### Library changes

[`BufReader::buffer`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.buffer
[`BufWriter::buffer`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.buffer
[`Cell::from_mut`]: https://doc.rust-lang.org/std/cell/struct.Cell.html#method.from_mut
[`Cell::as_slice_of_cells`]: https://doc.rust-lang.org/std/cell/struct.Cell.html#method.as_slice_of_cells
[`DoubleEndedIterator::nth_back`]: https://doc.rust-lang.org/std/iter/trait.DoubleEndedIterator.html#method.nth_back
[`Option::xor`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.xor
[`Wrapping::reverse_bits`]: https://doc.rust-lang.org/std/num/struct.Wrapping.html#method.reverse_bits
[`{i,u}{8,16,32,64,128,size}::reverse_bits`]: https://doc.rust-lang.org/std/primitive.u8.html#method.reverse_bits
[`slice::copy_within`]: https://doc.rust-lang.org/std/primitive.slice.html#method.copy_within

In Rust 1.37.0 there have been a number of standard library stabilizations:

- [`BufReader::buffer`] and [`BufWriter::buffer`]
- [`Cell::from_mut`]
- [`Cell::as_slice_of_cells`]
- [`DoubleEndedIterator::nth_back`]
- [`Option::xor`]
- [`{i,u}{8,16,32,64,128,size}::reverse_bits`] and [`Wrapping::reverse_bits`]
- [`slice::copy_within`]

### Other changes

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-137-2019-08-15
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-137

There are other changes in the Rust 1.37 release: check out what changed in [Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.37.0

Many people came together to create Rust 1.37.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.37.0/)

## New sponsors of Rust infrastructure

We'd like to thank two new sponsors of Rust's infrastructure who provided the resources needed to make Rust 1.37.0 happen: Amazon Web Services (AWS) and Microsoft Azure.

- AWS has provided hosting for release artifacts (compilers, libraries, tools, and source code), serving those artifacts to users through CloudFront, preventing regressions with Crater on EC2, and managing other Rust-related infrastructure hosted on AWS.
- Microsoft Azure has sponsored builders for Rust’s CI infrastructure, notably the extremely resource intensive rust-lang/rust repository.
