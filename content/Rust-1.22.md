+++
path = "2017/11/22/Rust-1.22"
title = "Announcing Rust 1.22 (and 1.22.1)"
authors = ["The Rust Core Team"]
aliases = [
    "2017/11/22/Rust-1.22.html",
    "releases/1.22.0",
]

[extra]
release = true
+++

The Rust team is happy to announce *two* new versions of Rust, 1.22.0 and
1.22.1. Rust is a systems programming language focused on safety, speed, and
concurrency.

> Wait, two versions? At the last moment, we [discovered a late-breaking
> issue with the new macOS High
> Sierra](https://github.com/rust-lang/rust/pull/46183) in 1.22.0, and for
> various reasons, decided to release 1.22.0 as usual, but also put out a
> 1.22.1 with the patch. The bug is actually in Cargo, not `rustc`, and only
> affects users on macOS High Sierra.

If you have a previous version of Rust installed via rustup, getting Rust
1.22.1 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.22.0][notes] and 1.22.1 on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1220-2017-11-22

## What's in 1.22.0 and 1.22.1 stable

The headline feature for this release is one many have been anticipating for
a long time: you can now [use `?` with `Option<T>`]! About a year ago, in
[Rust 1.13], we introduced the `?` operator for working with `Result<T, E>`.
Ever since then, there's been discussion about how far `?` should go: should
it stay only for results? Should it be user-extensible? Should it be
usable with `Option<T>`?

In Rust 1.22, basic usage of `?` with `Option<T>` is now stable.
This code will now compile:

```rust
fn add_one_to_first(list: &[u8]) -> Option<u8> {
    // get first element, if exists
    // return None if it doesn't
    let first = list.get(0)?;
    Some(first + 1)
}

assert_eq!(add_one_to_first(&[10, 20, 30]), Some(11));
assert_eq!(add_one_to_first(&[]), None);
```

However, this functionality is still a bit limited; you cannot yet write
code that mixes results and options with `?` in the same function, for
example. This will be possible in the future, and already is in nightly
Rust; expect to hear more about this in a future release.

[use `?` with `Option<T>`]: https://github.com/rust-lang/rust/pull/42526
[Rust 1.13]: https://blog.rust-lang.org/2016/11/10/Rust-1.13.html

Types that implement `Drop` are [now allowed in `const` and `static`
items](https://github.com/rust-lang/rust/pull/44456). Like this:

```rust
struct Foo {
    a: u32,
}

impl Drop for Foo {
    fn drop(&mut self) {}
}

const F: Foo = Foo { a: 0 };
static S: Foo = Foo { a: 0 };
```

This change doesn't bring much on its own, but as we improve our
ability to compute things at compile-time, more and more will be
possible in `const` and `static`.

Additionally, some small quality-of-life improvements:

[Two] recent [compiler changes] should speed up compiles in debug mode. We
don't have any specific numbers to commit to with these changes, but as
always, compile times are very important to us, and we're continuing to
work on improving them.

[Two]: https://github.com/rust-lang/rust/pull/45075
[compiler changes]: https://github.com/rust-lang/rust/pull/45064

[`T op= &T` now works for primitive types][add], which is a fancy way of saying:

```rust
let mut x = 2;
let y = &8;

// this didn't work, but now does
x += y;
```

Previously, you'd have needed to write `x += *y` in order to de-reference, so
this solves a small papercut.

[add]: https://github.com/rust-lang/rust/pull/44287

[Backtraces are improved on MacOS](https://github.com/rust-lang/rust/pull/44251).

You can now [create `compile-fail` tests in Rustdoc], like this:

```
/// ```compile_fail
/// let x = 5;
/// x += 2; // shouldn't compile!
/// ```
```

Please note that these kinds of tests can be more fragile than others, as
additions to Rust may cause code to compile when it previously would not.
Consider the first announcement with `?`, for example: that code would fail
to compile on Rust 1.21, but compile successfully on Rust 1.22, causing your
test suite to start failing.

[create `compile-fail` tests in Rustdoc]: https://github.com/rust-lang/rust/pull/43949

Finally, we [removed support for the `le32-unknown-nacl`
target](https://github.com/rust-lang/rust/pull/45041). Google itself [has
deprecated
PNaCl](https://blog.chromium.org/2017/05/goodbye-pnacl-hello-webassembly.html),
instead throwing its support behind [WebAssembly](http://webassembly.org/).
You can already compile Rust code to WebAssembly today, and you can expect
to hear more developments regarding this in future releases.

See the [detailed release notes][notes] for more.

### Library stabilizations

A few new APIs were stabilized this release:

- [`Box<Error>` now impls `From<Cow<str>>`][44466]
- [`std::mem::Discriminant` is now guaranteed to be `Send + Sync`][45095]
- [`fs::copy` now returns the length of the main stream on NTFS.][44895]
- [Properly detect overflow in `Instant += Duration`.][44220]
- [impl `Hasher` for `{&mut Hasher, Box<Hasher>}`][44015]
- [impl `fmt::Debug` for `SplitWhitespace`.][44303]

[44466]: https://github.com/rust-lang/rust/pull/44466
[45095]: https://github.com/rust-lang/rust/pull/45095
[44895]: https://github.com/rust-lang/rust/pull/44895
[44220]: https://github.com/rust-lang/rust/pull/44220
[44015]: https://github.com/rust-lang/rust/pull/44015
[44303]: https://github.com/rust-lang/rust/pull/44303

See the [detailed release notes][notes] for more.

### Cargo features

If you have a big example to show your users, Cargo has grown
the ability to [build multi-file
examples](https://github.com/rust-lang/cargo/pull/4496) by
creating a subdirectory inside `examples` that contains a
`main.rs`.

Cargo now has the ability to [vendor git repositories](https://github.com/rust-lang/cargo/pull/3992).

See the [detailed release notes][notes] for more.

## Contributors to 1.22.0 and 1.22.1

Many people came together to create Rust 1.22. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.22.0) (and
[Thanks again!](https://thanks.rust-lang.org/rust/1.22.1))
