+++
path = "2021/05/06/Rust-1.52.0"
title = "Announcing Rust 1.52.0"
authors = ["The Rust Release Team"]
aliases = ["2021/05/06/Rust-1.52.0.html"]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.52.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.52.0 is as easy as:

```console
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.52.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1520-2021-05-06

## What's in 1.52.0 stable

The most significant change in this release is not to the language or standard
libraries, but rather an enhancement to tooling support for Clippy.

Previously, running `cargo check` followed by `cargo clippy` wouldn't actually
run Clippy: the build caching in Cargo didn't differentiate between the two. In
1.52, however, this has been fixed, which means that users will get the expected
behavior independent of the order in which they run the two commands.

### Stabilized APIs

The following methods were stabilized.

- [`Arguments::as_str`]
- [`char::MAX`]
- [`char::REPLACEMENT_CHARACTER`]
- [`char::UNICODE_VERSION`]
- [`char::decode_utf16`]
- [`char::from_digit`]
- [`char::from_u32_unchecked`]
- [`char::from_u32`]
- [`slice::partition_point`]
- [`str::rsplit_once`]
- [`str::split_once`]

The following previously stable APIs are now `const`.

- [`char::len_utf8`]
- [`char::len_utf16`]
- [`char::to_ascii_uppercase`]
- [`char::to_ascii_lowercase`]
- [`char::eq_ignore_ascii_case`]
- [`u8::to_ascii_uppercase`]
- [`u8::to_ascii_lowercase`]
- [`u8::eq_ignore_ascii_case`]

[`char::MAX`]: https://doc.rust-lang.org/std/primitive.char.html#associatedconstant.MAX
[`char::REPLACEMENT_CHARACTER`]: https://doc.rust-lang.org/std/primitive.char.html#associatedconstant.REPLACEMENT_CHARACTER
[`char::UNICODE_VERSION`]: https://doc.rust-lang.org/std/primitive.char.html#associatedconstant.UNICODE_VERSION
[`char::decode_utf16`]: https://doc.rust-lang.org/std/primitive.char.html#method.decode_utf16
[`char::from_u32`]: https://doc.rust-lang.org/std/primitive.char.html#method.from_u32
[`char::from_u32_unchecked`]: https://doc.rust-lang.org/std/primitive.char.html#method.from_u32_unchecked
[`char::from_digit`]: https://doc.rust-lang.org/std/primitive.char.html#method.from_digit
[`Arguments::as_str`]: https://doc.rust-lang.org/stable/std/fmt/struct.Arguments.html#method.as_str
[`str::split_once`]: https://doc.rust-lang.org/stable/std/primitive.str.html#method.split_once
[`str::rsplit_once`]: https://doc.rust-lang.org/stable/std/primitive.str.html#method.rsplit_once
[`slice::partition_point`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.partition_point
[`char::len_utf8`]: https://doc.rust-lang.org/stable/std/primitive.char.html#method.len_utf8
[`char::len_utf16`]: https://doc.rust-lang.org/stable/std/primitive.char.html#method.len_utf16
[`char::to_ascii_uppercase`]: https://doc.rust-lang.org/stable/std/primitive.char.html#method.to_ascii_uppercase
[`char::to_ascii_lowercase`]: https://doc.rust-lang.org/stable/std/primitive.char.html#method.to_ascii_lowercase
[`char::eq_ignore_ascii_case`]: https://doc.rust-lang.org/stable/std/primitive.char.html#method.eq_ignore_ascii_case
[`u8::to_ascii_uppercase`]: https://doc.rust-lang.org/stable/std/primitive.u8.html#method.to_ascii_uppercase
[`u8::to_ascii_lowercase`]: https://doc.rust-lang.org/stable/std/primitive.u8.html#method.to_ascii_lowercase
[`u8::eq_ignore_ascii_case`]: https://doc.rust-lang.org/stable/std/primitive.u8.html#method.eq_ignore_ascii_case

### Other changes

There are other changes in the Rust 1.52.0 release: check out what changed in [Rust](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1520-2021-05-06), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-152-2021-05-06), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-152).

### Contributors to 1.52.0

Many people came together to create Rust 1.52.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.52.0/)
