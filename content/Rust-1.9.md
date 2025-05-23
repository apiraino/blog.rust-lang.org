+++
path = "2016/05/26/Rust-1.9"
title = "Announcing Rust 1.9"
authors = ["The Rust Core Team"]
aliases = [
    "2016/05/26/Rust-1.9.html",
    "releases/1.9.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.9. Rust is a
systems programming language focused on safety, speed, and concurrency.

As always, you can [install Rust 1.9][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.9][notes] on GitHub.
About 1000 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-190-2016-05-26

### What's in 1.9 stable

#### Controlled unwinding

The biggest shift in Rust 1.9 is the stabilization of the `std::panic` module,
which includes methods for halting the unwinding process started by a panic:

```rust
use std::panic;

let result = panic::catch_unwind(|| {
    println!("hello!");
});
assert!(result.is_ok());

let result = panic::catch_unwind(|| {
    panic!("oh no!");
});
assert!(result.is_err());
```

This new API was defined in [RFC 1236].

[`std::panic`]: https://doc.rust-lang.org/stable/std/panic/index.html
[RFC 1236]: https://github.com/rust-lang/rfcs/pull/1236

In general, Rust distinguishes between two ways that an operation can fail:

- Due to an *expected problem*, like a file not being found.
- Due to an *unexpected problem*, like an index being out of bounds for an array.

Expected problems usually arise from conditions that are outside of your
control; robust code should be prepared for anything its environment might throw
at it. In Rust, expected problems are handled via [the `Result` type][result],
which allows a function to return information about the problem to its caller,
which can then handle the error in a fine-grained way.

[result]: https://static.rust-lang.org/doc/master/std/result/index.html

Unexpected problems are *bugs*: they arise due to a contract or assertion being
violated. Since they are unexpected, it doesn't make sense to handle them in a
fine-grained way. Instead, Rust employs a "fail fast" approach by *panicking*,
which by default unwinds the stack (running destructors but no other code) of
the thread which discovered the error. Other threads continue running, but will
discover the panic any time they try to communicate with the panicked thread
(whether through channels or shared memory). Panics thus abort execution up to
some "isolation boundary", with code on the other side of the boundary still
able to run, and perhaps to "recover" from the panic in some very coarse-grained
way. A server, for example, does not necessarily need to go down just because of
an assertion failure in one of its threads.

The new `catch_unwind` API offers a way to introduce new isolation boundaries
*within a thread*. There are a couple of key motivating examples:

* Embedding Rust in other languages
* Abstractions that manage threads

For the first case, unwinding across a language boundary is undefined behavior,
and often leads to segfaults in practice. Allowing panics to be caught means
that you can safely expose Rust code via a C API, and translate unwinding into
an error on the C side.

For the second case, consider a threadpool library. If a thread in the pool
panics, you generally don't want to kill the thread itself, but rather catch the
panic and communicate it to the client of the pool. The `catch_unwind` API is
paired with `resume_unwind`, which can then be used to restart the panicking
process on the client of the pool, where it belongs.

In both cases, you're introducing a new isolation boundary within a thread, and
then translating the panic into some other form of error elsewhere.

A final point: why `catch_unwind` rather than `catch_panic`? We are
[in the process][abort] of adding an additional strategy for panics: aborting
the entire process (possibly after running a general [hook]). For some
applications, this is the most reasonable way to deal with a programmer error,
and avoiding unwinding can have performance and code size wins.

[hook]: https://github.com/rust-lang/rfcs/pull/1328
[abort]: https://github.com/rust-lang/rfcs/pull/1513

#### Deprecation warnings

We introduced a new attribute for library authors: `#[deprecated]`. This attribute
allows you to tag an API with a deprecation warning, which users of their crate
will receive whenever they use the API, directing them to a replacement API.
Deprecation warnings have long been a part of the standard library, but thanks
to [RFC 1270] they're now usable ecosystem-wide.

[RFC 1270]: https://github.com/rust-lang/rfcs/blob/master/text/1270-deprecation.md

#### New targets

We now publish standard library binaries for several new targets:

- `mips-unknown-linux-musl`,
- `mipsel-unknown-linux-musl`, and
- `i586-pc-windows-msvc`.

The first two targets are particularly interesting from a cross-compilation
perspective; see the [recent blog post on `rustup`][rustup] for more details.

[rustup]: https://blog.rust-lang.org/2016/05/13/rustup.html

#### Compile time improvements

[The time complexity of comparing variables for equivalence][compare] during
type unification is reduced from O(n!) to O(n). As a result, some programming
patterns compile much, much more quickly.

[compare]: https://github.com/rust-lang/rust/pull/32062

#### Rolling out use of specialization

This release sees some of the first use of [specialization] within the standard
library. Specialization, which is currently available only on [nightly], allows
generic code to automatically be specialized based on more specific type
information.

One example where this comes up in the standard library: conversion from a
string slice (`&str`) to an owned `String`. One method, `to_string`, comes from
a generic API which was previously relatively slow, while the custom `to_owned`
implementation provided better performance. Using specialization, these two
functions are [now equivalent].

With this simple test of specialization under our belt, we have more performance
improvements on the way in upcoming releases.

[specialization]: https://github.com/rust-lang/rfcs/pull/1210
[nightly]: https://blog.rust-lang.org/2014/10/30/Stability.html
[now equivalent]: https://github.com/rust-lang/rust/pull/32586

#### Library stabilizations

About 80 library functions and methods are now stable in 1.9.  The most major is
the `std::panic` module, described earlier, but there's a lot more too:

**Networking**

* `TcpStream`, `TcpListener`, and `UdpSocket` gained a number of methods for
  configuring the connection.
* `SocketAddr` and its variants gained `set_ip()` and `set_port()` conveniences.

**Collections**

* `BTreeSet` and `HashSet` gained the `take()`, `replace()`, and `get()`
  methods, which make it possible to recover ownership of the original key.
* `OsString` gained a few methods, bringing it closer to parity with `String`.
* Slices gained `copy_from_slice()`, a safe form of `memcpy`.

**Encoding**

* `char` gained the ability to decode into UTF-16.

**Pointers**

* Raw pointers gained `as_ref()` and `as_mut()`, which returns an `Option<&T>`,
  translating null pointers into `None`.
* `ptr::{read,write}_volatile()` allow for volatile reading and writing from a
  raw pointer.

Finally, many of the types in `libcore` did not contain a `Debug`
implementation. [This was fixed](https://github.com/rust-lang/rust/pull/32054)
in the 1.9 release.

See the [detailed release notes][notes] for more.

#### Cargo features

There were two major changes to Cargo:

First, Cargo
[can now be run concurrently](https://github.com/rust-lang/cargo/pull/2486).

Second, a new flag, `RUSTFLAGS`,
[was added](https://github.com/rust-lang/cargo/pull/2241). This flag allows you
to specify arbitrary flags to be passed to `rustc` through an environment
variable, which is useful for packagers, for example.

See the [detailed release notes][notes] for more.

### Contributors to 1.9

We had 127 individuals contribute to 1.9. Thank you so much!

* Aaron Turon
* Abhishek Chanda
* Adolfo Ochagavía
* Aidan Hobson Sayers
* Alan Somers
* Alejandro Wainzinger
* Aleksey Kladov
* Alex Burka
* Alex Crichton
* Amanieu d'Antras
* Andrea Canciani
* Andreas Linz
* Andrew Cantino
* Andrew Horton
* Andrew Paseltiner
* Andrey Cherkashin
* Angus Lees
* Ariel Ben-Yehuda
* Benjamin Herr
* Björn Steinbrink
* Brian Anderson
* Brian Bowman
* Christian Wesselhoeft
* Christopher Serr
* Corey Farwell
* Craig M. Brandenburg
* Cyryl Płotnicki-Chudyk
* Daniel J Rollins
* Dave Huseby
* David AO Lozano
* David Henningsson
* Devon Hollowood
* Dirk Gadsden
* Doug Goldstein
* Eduard Burtescu
* Eduard-Mihai Burtescu
* Eli Friedman
* Emanuel Czirai
* Erick Tryzelaar
* Evan
* Felix S. Klock II
* Florian Berger
* Geoff Catlin
* Guillaume Gomez
* Gökhan Karabulut
* JP Sugarbroad
* James Miller
* Jeffrey Seyfried
* John Talling
* Jonas Schievink
* Jonathan S
* Jorge Aparicio
* Joshua Holmer
* Kai Noda
* Kamal Marhubi
* Katze
* Kevin Brothaler
* Kevin Butler
* Manish Goregaokar
* Markus Westerlind
* Marvin Löbel
* Masood Malekghassemi
* Matt Brubeck
* Michael Huynh
* Michael Neumann
* Michael Woerister
* Ms2ger
* NODA, Kai
* Nathan Kleyn
* Nick Cameron
* Niko Matsakis
* Noah
* Novotnik, Petr
* Oliver Middleton
* Oliver Schneider
* Philipp Oppermann
* Piotr Czarnecki
* Pyfisch
* Richo Healey
* Ruud van Asseldonk
* Scott Olson
* Sean McArthur
* Sebastian Wicki
* Seo Sanghyeon
* Simon Sapin
* Simonas Kazlauskas
* Steve Klabnik
* Steven Allen
* Steven Fackler
* Stu Black
* Sébastien Marie
* Tang Chenglong
* Ted Horst
* Ticki
* Tim Montague
* Tim Neumann
* Timon Van Overveldt
* Tobias Bucher
* Tobias Müller
* Todd Lucas
* Tom Tromey
* Tshepang Lekhonkhobe
* Ulrik Sverdrup
* Vadim Petrochenkov
* Valentin Lorentz
* Varun Vats
* Wang Xuerui
* Wangshan Lu
* York Xiang
* arcnmx
* ashleysommer
* bors
* ggomez
* gohyda
* ituxbag
* mitaa
* nicholasf
* petevine
* pierzchalski
* pravic
* srinivasreddy
* tiehuis
* ubsan
* vagrant
* vegai
* vlastachu
* Валерий Лашманов
