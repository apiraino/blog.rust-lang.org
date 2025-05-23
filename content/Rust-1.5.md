+++
path = "2015/12/10/Rust-1.5"
title = "Announcing Rust 1.5"
authors = ["The Rust Core Team"]
aliases = [
    "2015/12/10/Rust-1.5.html",
    "releases/1.5.0",
]

[extra]
release = true
+++

Today we're releasing [Rust 1.5 stable][install]. This post gives the
highlights, and you can find the full details in the
[release notes][notes].

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-150-2015-12-10

### What's in 1.5 stable

The biggest news with Rust 1.5 is the introduction of `cargo install`,
a new subcommand that installs Cargo application packages on the local
system. This tool offers a painless way to distribute Rust applications.

The community is already taking advantage of `cargo install` to
install applications like
[rustfmt](https://github.com/rust-lang-nursery/rustfmt), the
work-in-progress code formatting tool for Rust. Moreover, `cargo install`
can also be used to install new subcommands for Cargo itself:

* `cargo-check`: statically check a project, but don't build a binary.
* `cargo-edit`: add or remove dependencies for a project through the command line.
* `cargo-graph`: build dependency graphs for a project using GraphViz.
* `cargo-watch`: automatically re-run a Cargo command when the project changes.

(You can find more with a [crates.io search](https://crates.io/search?q=subcommand).)

In addition to these tooling changes, Rust 1.5 sees a large number of
library API stabilizations, especially around the interaction of paths
and the file system.

Finally, there were a few improvements to compile times, and crate
metadata shrunk
[by about 20%](https://github.com/rust-lang/rust/pull/28521).

### Contributors to 1.5

The Rust community continues to do incredible work, and we'd like to
thank the 152 contributors to this release:

- Aaron Turon
- Adolfo Ochagavía
- Ahmed Charles
- Aidan Hobson Sayers
- Aleksey Kladov
- Alex Burka
- Alex Crichton
- Alex Gaynor
- Alexis Beingessner
- Alfie John
- Amit Aryeh Levy
- Andre Bogus
- Andrea Canciani
- Andreas Sommer
- Andrew Chin
- Andrew Paseltiner
- Ariel Ben-Yehuda
- Barosl Lee
- Bastien Dejean
- Ben S
- Ben Sago
- Björn Steinbrink
- Boris Egorov
- Brian Anderson
- Bryce Van Dyk
- Carlos Liam
- Carol (Nichols || Goulding)
- Charlotte Spencer
- Chris C Cerami
- Chris Drake
- Chris Wong
- Colin Wallace
- Corentin Henry
- Corey Farwell
- Craig Hills
- Cristi Cobzarenco
- Cristian Kubis
- Dan W.
- Daniel Carral
- Daniel Keep
- Dato Simó
- David Elliott
- David Ripton
- David Szotten
- DenisKolodin
- Dominik Inführ
- Dongie Agnir
- Eduard Burtescu
- Eli Friedman
- Eljay
- Emanuel Czirai
- Fabiano Beselga
- Felix S. Klock II
- Florian Hahn
- Florian Hartwig
- Garming Sam
- Gavin Baker
- Gleb Kozyrev
- Guillaume Gomez
- Huon Wilson
- Irving A.J. Rivas Z.
- J. Ryan Stinnett
- Jack Wilson
- James Bell
- James McGlashan
- Jan Likar
- Jan-Erik Rediger
- Jed Davis
- Jethro Beekman
- John Hodge
- Jonas Schievink
- Jonathan Hansford
- Jorge Aparicio
- Jose Narvaez
- Joseph Caudle
- Keshav Kini
- Kevin Butler
- Kevin Yap
- Kyle Robinson Young
- Lee Jeffery
- Lee Jenkins
- Lennart Kudling
- Liigo Zhuang
- Luqman Aden
- Manish Goregaokar
- Marcello Seri
- Marcus Klaas
- Martin Pool
- Matt Brubeck
- Michael Howell
- Michael Layzell
- Michael Pankov
- Ms2ger
- Nick Cameron
- Nick Hamann
- Nick Howell
- Niko Matsakis
- Oliver Schneider
- Peter Atashian
- Peter Marheine
- Philipp Oppermann
- Remi Rampin
- Reza Akhavan
- Ricardo Signes
- Richard Diamond
- Robert Gardner
- Robin Kruppe
- Ruud van Asseldonk
- Ryan Scheel
- Scott Olson
- Sean Bowe
- Sebastian Wicki
- Seeker14491
- Seo Sanghyeon
- Simon Mazur
- Simon Sapin
- Simonas Kazlauskas
- Stefan O'Rear
- Steve Klabnik
- Steven Allen
- Steven Fackler
- Sébastien Marie
- Ted Mielczarek
- Tobias Bucher
- Tshepang Lekhonkhobe
- Ulrik Sverdrup
- Utkarsh Kukreti
- Vadim Chugunov
- Vadim Petrochenkov
- Vitali Haravy
- Vladimir Rutsky
- Wesley Wiser
- Will Speak
- William Throwe
- Willy Aguirre
- Xavier Shay
- Yoshito Komatsu
- arcnmx
- arthurprs
- billpmurphy
- bors
- christopherdumas
- critiqjo
- glendc
- kickinbahk
- llogiq
- mdinger
- nwin
- nxnfufunezn
- panicbit
- skeleten
- whitequark
