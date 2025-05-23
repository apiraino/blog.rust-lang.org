+++
path = "2019/05/14/Rust-1.34.2"
title = "Announcing Rust 1.34.2"
authors = ["The Rust Release Team"]
aliases = [
    "2019/05/14/Rust-1.34.2.html",
    "releases/1.34.2",
]

[extra]
release = true
+++

The Rust team has published a new point release of Rust, 1.34.2. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.34.2 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1342-2019-05-14

## What's in 1.34.2 stable

Sean McArthur reported a [security vulnerability][ml] affecting the standard
library that caused the [`Error::downcast`][Error::downcast] family of methods
to perform unsound casts when a manual implementation of the
[`Error::type_id`][Error::type_id] method returned the wrong
[`TypeId`][TypeId], leading to security issues such as out of bounds
reads/writes/etc.

The [`Error::type_id`][Error::type_id] method was recently stabilized as part
of Rust 1.34.0. This point release **destabilizes** it, preventing any code on
the stable and beta channels to implement or use it, awaiting future plans that
will be discussed in [issue #60784][60784].

An in-depth explanation of this issue was posted in yesterday's [security
advisory][ml]. The assigned CVE for the vulnerability is [CVE-2019-12083][cve].

[ml]: https://groups.google.com/d/msg/rustlang-security-announcements/aZabeCMUv70/-2Y6-SL6AQAJ
[Error::downcast]: https://doc.rust-lang.org/stable/std/error/trait.Error.html#method.downcast
[Error::type_id]: https://doc.rust-lang.org/stable/std/error/trait.Error.html#method.type_id
[TypeId]: https://doc.rust-lang.org/stable/std/any/struct.TypeId.html
[60784]: https://github.com/rust-lang/rust/issues/60784
[cve]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-12083
