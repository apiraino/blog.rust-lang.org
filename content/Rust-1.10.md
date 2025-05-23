+++
path = "2016/07/07/Rust-1.10"
title = "Announcing Rust 1.10"
authors = ["The Rust Core Team"]
aliases = [
    "2016/07/07/Rust-1.10.html",
    "releases/1.10.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.10. Rust is a
systems programming language focused on safety, speed, and concurrency.

As always, you can [install Rust 1.10][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.10][notes] on GitHub.
1276 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1100-2016-07-07

### What's in 1.10 stable

Rust 1.10 contains one of the most-requested features in Rust: through the [`-C
panic=abort` flag] or a [setting in your `Cargo.toml`], when a `panic!`
happens, it will abort rather than unwind. Why would you want to do this?
Remember that [panics are for unexpected problems], and for many applications,
aborting is a reasonable choice. With an abort, less code gets generated,
meaning that binary sizes are a bit smaller, and compilation time is
ever-so-slightly faster. Some very rough numbers are 10% smaller binaries, and
10% faster compilation time. This feature was defined in [RFC 1513].

[`-C panic=abort` flag]: https://github.com/rust-lang/rust/pull/32900
[setting in your `Cargo.toml`]: https://github.com/rust-lang/cargo/pull/2687
[panics are for unexpected problems]: https://blog.rust-lang.org/2016/05/26/Rust-1.9.html
[RFC 1513]: https://github.com/rust-lang/rfcs/blob/master/text/1513-less-unwinding.md

The second big feature in 1.10 is a new crate type: [`cdylib`]. The existing
dylib dynamic library format will now be used solely for writing a dynamic
library to be used within a Rust project, while `cdylib`s will be used when
compiling Rust code as a dynamic library to be embedded in another language.
With the 1.10 release, `cdylib`s are supported by the compiler, but not yet in
Cargo. This format was defined in [RFC 1510].

[`cdylib`]: https://github.com/rust-lang/rust/pull/33553
[RFC 1510]: https://github.com/rust-lang/rfcs/blob/master/text/1510-rdylib.md

In addition, [a number of performance improvements landed in the
compiler](https://github.com/rust-lang/rust/blob/master/RELEASES.md#performance),
and so did [a number of usability
improvements](https://github.com/rust-lang/rust/blob/master/RELEASES.md#usability)
across the documentation, `rustdoc` itself, and various error messages.

Finally, there's a large change to the way that we develop Rust that won't
impact Rust users directly, but will help those distributing Rust
significantly. Rust is implemented in Rust, which means that to build a copy of
Rust, you need a copy of Rust. This is commonly referred to as 'bootstrapping'.
Historically, we would do this by "snapshotting" a specific version of the
compiler, and always bootstrapping from that; the snapshot would periodically
be updated, as needed. Furthermore, since the Rust compiler uses unstable Rust
features, in order to build a copy of the stable compiler, you would need a
specific nightly version of the Rust compiler. This has served us well for
years, but we've outgrown it now. The main drawback to this approach is that it
requires downloading a snapshot binary, which is not ideal for an important
constituency: Linux distributions. In particular, distros generally want to be
able to build the latest version of Rust using only previously-packaged
versions that they have produced, rather than via untrusted binaries. As such,
we have modified our build system so that Rust 1.10 builds with Rust 1.9. In
the future, this pattern will continue; Rust 1.11 will be built with Rust 1.10.
Furthermore, you can use the stable compiler to build the compiler. This
simplifies everything around bootstrapping, and helps distribution maintainers
significantly, as they no longer need two packages. You can find more details
about this change [in its pull
request](https://github.com/rust-lang/rust/pull/32942).

See the [detailed release notes][notes] for more.

#### Library stabilizations

Roughly 70 APIs were made stable in this release. They break down into these rough
groups:

* [`std::os::windows::fs::OpenOptionsExt`], for Windows-specific file operations.
* The ability to [register and unregister panic hooks] with `std::panic::{set,take}_hook`.
* [`CStr::from_bytes_with_nul`], to create a `CStr` from a byte slice ([and an unchecked variant]).
* Small improvements to [`std::fs::Metadata`].
* [`compare_exchange` for various atomic types].
* A lot of [UNIX-specific networking capabilities] via
  `std::os::unix::net::{UnixStream, UnixListener, UnixDatagram, SocketAddr}`.

[`std::os::windows::fs::OpenOptionsExt`]: https://github.com/rust-lang/rfcs/pull/1252
[register and unregister panic hooks]: https://github.com/rust-lang/rfcs/pull/1328
[`CStr::from_bytes_with_nul`]: https://doc.rust-lang.org/std/ffi/struct.CStr.html#method.from_bytes_with_nul
[and an unchecked variant]: https://doc.rust-lang.org/std/ffi/struct.CStr.html#method.from_bytes_with_nul_unchecked
[`std::fs::Metadata`]: https://doc.rust-lang.org/std/fs/struct.Metadata.html
[`compare_exchange` for various atomic types]: https://github.com/rust-lang/rfcs/pull/1443
[UNIX-specific networking capabilities]: https://github.com/rust-lang/rfcs/pull/1479

In addition, `Default` was implemented for `&CStr`, `CString`,
`UnsafeCell`, `fmt::Error`, `Condvar`, `Mutex`, and `RwLock`.

Finally, on Linux, if HashMaps can't be initialized with `getrandom` they will
[fall back to `/dev/urandom` temporarily] to avoid blocking during early boot.

[fall back to `/dev/urandom` temporarily]: https://github.com/rust-lang/rust/pull/33086

See the [detailed release notes][notes] for more.

#### Cargo features

Cargo has received a number of small improvements in this release.

* The aforementioned [`profile.*.panic`] option can control how you'd like
  panics implemented for your project.
* Cargo now [reports its status to stderr rather than stdout].
* Rust keywords are now [banned from crate names].
* The [`--force` flag] has been added to `cargo install`.
* `cargo test` now takes a [`--doc` flag] for running only documentation tests.
* [`cargo --explain` was added], mirroring `rustc --explain`.

[`profile.*.panic`]: https://github.com/rust-lang/cargo/pull/2687
[banned from crate names]: https://github.com/rust-lang/cargo/pull/2707
[reports its status to stderr rather than stdout]: https://github.com/rust-lang/cargo/pull/2693
[`--force` flag]: https://github.com/rust-lang/cargo/pull/2405
[`--doc` flag]: https://github.com/rust-lang/cargo/pull/2578
[`cargo --explain` was added]: https://github.com/rust-lang/cargo/pull/2551

See the [detailed release notes][notes] for more.

### Contributors to 1.10

We had 139 individuals contribute to 1.10. Thank you so much!

* Adolfo Ochagavía
* Alan Somers
* Alec S
* Alex Burka
* Alex Crichton
* Alex Ozdemir
* Amanieu d'Antras
* Andrea Canciani
* Andrew Paseltiner
* Andrey Tonkih
* Andy Russell
* Anton Blanchard
* Ariel Ben-Yehuda
* Barosl Lee
* benaryorg
* billyevans
* Björn Steinbrink
* bnewbold
* bors
* Brandon Edens
* Brayden Winterton
* Brian Anderson
* Brian Campbell
* Brian Green
* c4rlo
* Christopher Serr
* Corey Farwell
* Cristian Oliveira
* Cyryl Płotnicki-Chudyk
* Dan Fockler
* Daniel Campoverde [alx741]
* Dave Huseby
* David Hewitt
* David Tolnay
* Deepak Kannan
* Demetri Obenour
* Doug Goldstein
* Eduard Burtescu
* Eduard-Mihai Burtescu
* Ergenekon Yigit
* Fabrice Desré
* Felix S. Klock II
* Florian Berger
* Garrett Squire
* Geordon Worley
* Georg Brandl
* ggomez
* Gigih Aji Ibrahim
* Guillaume Bonnet
* Guillaume Gomez
* Haiko Schol
* Jake Goulding
* James Miller
* jbranchaud
* Jeffrey Seyfried
* jethrogb
* jocki84
* Johannes Oertel
* Jonas Schievink
* jonathandturner
* Jonathan S
* Jonathan Turner
* JP Sugarbroad
* Kaiyin Zhong
* Kamal Marhubi
* Kevin Butler
* Léo Testard
* Luca Bruno
* Lukas Kalbertodt
* Lukas Pustina
* Luqman Aden
* Manish Goregaokar
* Marcus Klaas
* mark-summerfield
* Masood Malekghassemi
* Matt Brubeck
* Matt Kraai
* Maxim Samburskiy
* Michael Howell
* Michael Tiller
* Michael Woerister
* mitaa
* mrmiywj
* Ms2ger
* Murarth
* Nerijus Arlauskas
* Nick Cameron
* Nick Fitzgerald
* Nick Hamann
* Nick Platt
* Niko Matsakis
* Oliver 'ker' Schneider
* Oliver Middleton
* Oliver Schneider
* Patrick Walton
* Pavel Sountsov
* Philipp Matthias Schaefer
* Philipp Oppermann
* pierzchalski
* Postmodern
* pravic
* Pyry Kontio
* Raph Levien
* Rémy Rakic
* rkjnsn
* Robert Habermeier
* Robin Kruppe
* Sander Maijers
* Scott Olson
* Sean Gillespie
* Sébastien Marie
* Seo Sanghyeon
* silvo38
* Simonas Kazlauskas
* Simon Wollwage
* Stefan Schindler
* Stephen Mather
* Steve Klabnik
* Steven Burns
* Steven Fackler
* Szabolcs Berecz
* Tamir Duberstein
* Tang Chenglong
* Taylor Cramer
* Ticki
* Timon Van Overveldt
* Timothy McRoy
* Tobias Bucher
* Tobias Müller
* Tomáš Hübelbauer
* Tomoki Aonuma
* Tshepang Lekhonkhobe
* Ulrik Sverdrup
* User
* Vadim Chugunov
* Vadim Petrochenkov
* Val Vanderschaegen
* Wang Xuerui
* York Xiang
