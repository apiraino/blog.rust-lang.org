+++
path = "2018/11/08/Rust-1.30.1"
title = "Announcing Rust 1.30.1"
authors = ["The Rust Release Team"]
aliases = [
    "2018/11/08/Rust-1.30.1.html",
    "releases/1.30.1",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.30.1. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.30.1 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.30.1][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1301-2018-11-08

## What's in 1.30.1 stable

This patch release fixes broken Cargo progress bars in MSYS terminals on
Windows by [capping the progress bar width to 60 columns][cargo/6122]. This doesn't affect
other terminal emulators (like `cmd.exe` or PowerShell).

This patch release also [fixes a compiler panic][54199] that happened while building the
docs of some crates in Rust 1.30.0. The crates impacted were widely used, so
this change impacted a considerable amount of users, which made it sufficiently
prominent for us to issue a point release.


[cargo/6122]: https://github.com/rust-lang/cargo/pull/6122
[54199]: https://github.com/rust-lang/rust/pull/54199
