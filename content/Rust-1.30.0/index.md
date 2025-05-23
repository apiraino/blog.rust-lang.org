+++
path = "2018/10/25/Rust-1.30.0"
title = "Announcing Rust 1.30"
authors = ["The Rust Core Team"]
aliases = [
    "2018/10/25/Rust-1.30.0.html",
    "releases/1.30.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.30.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.30.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.30.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1300-2018-10-25

## What's in 1.30.0 stable

Rust 1.30 is an exciting release with a number of features. On Monday, expect another
blog post asking you to check out Rust 1.31's beta; Rust 1.31 will be the first release
of "Rust 2018." For more on that concept, please see our previous post
["What is Rust 2018"](https://blog.rust-lang.org/2018/07/27/what-is-rust-2018.html).

## Procedural Macros

Way back in [Rust 1.15], we announced the ability to define "custom derives." For example,
with `serde_derive`, you could

```rust
#[derive(Serialize, Deserialize, Debug)]
struct Pet {
    name: String,
}
```

And convert a `Pet` to and from JSON using `serde_json` because `serde_derive`
defined `Serialize` and `Deserialize` in a procedural macro.

Rust 1.30 expands on this by adding the ability to define two other kinds of
advanced macros, "attribute-like procedural macros" and "function-like
procedural macros."

Attribute-like macros are similar to custom derive macros, but instead of generating code
for only the `#[derive]` attribute, they allow you to create new, custom attributes of
your own. They're also more flexible: derive only works for structs and enums, but
attributes can go on other places, like functions. As an example of using an
attribute-like macro, you might have something like this when using a web application
framework:

```
#[route(GET, "/")]
fn index() {
```

This `#[route]` attribute would be defined by the framework itself, as a
procedural macro. Its signature would look like this:

```
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Here, we have two input `TokenStreams`: the first is for the contents of the
attribute itself, that is, the `GET, "/"` stuff. The second is the body of the
thing the attribute is attached to, in this case, `fn index() {}` and the rest
of the function's body.

Function-like macros define macros that look like function calls. For
example, the [`gnome-class` crate](https://gitlab.gnome.org/federico/gnome-class)
has a procedural macro that defines `GObject` classes in Rust:

```rust
gobject_gen!(
    class MyClass: GObject {
        foo: Cell<i32>,
        bar: RefCell<String>,
    }

    impl MyClass {
        virtual fn my_virtual_method(&self, x: i32) {
            // ... do something with x ...
        }
    }
)
```

This looks like a function that is taking a bunch of code as an argument.
This macro would be defined like this:

```
#[proc_macro]
pub fn gobject_gen(input: TokenStream) -> TokenStream {
```

This is similar to the derive macro's signature: we get the tokens that
are inside of the parentheses and return the code we want to generate.

### `use` and macros

You can now [bring macros into scope with the `use` keyword][externmacro]. For example,
to use `serde-json`'s `json` macro, you used to write:

```rust
#[macro_use]
extern crate serde_json;

let john = json!({
    "name": "John Doe",
    "age": 43,
    "phones": [
        "+44 1234567",
        "+44 2345678"
    ]
});
```

But now, you'd write

```rust
extern crate serde_json;

use serde_json::json;

let john = json!({
    "name": "John Doe",
    "age": 43,
    "phones": [
        "+44 1234567",
        "+44 2345678"
    ]
});
```

This brings macros more in line with other items and removes the need for
`macro_use` annotations.

[externmacro]: https://github.com/rust-lang/rust/pull/50911/
[Rust 1.15]: https://blog.rust-lang.org/2017/02/02/Rust-1.15.html

Finally, the [`proc_macro` crate](https://doc.rust-lang.org/stable/proc_macro/)
is made stable, which gives you the needed APIs to write these sorts of macros.
It also has significantly improved the APIs for errors, and crates like `syn` and
`quote` are already using them. For example, before:

```rust
#[derive(Serialize)]
struct Demo {
    ok: String,
    bad: std::thread::Thread,
}
```

used to give this error:

```
error[E0277]: the trait bound `std::thread::Thread: _IMPL_SERIALIZE_FOR_Demo::_serde::Serialize` is not satisfied
 --> src/main.rs:3:10
  |
3 | #[derive(Serialize)]
  |          ^^^^^^^^^ the trait `_IMPL_SERIALIZE_FOR_Demo::_serde::Serialize` is not implemented for `std::thread::Thread`
```

Now it will give this one:

```
error[E0277]: the trait bound `std::thread::Thread: serde::Serialize` is not satisfied
 --> src/main.rs:7:5
  |
7 |     bad: std::thread::Thread,
  |     ^^^ the trait `serde::Serialize` is not implemented for `std::thread::Thread`
```

## Module system improvements

The module system has long been a pain point of new Rustaceans; several of
its rules felt awkward in practice. These changes are the first steps we're
taking to make the module system feel more straightforward.

There's two changes to `use` in addition to the aforementioned change for
macros. The first is that [external crates are now in the
prelude][nocoloncolon], that is:

```rust
// old
let json = ::serde_json::from_str("...");

// new
let json = serde_json::from_str("...");
```

The trick here is that the 'old' style wasn't always needed, due to the way Rust's
module system worked:

```rust
extern crate serde_json;

fn main() {
    // this works just fine; we're in the crate root, so `serde_json` is in
    // scope here
    let json = serde_json::from_str("...");
}

mod foo {
    fn bar() {
        // this doesn't work; we're inside the `foo` namespace, and `serde_json`
        // isn't declared there
        let json = serde_json::from_str("...");

    }

    // one option is to `use` it inside the module
    use serde_json;

    fn baz() {
        // the other option is to use `::serde_json`, so we're using an absolute path
        // rather than a relative one
        let json = ::serde_json::from_str("...");
    }
}
```

Moving a function to a submodule and having some of your code break was not a great
experience. Now, it will check the first part of the path and see if it's an `extern
crate`, and if it is, use it regardless of where you're at in the module hierarchy.

[nocoloncolon]: https://github.com/rust-lang/rust/pull/54404/

Finally, [`use` also supports bringing items into scope with paths starting with
`crate`][usecrate]:

```rust
mod foo {
    pub fn bar() {
        // ...
    }
}

// old
use ::foo::bar;
// or
use foo::bar;

// new
use crate::foo::bar;
```

The `crate` keyword at the start of the path indicates that you would like the path to
start at your crate root. Previously, paths specified after `use` would always start at
the crate root, but paths referring to items directly would start at the local path,
meaning the behavior of paths was inconsistent:

```rust
mod foo {
    pub fn bar() {
        // ...
    }
}

mod baz {
    pub fn qux() {
        // old
        ::foo::bar();
        // does not work, which is different than with `use`:
        // foo::bar();

        // new
        crate::foo::bar();
    }
}
```

Once this style becomes widely used, this will hopefully make absolute paths a bit more
clear and remove some of the ugliness of leading `::`.

All of these changes combined lead to a more straightforward understanding of how paths
resolve. Wherever you see a path like `a::b::c` someplace other than a `use` statement,
you can ask:

* Is `a` the name of a crate? Then we're looking for `b::c` inside of it.
* Is `a` the keyword `crate`? Then we're looking for `b::c` from the root of our crate.
* Otherwise, we're looking for `a::b::c` from the current spot in the module hierarchy.

The old behavior of `use` paths always starting from the crate root still applies. But
after making a one-time switch to the new style, these rules will apply uniformly to
paths everywhere, and you'll need to tweak your imports much less when moving code around.

[usecrate]: https://github.com/rust-lang/rust/pull/54404/

## Raw Identifiers

[You can now use keywords as identifiers][rawidents] with some new syntax:

```rust
// define a local variable named `for`
let r#for = true;

// define a function named `for`
fn r#for() {
    // ...
}

// call that function
r#for();
```

This doesn't have many use cases today, but will once you are trying to use a Rust 2015
crate with a Rust 2018 project and vice-versa because the set of keywords will be
different in the two editions; we'll explain more in the upcoming blog post about
Rust 2018.

[rawidents]: https://github.com/rust-lang/rust/pull/53236/

## `no_std` applications

Back in Rust 1.6, we announced the [stabilization of `no_std` and
`libcore`](https://blog.rust-lang.org/2016/01/21/Rust-1.6.html) for building
projects without the standard library. There was a twist, though: you could
only build libraries, but not applications.

With Rust 1.30, you can [use the `#[panic_handler]`][panichandler] attribute
to implement panics yourself. This now means that you can build applications,
not just libraries, that don't use the standard library.

[panichandler]: https://github.com/rust-lang/rust/pull/51366/
## Other things

Finally, you can now [match on visibility keywords, like `pub`, in
macros][viskeyword] using the `vis` specifier. Additionally, "tool
attributes" like `#[rustfmt::skip]` [are now
stable](https://github.com/rust-lang/rust/pull/53459/). Tool *lints*
like `#[allow(clippy::something)]` are not yet stable, however.

[viskeyword]: https://github.com/rust-lang/rust/pull/53370/

See the [detailed release notes][notes] for more.

### Library stabilizations

A few new APIs were [stabilized for this
release](https://github.com/rust-lang/rust/blob/master/RELEASES.md#stabilized-apis):

* `Ipv4Addr::{BROADCAST, LOCALHOST, UNSPECIFIED}`
* `Ipv6Addr::{LOCALHOST, UNSPECIFIED}`
* `Iterator::find_map`

Additionally, the standard library has long had functions like `trim_left` to eliminate
whitespace on one side of some text. However, when considering right-to-left (RTL)
languages, the meaning of "right" and "left" gets confusing. As such, we're introducing
new names for these APIs:

* `trim_left` -> `trim_start`
* `trim_right` -> `trim_end`
* `trim_left_matches` -> `trim_start_matches`
* `trim_right_matches` -> `trim_end_matches`

We plan to deprecate (but not remove, of course) the old names in Rust 1.33.

See the [detailed release notes][notes] for more.

### Cargo features

The largest feature of Cargo in this release is that we now [have a progress
bar!](https://github.com/rust-lang/cargo/pull/5995/)

![demo gif](demo.gif)

See the [detailed release notes][notes] for more.

## Contributors to 1.30.0

Many people came together to create Rust 1.30. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.30.0)
