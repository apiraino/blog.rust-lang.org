+++
path = "2017/02/02/Rust-1.15"
title = "Announcing Rust 1.15"
authors = ["The Rust Core Team"]
aliases = [
    "2017/02/02/Rust-1.15.html",
    "releases/1.15.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.15.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed, getting Rust 1.15 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.15.0][notes] on GitHub. 1443 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1150-2017-02-02

### What's in 1.15.0 stable

Rust 1.15 sees an _extremely_ eagerly-awaited feature land on stable: custom
derive! To review, in Rust, you've always been able to automatically implement
some traits through the `derive` attribute:

```rust
#[derive(Debug)]
struct Pet {
    name: String,
}
```

The `Debug` trait is then implemented for `Pet`, with vastly less boilerplate.
However, this only worked for traits provided as part of the standard library;
it was not customizable. With Rust 1.15, it now is. That means, if you want to
turn your `Pet` into JSON, it's as easy as adding [Serde][serde] to your
`Cargo.toml`:

```toml
[dependencies]
serde = "0.9"
serde_derive = "0.9"
serde_json = "0.9"
```

[serde]: https://serde.rs

And adding another trait to your `Pet`:

```rust
#[macro_use]
extern crate serde_derive;

extern crate serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct Pet {
    name: String,
}

fn main() {
    let pet = Pet { name: String::from("Ferris") };

    let serialized = serde_json::to_string(&pet).unwrap();
    println!("serialized = {}", serialized);

    let deserialized: Pet = serde_json::from_str(&serialized).unwrap();
    println!("deserialized = {:?}", deserialized);
}
```

This will output:

```
serialized = {"name":"Ferris"}
deserialized = Pet { name: "Ferris" }
```

Another common use-case is [Diesel][diesel]. Say we had a database of `Pet`s.
We could fetch them like this:

```rust
// some extern crate and use lines elided here

#[derive(Queryable)]
struct Pet {
    name: String,
}

fn main() {
    use diesel_demo::schema::pets::dsl::*;

    let connection = establish_connection();
    let results = pets
        .limit(5)
        .load::<Pet>(&connection)
        .expect("Error loading pets");

    println!("Displaying {} pets", results.len());
    for pet in results {
        println!("{}", pet.name);
    }
}
```

For full instructions, see [the website][diesel].

[diesel]: https://diesel.rs

These kinds of libraries are extremely powerful, but rely on custom derive for
ergonomics. While these libraries _worked_ on Rust stable previously, they were
not as nice to use, so much so that we often heard from users "I only use
nightly because of Serde and Diesel." The use of custom derive is one of the
most widely used nightly-only features. As such, [RFC 1681] was opened in July
of last year to support this use-case. The RFC was merged in August, underwent
a lot of development and testing, and now reaches stable today!

[RFC 1681]: https://github.com/rust-lang/rfcs/pull/1681

To find out how to write your own custom derives, see [the chapter of "The Rust
Programming Language"](https://doc.rust-lang.org/book/procedural-macros.html).

While we've said "Serde and Diesel" a number of times here, there's a lot of
other cool things you can do with custom derive: see
[`derive-new`](https://crates.io/crates/derive-new) for another example. See
[the `syn` crate's reverse dependencies for more.][syn-deps] (`syn` is
important for writing custom derives, see the book chapter, linked above, for
more.) Custom derive was also known as "macros 1.1", as it includes the
infrastructure for supporting even more compile-time powers of Rust, nicknamed
"macros 2.0." Expect to hear more about this space in future releases.

[syn-deps]: https://crates.io/crates/syn/reverse_dependencies

#### Other improvements

The build system for Rust [has been re-written in Rust, using
Cargo][rustbuild]. It is now the default. This process has been long, but has
finally borne fruit. Given that all Rust development happens on the master
branch, we've been using it since December of last year, and it's working well.
There is an open PR [to remove the Makefiles entirely][rustbuild-only], landing
in Rust 1.17. This will pave the way for `rustc` to use packages from
`crates.io` in the compiler like any other Rust project, and is a further
demonstration of the maturity of Cargo.

[rustbuild]: https://github.com/rust-lang/rust/pull/37817
[rustbuild-only]: https://github.com/rust-lang/rust/pull/39431

Rust has gained [Tier 3 support][tiers] for [`i686-unknown-openbsd`], [`MSP430`],
and [`ARMv5TE`].

[tiers]: https://forge.rust-lang.org/platform-support.html
[`i686-unknown-openbsd`]: https://github.com/rust-lang/rust/pull/38086
[`MSP430`]: https://github.com/rust-lang/rust/pull/37672
[`ARMv5TE`]: https://github.com/rust-lang/rust/pull/37615

[A number of compiler performance improvements have
landed](https://github.com/rust-lang/rust/blob/master/RELEASES.md#compiler-performance).
We continue to work on making the compiler faster. Expect to see more in the
future!

As a smaller improvement, [`?Sized` can now be used in `where`
clauses](https://github.com/rust-lang/rust/pull/37791). In other words:

```rust
struct Foo<T: ?Sized> {
    f: T,
}

struct Foo<T> where T: ?Sized {
    f: T,
}
```

This second form is now accepted, and is equivalent to the first.


See the [detailed release notes][notes] for more.

#### Library stabilizations

The `slice::sort` algorithm [has been rewritten][38192], and is much, much,
much faster. It is a hybrid merge sort, drawing influences from Timsort.
Previously it was a straightforward merge sort.

If you had a `Vec<T>` where `T: Copy`, and you called `extend` on it,
your code will [now be a lot faster][38182].

Speaking of things getting faster, [`chars().count()`][37888],
[`chars().last()`, and `char_indices().last()`][37882] are too!

[Chinese characters now display correctly in `fmt::Debug`][37855].

[38192]: https://github.com/rust-lang/rust/pull/38192
[38182]: https://github.com/rust-lang/rust/pull/38182
[37888]: https://github.com/rust-lang/rust/pull/37888
[37882]: https://github.com/rust-lang/rust/pull/37882
[37855]: https://github.com/rust-lang/rust/pull/37855

There were a number of functions stabilized as well:

* [`std::iter::Iterator::min_by`] and [`std::iter::Iterator::max_by`]
* [`std::os::*::fs::FileExt`]
* [`std::sync::atomic::Atomic*::get_mut`] and [`std::sync::atomic::Atomic*::into_inner`]
* [`std::vec::IntoIter::as_slice`] and [`std::vec::IntoIter::as_mut_slice`]
* [`std::sync::mpsc::Receiver::try_iter`]
* [`std::os::unix::process::CommandExt::before_exec`]
* [`std::rc::Rc::strong_count`] and [`std::rc::Rc::weak_count`]
* [`std::sync::Arc::strong_count`] and [`std::sync::Arc::weak_count`]
* [`std::char::encode_utf8`] and [`std::char::encode_utf16`]
* [`std::cell::Ref::clone`]
* [`std::io::Take::into_inner`]

[`std::iter::Iterator::min_by`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.min_by
[`std::iter::Iterator::max_by`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.max_by
[`std::os::*::fs::FileExt`]: https://doc.rust-lang.org/std/os/unix/fs/trait.FileExt.html
[`std::sync::atomic::Atomic*::get_mut`]: https://doc.rust-lang.org/std/sync/atomic/struct.AtomicU8.html#method.get_mut
[`std::sync::atomic::Atomic*::into_inner`]: https://doc.rust-lang.org/std/sync/atomic/struct.AtomicU8.html#method.into_inner
[`std::vec::IntoIter::as_slice`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html#method.as_slice
[`std::vec::IntoIter::as_mut_slice`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html#method.as_mut_slice
[`std::sync::mpsc::Receiver::try_iter`]: https://doc.rust-lang.org/std/sync/mpsc/struct.Receiver.html#method.try_iter
[`std::os::unix::process::CommandExt::before_exec`]: https://doc.rust-lang.org/std/os/unix/process/trait.CommandExt.html#tymethod.before_exec
[`std::rc::Rc::strong_count`]: https://doc.rust-lang.org/std/rc/struct.Rc.html#method.strong_count
[`std::rc::Rc::weak_count`]: https://doc.rust-lang.org/std/rc/struct.Rc.html#method.weak_count
[`std::sync::Arc::strong_count`]: https://doc.rust-lang.org/std/sync/struct.Arc.html#method.strong_count
[`std::sync::Arc::weak_count`]: https://doc.rust-lang.org/std/sync/struct.Arc.html#method.weak_count
[`std::char::encode_utf8`]: https://doc.rust-lang.org/std/primitive.char.html#method.encode_utf8
[`std::char::encode_utf16`]: https://doc.rust-lang.org/std/primitive.char.html#method.encode_utf16
[`std::cell::Ref::clone`]: https://doc.rust-lang.org/std/cell/struct.Ref.html#method.clone
[`std::io::Take::into_inner`]: https://doc.rust-lang.org/std/io/struct.Take.html#method.into_inner

See the [detailed release notes][notes] for more.

#### Cargo features

Cargo will now [emit a warning][cargo/3361] if you have a file named `build.rs`
at the top level of a package, but don't have a `build = "build.rs"`
annotation. This is in anticipation of inferring that `build.rs` at the top
level is always a build script, but is a warning right now for compatibility
reasons. Previously, all build scripts required configuration, but this
convention was strong within the community, so we're going to encode it into
Cargo.

[cargo/3361]: https://github.com/rust-lang/cargo/pull/3361

In this release, [Cargo build scripts no longer have access to the `OUT_DIR`
environment variable at build time via `env!("OUT_DIR")`][cargo/3368]. They
should instead check the variable at runtime with `std::env`. That the value
was set at build time was a bug, and incorrect when cross-compiling. Please
check what your packages are doing and update to use `std::env`!

[cargo/3368]:  https://github.com/rust-lang/cargo/pull/3368

The `cargo test` command has [gained support for a `--all` flag][cargo/3221],
which is useful when you have a workspace.

[cargo/3221]: https://github.com/rust-lang/cargo/pull/3221

We now [Compile statically against the MSVC CRT][cargo/3363] on Windows, and
[Link OpenSSL statically][cargo/3311] on Mac OS X.

[cargo/3363]: https://github.com/rust-lang/cargo/pull/3363
[cargo/3311]: https://github.com/rust-lang/cargo/pull/3311

See the [detailed release notes][notes] for more.

### Contributors to 1.15.0

In this part of the release announcements, we usually post a list of
contributors. However, we've recently started a new initiative, "Thanks!", to
do this in a more comprehensive way. One issue with this section is that it
only counted contributions to the `rust-lang/rust` repository; those who
committed to Cargo weren't thanked, for example. We also had to manually
generate this list, which wasn't terrible, but running the correct git commands
to determine who contributed is exactly what code is good for!

As such, you can now visit
[https://thanks.rust-lang.org/](https://thanks.rust-lang.org/) to see more
comprehensive contribution calculations. If you prefer, we also have an alias
at [https://❤.rust-lang.org](https://❤.rust-lang.org) as well. For now, this
will only show what we've shown in previous release posts. We do have one
additional feature, which is an all-time contributions list, sorted by commit
count. That's located here:
[https://thanks.rust-lang.org/rust/all-time](https://thanks.rust-lang.org/rust/all-time)

We have done some of the needed backend work to enable more repositories than
only `rust-lang/rust`, but it's not quite done yet. If you'd like to get
involved, please [check out thanks on
GitHub](https://github.com/rust-lang-nursery/thanks)!

We had 137 individuals contribute to Rust 1.15.
[Thanks!](https://thanks.rust-lang.org/rust/1.15.0)
