+++
path = "2020/08/27/Rust-1.46.0"
title = "Announcing Rust 1.46.0"
authors = ["The Rust Release Team"]
aliases = [
    "2020/08/27/Rust-1.46.0.html",
    "releases/1.46.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.46.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.46.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.46.0][notes] on GitHub.

[install]: https://www.rust-lang.org/tools/install
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1460-2020-08-27

## What's in 1.46.0 stable

This release enables quite a lot of new things to appear in `const fn`, two
new standard library APIs, and one feature useful for library authors. See
the [detailed release notes][notes] to learn about other changes not covered
by this post.

### `const fn` improvements

There are [several core language features] you can now use in a `const fn`:

* `if`, `if let`, and `match`
* `while`, `while let`, and `loop`
* the `&&` and `||` operators

You can also [cast to a slice][cast-to-slice]:

```rust
const fn foo() {
  let x = [1, 2, 3, 4, 5];

  // cast the array to a slice
  let y: &[_] = &x;
}
```

While these features may not feel *new*, given that you could use them all
outside of `const fn`, they add a lot of compile-time computation power! As
an example, the [`const-sha1` crate][sha1] can let you compute SHA-1 hashes
at compile time. This led to a [40x performance improvement][const-perf] in
Microsoft's WinRT bindings for Rust.

[several core language features]: https://github.com/rust-lang/rust/pull/72437/
[cast-to-slice]: https://github.com/rust-lang/rust/pull/73862/
[sha1]: https://github.com/rylev/const-sha1
[const-perf]: https://github.com/microsoft/winrt-rs/pull/279#issuecomment-668436700


### `#[track_caller]`

Back in March, the release of Rust 1.42 introduced [better error messages when `unwrap` and related functions would panic][better-errors]. At the time, we mentioned that the way
this was implemented was not yet stable. Rust 1.46 stabilizes this feature.

[better-errors]: https://blog.rust-lang.org/2020/03/12/Rust-1.42.html#useful-line-numbers-in-option-and-result-panic-messages

This attribute is called `#[track_caller]`, which was originally proposed in
[RFC 2091][rfc-2091] way back in July of 2017! If you're writing a function
like `unwrap` that may panic, you can put this annotation on your functions,
and the default panic formatter will use its caller as the location in its
error message. For example, here is `unwrap` previously:

```rust
pub fn unwrap(self) -> T {
    match self {
        Some(val) => val,
        None => panic!("called `Option::unwrap()` on a `None` value"),
    }
}
```

It now looks like this:

```rust
#[track_caller]
pub fn unwrap(self) -> T {
    match self {
        Some(val) => val,
        None => panic!("called `Option::unwrap()` on a `None` value"),
    }
}
```

That's it!

If you are implementing a panic hook yourself, you can use the [caller] method
on `std::panic::Location` to get access to this information.

[rfc-2091]: https://github.com/rust-lang/rfcs/pull/2091
[caller]: https://doc.rust-lang.org/stable/std/panic/struct.Location.html#method.caller

### Library changes

Keeping with the theme of `const fn` improvements, [`std::mem::forget` is now
a `const fn`][forget]. Additionally, two new APIs were stabilized this release:

* [`Option::zip`][zip]
* [`vec::Drain::as_slice`][as_slice]

[forget]: https://github.com/rust-lang/rust/pull/73887/
[zip]: https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.zip
[as_slice]:  https://doc.rust-lang.org/stable/std/vec/struct.Drain.html#method.as_slice

See the [detailed release notes][notes] for more.

### Other changes

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-146-2020-08-27
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-146

There are other changes in the Rust 1.46.0 release: check out what changed in
[Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.46.0

Many people came together to create Rust 1.46.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.46.0/)
