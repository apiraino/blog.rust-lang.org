+++
path = "2015/10/29/Rust-1.4"
title = "Announcing Rust 1.4"
authors = ["The Rust Core Team"]
aliases = [
    "2015/10/29/Rust-1.4.html",
    "releases/1.4.0",
]

[extra]
release = true
+++

Choo choo! The trains have kept rolling, and today, we’re happy to announce the
release of Rust 1.4, the newest stable release. Rust is a systems programming
language focused on safety, speed, and concurrency.

As always, you can [install Rust 1.4][install] from the appropriate page on our
website, and check out the [detailed release notes for 1.4][notes] on GitHub as
well. About 1200 patches were landed in this release.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/8ab8581f6921bc7a8e3fa4defffd2814372dcb15/RELEASES.md#version-140-october-2015

### What's in 1.4 stable

The story of 1.4 is mostly one of improvements and stabilizations, rather than
new features.

However, there is one particular change which is a language fix that enables a
new feature: [RFC 1214, “Clarify (and improve) rules for projections and
well-formedness”](https://github.com/rust-lang/rfcs/pull/1214). While that’s
a deeply technical title, the TL;DR is that we found some weaknesses in the
definition and implementation of a few aspects of the type system. This RFC
fixes these problems. Given that changes to the type system like this can cause
regressions, but fixes like this are important for soundness, Rust 1.4 will
warn on any code that violates the new rules, but still compile. These warnings
will turn into errors in Rust 1.5. However, given the train model, the
community has had time to deal with these changes while 1.4 was in beta, and
the small number of crates we were aware of have already been fixed.

These soundness fixes enable the return of the ‘scoped threads’ feature, in
which you can create threads that reference data stored on the stack in a
safe manner. A few crates have implemented this feature, most notably
[crossbeam] and [scoped_threadpool]. See their documentation for more
information.

[crossbeam]: https://crates.io/crates/crossbeam
[scoped_threadpool]: https://crates.io/crates/scoped_threadpool

[RFC 1212](https://github.com/rust-lang/rfcs/blob/master/text/1212-line-endings.md)
is also in this release, which changes all functions dealing with reading
‘lines’ to treat both `\n` and `\r\n` as a valid line-ending. This was
determined during the RFC process to be a bugfix, but we’re mentioning it
here to raise awareness. The older behavior of only dealing with `\n` made
for surprising behavior, where your crate would work well on Linux and Mac OS
X, but fail on Windows. This fix brings these functions more in-line (😉)
with expectations.

Rust 1.4 marks an upgrade in our Windows support: Windows builds targeting the
64-bit MSVC ABI and linker (instead of GNU) are now supported and recommended
for general use, and will appear on the downloads page for the first time.
Thank you to all who have helped us work out the kinks since support initially
landed in Rust 1.2.

Here’s a summary of library changes:

* 48 APIs were stabilized.
* Eight APIs were deprecated.
* Two were made faster.
* Over ten various types implement new traits.

See the [release notes][libnotes] for exact details.

[libnotes]: https://github.com/brson/rust/blob/relnotes/RELEASES.md#libraries

The compiler [no longer uses
`morestack`](https://github.com/rust-lang/rust/pull/27338), which was a
holdover implementation detail from long, long ago. We now use guard pages
and stack probes instead, though stack probes are only implemented on Windows
so far.

Finally, one major Cargo improvement: [`cargo update` will now print extra
information about what it is
changing.](https://github.com/rust-lang/cargo/pull/1931) For example:

```
$ cargo update
    Updating registry `https://github.com/rust-lang/crates.io-index`
    Updating libc v0.1.8 -> v0.1.10
    Updating memchr v0.1.3 -> v0.1.5
    Updating num v0.1.26 -> v0.1.27
    Updating rand v0.3.9 -> v0.3.10
    Updating rustc-serialize v0.3.15 -> v0.3.16
```

Before, it would do this job silently.

### Contributors to 1.4

Rust is a community-driven project, and we're very appreciative of the work of
the 127 contributors who made 1.4 happen. Thank you!

- Adam Crume
- Aidan Hobson Sayers
- Aleksey Kladov
- Alex Burka
- Alex Crichton
- Alex Ozdemir
- AlexDenisov
- Alexis Beingessner
- Alisdair Owens
- Andre Bogus
- Andrea Canciani
- Andrew Paseltiner
- Ariel Ben-Yehuda
- Artem Shitov
- Barosl Lee
- benshu
- Björn Steinbrink
- bors
- Brian Anderson
- Cesar Eduardo Barros
- Chris Krycho
- Chris Morgan
- Chris Nixon
- Chris Wong
- christopherdumas
- Cody P Schafer
- Corey Farwell
- Daan Rijks
- Dave Huseby
- diaphore
- Diggory Blake
- Dong Zhou
- Dylan McKay
- Elaine "See More" Nemo
- Eli Friedman
- Eljay
- Erick Tryzelaar
- Felix S. Klock II
- Garming Sam
- Georg Brandl
- Gleb Kozyrev
- Guillaume Gomez
- Hunan Rostomyan
- Huon Wilson
- Ivan Jager
- Jack Wilson
- Jake Goulding
- Jake Kerr
- Jake Shadle
- James Miller
- Jan Likar
- Jared Roesch
- Jeehoon Kang
- John Thomas
- Jonas Schievink
- Jørn Lode
- Jose Narvaez
- jotomicron
- Kang Seonghoon
- Kornel Lesiński
- Lee Jeffery
- Leif Arne Storset
- Lennart Kudling
- llogiq
- Manish Goregaokar
- Marc-Antoine Perennou
- Marcus Klaas
- Marko Lalic
- Martin Wernstål
- Matěj Grabovský
- Matej Lach
- Matt Brubeck
- Matt Friedman
- Michael Choate
- Michael Layzell
- Michael Macias
- Michael McConville
- Michael Neumann
- Mickaël Salaün
- midinastasurazz
- Mike Marcacci
- mitaa
- Ms2ger
- Murarth
- Nathan Kleyn
- Nicholas Seckar
- Nick Cameron
- Nick Howell
- Niko Matsakis
- Nikolay Kondratyev
- Niranjan Padmanabhan
- Overmind JIANG
- Pascal Hertleif
- Peter Reid
- Remi Rampin
- Richard Diamond
- Robin Kruppe
- Ruby
- Ryo Munakata
- Scott Olson
- Sean Bowe
- Sean McArthur
- Sébastien Marie
- Simon Mazur
- Simon Sapin
- Simonas Kazlauskas
- Stepan Koltsov
- Steve Klabnik
- Steven Fackler
- Sylvestre Ledru
- Taliesin Beynon
- Tamir Duberstein
- Tim Cuthbertson
- Tim JIANG
- Tim Neumann
- Tobias Bucher
- Tshepang Lekhonkhobe
- Ulrik Sverdrup
- Vadim Chugunov
- Vadim Petrochenkov
- Viacheslav Chimishuk
- Victor Berger
- Vincent Bernat
- Vladimir Rutsky
- w00ns
- William Throwe
- Without Boats
- Xiao Chuan Yu
