+++
path = "2022/01/20/Rust-1.58.1"
title = "Announcing Rust 1.58.1"
authors = ["The Rust Release Team"]
aliases = [
    "2022/01/20/Rust-1.58.1.html",
    "releases/1.58.1",
]

[extra]
release = true
+++

The Rust team has published a new point release of Rust, 1.58.1. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.58.1 is as easy as:

```
rustup update stable
```

If you don't have it already, you can [get `rustup`][rustup] from the
appropriate page on our website.

[rustup]: https://www.rust-lang.org/install.html

## What's in 1.58.1 stable

Rust 1.58.1 fixes a race condition in the `std::fs::remove_dir_all` standard
library function. This security vulnerability is tracked as [CVE-2022-21658],
and you can read more about it [on the advisory we published earlier
today][advisory]. We recommend all users to update their toolchain immediately
and rebuild their programs with the updated compiler.

Rust 1.58.1 also addresses several regressions in diagnostics and tooling introduced in Rust 1.58.0:

* The `non_send_fields_in_send_ty` Clippy lint was discovered to have too many
  false positives and has been moved to the experimental lints group (called
  "nursery").
* The `useless_format` Clippy lint has been updated to handle captured
  identifiers in format strings, introduced in Rust 1.58.0.
* A regression in Rustfmt preventing generated files from being formatted when
  passed through the standard input has been fixed.
* An incorrect error message displayed by `rustc` in some cases has been fixed.

You can find more detailed information on the specific regressions in the
[release notes].

[CVE-2022-21658]: https://www.cve.org/CVERecord?id=CVE-2022-21658
[advisory]: https://blog.rust-lang.org/2022/01/20/cve-2022-21658.html
[release notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1581-2022-01-20

### Contributors to 1.58.1

Many people came together to create Rust 1.58.1. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.58.1/)
