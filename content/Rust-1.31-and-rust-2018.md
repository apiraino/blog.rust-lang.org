+++
path = "2018/12/06/Rust-1.31-and-rust-2018"
title = "Announcing Rust 1.31 and Rust 2018"
authors = ["The Rust Core Team"]
aliases = [
    "2018/12/06/Rust-1.31-and-rust-2018.html",
    "releases/1.31.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.31.0, and "Rust
2018" as well. Rust is a programming language that empowers everyone to build
reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.31.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.31.0][notes] on GitHub.

[install]: https://www.rust-lang.org/tools/install
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1310-2018-12-06

## What's in 1.31.0 stable

Rust 1.31 may be the most exciting release since Rust 1.0! Included in this release is the
first iteration of "Rust 2018," but there's more than just that! This is going to be a long
post, so here's a table of contents:

* [Rust 2018](#rust-2018)
  * [Non-lexical lifetimes](#non-lexical-lifetimes)
  * [Module system changes](#module-system-changes)
* [More lifetime elision rules](#more-lifetime-elision-rules)
* [`const fn`](#const-fn)
* [New tools](#new-tools)
* [Tool Lints](#tool-lints)
* [Documentation](#documentation)
* [Domain working groups](#domain-working-groups)
* [New website](#new-website)
* [Library stabilizations](#library-stabilizations)
* [Cargo features](#cargo-features)
* [Contributors](#contributors-to-1-31-0)

### Rust 2018

We wrote about Rust 2018 [first in
March](https://blog.rust-lang.org/2018/03/12/roadmap.html), [and then in
July](https://blog.rust-lang.org/2018/07/27/what-is-rust-2018.html).
For some more background about the *why* of Rust 2018, please go read those
posts; there's a lot to cover in the release announcement, and so we're going
to focus on the *what* here. There's also a [post on Mozilla Hacks][hacks] as
well!

[hacks]: https://hacks.mozilla.org/2018/12/rust-2018-is-here/

Briefly, Rust 2018 is an opportunity to bring
all of the work we've been doing over the past three years together, and create
a cohesive package. This is more than just language features, it also includes

* Tooling (IDE support, `rustfmt`, Clippy)
* Documentation
* Domain working groups work
* A new web site

We'll be covering all of this and more in this post.

Let's create a new project with Cargo:

```
$ cargo new foo
```

Here's the contents of `Cargo.toml`:

```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
```

A new key has been added under `[package]`: `edition`. Note that it has been
set to `2018`. You can also set it to `2015`, which is the default if the key
does not exist.

By using Rust 2018, some new features are unlocked that are not allowed in
Rust 2015.

It is important to note that each package can be in either 2015 or
2018 mode, and they work seamlessly together. Your 2018 project can use 2015
dependencies, and a 2015 project can use 2018 dependencies. This ensures that
we don't split the ecosystem, and all of these new things are opt-in,
preserving compatibility for existing code. Furthermore, when you do choose
to migrate Rust 2015 code to Rust 2018, the changes can be made
automatically, via `cargo fix`.

What kind of new features, you may ask? Well, first, features get added to
Rust 2015 unless they require some sort of incompatibility with 2015's
features. As such, most of the language is available everywhere. You can
check out [the edition
guide](https://doc.rust-lang.org/edition-guide) to check each
feature's minimum `rustc` version as well as edition requirements. However,
there are a few big-ticket features we'd like to mention here: non-lexical
lifetimes, and some module system improvements.

#### Non-lexical lifetimes

If you've been following Rust's development over the past few years, you may
have heard the term "NLL" or "non-lexical lifetimes" thrown around. This is
jargon, but it has a straightforward translation into simpler terms: the
borrow checker has gotten smarter, and now accepts some valid code that it
previously rejected. Consider this example:

```rust
fn main() {
    let mut x = 5;

    let y = &x;

    let z = &mut x;
}
```

In older Rust, this is a compile-time error:

```
error[E0502]: cannot borrow `x` as mutable because it is also borrowed as immutable
 --> src/main.rs:5:18
  |
4 |     let y = &x;
  |              - immutable borrow occurs here
5 |     let z = &mut x;
  |                  ^ mutable borrow occurs here
6 | }
  | - immutable borrow ends here
```

This is because lifetimes follow "lexical scope"; that is, the borrow from `y`
is considered to be held until `y` goes out of scope at the end of main, even
though we never use `y` again. This code is fine, but the borrow checker could
not handle it.

Today, this code will compile just fine.

What if we did use `y`, like this for example:

```rust
fn main() {
    let mut x = 5;
    let y = &x;
    let z = &mut x;

    println!("y: {}", y);
}
```

Older Rust will give you this error:

```
error[E0502]: cannot borrow `x` as mutable because it is also borrowed as immutable
 --> src/main.rs:5:18
  |
4 |     let y = &x;
  |              - immutable borrow occurs here
5 |     let z = &mut x;
  |                  ^ mutable borrow occurs here
...
8 | }
  | - immutable borrow ends here
```

With Rust 2018, this error changes for the better:

```
error[E0502]: cannot borrow `x` as mutable because it is also borrowed as immutable
 --> src/main.rs:5:13
  |
4 |     let y = &x;
  |             -- immutable borrow occurs here
5 |     let z = &mut x;
  |             ^^^^^^ mutable borrow occurs here
6 |
7 |     println!("y: {}", y);
  |                       - borrow later used here
```

Instead of pointing to where `y` goes out of scope, it shows you where the
conflicting borrow occurs. This makes these sorts of errors far easier to
debug.

In Rust 1.31, this feature is exclusive to Rust 2018. We plan to backport it
to Rust 2015 at a later date.

#### Module system changes

The module system can be a struggle for people first learning Rust.
Everyone has their own things that take time to master, of course, but
there's a root cause for why it's so confusing to many: while there are
simple and consistent rules defining the module system, their consequences
can feel inconsistent, counterintuitive and mysterious.

As such, the 2018 edition of Rust introduces a few changes to how paths work,
but they end up simplifying the module system, to make it more clear as to
what is going on.

Here's a brief summary:

* `extern crate` is no longer needed in almost all circumstances.
* You can import macros with `use`, rather than a `#[macro_use]` attribute.
* Absolute paths begin with a crate name, where the keyword `crate` refers to the current crate.
* A `foo.rs` and `foo/` subdirectory may coexist; `mod.rs` is no longer needed when placing submodules in a subdirectory.

These may seem like arbitrary new rules when put this way, but the mental
model is now significantly simplified overall.

There's a *lot* of details here, so please read [the edition
guide](https://doc.rust-lang.org/edition-guide/rust-2018/module-system/path-clarity.html)
for full details.

### More lifetime elision rules

Let's talk about a feature that's available in both editions: we've added
some additional elision rules for `impl` blocks and function definitions.
Code like this:

```rust
impl<'a> Reader for BufReader<'a> {
    // methods go here
}
```

can now be written like this:

```rust
impl Reader for BufReader<'_> {
    // methods go here
}
```

The `'_` lifetime still shows that `BufReader` takes a parameter, but we
don't need to create a name for it anymore.

Lifetimes are still required to be defined in structs. However, we no longer
require as much boilerplate as before:

```rust
// Rust 2015
struct Ref<'a, T: 'a> {
    field: &'a T
}

// Rust 2018
struct Ref<'a, T> {
    field: &'a T
}
```

The `: 'a` is inferred. You can still be explicit if you prefer. We're
considering some more options for elision here in the future, but have no
concrete plans yet.

### `const fn`

There's several ways to define a function in Rust: a regular function with
`fn`, an unsafe function with `unsafe fn`, an external function with `extern fn`.
This release adds a new way to qualify a function: `const fn`. It looks like
this:

```rust
const fn foo(x: i32) -> i32 {
    x + 1
}
```

A `const fn` can be called like a regular function, but it can also be used
in any constant context. When it is, it is evaluated at compile time, rather
than at run time. As an example:

```rust
const SIX: i32 = foo(5);
```

This will execute `foo` at compile time, and set `SIX` to `6`.

`const fn`s cannot do everything that normal `fn`s can do; they must
have deterministic output. This is important for soundness reasons.
Currently, `const fn`s can do a minimal subset of operations. Here's
some examples of what you can do:

* Arithmetic and comparison operators on integers
* All boolean operators except for `&&` and `||`
* Constructing arrays, structs, enums, and tuples
* Calls to other `const fn`s
* Index expressions on arrays and slices
* Field accesses on structs and tuples
* Reading from constants (but not statics, not even taking a reference to a static)
* `&` and `*` of references
* Casts, except for raw pointer to integer casts

We'll be growing the abilities of `const fn`, but we've decided that
this is enough useful stuff to start shipping the feature itself.

For full details, please see [the
reference](https://doc.rust-lang.org/reference/items/functions.html#const-functions).

### New tools

The 2018 edition signals a new level of maturity for Rust's tools ecosystem.
Cargo, Rustdoc, and Rustup have been crucial tools since 1.0; with the 2018
edition, there is a new generation of tools ready for all users: Clippy,
Rustfmt, and IDE support.

Rust's linter, [`clippy`](https://github.com/rust-lang/rust-clippy/), is
now available on stable Rust. You can install it via `rustup component add
clippy` and run it with `cargo clippy`. Clippy is now considered 1.0, which
carries the same lint stability guarantees as rustc. New lints may be added,
and lints may be modified to add more functionality, however lints may never
be removed (only deprecated). This means that code that compiles under clippy
will continue to compile under clippy (provided there are no lints set to
error via `deny`), but may throw new warnings.

[Rustfmt](https://github.com/rust-lang/rustfmt) is a tool for formatting Rust
code. Automatically formatting your code lets you save time and arguments by
using the [official Rust
style](https://github.com/rust-lang/rfcs/blob/master/style-guide/README.md).
You can install with `rustup component add rustfmt` and use it with `cargo
fmt`.

This release includes Rustfmt 1.0. From now on we guarantee backwards
compatibility for Rustfmt: if you can format your code today, then the
formatting will not change in the future (only with the default options).
Backwards compatibility means that running Rustfmt on your CI is practical
(use `cargo fmt -- --check`). Try that and 'format on save' in your editor to
revolutionize your workflow.

IDE support is one of the most requested tooling features for Rust. There are
now multiple, high quality options:

* [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust)
* [IntelliJ](https://plugins.jetbrains.com/plugin/8182-rust)
* [Atom](https://github.com/rust-lang-nursery/atom-ide-rust)
* [Sublime Text 3](https://github.com/rust-lang/rust-enhanced)
* [Eclipse](https://www.eclipse.org/downloads/packages/release/photon/r/eclipse-ide-rust-developers-includes-incubating-components)

Work on IDE support is not finished, in particular code completion is not up
to scratch in the RLS-based editors. However, if you mainly want support for
types, documentation, and 'go to def', etc. then you should be happy.

If you have problems installing any of the tools with Rustup, try running
`rustup self update`, and then try again.

### Tool lints

In [Rust 1.30](https://blog.rust-lang.org/2018/10/25/Rust-1.30.0.html), we
stabilized "tool attributes", like `#[rustfmt::skip]`. In Rust 1.31, we're
stabilizing something similar: "tool lints," like
`#[allow(clippy::bool_comparison)]` These give a namespace to lints, so that it's
more clear which tool they're coming from.

If you previously used Clippy's lints, you can migrate like this:

```rust
// old
#![cfg_attr(feature = "cargo-clippy", allow(bool_comparison))]

// new
#![allow(clippy::bool_comparison)]
```

You don't need `cfg_attr` anymore! You'll also get warnings that can help you
update to the new style.

### Documentation

Rustdoc has seen a number of improvements this year, and we also shipped a
complete re-write of the "The Rust Programming Language." Additionally, you
can [buy a dead-tree copy from No Starch Press](https://nostarch.com/rust)!

We had previously called this the "second edition" of the book, but since
it's the first edition in print, that was confusing. We also want to
periodically update the print edition as well. In the end, after many
discussions with No Starch, we're going to be updating the book on the
website with each release, and No Starch will periodically pull in our
changes and print them. The book has been selling quite well so far, raising
money for [Black Girls Code](http://www.blackgirlscode.com/).

You can find the new TRPL [here](https://doc.rust-lang.org/beta/book/).

### Domain working groups

We announced the formation of four working groups this year:

* Network services
* Command-line applications
* WebAssembly
* Embedded devices

Each of these groups has been working very hard on a number of things to
make Rust awesome in each of these domains. Some highlights:

* Network services has been shaking out the Futures interface, and async/await
  on top of it. This hasn't shipped yet, but we're close!
* The CLI working group has been working on libraries and documentation for making awesome
  command-line applications
* The WebAssembly group has been shipping a ton of world-class tooling for using Rust with wasm.
* Embedded devices has gotten ARM development working on stable Rust!

You can find out more about this work on the new website!

### New Website

[Last
week](https://blog.rust-lang.org/2018/11/29/a-new-look-for-rust-lang-org.html)
we announced a new iteration of the web site. It's now been promoted to
rust-lang.org itself!

There's still a ton of work to do, but we're proud of the year of work that it
took by many people to get it shipped.

### Library stabilizations

A bunch of `From` implementations have been added:

* `u8` now implements `From<NonZeroU8>`, and likewise for the other numeric types and their `NonZero` equivalents
* `Option<&T>` implements `From<&Option<T>>`, and likewise for `&mut`

Additionally, these functions have been stabilized:

* [`slice::align_to`](https://doc.rust-lang.org/std/primitive.slice.html#method.align_to) and its mutable counterpart
* [`slice::chunks_exact`](https://doc.rust-lang.org/std/primitive.slice.html#method.chunks_exact),
  as well as its mutable and `r` counterparts (like
  [`slice::rchunks_exact_mut`](https://doc.rust-lang.org/std/primitive.slice.html#method.rchunks_mut)) in all combinations

See the [detailed release notes][notes] for more.

### Cargo features

Cargo will now download packages in parallel using HTTP/2.

Additionally, now that `extern crate` is not usually required, it would be
jarring to do `extern crate foo as baz;` to rename a crate. As such, you can
do so in your `Cargo.toml`, like this:

```toml
[dependencies]
baz = { version = "0.1", package = "foo" }
```

or, the equivalent

```toml
[dependencies.baz]
version = "0.1"
package = "foo"
```

Now, the `foo` package will be able to be used via `baz` in your code.

See the [detailed release notes][notes] for more.

## Contributors to 1.31.0

At the end of release posts, we normally thank [the people who contributed to
this release](https://thanks.rust-lang.org/rust/1.31.0). But for this
release, more so than others, this list does not truly capture the amount of
work and the number of people who have contributed. Each release is only six
weeks, but this release is the culmination of three years of effort, in
countless repositories, by numerous people. It's been a pleasure to work with
you all, and we look forward to continuing to grow in the next three years.
