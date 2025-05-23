+++
path = "2018/01/04/Rust-1.23"
title = "Announcing Rust 1.23"
authors = ["The Rust Core Team"]
aliases = [
    "2018/01/04/Rust-1.23.html",
    "releases/1.23.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.23.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.23.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.23.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1230-2018-01-04

## What's in 1.23.0 stable

New year, new Rust! For our first improvement today, we now [avoid some unnecessary
copies](https://github.com/rust-lang/rust/pull/45380) in certain situations.
We've seen memory usage of using `rustc` to drop 5-10% with this change; it may
be different with your programs.

The documentation team has been on a long journey to move `rustdoc` to use
[CommonMark]. Previously, `rustdoc` never guaranteed which markdown rendering
engine it used, but we're finally committing to CommonMark. As part of this
release, we render the documentation with our previous renderer, [Hoedown],
but also render it with a CommonMark compliant renderer, and [warn if there
are any differences]. There should be a way for you to modify the syntax you
use to render correctly under both; we're not aware of any situations where
this is impossible. Docs team member Guillaume Gomez has [written a blog post]
showing some common differences and how to solve them. In a future release,
we will switch to using the CommonMark renderer by default. This [warning
landed in nightly in May of last year], and has been on by default [since
October of last year], so many crates have already fixed any issues that
they've found.

[CommonMark]: http://commonmark.org/
[Hoedown]: https://github.com/hoedown/hoedown
[warn if there are any differences]: https://github.com/rust-lang/rust/pull/45324
[written a blog post]: https://blog.guillaume-gomez.fr/articles/2017-09-18+New+rustdoc+rendering+common+errors
[warning landed in nightly in May of last year]: https://github.com/rust-lang/rust/pull/41991
[since October of last year]: https://github.com/rust-lang/rust/pull/45324

In other documentation news, historically, Cargo's docs have been a bit strange.
Rather than being on [doc.rust-lang.org](https://doc.rust-lang.org),
they've been at [doc.crates.io](http://doc.crates.io).
With this release, [that's changing](https://github.com/rust-lang/rust/pull/45692).
You can now find Cargo's docs at [doc.rust-lang.org/cargo](https://doc.rust-lang.org/cargo).
Additionally, they've been converted to the same format as our other long-form documentation.
We'll be adding a redirect from `doc.crates.io` to this page, and you can expect to see more
improvements and updates to Cargo's docs throughout the year.

See the [detailed release notes][notes] for more.

### Library stabilizations

As of Rust 1.0, a trait named [`AsciiExt`] existed to provide ASCII related functionality
on `u8`, `char`, `[u8]`, and `str`. To use it, you'd write code like this:

```rust
use std::ascii::AsciiExt;

let ascii = 'a';
let non_ascii = '❤';
let int_ascii = 97;

assert!(ascii.is_ascii());
assert!(!non_ascii.is_ascii());
assert!(int_ascii.is_ascii());
```

In Rust 1.23, these methods are now defined directly on those types, and so you no longer need
to import the trait. Thanks to our stability guarantees, this trait still exists, so if you'd
like to still support Rust versions before Rust 1.23, you can do this:

```rust
#[allow(unused_imports)]
use std::ascii::AsciiExt;
```

…to suppress the related warning. Once you drop support for older Rusts, you
can remove both lines, and everything will continue to work.

[`AsciiExt`]: https://doc.rust-lang.org/std/ascii/trait.AsciiExt.html

Additionally, a few new APIs were stabilized this release:

* The various [`std::sync::atomic
  types`](https://doc.rust-lang.org/std/sync/atomic/index.html#structs)
  now implement `From` their non-atomic types. For example, `let x = AtomicBool::from(true);`.
* [`()` now implements `FromIterator<()>`](https://github.com/rust-lang/rust/pull/45379); check the PR for
  a neat use-case.
* [`RwLock<T>` has had its `Send` restriction lifted](https://github.com/rust-lang/rust/pull/45682)

See the [detailed release notes][notes] for more.

### Cargo features

`cargo check` can now [check your unit tests](https://github.com/rust-lang/cargo/pull/4592).

`cargo uninstall` can now [uninstall more than one package in one command](https://github.com/rust-lang/cargo/pull/4561).

See the [detailed release notes][notes] for more.

## Contributors to 1.23.0

Many people came together to create Rust 1.23. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.23.0)
