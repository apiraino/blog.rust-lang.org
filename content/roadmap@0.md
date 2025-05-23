+++
path = "2017/02/06/roadmap"
title = "Rust's 2017 roadmap"
authors = ["Aaron Turon"]
description = "What the Rust community hopes to get done in 2017"
aliases = ["2017/02/06/roadmap.html"]
+++

Starting with 2017, Rust is following an [open roadmap process] for setting our
aims for the year. The process is coordinated with [the survey] and
[production user outreach], to make sure our goals are aligned with the needs of
Rust's users. It culminates in a [community-wide discussion] and ultimately
[an RFC] laying out a vision.

[open roadmap process]: https://github.com/rust-lang/rfcs/pull/1728
[the survey]: https://blog.rust-lang.org/2016/06/30/State-of-Rust-Survey-2016.html
[production user outreach]: https://internals.rust-lang.org/t/2016-rust-commercial-user-survey-results/4317
[community-wide discussion]: https://internals.rust-lang.org/t/setting-our-vision-for-the-2017-cycle/3958?u=aturon
[an RFC]: https://github.com/rust-lang/rfcs/pull/1774

**This year, the overarching theme is *productivity*, especially for early-stage
Rust users**. From tooling to libraries to documentation to the core language,
we want to make it easier to get things done with Rust.

A focus on productivity might seem at odds with some of Rust's other
goals. After all, Rust has focused on reliability and performance, and it's easy
to imagine that achieving those aims forces compromises elsewhere—like the
learning curve, or developer productivity. Is "fighting with the borrow checker"
an inherent rite of passage for budding Rustaceans? Does removing papercuts and
small complexities entail glossing over safety holes or performance cliffs?

Our approach with Rust has always been to bend the curve around tradeoffs, as
embodied in the various pillars we've talked about on this blog:

- [Memory safety without garbage collection](https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html)
- [Concurrency without data races](https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html)
- [Abstraction without overhead](https://blog.rust-lang.org/2015/05/11/traits.html)
- [Stability without stagnation](https://blog.rust-lang.org/2014/10/30/Stability.html)

In the core language we've sometimes been able to leverage things like Rust's
ownership model to make features easier to use. For example, [closures in Rust],
unlike those in C++, do not require explicit "capture clauses"; Rust's ownership
tracking means that we can automatically infer whether to move or borrow data
into the closure, without sacrificing reliability or performance. We've also
been working to make the language easier to learn by improving the
[compiler's error messages]. And [Cargo] is an example of early focus on
productivity that has only enhanced the power of systems programming with Rust,
allowing for [OS](https://os.phil-opp.com/)
[projects](https://intermezzos.github.io/) to easily build and leverage an
ecosystem of shared libraries. There's so much more we can do along these lines!

[Cargo]: https://blog.rust-lang.org/2016/05/05/cargo-pillars.html
[closures in Rust]: https://huonw.github.io/blog/2015/05/finding-closure-in-rust/
[compiler's error messages]: https://blog.rust-lang.org/2016/08/10/Shape-of-errors-to-come.html

In short, **productivity should be a core value of Rust**, and we should work
creatively to improve it while retaining Rust's other core values. By the end of
2017, we want to have earned the slogan:

- Rust: fast, reliable, productive—pick three.

## The roadmap

With that framing in mind, here's Rust's vision for 2017 in a nutshell. Each
statement links to a corresponding tracker with more details:

* [**Rust should have a lower learning curve**](https://github.com/rust-lang/rust-roadmap/issues/3). Plans
  include a [new book](https://github.com/aturon/rust-roadmap/issues/7),
  collecting examples and patterns, improving errors,
  [improving the core language](https://github.com/aturon/rust-roadmap/issues/17),
  and [building IDEs](https://github.com/rust-lang/rust-roadmap/issues/2).

* [**Rust should have a pleasant edit-compile-debug cycle**](https://github.com/rust-lang/rust-roadmap/issues/1). Plans
  include
  [incremental compilation](https://blog.rust-lang.org/2016/09/08/incremental.html) and
  a [trait system overhaul](https://github.com/aturon/rust-roadmap/issues/8).

* [**Rust should provide a solid, but basic IDE experience**](https://github.com/rust-lang/rust-roadmap/issues/2). Plans
  are focused on the
  [Rust language service](https://github.com/aturon/rust-roadmap/issues/6).

* [**Rust should provide easy access to high quality crates**](https://github.com/rust-lang/rust-roadmap/issues/9). Plans
  include
  [crate categories](https://www.reddit.com/r/rust/comments/5r72aj/cratesio_has_categories/),
  [crate ranking](https://github.com/rust-lang/rfcs/pull/1824), open-ended
  testing and benchmarking frameworks, new API and documentation guidelines, and
  [guidelines for unsafe code](https://github.com/rust-lang/rfcs/pull/1643).

* [**Rust should be well-equipped for writing robust, high-scale servers**](https://github.com/rust-lang/rust-roadmap/issues/10). Plans
  mostly focus on the [Tokio project](https://tokio.rs/) for asynchronous I/O and
  potentially
  [async/await notation](https://github.com/rust-lang/rfcs/pull/1823).

* [**Rust should have 1.0-level crates for essential tasks**](https://github.com/rust-lang/rust-roadmap/issues/11). Plans
  are under way to focus the libs team and the community on a number of
  important existing crates, to help polish them to 1.0 quality by the end of
  the year.

* [**Rust should integrate easily into large build systems**](https://github.com/rust-lang/rust-roadmap/issues/12). Plans
  include working with large companies incorporating Rust to understand how best
  to equip Cargo to smooth the process.

* [**Rust's community should provide mentoring at all levels**](https://github.com/rust-lang/rust-roadmap/issues/13). Plans
  include the
  [RustBridge project](https://github.com/rust-community/rustbridge/) and new
  [team shepherds](https://internals.rust-lang.org/t/language-team-shepherds/4595).

In addition to these primary goals, we have highlighted two areas we'd like to explore, but where the end point is less clear:

* [Integration with other languages, running the gamut from C to JavaScript](https://github.com/rust-lang/rust-roadmap/issues/14).
* [Usage in resource-constrained environments](https://github.com/rust-lang/rust-roadmap/issues/15).

The various Rust subteams are actively working on projects tied to these
goals. You can track progress or jump in by watching and commenting on the
[roadmap tracker](https://github.com/rust-lang/rust-roadmap). And a fully
detailed rationale for the roadmap is in
[the RFC](https://github.com/rust-lang/rfcs/pull/1774).

As the year progresses, expect to see many more blog posts announcing
roadmap-related initiatives and milestones. Around the end of the year, we'll
publish a **retrospective** aggregating the progress of the year and providing a
guide to the current state of Rust.

In the meantime—please
[get involved](https://github.com/rust-lang/rust-roadmap)! We have ambitious
goals for the year, and we need all the help we can get.
