+++
path = "2020/11/19/Rust-1.48"
title = "Announcing Rust 1.48.0"
authors = ["The Rust Release Team"]
aliases = ["2020/11/19/Rust-1.48.html"]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.48.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.48.0 is as easy as:

```console
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.48.0][notes] on GitHub.

[install]: https://www.rust-lang.org/tools/install
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1480-2020-11-19

## What's in 1.48.0 stable

The star of this release is Rustdoc, with a few changes to make writing
documentation even easier! See the [detailed release notes][notes] to learn
about other changes not covered by this post.

### Easier linking in rustdoc

Rustdoc, the library documentation tool included in the Rust distribution,
lets you write documentation in Markdown. This makes it very easy to use, but
also has some pain points. Let's say that you are writing some documentation
for some Rust code that looks like this:

```rust
pub mod foo {
    pub struct Foo;
}

pub mod bar {
    pub struct Bar;
}
```

We have two modules, each with a struct inside. Imagine we wanted to use these
two structs together; we may want to note this in the documentation. So we'd
write some docs that look like this:

```rust
pub mod foo {
    /// Some docs for `Foo`
    ///
    /// You may want to use `Foo` with `Bar`.
    pub struct Foo;
}

pub mod bar {
    /// Some docs for `Bar`
    ///
    /// You may want to use `Bar` with `Foo`.
    pub struct Bar;
}
```

That's all well and good, but it would be really nice if we could link to these
other types. That would make it much easier for the users of our library to
navigate between them in our docs.

The problem here is that Markdown doesn't know anything about Rust, and the
URLs that rustdoc generates. So what Rust programmers have had to do is write
those links out manually:

```rust
pub mod foo {
    /// Some docs for `Foo`
    ///
    /// You may want to use `Foo` with [`Bar`].
    ///
    /// [`Bar`]: ../bar/struct.Bar.html
    pub struct Foo;
}

pub mod bar {
    /// Some docs for `Bar`
    ///
    /// You may want to use `Bar` with [`Foo`].
    ///
    /// [`Foo`]: ../foo/struct.Foo.html
    pub struct Bar;
}
```

Note that we've also had to use relative links, so that this works offline.
Not only is this process tedious, and error prone, but it's also just wrong
in places. If we put a `pub use bar::Bar` in our crate root, that would
re-export `Bar` in our root. Now our links are wrong. But if we fix them,
then they end up being wrong when we navigate to the `Bar` that lives inside
the module. You can't actually write these links by hand, and have them all
be accurate.

In this release, you can use some syntax to let rustdoc know that you're
trying to link to a type, and it will generate the URLs for you. Here's
two different examples, based off of our code before:

```rust
pub mod foo {
    /// Some docs for `Foo`
    ///
    /// You may want to use `Foo` with [`Bar`](crate::bar::Bar).
    pub struct Foo;
}

pub mod bar {
    /// Some docs for `Bar`
    ///
    /// You may want to use `Bar` with [`crate::foo::Foo`].
    pub struct Bar;
}
```

The first example will show the same text as before; but generate the proper
link to the `Bar` type. The second will link to `Foo`, but will show the whole
`crate::foo::Foo` as the link text.

There are a bunch of options you can use here. Please see the ["Linking to
items by name"][intra-docs] section of the rustdoc book for more. There is also
a post on Inside Rust [on the history of this feature][intra-history], written
by some of the contributors behind it!

[intra-docs]: https://doc.rust-lang.org/rustdoc/write-documentation/linking-to-items-by-name.html
[intra-history]: https://blog.rust-lang.org/inside-rust/2020/09/17/stabilizing-intra-doc-links.html

### Adding search aliases

[You can now specify `#[doc(alias = "<alias>")]` on items to add search
aliases when searching through `rustdoc`'s UI.][75740] This is a smaller change,
but still useful. It looks like this:

```rust
#[doc(alias = "bar")]
struct Foo;
```

With this annotation, if we search for "bar" in rustdoc's search, `Foo` will
come up as part of the results, even though our search text doesn't have
"Foo" in it.

An interesting use case for aliases is FFI wrapper crates, where each Rust
function could be aliased to the C function it wraps. Existing users of the
underlying C library would then be able to easily search the right Rust
functions!

[75740]: https://github.com/rust-lang/rust/pull/75740/

### Library changes

The most significant API change is kind of a mouthful: `[T; N]: TryFrom<Vec<T>>`
is now stable. What does this mean? Well, you can use this to try and turn
a vector into an array of a given length:

```rust
use std::convert::TryInto;

let v1: Vec<u32> = vec![1, 2, 3];

// This will succeed; our vector has a length of three, we're trying to
// make an array of length three.
let a1: [u32; 3] = v1.try_into().expect("wrong length");

// But if we try to do it with a vector of length five...
let v2: Vec<u32> = vec![1, 2, 3, 4, 5];

// ... this will panic, since we have the wrong length.
let a2: [u32; 3] = v2.try_into().expect("wrong length");
```

In the last release, we talked about the standard library being able to use
const generics. This is a good example of the kinds of APIs that we can add
with these sorts of features. Expect to hear more about the stabilization of
const generics soon.

Additionally, five new APIs were stabilized this release:

- [`slice::as_ptr_range`]
- [`slice::as_mut_ptr_range`]
- [`VecDeque::make_contiguous`]
- [`future::pending`]
- [`future::ready`]

The following previously stable APIs have now been made `const`:

- [`Option::is_some`]
- [`Option::is_none`]
- [`Option::as_ref`]
- [`Result::is_ok`]
- [`Result::is_err`]
- [`Result::as_ref`]
- [`Ordering::reverse`]
- [`Ordering::then`]

See the [detailed release notes][notes] for more.

[`Option::is_some`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.is_some
[`Option::is_none`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.is_none
[`Option::as_ref`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.as_ref
[`Result::is_ok`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.is_ok
[`Result::is_err`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.is_err
[`Result::as_ref`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref
[`Ordering::reverse`]: https://doc.rust-lang.org/std/cmp/enum.Ordering.html#method.reverse
[`Ordering::then`]: https://doc.rust-lang.org/std/cmp/enum.Ordering.html#method.then
[`slice::as_ptr_range`]: https://doc.rust-lang.org/std/primitive.slice.html#method.as_ptr_range
[`slice::as_mut_ptr_range`]: https://doc.rust-lang.org/std/primitive.slice.html#method.as_mut_ptr_range
[`VecDeque::make_contiguous`]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html#method.make_contiguous
[`future::pending`]: https://doc.rust-lang.org/std/future/fn.pending.html
[`future::ready`]: https://doc.rust-lang.org/std/future/fn.ready.html

### Other changes

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-148-2020-11-19
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-148

There are other changes in the Rust 1.48.0 release: check out what changed in
[Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.48.0

Many people came together to create Rust 1.48.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.48.0/)
