+++
path = "2018/03/12/roadmap"
title = "Rust's 2018 roadmap"
authors = ["The Rust Core Team"]
aliases = ["2018/03/12/roadmap.html"]
+++

Each year the Rust community [comes together][roadmap-process] to set out a
roadmap. This year, in addition to the [survey], we put out
a [call for blog posts][blog-2018] in December, which resulted
in [100 blog posts][read-rust] written over the span of a few weeks. The end
result is the recently-merged [2018 roadmap RFC][rfc].

[roadmap-process]: https://github.com/rust-lang/rfcs/pull/1728
[survey]: https://blog.rust-lang.org/2017/09/05/Rust-2017-Survey-Results.html
[blog-2018]: https://blog.rust-lang.org/2018/01/03/new-years-rust-a-call-for-community-blogposts.html
[read-rust]: https://readrust.net/rust-2018/
[rfc]: https://github.com/rust-lang/rfcs/pull/2314

## Rust: 2018 edition

**This year, we will deliver _Rust 2018_, marking the first major new edition of
Rust since 1.0** (aka Rust 2015).

We will continue to publish releases every six weeks as usual. But we will
designate a release in the latter third of the year (Rust 1.29 - 1.31) as *Rust
2018*. This new "edition" of Rust will be the culmination of feature
stabilization throughout the year, and will ship with polished documentation,
tooling, and libraries that tie in to those features.

The idea of editions is to signify major steps in Rust’s evolution, where a
collection of new features or idioms, taken as a whole, changes the experience
of using Rust. They’re a chance, every few years, to take stock of the work
we’ve delivered in six-week increments. To tell a bigger story about where Rust
is going. And to ship the whole stack as a polished product.

We expect that each edition will have a core theme or focus. Thinking of 1.0 as
"Rust 2015", we have:

