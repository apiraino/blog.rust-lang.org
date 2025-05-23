+++
path = "2024/03/28/Rust-1.77.1"
title = "Announcing Rust 1.77.1"
authors = ["The Rust Release Team"]
aliases = [
    "2024/03/28/Rust-1.77.1.html",
    "releases/1.77.1",
]

[extra]
release = true
+++

The Rust team has published a new point release of Rust, 1.77.1. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.77.1 is as easy as:

```
rustup update stable
```

If you don't have it already, you can [get `rustup`][rustup] from the
appropriate page on our website.

[rustup]: https://www.rust-lang.org/install.html

## What's in 1.77.1

Cargo enabled [stripping of debuginfo in release builds by default](https://github.com/rust-lang/cargo/pull/13257)
in Rust 1.77.0. However, due to a pre-existing issue, debuginfo stripping does not [behave](https://github.com/rust-lang/rust/issues/122857) in the expected way on Windows with the MSVC toolchain.

Rust 1.77.1 therefore [disables](https://github.com/rust-lang/cargo/pull/13630) the new Cargo behavior on Windows for
targets that use MSVC. There are no changes for other targets. We plan to eventually re-enable debuginfo stripping in
release mode in a later Rust release.

### Contributors to 1.77.1

Many people came together to create Rust 1.77.1. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.77.1/)
