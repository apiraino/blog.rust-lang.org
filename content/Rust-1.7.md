+++
path = "2016/03/02/Rust-1.7"
title = "Announcing Rust 1.7"
authors = ["The Rust Core Team"]
aliases = [
    "2016/03/02/Rust-1.7.html",
    "releases/1.7.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.7. Rust is a
systems programming language focused on safety, speed, and concurrency.

As always, you can [install Rust 1.7][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.7][notes] on GitHub.
About 1300 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-170-2016-03-03

### What's in 1.7 stable

This release is primarily about library features. While we have several
language features cooking for future releases, the timeframe in which 1.7 was
developed included the holidays, which means less time for commenting on GitHub
and more time for spending with loved ones.

#### Library stabilizations

About 40 library functions and methods are now stable in 1.7. One of the
largest APIs stabilized  was support for custom hash algorithms in the standard
library’s `HashMap<K, V>` type. Previously all hash maps would use [SipHash] as
the hashing algorithm, which provides protection against DOS attacks by
default. SipHash, however, is [not very fast] at hashing small keys. As shown,
however, the [FNV hash algorithm] is much faster for these size of inputs. This
means that by switching hash algorithms for types like `HashMap<usize, V>`
there can be a significant speedup so long as the loss of DOS protection is
acceptable.

[Siphash]: https://en.wikipedia.org/wiki/SipHash
[not very fast]: https://cglab.ca/~abeinges/blah/hash-rs/
[FNV hash algorithm]: https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function

To see this in action, you can check out the [fnv crate] on [crates.io] and
create a `HashMap` via:

```rust
extern crate fnv;

use std::collections::HashMap;
use std::hash::BuildHasherDefault;
use fnv::FnvHasher;

type MyHasher = BuildHasherDefault<FnvHasher>;

fn main() {
    let mut map: HashMap<_, _, MyHasher> = HashMap::default();
    map.insert(1, "Hello");
    map.insert(2, ", world!");
    println!("{:?}", map);
}
```

[fnv crate]: https://crates.io/crates/fnv
[crates.io]: https://crates.io


Note that most of the time you don’t even need to specify the hasher as type
inference will take care of it, so `HashMap::default()` should be all you need
to get up to 2x faster hashes. It’s also worth pointing out that [`Hash`] trait
is agnostic to the hashing algorithm used, so no changes are needed to the
types being inserted into hash maps to reap the benefits!

[`Hash`]: https://doc.rust-lang.org/std/hash/trait.Hash.html

Other notable improvements include:

* `<[T]>::clone_from_slice()`, an efficient way to copy the data from one slice
  and put it into another slice.
* Various convenience methods on `Ipv4Addr` and `Ipv6Addr`, such as `is_loopback()`,
  which returns `true` or `false` if the address is a loopback address according to
  RFC 6890.
* Various improvements to `CString`, used for FFI.
* checked, saturated, and overflowing operations for various numeric types.
  These aren’t counted in that ‘40’ number above, because there are a _lot_ of
  them, but they all do the same thing.

See the [detailed release notes][notes] for more.

#### Cargo features

There were a few small updates to Cargo:

* An [improvement to build scripts] that allows them to precisely inform Cargo
  about dependencies to ensure that they’re only rerun when those files change.
  This should help development quite a bit in repositories with build scripts.
* A [modification to the `cargo rustc` subcommand], which allows specifying
  profiles to pull in dev-dependencies during testing and such.


[improvement to build scripts]: https://github.com/rust-lang/cargo/pull/2279
[modification to the `cargo rustc` subcommand]: https://github.com/rust-lang/cargo/pull/2224

### Contributors to 1.7

We had 144 individuals contribute to 1.7. Thank you so much!

* Aaron Turon
* Adam Perry
* Adrian Heine
* Aidan Hobson Sayers
* Aleksey Kladov
* Alexander Lopatin
* Alex Burka
* Alex Crichton
* Ali Clark
* Amanieu d’Antras
* Andrea Bedini
* Andrea Canciani
* Andre Bogus
* Andrew Barchuk
* Andrew Paseltiner
* angelsl
* Anton Blanchard
* arcnmx
* Ariel Ben-Yehuda
* arthurprs
* ashleysommer
* Barosl Lee
* Benjamin Herr
* Björn Steinbrink
* bors
* Brandon W Maister
* Brian Anderson
* Brian Campbell
* Carlos E. Garcia
* Chad Shaffer
* Corey Farwell
* Daan Sprenkels
* Daniel Campbell
* Daniel Robertson
* Dave Hodder
* Dave Huseby
* dileepb
* Dirk Gadsden
* Eduard Burtescu
* Erick Tryzelaar
* est31
* Evan
* Fabrice Desré
* fbergr
* Felix Gruber
* Felix S. Klock II
* Florian Hahn
* Geoff Catlin
* Geoffrey Thomas
* Georg Brandl
* ggomez
* Gleb Kozyrev
* Gökhan Karabulut
* Greg Chapple
* Guillaume Bonnet
* Guillaume Gomez
* Ivan Kozik
* Jack O’Connor
* Jeffrey Seyfried
* Johan Lorenzo
* Johannes Oertel
* John Hodge
* John Kåre Alsaker
* Jonas Schievink
* Jonathan Reem
* Jonathan S
* Jorge Aparicio
* Josh Stone
* Kamal Marhubi
* Katze
* Keith Yeung
* Kenneth Koski
* Kevin Stock
* Luke Jones
* Manish Goregaokar
* Marc Bowes
* Marvin Löbel
* Masood Malekghassemi
* Matt Brubeck
* Mátyás Mustoha
* Michael Huynh
* Michael Neumann
* Michael Woerister
* mitaa
* mopp
* Nathan Kleyn
* Nicholas Mazzuca
* Nick Cameron
* Nikita Baksalyar
* Niko Matsakis
* NODA, Kai
* nxnfufunezn
* Olaf Buddenhagen
* Oliver ‘ker’ Schneider
* Oliver Middleton
* Oliver Schneider
* Pascal Hertleif
* Paul Dicker
* Paul Smith
* Peter Atashian
* Peter Kolloch
* petevine
* Pierre Krieger
* Piotr Czarnecki
* Prayag Verma
* qpid
* Ravi Shankar
* Reeze Xia
* Richard Bradfield
* Robin Kruppe
* rphmeier
* Ruud van Asseldonk
* Ryan Thomas
* Sandeep Datta
* Scott Olson
* Scott Whittaker
* Sean Leffler
* Sean McArthur
* Sebastian Hahn
* Sebastian Wicki
* Sébastien Marie
* Seo Sanghyeon
* Sergey Veselkov
* Simonas Kazlauskas
* Simon Sapin
* Stepan Koltsov
* Stephan Hügel
* Steve Klabnik
* Steven Allen
* Steven Fackler
* Tamir Duberstein
* tgor
* Thomas Wickham
* Thomas Winwood
* Tobias Bucher
* Toby Scrace
* Tomasz Miąsko
* tormol
* Tshepang Lekhonkhobe
* Ulrik Sverdrup
* Vadim Petrochenkov
* Vincent Esche
* Vlad Ureche
* Wangshan Lu
* Wesley Wiser
