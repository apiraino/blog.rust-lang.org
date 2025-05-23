+++
path = "2016/01/21/Rust-1.6"
title = "Announcing Rust 1.6"
authors = ["The Rust Core Team"]
aliases = [
    "2016/01/21/Rust-1.6.html",
    "releases/1.6.0",
]

[extra]
release = true
+++

Hello 2016! We’re happy to announce the first Rust release of the year, 1.6.
Rust is a systems programming language focused on safety, speed, and
concurrency.

As always, you can [install Rust 1.6][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.6][notes] on GitHub.
About 1100 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-160-2016-01-21

### What's in 1.6 stable

This release contains a number of small refinements, one major feature, and
a change to [Crates.io](https://crates.io).

#### libcore stabilization

The largest new feature in 1.6 is that [`libcore`] is now stable! Rust’s
standard library is two-tiered: there’s a small core library, `libcore`, and
the full standard library, `libstd`, that builds on top of it. `libcore` is
completely platform agnostic, and requires only a handful of external symbols
to be defined. Rust’s `libstd` builds on top of `libcore`, adding support for
memory allocation, I/O, and concurrency. Applications using Rust in the
embedded space, as well as those writing operating systems, often eschew
`libstd`, using only `libcore`.

[`libcore`]: https://doc.rust-lang.org/nightly/core/

`libcore` being stabilized is a major step towards being able to write the
lowest levels of software using stable Rust. There’s still future work to be
done, however. This will allow for a library ecosystem to develop around
`libcore`, but _applications_ are not fully supported yet. Expect to hear more
about this in future release notes.

#### Library stabilizations

About 30 library functions and methods are now stable in 1.6. Notable
improvements include:

The `drain()` family of functions on collections. These methods let you move
elements out of a collection while allowing them to retain their backing
memory, reducing allocation in certain situations.

A number of implementations of `From` for converting between standard library
types, mainly between various integral and floating-point types.

Finally, `Vec::extend_from_slice()`, which was previously known as
`push_all()`. This method has a significantly faster implementation than the
more general `extend()`.

See the [detailed release notes][notes] for more.

#### Crates.io disallows wildcards

If you maintain a crate on [Crates.io](https://crates.io), you might have seen
a warning: newly uploaded crates are no longer allowed to use a wildcard when
describing their dependencies. In other words, this is not allowed:

```toml
[dependencies]
regex = "*"
```

Instead, you must actually specify [a specific version or range of
versions][versions], using one of the `semver` crate’s various options: `^`,
`~`, or `=`.

[versions]: https://doc.crates.io/crates-io.html#using-cratesio-based-crates

A wildcard dependency means that you work with any possible version of your
dependency. This is highly unlikely to be true, and causes unnecessary breakage
in the ecosystem. We’ve been advertising this change as a warning for some time;
now it’s time to turn it into an error.

### Contributors to 1.6

We had 132 individuals contribute to 1.6. Thank you so much!

* Aaron Turon
* Adam Badawy
* Aleksey Kladov
* Alexander Bulaev
* Alex Burka
* Alex Crichton
* Alex Gaynor
* Alexis Beingessner
* Amanieu d'Antras
* Amit Saha
* Andrea Canciani
* Andrew Paseltiner
* androm3da
* angelsl
* Angus Lees
* Antti Keränen
* arcnmx
* Ariel Ben-Yehuda
* Ashkan Kiani
* Barosl Lee
* Benjamin Herr
* Ben Striegel
* Bhargav Patel
* Björn Steinbrink
* Boris Egorov
* bors
* Brian Anderson
* Bruno Tavares
* Bryce Van Dyk
* Cameron Sun
* Christopher Sumnicht
* Cole Reynolds
* corentih
* Daniel Campbell
* Daniel Keep
* Daniel Rollins
* Daniel Trebbien
* Danilo Bargen
* Devon Hollowood
* Doug Goldstein
* Dylan McKay
* ebadf
* Eli Friedman
* Eric Findlay
* Erik Davidson
* Felix S. Klock II
* Florian Hahn
* Florian Hartwig
* Gleb Kozyrev
* Guillaume Gomez
* Huon Wilson
* Igor Shuvalov
* Ivan Ivaschenko
* Ivan Kozik
* Ivan Stankovic
* Jack Fransham
* Jake Goulding
* Jake Worth
* James Miller
* Jan Likar
* Jean Maillard
* Jeffrey Seyfried
* Jethro Beekman
* John Kåre Alsaker
* John Talling
* Jonas Schievink
* Jonathan S
* Jose Narvaez
* Josh Austin
* Josh Stone
* Joshua Holmer
* JP Sugarbroad
* jrburke
* Kevin Butler
* Kevin Yeh
* Kohei Hasegawa
* Kyle Mayes
* Lee Jeffery
* Manish Goregaokar
* Marcell Pardavi
* Markus Unterwaditzer
* Martin Pool
* Marvin Löbel
* Matt Brubeck
* Matthias Bussonnier
* Matthias Kauer
* mdinger
* Michael Layzell
* Michael Neumann
* Michael Sproul
* Michael Woerister
* Mihaly Barasz
* Mika Attila
* mitaa
* Ms2ger
* Nicholas Mazzuca
* Nick Cameron
* Niko Matsakis
* Ole Krüger
* Oliver Middleton
* Oliver Schneider
* Ori Avtalion
* Paul A. Jungwirth
* Peter Atashian
* Philipp Matthias Schäfer
* pierzchalski
* Ravi Shankar
* Ricardo Martins
* Ricardo Signes
* Richard Diamond
* Rizky Luthfianto
* Ryan Scheel
* Scott Olson
* Sean Griffin
* Sebastian Hahn
* Sébastien Marie
* Seo Sanghyeon
* Simonas Kazlauskas
* Simon Sapin
* Stepan Koltsov
* Steve Klabnik
* Steven Fackler
* Tamir Duberstein
* Tobias Bucher
* Toby Scrace
* Tshepang Lekhonkhobe
* Ulrik Sverdrup
* Vadim Chugunov
* Vadim Petrochenkov
* William Throwe
* xd1le
* Xmasreturns
