+++
path = "2015/05/15/Rust-1.0"
title = "Announcing Rust 1.0"
authors = ["The Rust Core Team"]
aliases = [
    "2015/05/15/Rust-1.0.html",
    "releases/1.0.0",
]

[extra]
release = true
+++

Today we are very proud to announce the
[1.0 release of Rust][relnotes], a new programming language aiming to
make it easier to build reliable, efficient systems. **Rust combines
low-level control over performance with high-level convenience and
safety guarantees**. Better yet, it achieves these goals without
requiring a garbage collector or runtime, making it possible to
[use Rust libraries as a "drop-in replacement" for C][ffi]. If you'd
like to experiment with Rust, the
["Getting Started" section of the Rust book][book] is your best bet
(if you prefer to use an e-reader, Pascal Hertleif maintains
[unofficial e-book versions][ebook] as well).

What makes Rust different from other languages is its type system,
which represents a refinement and codification of "best practices"
that have been hammered out by generations of C and C++
programmers. As such, Rust has something to offer for both experienced
systems programmers and newcomers alike: experienced programmers will
find they save time they would have spent debugging, whereas newcomers
can write low-level code without worrying about minor mistakes leading
to mysterious crashes.

### What does it mean for Rust to be 1.0?

The current Rust language is the result of a lot of iteration and
experimentation. The process has worked out well for us: Rust today is
both simpler and more powerful than we originally thought would be
possible. But all that experimentation also made it difficult to
maintain projects written in Rust, since the language and standard
library were constantly changing.

**The 1.0 release marks the end of that churn.** This release is the
official beginning of our [commitment to stability][stable], and as
such it offers a firm foundation for building applications and
libraries. From this point forward, breaking changes are largely out
of scope (some [minor][minor] [caveats] apply, such as compiler bugs).

That said, releasing 1.0 doesn't mean that the Rust language is
"done". We have many [improvements in store][priorities]. In fact, the
Nightly builds of Rust already demonstrate [improvements to][24965]
[compile][24615] [times][25323] (with more to come) and includes work
on new APIs and language features, like [`std::fs`][1044] and
[associated constants][23606].

To help ensure that compiler and language improvements make their way
out into the ecosystem at large as quickly as possible, we've adopted
a [train-based][train] release model. This means that we'll be issuing
regular releases every six weeks, just like the Firefox and Chrome web
browsers. **To kick off that process, we are also releasing Rust 1.1
beta today, simultaneously with Rust 1.0.**

### Cargo and crates.io

Building a real project is about more than just writing code -- it's
also about managing dependencies. [Cargo][cargo], the Rust package
manager and build system, is designed to make this easy.  Using Cargo,
downloading and installing new libraries is as simple as adding one
line to your manifest.

Of course, to use a dependency, you first have to find it. This is
where [crates.io] comes in -- crates.io is a central package
repository for Rust code. It makes it easy to search for other
people's packages or to publish your own.

Since we [announced cargo and crates.io][cargo] approximately six
months ago, the number of packages has been growing
steadily. Nonetheless, it's still early days, and there are still lots
of great packages yet to be written. If you're interested in building
a library that will take the Rust world by storm, there's no time like
the present!

### Open Source and Open Governance

Rust has been an open-source project from the start. Over the last few
years, we've been constantly looking for ways to make our governance
more open and community driven. Since we introduced the
[RFC process][rfcs] a little over a year ago, all major decisions
about Rust are written up and discussed in the open in the form of an
RFC. Recently, we adopted a [new governance model][1068], which
establishes a set of subteams, each responsible for RFCs in one
particular area. If you'd like help shape the future of Rust, we
encourage you to get involved, either by uploading libraries to
[crates.io], commenting on RFCs, or
[writing code for Rust itself][contributing].

We'd like to give a special thank you to the following people, each of
whom contributed changes since our previous release (the
[complete list of contributors][AUTHORS] is here):

