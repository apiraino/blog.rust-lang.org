+++
layout = "post"
date = 2021-11-01
title = "Announcing Rust 1.56.1"
author = "The Rust Security Response WG"
release = true
+++

The Rust team has published a new point release of Rust, 1.56.1. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.56.1 is as easy as:

```
rustup update stable
```

If you don't have it already, you can [get `rustup`][rustup] from the
appropriate page on our website.

[rustup]: https://www.rust-lang.org/install.html

## What's in 1.56.1 stable

Rust 1.56.1 introduces two new lints to mitigate the impact of a security
concern recently disclosed, [CVE-2021-42574]. We recommend all users upgrade
immediately to ensure their codebase is not affected by the security issue.

You can learn more about the security issue in [the advisory][advisory].

[advisory]: https://blog.rust-lang.org/2021/11/01/cve-2021-42574.html
[CVE-2021-42574]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-42574
