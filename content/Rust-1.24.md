+++
path = "2018/02/15/Rust-1.24"
title = "Announcing Rust 1.24"
authors = ["The Rust Core Team"]
aliases = [
    "2018/02/15/Rust-1.24.html",
    "releases/1.24.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.24.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.24.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.24.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1240-2018-02-15

## What's in 1.24.0 stable

This release contains two very exciting new features: `rustfmt` and incremental compilation!

#### rustfmt

For years now, we've wanted a tool that automatically can reformat your Rust code to some sort
of "standard style." With this release, we're happy to announce that a *preview* of `rustfmt`
can be used with 1.24 stable. To give it a try, do this:

```bash
$ rustup component add rustfmt-preview
```

There are two important aspects here: first, you're using `rustup component
add` instead of `cargo install` here. If you've previously used `rustfmt` via
`cargo install`, you should uninstall it first. Second, this is a preview, as
it says in the name. `rustfmt` is not at 1.0 yet, and some stuff is being
tweaked, and bugs are being fixed. Once `rustfmt` hits 1.0, we'll be
releasing a `rustfmt` component and deprecating `rustfmt-preview`.

In the near future, we plan on writing a post about this release strategy, as it's big
enough for its own post, and is broader than just this release.

For more, please check out [`rustfmt` on GitHub](https://github.com/rust-lang-nursery/rustfmt).

#### Incremental compilation

Back in September of 2016 (!!!), we blogged about [Incremental Compilation](https://blog.rust-lang.org/2016/09/08/incremental.html).
While that post goes into the details, the idea is basically this: when you're working on
a project, you often compile it, then change something small, then compile again. Historically,
the compiler has compiled your *entire* project, no matter how little you've changed the code.
The idea with incremental compilation is that you only need to compile the code you've actually
changed, which means that that second build is faster.

As of Rust 1.24, this is now [turned on by default](https://github.com/rust-lang/cargo/pull/4817).
This means that your builds should get faster! Don't forget about `cargo check` when trying
to get the lowest possible build times.

This is still not the end story for compiler performance generally, nor incremental compilation
specifically. We have a lot more work planned in the future. For example, another change
related to performance hit stable this release:
[`codegen-units` is now set to 16 by default](https://github.com/rust-lang/rust/pull/46910).
One small note about this change: it makes builds faster, but makes the final binary a bit
slower. For maximum speed, setting `codegen-units` to `1` in your `Cargo.toml` is needed
to eke out every last drop of performance.

More to come!

#### Other good stuff

There's one other change we'd like to talk about here: undefined behavior. Rust generally
strives to minimize undefined behavior, having none of it in safe code, and as little as
possible in unsafe code. One area where you could invoke UB is when a `panic!` goes
across an FFI boundary. In other words, this:

```rust
extern "C" fn panic_in_ffi() {
    panic!("Test");
}
```

This cannot work, as the exact mechanism of how panics work would have to be reconciled
with how the `"C"` ABI works, in this example, or any other ABI in other examples.

In Rust 1.24, [this code will now abort](https://github.com/rust-lang/rust/pull/46833)
instead of producing undefined behavior.

See the [detailed release notes][notes] for more.

### Library stabilizations

If you're a fan of `str::find`, which is used to find a given `char` inside of a `&str`, you'll be
happy to see this pull request: [it's now 10x faster](https://github.com/rust-lang/rust/pull/46735)!
This is thanks to `memchr`. `[u8]::contains` [uses it too](https://github.com/rust-lang/rust/pull/46713),
though it doesn't get such an extreme speedup.

Additionally, a few new APIs were stabilized this release:

* [`RefCell::replace`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.replace)
* [`RefCell::swap`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.swap)
* [`std::sync::atomic::spin_loop_hint`](https://doc.rust-lang.org/std/sync/atomic/fn.spin_loop_hint.html)

Finally, these functions may now be used inside a constant expression, for example, to initialize a `static`:

* `Cell`, `RefCell`, and `UnsafeCell`'s `new` functions
* The `new` functions of the various `Atomic` integer types
* `{integer}::min_value` and `max_value`
* `mem`'s `size_of` and `align_of`
* `ptr::null` and `null_mut`

See the [detailed release notes][notes] for more.

### Cargo features

The big feature of this release was turning on incremental compilation by default, as mentioned above.

See the [detailed release notes][notes] for more.

## Contributors to 1.24.0

Many people came together to create Rust 1.24. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.24.0)