- `Aaron Gallagher`
- `Aaron Turon`
- `Abhishek Chanda`
- `Adolfo Ochagavía`
- `Alex Burka`
- `Alex Crichton`
- `Alex Quach`
- `Alexander Polakov`
- `Andrea Canciani`
- `Andreas Martens`
- `Andreas Tolfsen`
- `Andrei Oprea`
- `Andrew Paseltiner`
- `Andrew Seidl`
- `Andrew Straw`
- `Andrzej Janik`
- `Aram Visser`
- `Ariel Ben-Yehuda`
- `Augusto Hack`
- `Avdi Grimm`
- `Barosl Lee`
- `Ben Ashford`
- `Ben Gesoff`
- `Björn Steinbrink`
- `Brad King`
- `Brendan Graetz`
- `Brett Cannon`
- `Brian Anderson`
- `Brian Campbell`
- `Carlos Galarza`
- `Carol (Nichols || Goulding)`
- `Carol Nichols`
- `Chris Morgan`
- `Chris Wong`
- `Christopher Chambers`
- `Clark Gaebel`
- `Cole Reynolds`
- `Colin Walters`
- `Conrad Kleinespel`
- `Corey Farwell`
- `Dan Callahan`
- `Dave Huseby`
- `David Reid`
- `Diggory Hardy`
- `Dominic van Berkel`
- `Dominick Allen`
- `Don Petersen`
- `Dzmitry Malyshau`
- `Earl St Sauver`
- `Eduard Burtescu`
- `Erick Tryzelaar`
- `Felix S. Klock II`
- `Florian Hahn`
- `Florian Hartwig`
- `Franziska Hinkelmann`
- `FuGangqiang`
- `Garming Sam`
- `Geoffrey Thomas`
- `Geoffry Song`
- `Gleb Kozyrev`
- `Graydon Hoare`
- `Guillaume Gomez`
- `Hajime Morrita`
- `Hech`
- `Heejong Ahn`
- `Hika Hibariya`
- `Huon Wilson`
- `Igor Strebezhev`
- `Isaac Ge`
- `J Bailey`
- `Jake Goulding`
- `James Miller`
- `James Perry`
- `Jan Andersson`
- `Jan Bujak`
- `Jan-Erik Rediger`
- `Jannis Redmann`
- `Jason Yeo`
- `Johann`
- `Johann Hofmann`
- `Johannes Oertel`
- `John Gallagher`
- `John Van Enk`
- `Jonathan S`
- `Jordan Humphreys`
- `Joseph Crail`
- `Josh Triplett`
- `Kang Seonghoon`
- `Keegan McAllister`
- `Kelvin Ly`
- `Kevin Ballard`
- `Kevin Butler`
- `Kevin Mehall`
- `Krzysztof Drewniak`
- `Lee Aronson`
- `Lee Jeffery`
- `Liam Monahan`
- `Liigo Zhuang`
- `Luke Gallagher`
- `Luqman Aden`
- `Manish Goregaokar`
- `Manuel Hoffmann`
- `Marin Atanasov Nikolov`
- `Mark Mossberg`
- `Marvin Löbel`
- `Mathieu Rochette`
- `Mathijs van de Nes`
- `Matt Brubeck`
- `Michael Alexander`
- `Michael Macias`
- `Michael Park`
- `Michael Rosenberg`
- `Michael Sproul`
- `Michael Woerister`
- `Michael Wu`
- `Michał Czardybon`
- `Mickaël Salaün`
- `Mike Boutin`
- `Mike Sampson`
- `Ms2ger`
- `Nelo Onyiah`
- `Nicholas`
- `Nicholas Mazzuca`
- `Nick Cameron`
- `Nick Hamann`
- `Nick Platt`
- `Niko Matsakis`
- `Oak`
- `Oliver Schneider`
- `P1start`
- `Pascal Hertleif`
- `Paul Banks`
- `Paul Faria`
- `Paul Quint`
- `Pete Hunt`
- `Peter Marheine`
- `Phil Dawes`
- `Philip Munksgaard`
- `Piotr Czarnecki`
- `Piotr Szotkowski`
- `Poga Po`
- `Przemysław Wesołek`
- `Ralph Giles`
- `Raphael Speyer`
- `Remi Rampin`
- `Ricardo Martins`
- `Richo Healey`
- `Rob Young`
- `Robin Kruppe`
- `Robin Stocker`
- `Rory O’Kane`
- `Ruud van Asseldonk`
- `Ryan Prichard`
- `Scott Olson`
- `Sean Bowe`
- `Sean McArthur`
- `Sean Patrick Santos`
- `Seo Sanghyeon`
- `Shmuale Mark`
- `Simon Kern`
- `Simon Sapin`
- `Simonas Kazlauskas`
- `Sindre Johansen`
- `Skyler`
- `Steve Klabnik`
- `Steven Allen`
- `Swaroop C H`
- `Sébastien Marie`
- `Tamir Duberstein`
- `Tero Hänninen`
- `Theo Belaire`
- `Theo Belaire`
- `Thiago Carvalho`
- `Thomas Jespersen`
- `Tibor Benke`
- `Tim Cuthbertson`
- `Tincan`
- `Ting-Yu Lin`
- `Tobias Bucher`
- `Toni Cárdenas`
- `Tshepang Lekhonkhobe`
- `Ulrik Sverdrup`
- `Vadim Chugunov`
- `Vadim Petrochenkov`
- `Valerii Hiora`
- `Wangshan Lu`
- `Wei-Ming Yang`
- `Will`
- `Will Hipschman`
- `Wojciech Ogrodowczyk`
- `Xue Fuqiao`
- `Xuefeng Wu`
- `York Xiang`
- `Young Wu`
- `bcoopers`
- `critiqjo`
- `diwic`
- `fenduru`
- `gareins`
- `github-monoculture`
- `inrustwetrust`
- `jooert`
- `kgv`
- `klutzy`
- `kwantam`
- `leunggamciu`
- `mdinger`
- `nwin`
- `pez`
- `robertfoss`
- `rundrop1`
- `sinkuu`
- `tynopex`
- `Łukasz Niemier`
- `らいどっと`

