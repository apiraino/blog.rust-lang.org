+++
path = "2016/04/14/Rust-1.8"
title = "Announcing Rust 1.8"
authors = ["The Rust Core Team"]
aliases = [
    "2016/04/14/Rust-1.8.html",
    "releases/1.8.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.8. Rust is a
systems programming language focused on safety, speed, and concurrency.

As always, you can [install Rust 1.8][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.8][notes] on GitHub.
About 1400 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-180-2016-04-14

### What's in 1.8 stable

There are two new features in Rust 1.8, as well as good news for Windows users!
Additionally, work is underway to replace our `make`-based build system with
one based on Cargo.

The first feature is that the various “operator equals” operators, such as `+=`
and `-=`, are now overloadable via various traits. This change was accepted in
[RFC 953], and looks like this:

```rust
use std::ops::AddAssign;

#[derive(Debug)]
struct Count { 
    value: i32,
}
    
impl AddAssign for Count {
    fn add_assign(&mut self, other: Count) {
        self.value += other.value;
    }
}   

fn main() {
    let mut c1 = Count { value: 1 };
    let c2 = Count { value: 5 };

    c1 += c2;

    println!("{:?}", c1);
}
```

[RFC 953]: https://github.com/rust-lang/rfcs/blob/master/text/0953-op-assign.md

This will print out `Count { value: 6 }`. Like the other operator traits, an
associated type allows you to use different types on each side of the operator,
as well. See the RFC for more details.

The second feature is very small, and comes from [RFC 218]. Before Rust 1.8, a
`struct` with no fields did not have curly braces:

```rust
struct Foo; // works
struct Bar { } // error
```

[RFC 218]: https://github.com/rust-lang/rfcs/blob/master/text/0218-empty-struct-with-braces.md

The second form is no longer an error. This was originally disallowed for
consistency with other empty declarations, as well as a parsing ambiguity.
However, that ambiguity is non-existent in post-1.0 Rust. Macro authors saw
additional complexity due to needing a special-case, as well. Finally, users
who do active development would sometimes switch between empty and non-empty
versions of a struct, and the extra work and diffs involved was less than
ideal.

On the Windows front, 32-bit MSVC builds [now implement unwinding]. This moves
`i686-pc-windows-msvc` to a Tier 1 platform.

[now implement unwinding]: https://github.com/rust-lang/rust/pull/30448

Finally, we have used `make` to build Rust for a very long time. However,
we already have a wonderful tool for building Rust programs: Cargo. In Rust
1.8, [initial support landed] for a new build system that’s written in Rust,
and based on Cargo. It is not yet the default, and there is much more work to
do. We will talk about this in release notes more once it’s completely done,
for now, please read the GitHub issue for more details.

[initial support landed]: https://github.com/rust-lang/rust/pull/31123

#### Library stabilizations

About 20 library functions and methods are now stable in 1.8. There are three
major groups of changes: UTF-16 related string methods, various APIs related to
time, and the various traits needed for operator overloading mentioned in the
language section.

See the [detailed release notes][notes] for more.

#### Cargo features

There were a few updates to Cargo:

* [`cargo init`](https://github.com/rust-lang/cargo/pull/2081) can be used to
  start a Cargo project in your current working directory, rather than making a
  new subdirectory like `cargo new`.
* [`cargo metadata`](https://github.com/rust-lang/cargo/pull/2196) is another
  new subcommand for fetching metadata about a project.
* `.cargo/config` now has [keys for `-v` and
  `--color`](https://github.com/rust-lang/cargo/pull/2397)
* Cargo’s ability to have target-specific dependencies [was
  enhanced](https://github.com/rust-lang/cargo/pull/2328).


See the [detailed release notes][notes] for more.

### Contributors to 1.8

We had 126 individuals contribute to 1.8. Thank you so much!

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
* arcnmx
* Ariel Ben-Yehuda
* ashleysommer
* Benjamin Herr
* Валерий Лашманов
* Björn Steinbrink
* bors
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
* ggomez
* gohyda
* Gökhan Karabulut
* Guillaume Gomez
* ituxbag
* James Miller
* Jeffrey Seyfried
* John Talling
* Jonas Schievink
* Jonathan S
* Jorge Aparicio
* Joshua Holmer
* JP Sugarbroad
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
* mitaa
* Ms2ger
* Nathan Kleyn
* nicholasf
* Nick Cameron
* Niko Matsakis
* Noah
* NODA, Kai
* Novotnik, Petr
* Oliver Middleton
* Oliver Schneider
* petevine
* Philipp Oppermann
* pierzchalski
* Piotr Czarnecki
* pravic
* Pyfisch
* Richo Healey
* Ruud van Asseldonk
* Scott Olson
* Sean McArthur
* Sebastian Wicki
* Sébastien Marie
* Seo Sanghyeon
* Simonas Kazlauskas
* Simon Sapin
* srinivasreddy
* Steve Klabnik
* Steven Allen
* Steven Fackler
* Stu Black
* Tang Chenglong
* Ted Horst
* Ticki
* tiehuis
* Tim Montague
* Tim Neumann
* Timon Van Overveldt
* Tobias Bucher
* Tobias Müller
* Todd Lucas
* Tom Tromey
* Tshepang Lekhonkhobe
* ubsan
* Ulrik Sverdrup
* Vadim Petrochenkov
* vagrant
* Valentin Lorentz
* Varun Vats
* vegai
* vlastachu
* Wangshan Lu
* York Xiang
