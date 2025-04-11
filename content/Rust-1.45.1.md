+++
path = "2020/07/30/Rust-1.45.1"
title = "Announcing Rust 1.45.1"
authors = ["The Rust Release Team"]
aliases = ["2020/07/30/Rust-1.45.1.html"]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.45.1. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.45.1 is as easy as:

```console
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.45.1][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1451-2020-07-30

## What's in 1.45.1 stable

1.45.1 contains a collection of fixes, including one soundness fix. All patches
in 1.45.1 address bugs that affect only the 1.45.0 release; prior releases are
not affected by the bugs fixed in this release.

### Fix const propagation with references

In Rust 1.45.0, `rustc`'s const propagation pass did not properly handle
encountering references when determining whether to propagate a given constant,
which could lead to incorrect behavior. Our releases are run through [crater],
and we did not detect it, which helps us be fairly confident that this affects a
very small set of code in the wild (if any).

The conditions necessary to cause this bug are highly unlikely to occur in
practice: the code must have inputs consisting of entirely constant values and
no control flow or function calls in between.

```rust
struct Foo {
    x: u32,
}

fn main() {
    let mut foo = Foo { x: 42 };
    let x = &mut foo.x;
    *x = 13;
    let y = foo;
    println!("{}", y.x); // -> 42; expected result: 13
}
```

## Contributors to 1.45.1

Many people came together to create Rust 1.45.1. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.45.1/)

[crater]: https://github.com/rust-lang/crater
