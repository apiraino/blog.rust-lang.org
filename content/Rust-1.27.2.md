+++
path = "2018/07/20/Rust-1.27.2"
title = "Announcing Rust 1.27.2"
authors = ["The Rust Core Team"]
aliases = [
    "2018/07/20/Rust-1.27.2.html",
    "releases/1.27.2",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.27.2. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.27.2 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.27.2][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1272-2018-07-20

## What's in 1.27.2 stable

This patch release fixes a bug in the borrow checker verification of `match` expressions. This bug
was introduced in 1.27.1 with a different bugfix for [match ergonomics].

```rust
fn transmute_lifetime<'a, 'b, T>(t: &'a (T,)) -> &'b T {
    match (&t, ()) {
        ((t,), ()) => t,
    }
}

fn main() {
    let x = {
        let y = Box::new((42,));
        transmute_lifetime(&y)
    };

    println!("{}", x);
}
```

1.27.2 will reject the above code.

## Concern over numerous patches to the match ergonomics feature

Users have expressed concern with the frequency of patch releases to fix bugs in the match
ergonomics verification by the current borrow checker on a variety of Rust's forums. There are two
primary reasons for the increased rate of patch releases: significantly higher bandwidth and the
age of the currently used borrow checker.

With the formation of the Release team, Rust's ability to generate patch releases has
greatly increased. This means that the investment from the compiler and core teams required to make
a patch release is greatly reduced, which also makes such a patch release more likely to happen.

The current borrow checker has been around for years now, and is beginning to show its age.  The
work on a better, more precise borrow checker is underway, and it has detected all of these bugs.
This work is planned to be stabilized in the next few releases, so expect to hear more about it
soon.

Together, the lack of good maintenance on the current borrow checker and an increased capacity for
releases make it feasible for us to ship patch releases on a more rapid and frequent basis.

[match ergonomics]: https://blog.rust-lang.org/2018/05/10/Rust-1.26.html#nicer-match-bindings
