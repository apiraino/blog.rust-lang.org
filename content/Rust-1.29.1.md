+++
path = "2018/09/25/Rust-1.29.1"
title = "Announcing Rust 1.29.1"
authors = ["The Rust Core Team"]
aliases = [
    "2018/09/25/Rust-1.29.1.html",
    "releases/1.29.1",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.29.1. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.29.1 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.29.1][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1291-2018-09-25

## What's in 1.29.1 stable

A [security vulnerability][vuln] was found in the standard library where if a
large number was passed to `str::repeat` it could cause a buffer overflow
after an integer overflow. If you do not call the `str::repeat` function you
are not affected. This has been addressed by unconditionally panicking in
`str::repeat` on integer overflow. More details about this can be found in the
[security announcement][vuln].

[vuln]: https://blog.rust-lang.org/2018/09/21/Security-advisory-for-std.html
