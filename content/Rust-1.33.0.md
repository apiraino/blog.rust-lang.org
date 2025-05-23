+++
path = "2019/02/28/Rust-1.33.0"
title = "Announcing Rust 1.33.0"
authors = ["The Rust Release Team"]
aliases = [
    "2019/02/28/Rust-1.33.0.html",
    "releases/1.33.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.33.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.33.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.33.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1330-2019-02-28

## What's in 1.33.0 stable

The two largest features in this release are significant improvements to
`const fn`s, and the stabilization of a new concept: "pinning."

### `const fn` improvements

With `const fn`, you can [now do way more
things!](https://github.com/rust-lang/rust/pull/57175/) Specifically:

* irrefutable destructuring patterns (e.g. `const fn foo((x, y): (u8, u8)) { ... }`)
* `let` bindings (e.g. `let x = 1;`)
* mutable `let` bindings (e.g. `let mut x = 1;`)
* assignment (e.g. `x = y`) and assignment operator (e.g. `x += y`)
  expressions, even where the assignment target is a projection (e.g. a struct
  field or index operation like `x[3] = 42`)
* expression statements (e.g. `3;`)

You're also [able to call `const unsafe fn`s inside a `const
fn`](https://github.com/rust-lang/rust/pull/57067/), like this:

```rust
const unsafe fn foo() -> i32 { 5 }
const fn bar() -> i32 {
    unsafe { foo() }
}
```

With these additions, many more functions in the standard library are able to
be marked as `const`. We'll enumerate those in the library section below.

### Pinning

This release introduces a new concept for Rust programs, implemented as two
types: the [`std::pin::Pin<P>`
type](https://doc.rust-lang.org/std/pin/struct.Pin.html), and the [`Unpin`
marker trait](https://doc.rust-lang.org/std/marker/trait.Unpin.html). The core
idea is elaborated on in [the docs for
`std::pin`](https://doc.rust-lang.org/std/pin/index.html):

> It is sometimes useful to have objects that are guaranteed to not move, in
> the sense that their placement in memory does not change, and can thus be
> relied upon. A prime example of such a scenario would be building
> self-referential structs, since moving an object with pointers to itself will
> invalidate them, which could cause undefined behavior.
>
> A `Pin<P>` ensures that the pointee of any pointer type `P` has a stable location
> in memory, meaning it cannot be moved elsewhere and its memory cannot be
> deallocated until it gets dropped. We say that the pointee is "pinned".

This feature will largely be used by library authors, and so we won't talk a
lot more about the details here. Consult the docs if you're interested in
digging into the details. However, the stabilization of this API is important
to Rust users generally because it is a significant step forward towards a
highly anticipated Rust feature: `async`/`await`. We're not quite there yet,
but this stabilization brings us one step closer. You can track all of the
necessary features at [areweasyncyet.rs](https://areweasyncyet.rs/).

### Import as `_`

[You can now import an item as
`_`](https://github.com/rust-lang/rust/pull/56303/). This allows you to
import a trait's impls, and not have the name in the namespace. e.g.

```rust
use std::io::Read as _;

// Allowed as there is only one `Read` in the module.
pub trait Read {}
```

See the [detailed release notes][notes] for more details.

### Library stabilizations

Here's all of the stuff that's been made `const`:

- [The methods `overflowing_{add, sub, mul, shl, shr}` are now `const`
  functions for all numeric types.][57566]
- [The methods `rotate_left`, `rotate_right`, and `wrapping_{add, sub, mul, shl, shr}`
  are now `const` functions for all numeric types.][57105]
- [The methods `is_positive` and `is_negative` are now `const` functions for
  all signed numeric types.][57105]
- [The `get` method for all `NonZero` types is now `const`.][57167]
- [The methods `count_ones`, `count_zeros`, `leading_zeros`, `trailing_zeros`,
  `swap_bytes`, `from_be`, `from_le`, `to_be`, `to_le` are now `const` for all
  numeric types.][57234]
- [`Ipv4Addr::new` is now a `const` function][57234]

[57566]: https://github.com/rust-lang/rust/pull/57566
[57105]: https://github.com/rust-lang/rust/pull/57105
[57105]: https://github.com/rust-lang/rust/pull/57105
[57167]: https://github.com/rust-lang/rust/pull/57167
[57234]: https://github.com/rust-lang/rust/pull/57234
[57234]: https://github.com/rust-lang/rust/pull/57234

Additionally, these APIs have become stable:

- [`unix::FileExt::read_exact_at`] and [`unix::FileExt::write_all_at`]
- [`Option::transpose`] and [`Result::transpose`]
- [`convert::identity`]
- [`pin::Pin`] and [`marker::Unpin`] (mentioned above)
- [`marker::PhantomPinned`]
- [`Vec::resize_with`] and [`VecDeque::resize_with`]
- [`Duration::as_millis`], [`Duration::as_micros`], and [`Duration::as_nanos`]

[`unix::FileExt::read_exact_at`]: https://doc.rust-lang.org/std/os/unix/fs/trait.FileExt.html#method.read_exact_at
[`unix::FileExt::write_all_at`]: https://doc.rust-lang.org/std/os/unix/fs/trait.FileExt.html#method.write_all_at
[`Option::transpose`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.transpose
[`Result::transpose`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.transpose
[`convert::identity`]: https://doc.rust-lang.org/std/convert/fn.identity.html
[`pin::Pin`]: https://doc.rust-lang.org/std/pin/struct.Pin.html
[`marker::Unpin`]: https://doc.rust-lang.org/stable/std/marker/trait.Unpin.html
[`marker::PhantomPinned`]: https://doc.rust-lang.org/nightly/std/marker/struct.PhantomPinned.html
[`Vec::resize_with`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.resize_with
[`VecDeque::resize_with`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.resize_with
[`Duration::as_millis`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.as_millis
[`Duration::as_micros`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.as_micros
[`Duration::as_nanos`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.as_nanos

See the [detailed release notes][notes] for more details.

### Cargo features

[Cargo should now rebuild a crate if a file was modified during the initial
build.](https://github.com/rust-lang/cargo/pull/6484/)

See the [detailed release notes][notes] for more.

### Crates.io

[As previously announced][urlo-ann], coinciding with this release, crates.io
will require that you have a verified email address to publish. Starting at
2019-03-01 00:00 UTC, if you don't have a verified email address and run `cargo
publish`, you'll get an error.

This ensures we can comply with DMCA procedures. If you haven't heeded the
warnings cargo printed during the last release cycle, head on over to
[crates.io/me][me] to set and verify your email address. This email address
will never be displayed publicly and will only be used for crates.io operations.

[urlo-ann]: https://users.rust-lang.org/t/a-verified-email-address-will-be-required-to-publish-to-crates-io-starting-on-2019-02-28/22425
[me]: https://crates.io/me

## Contributors to 1.33.0

Many people came together to create Rust 1.33.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.33.0)