[stable]: https://blog.rust-lang.org/2014/10/30/Stability.html
[train]: https://blog.rust-lang.org/2014/12/12/1.0-Timeline.html
[traits]: https://blog.rust-lang.org/2015/05/11/traits.html
[rfcs]: https://github.com/rust-lang/rfcs/blob/master/README.md
[1068]: https://github.com/rust-lang/rfcs/pull/1068
[contributing]: https://github.com/rust-lang/rust/blob/master/CONTRIBUTING.md
[hb]: https://en.wikipedia.org/wiki/Heisenbug
[priorities]: https://internals.rust-lang.org/t/priorities-after-1-0/1901
[crates.io]: https://crates.io/
[cargo]: https://blog.rust-lang.org/2014/11/20/Cargo.html
[relnotes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-100-may-2015
[caveats]: https://github.com/rust-lang/rfcs/pull/1122
[book]: https://doc.rust-lang.org/1.0.0/book/getting-started.html
[ffi]: https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html
[AUTHORS]: https://github.com/rust-lang/rust/graphs/contributors
[23606]: https://github.com/rust-lang/rust/pull/23606/
[1044]: https://github.com/rust-lang/rfcs/pull/1044
[24965]: https://github.com/rust-lang/rust/pull/24965
[24615]: https://github.com/rust-lang/rust/pull/24615
[25323]: https://github.com/rust-lang/rust/pull/25323
[ebook]: https://killercup.github.io/trpl-ebook/
[minor]: https://github.com/rust-lang/rfcs/pull/1105
