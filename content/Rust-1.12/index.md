+++
path = "2016/09/29/Rust-1.12"
title = "Announcing Rust 1.12"
authors = ["The Rust Core Team"]
aliases = [
    "2016/09/29/Rust-1.12.html",
    "releases/1.12.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.12. Rust is
a systems programming language with the slogan "fast, reliable, productive:
pick three."

As always, you can [install Rust 1.12][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.12][notes] on GitHub.
1361 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1120-2016-09-29

### What's in 1.12 stable

The release of 1.12 might be one of the most significant Rust releases since
1.0. We have a lot to cover, but if you don't have time for that, here's a
summary:

The largest user-facing change in 1.12 stable is the new error message format
emitted by `rustc`. We've [previously talked] about this format and this is the
first stable release where they are broadly available. These error messages are
a result of the effort of many hours of [volunteer effort] to design, test, and
update every one of `rustc`s errors to the new format. We're excited to see
what you think of them:

![A new borrow-check error](borrowck-error.png)

The largest internal change in this release is moving to a new compiler backend
based on the new Rust [MIR]. While this feature does not result in anything
user-visible today, it paves the way for a number of future compiler
optimizations, and for some codebases it already results in improvements to
compile times and reductions in code size.

[previously talked]: https://blog.rust-lang.org/2016/08/10/Shape-of-errors-to-come.html
[volunteer effort]: https://github.com/rust-lang/rust/issues/35233
[MIR]: https://blog.rust-lang.org/2016/04/19/MIR.html

#### Overhauled error messages

With 1.12 we're introducing a new error format which helps to surface a lot of
the internal knowledge about why an error is occurring to you, the developer.
It does this by putting your code front and center, highlighting the parts
relevant to the error with annotations describing what went wrong.

For example, in 1.11 if a implementation of a trait didn't match the trait
declaration, you would see an error like the one below:

![An old mismatched trait
error](old-mismatched-trait-error.png)

In the new error format we represent the error by instead showing the points in
the code that matter the most. Here is the relevant line in the trait
declaration, and the relevant line in the implementation, using labels to
describe why they don't match:

![A new mismatched trait
error](mismatched-trait-error.png)

Initially, this error design was built to aid in understanding borrow-checking
errors, but we found, as with the error above, the format can be broadly
applied to a wide variety of errors. If you would like to learn more about the
design, check out the [previous blog post on the subject][err].

[err]: https://blog.rust-lang.org/2016/08/10/Shape-of-errors-to-come.html

Finally, you can also get these errors as JSON with a flag. Remember that error
we showed above, at the start of the post? Here's an example of attempting to
compile that code while passing the `--error-format=json` flag:

```bash
$ rustc borrowck-assign-comp.rs --error-format=json
{"message":"cannot assign to `p.x` because it is borrowed","level":"error","spans":[{"file_name":"borrowck-assign-comp.rs","byte_start":562,"byte_end":563,"line_start":15,"line_end":15,"column_start":14,"column_end":15,"is_primary":false,"text":[{"text":"    let q = &p;","highlight_start":14,"highlight_end":15}],"label":"borrow of `p.x` occurs here","suggested_replacement":null,"expansion":null}],"label":"assignment to borrowed `p.x` occurs here","suggested_replacement":null,"expansion":null}],"children":[],"rendered":null}
{"message":"aborting due to previous error","code":null,"level":"error","spans":[],"children":[],"rendered":null}
```

We've actually elided a bit of this for brevity's sake, but you get the idea.
This output is primarily for tooling; we are continuing to invest in supporting
IDEs and other useful development tools. This output is a small part of that
effort.

#### MIR code generation

The new Rust "mid-level IR", usually called "MIR", gives the compiler a simpler
way to think about Rust code than its previous way of operating entirely on the
Rust abstract syntax tree. It makes analysis and optimizations possible that
have historically been difficult to implement correctly. The first of many
upcoming changes to the compiler enabled by MIR is a rewrite of the pass that
generates LLVM IR, what `rustc` calls "translation", and after many months of
effort the MIR-based backend has proved itself ready for prime-time.

MIR exposes perfect information about the program's control flow, so the
compiler knows exactly whether types are moved or not. This means that it knows
statically whether or not the value's destructor needs to be run. In cases
where a value may or may not be moved at the end of a scope, the compiler now
simply uses a single bitflag on the stack, which is in turn easier for
optimization passes in LLVM to reason about. The end result is less work for
the compiler and less bloat at runtime. In addition, because MIR is a simpler
'language' than the full AST, it's also easier to implement compiler passes on,
and easier to verify that they are correct.

#### Other improvements

* Many minor improvements to the documentation.
* [`rustc` supports three new MUSL targets on ARM:
  `arm-unknown-linux-musleabi`, `arm-unknown-linux-musleabihf`, and
`armv7-unknown-linux-musleabihf`](https://github.com/rust-lang/rust/pull/35060).
  These targets produce statically-linked binaries. There are no binary release
  builds yet though.
* In error descriptions,
  [references](https://github.com/rust-lang/rust/pull/35611) and [unknown numeric
  types](https://github.com/rust-lang/rust/pull/35080) have more human-friendly errors.
* [The compiler can now be built against LLVM 3.9](https://github.com/rust-lang/rust/pull/35594)
* [Test binaries now support a `--test-threads` argument to specify the number
  of threads used to run tests, and which acts the same as the
  `RUST_TEST_THREADS` environment variable](https://github.com/rust-lang/rust/pull/35414)
* [The test runner now emits a warning when tests run over 60
  seconds](https://github.com/rust-lang/rust/pull/35405)
* [Rust releases now come with source packages that can be installed by rustup
  via `rustup component add
rust-src`](https://github.com/rust-lang/rust/pull/34366).
  The resulting source code can be used by tools and IDES, located in the
  sysroot under `lib/rustlib/src`.

See the [detailed release notes][notes] for more.

#### Library stabilizations

This release sees a number of small quality of life improvements for various
types in the standard library:

* [`Cell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.Cell.html#method.as_ptr)
  and
  [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
* `IpAddr`, `Ipv4Addr`, and `Ipv6Addr` have a few new methods.
* [`LinkedList`](https://doc.rust-lang.org/std/collections/linked_list/struct.LinkedList.html#method.contains)
  and
  [`VecDeque`](https://doc.rust-lang.org/std/collections/vec_deque/struct.VecDeque.html#method.contains)
  have a new `contains` method.
* [`iter::Product`](https://doc.rust-lang.org/std/iter/trait.Product.html) and
  [`iter::Sum`](https://doc.rust-lang.org/std/iter/trait.Sum.html)
* [`Option` implements `From` for its contained
  type](https://github.com/rust-lang/rust/pull/34828)
* [`Cell`, `RefCell` and `UnsafeCell` implement `From` for their contained
  type](https://github.com/rust-lang/rust/pull/35392)
* [`Cow<str>` implements `FromIterator` for `char`, `&str` and
  `String`](https://github.com/rust-lang/rust/pull/35064)
* [Sockets on Linux are correctly closed in subprocesses via
  `SOCK_CLOEXEC`](https://github.com/rust-lang/rust/pull/34946)
* [`String` implements
  `AddAssign`](https://github.com/rust-lang/rust/pull/34890)
* [Unicode definitions have been updated to
  9.0](https://github.com/rust-lang/rust/pull/34599)

See the [detailed release notes][notes] for more.

#### Cargo features

The biggest feature added to Cargo this cycle is
"[workspaces](https://github.com/rust-lang/cargo/pull/2759)." Defined in [RFC
1525](https://github.com/rust-lang/rfcs/blob/master/text/1525-cargo-workspace.md),
workspaces allow a group of Rust packages to share the same `Cargo.lock` file.
If you have a project that's split up into multiple packages, this makes it
much easier to keep shared dependencies on a single version. To enable this
feature, most multi-package projects need to add a single key, `[workspace]`,
to their top-level `Cargo.toml`, but more complex setups may require more
configuration.

Another significant feature is the ability to [override the
source of a crate](https://github.com/rust-lang/cargo/pull/2857). Using this
with tools like [cargo-vendor] and [cargo-local-registry] allow vendoring
dependencies locally in a robust fashion. Eventually this support will be the
foundation of supporting mirrors of [crates.io] as well.

[cargo-vendor]: https://github.com/alexcrichton/cargo-vendor
[cargo-local-registry]: https://github.com/alexcrichton/cargo-local-registry
[crates.io]: https://crates.io/

There are some other improvements as well:

* [Speed up noop registry
  updates](https://github.com/rust-lang/cargo/pull/2974)
* [Add a `--lib` flag to `cargo
  new`](https://github.com/rust-lang/cargo/pull/2921)
* [Indicate the compilation profile after
  compiling](https://github.com/rust-lang/cargo/pull/2909)
* [Add `--dry-run` to `cargo
  publish`](https://github.com/rust-lang/cargo/pull/2849)

See the [detailed release notes][notes] for more.

### Contributors to 1.12

We had 176 individuals contribute to 1.12. Thank you so much!

* Aaron Gallagher
* abhi
* Adam Medziński
* Ahmed Charles
* Alan Somers
* Alexander Altman
* Alexander Merritt
* Alex Burka
* Alex Crichton
* Amanieu d'Antras
* Andrea Pretto
* Andre Bogus
* Andrew
* Andrew Cann
* Andrew Paseltiner
* Andrii Dmytrenko
* Antti Keränen
* Aravind Gollakota
* Ariel Ben-Yehuda
* Bastien Dejean
* Ben Boeckel
* Ben Stern
* bors
* Brendan Cully
* Brett Cannon
* Brian Anderson
* Bruno Tavares
* Cameron Hart
* Camille Roussel
* Cengiz Can
* CensoredUsername
* cgswords
* Chiu-Hsiang Hsu
* Chris Stankus
* Christian Poveda
* Christophe Vu-Brugier
* Clement Miao
* Corey Farwell
* CrLF0710
* crypto-universe
* Daniel Campbell
* David
* decauwsemaecker.glen@gmail.com
* Diggory Blake
* Dominik Boehi
* Doug Goldstein
* Dridi Boukelmoune
* Eduard Burtescu
* Eduard-Mihai Burtescu
* Evgeny Safronov
* Federico Ravasio
* Felix Rath
* Felix S. Klock II
* Fran Guijarro
* Georg Brandl
* ggomez
* gnzlbg
* Guillaume Gomez
* hank-der-hafenarbeiter
* Hariharan R
* Isaac Andrade
* Ivan Nejgebauer
* Ivan Ukhov
* Jack O'Connor
* Jake Goulding
* Jakub Hlusička
* James Miller
* Jan-Erik Rediger
* Jared Manning
* Jared Wyles
* Jeffrey Seyfried
* Jethro Beekman
* Jonas Schievink
* Jonathan A. Kollasch
* Jonathan Creekmore
* Jonathan Giddy
* Jonathan Turner
* Jorge Aparicio
* José manuel Barroso Galindo
* Josh Stone
* Jupp Müller
* Kaivo Anastetiks
* kc1212
* Keith Yeung
* Knight
* Krzysztof Garczynski
* Loïc Damien
* Luke Hinds
* Luqman Aden
* m4b
* Manish Goregaokar
* Marco A L Barbosa
* Mark Buer
* Mark-Simulacrum
* Martin Pool
* Masood Malekghassemi
* Matthew Piziak
* Matthias Rabault
* Matt Horn
* mcarton
* M Farkas-Dyck
* Michael Gattozzi
* Michael Neumann
* Michael Rosenberg
* Michael Woerister
* Mike Hommey
* Mikhail Modin
* mitchmindtree
* mLuby
* Moritz Ulrich
* Murarth
* Nick Cameron
* Nick Massey
* Nikhil Shagrithaya
* Niko Matsakis
* Novotnik, Petr
* Oliver Forral
* Oliver Middleton
* Oliver Schneider
* Omer Sheikh
* Panashe M. Fundira
* Patrick McCann
* Paul Woolcock
* Peter C. Norton
* Phlogistic Fugu
* Pietro Albini
* Rahiel Kasim
* Rahul Sharma
* Robert Williamson
* Roy Brunton
* Ryan Scheel
* Ryan Scott
* saml
* Sam Payson
* Samuel Cormier-Iijima
* Scott A Carr
* Sean McArthur
* Sebastian Thiel
* Seo Sanghyeon
* Shantanu Raj
* ShyamSundarB
* silenuss
* Simonas Kazlauskas
* srdja
* Srinivas Reddy Thatiparthy
* Stefan Schindler
* Stephen Lazaro
* Steve Klabnik
* Steven Fackler
* Steven Walter
* Sylvestre Ledru
* Tamir Duberstein
* Terry Sun
* TheZoq2
* Thomas Garcia
* Tim Neumann
* Timon Van Overveldt
* Tobias Bucher
* Tomasz Miąsko
* trixnz
* Tshepang Lekhonkhobe
* ubsan
* Ulrik Sverdrup
* Vadim Chugunov
* Vadim Petrochenkov
* Vincent Prouillet
* Vladimir Vukicevic
* Wang Xuerui
* Wesley Wiser
* William Lee
* Ximin Luo
* Yojan Shrestha
* Yossi Konstantinovsky
* Zack M. Davis
* Zhen Zhang
* 吴冉波