-   Rust 2015: [stability](https://blog.rust-lang.org/2014/09/15/Rust-1.0.html)
-   Rust 2018: productivity

## What will be in Rust 2018?

The roadmap doesn’t say _for certain_ what will ship in Rust 2018, but we have a
pretty good idea, and we’ll cover the major suspects below.

### Documentation improvements

Part of the goal with the Rust 2018 release is to provide high quality
documentation for the full set of new and improved features and the idioms they
give rise to. [The Rust Programming Language book][trpl] has been completely
re-written over the last 18 months, and will be updated throughout the year as
features reach the stable compiler. [Rust By Example] will likewise undergo a
revamp this year. And there are numerous third party books, like [Programming
Rust], reaching print as well.

[trpl]: https://doc.rust-lang.org/nightly/book/second-edition/
[Programming Rust]: http://shop.oreilly.com/product/0636920040385.do
[Rust By Example]: https://rustbyexample.com/

### Language improvements

The most prominent language work in the pipeline stems from [2017’s ergonomics
initiative]. Almost all of the accepted RFCs from the initiative are available
on nightly today, and will be polished and stabilized over the next several
months. Among these productivity improvements are a few “headliners” that will
form the backbone of the release:

[2017’s ergonomics initiative]: https://blog.rust-lang.org/2017/03/02/lang-ergonomics.html

- **Ownership system improvements**, including making borrowing more flexible
  via “non-lexical lifetimes”, improved pattern matching integration, and more.
- **Trait system improvements**, including the long-awaited `impl Trait` syntax
  for dealing with types abstractly.
- **Module system improvements**, focused on increasing clarity and reducing
  complexity.
- **Generators/async/await:** work is rapidly progressing on first-class async
  programming support.

In addition, we anticipate a few more major features to stabilize prior to the
Rust 2018 release, including **SIMD**, **custom allocators**, and **macros 2.0**.

### Compiler improvements

As of [Rust 1.24](https://blog.rust-lang.org/2018/02/15/Rust-1.24.html),
incremental recompilation is available and enabled by default on the stable
compiler. This feature already makes rebuilds significantly faster than fresh
builds, but over the course of the year we expect continued improvements for
_both_ fresh and re-builds. Compiler performance should not be an obstacle to
productivity in Rust 2018.

### Tooling improvements

Rust 2018 will see high quality 1.0 releases of the [Rust Language Server] ("RLS",
which underlies much of our IDE integration story) and [`rustfmt`] (a standard
formatting tool for Rust code). We will continue to improve Cargo by stabilizing
custom registries, public dependencies, and a revised profile system. We’re also
expecting further work on [Cargo build system integration], [Xargo integration],
and [custom test frameworks], though it’s unclear as yet how many of these will
be complete prior to Rust 2018.

[Rust Language Server]: https://github.com/rust-lang-nursery/rls
[Cargo build system integration]: https://github.com/rust-lang/rfcs/pull/2136
[Xargo integration]: https://github.com/rust-lang/cargo/issues/4959
[custom test frameworks]: https://github.com/rust-lang/rfcs/pull/2318
[`rustfmt`]: https://github.com/rust-lang-nursery/rustfmt

### Library improvements

Building on [our work from last year][blitz], we will publish a 1.0 version of
the [Rust API guidelines book], continue pushing important libraries to 1.0
status, improve discoverability through a revamped cookbook effort, and make
heavy investments in libraries in specific domains—as we’ll see below.

[blitz]: https://blog.rust-lang.org/2017/05/05/libz-blitz.html
[Rust API guidelines book]: https://github.com/rust-lang-nursery/api-guidelines

### Web site improvements

As part of Rust 2018, we will completely overhaul the Rust web site, making it
useful for CTOs and engineers alike. It should be far easier to find information
to help evaluate Rust for your use case, and to stay up to date with the latest
tooling and ecosystem improvements.

### Four target domains

Part of our goal with Rust 2018 is to demonstrate Rust’s productivity in
specific domains of use. We’ve selected four such domains to invest in and
highlight this year:

- **Network services**. Rust’s reliability and low footprint make it an
  excellent match for network services and infrastructure, especially at high
  scale.
- **[Command-line apps]** (CLI). Rust’s portability, reliability, ergonomics, and ability to
  produce static binaries come together to great effect for writing CLI apps.
- **[WebAssembly]**. The “wasm” web standard allows shipping native-like binaries
  to all major browsers, but GC support is still years away. Rust
  is [extremely well positioned](https://mgattozzi.com/rust-wasm) to target this
  domain, and provides a reasonable on-ramp for programmers coming from JS.
- **[Embedded devices]**. Rust has the potential to make programming
  resource-constrained devices much more productive—and fun! We want embedded
  programming to reach first-class status this year.

[Command-line apps]: https://internals.rust-lang.org/t/announcing-the-cli-working-group/6872
[Embedded devices]: https://internals.rust-lang.org/t/announcing-the-embedded-devices-working-group/6839
[WebAssembly]: https://internals.rust-lang.org/t/come-join-the-rust-and-webassembly-working-group/6845

Each of these domains has a dedicated working group for the year. These WGs will
work in a cross-cutting fashion, interfacing with language, tooling, library,
and documentation work.

### Compatibility across editions

**TL;DR: Rust will continue its stability guarantee
of [hassle-free updates to new versions][stability]**.

[stability]: https://blog.rust-lang.org/2014/10/30/Stability.html

Editions will have a meaning for the compiler. You will be able to write:

```toml
edition = "2018"
```

in your Cargo.toml to _opt in_ to the new edition for your crate. Doing so may
introduce new keywords or otherwise require adjustments to code. However:

-   You can use _old_ editions indefinitely on _new_ compilers; **editions are
    opt-in**.
-   Editions are set on a _per-crate_ basis and can be mixed and matched; **you
    can be on a different edition from your dependencies**.
-   Warning-free code in one edition must compile, and have the same behavior, on
    the next.
-   Edition-related warnings, e.g. that an identifier will become a keyword in the
    next edition, must be easily fixable via an automated migration tool
    (rustfix). **Only a small minority of crates should require _any_
    manual work to opt in to a new edition**, and that manual work must be
    minimal.
-   Most new features are edition-independent, and will be usable on new compilers
    even when an older edition is selected.

In other words, the progression of new compiler versions is independent from
editions; you can migrate at your leisure, and don’t have to worry about ecosystem
compatibility; and edition migration is normally trivial.

## Additional 2018 goals

While the Rust 2018 release is our major focus this year, there are some
additional ongoing concerns that we want to give attention to.

### Better serving intermediate Rustaceans

One of the strongest messages we’ve heard from production users, and [the 2017
survey], is that people need more resources to take them from understanding
Rust’s concepts to knowing how to use them _effectively_. The roadmap does not
stipulate exactly what these resources should look like
— [probably there should be several kinds][intermediate] — but commits us as a
community to putting significant work into this space, and ending the year with
some solid new material.

[the 2017 survey]: https://blog.rust-lang.org/2017/09/05/Rust-2017-Survey-Results.html
[intermediate]: https://quietmisdreavus.net/code/2018/01/10/not-a-layer-cake-analogy/

### Community

**Connect and empower Rust's global community**. We will pursue
internationalization as a first-class concern, and proactively work to build
ties between Rust subcommunities currently separated by language, geography, or
culture. We will spin up and support Rust events worldwide, including further
growth of the RustBridge program.

**Grow Rust's teams and new leaders within them**. We will refactor the Rust
team structure to support more scale, agility, and leadership growth. We will
systematically invest in mentoring, both by creating more on-ramp resources and
through direct mentorship relationships.

## A call to action

As always in the Rust world, the goals laid out here will ultimately be the
result of a community-wide effort—maybe one including you! Here are some of the
teams where we could use the most help. Note that all IRC channels refer to the
irc.mozilla.org network.

- **WebAssembly WG**. Compiling Rust to WebAssembly should be _the_ best choice for fast code on the Web. Check out [rust-lang-nursery/rust-wasm](https://github.com/rust-lang-nursery/rust-wasm) to learn more and get involved!
- **CLI WG**. Writing CLI apps in Rust should be a frictionless experience--from finding the right libraries and writing concise integration tests up to cross-platform distribution. Join us at [rust-lang-nursery/cli-wg](https://github.com/rust-lang-nursery/cli-wg) and help us reach that goal!
- **Embedded Devices WG**. Quality, productivity, accessibility: Rust can change the embedded industry for the better. Let's get this process started in 2018! Join us at [https://github.com/rust-lang-nursery/embedded-wg](https://github.com/rust-lang-nursery/embedded-wg)
- **Ecosystem WG**. We'll be providing guidance and support to important crates throughout the ecosystem. Drop into the [WG-ecosystem room](https://gitter.im/rust-lang/WG-ecosystem) and we'll guide you to places that need help!
- **Dev Tools Team**. There are always interesting things to tackle with developer tools  (IDEs, Cargo, rustdoc, Clippy, Rustfmt, custom test frameworks, and more). Drop in to #rust-dev-tools and have a chat with the team!
- **Rustdoc Team**. With your help, we can make documentation better for everyone. Come join us in #rustdoc on IRC, and we can help you get started!
- **Release Team**. Drop by #rust-release on IRC to get involved with regression triage and release production!
- **Community Team**. We've kicked off several new Teams within the Community Team
  and are eager to add new members: Events, Content, Switchboard, RustBridge, Survey,
  and Localization! [Check out our team repo] or stop by our IRC channel, #rust-community,
  to learn more and get involved!

[Check out our team repo]: https://github.com/rust-community/team
