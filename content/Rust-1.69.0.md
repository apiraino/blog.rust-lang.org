+++
path = "2023/04/20/Rust-1.69.0"
title = "Announcing Rust 1.69.0"
authors = ["The Rust Release Team"]
aliases = [
    "2023/04/20/Rust-1.69.0.html",
    "releases/1.69.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a nice version of Rust, 1.69.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.69.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.69.0](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1690-2023-04-20) on GitHub.

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.69.0 stable

Rust 1.69.0 introduces no major new features. However, it contains many small improvements, including over 3,000 commits from over 500 contributors.

### Cargo now suggests to automatically fix some warnings

Rust 1.29.0 added the `cargo fix` subcommand to automatically fix some simple compiler warnings. Since then, the number of warnings that can be fixed automatically continues to steadily increase. In addition, support for automatically fixing some simple Clippy warnings has also been added.

In order to draw more attention to these increased capabilities, Cargo will now suggest running `cargo fix` or `cargo clippy --fix` when it detects warnings that are automatically fixable:

```
warning: unused import: `std::hash::Hash`
 --> src/main.rs:1:5
  |
1 | use std::hash::Hash;
  |     ^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `foo` (bin "foo") generated 1 warning (run `cargo fix --bin "foo"` to apply 1 suggestion)
```

Note that the full Cargo invocation shown above is only necessary if you want to precisely apply fixes to a single crate. If you want to apply fixes to all the default members of a workspace, then a simple `cargo fix` (with no additional arguments) will suffice.

### Debug information is not included in build scripts by default anymore

To improve compilation speed, Cargo now avoids emitting debug information in build scripts by default. There will be no visible effect when build scripts execute successfully, but backtraces in build scripts will contain less information.

If you want to debug a build script, you can add this snippet to your `Cargo.toml` to emit debug information again:

```toml
[profile.dev.build-override]
debug = true
[profile.release.build-override]
debug = true
```

### Stabilized APIs

- [`CStr::from_bytes_until_nul`](https://doc.rust-lang.org/stable/core/ffi/struct.CStr.html#method.from_bytes_until_nul)
- [`core::ffi::FromBytesUntilNulError`](https://doc.rust-lang.org/stable/core/ffi/struct.FromBytesUntilNulError.html)

These APIs are now stable in const contexts:

- [`SocketAddr::new`](https://doc.rust-lang.org/stable/std/net/enum.SocketAddr.html#method.new)
- [`SocketAddr::ip`](https://doc.rust-lang.org/stable/std/net/enum.SocketAddr.html#method.ip)
- [`SocketAddr::port`](https://doc.rust-lang.org/stable/std/net/enum.SocketAddr.html#method.port)
- [`SocketAddr::is_ipv4`](https://doc.rust-lang.org/stable/std/net/enum.SocketAddr.html#method.is_ipv4)
- [`SocketAddr::is_ipv6`](https://doc.rust-lang.org/stable/std/net/enum.SocketAddr.html#method.is_ipv6)
- [`SocketAddrV4::new`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV4.html#method.new)
- [`SocketAddrV4::ip`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV4.html#method.ip)
- [`SocketAddrV4::port`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV4.html#method.port)
- [`SocketAddrV6::new`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV6.html#method.new)
- [`SocketAddrV6::ip`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV6.html#method.ip)
- [`SocketAddrV6::port`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV6.html#method.port)
- [`SocketAddrV6::flowinfo`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV6.html#method.flowinfo)
- [`SocketAddrV6::scope_id`](https://doc.rust-lang.org/stable/std/net/struct.SocketAddrV6.html#method.scope_id)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1690-2023-04-20), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-169-2023-04-20), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-169).

## Contributors to 1.69.0

Many people came together to create Rust 1.69.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.69.0/)
