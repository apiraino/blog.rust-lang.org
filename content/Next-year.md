+++
path = "2015/08/14/Next-year"
title = "Rust in 2016"
authors = ["Nicholas Matsakis and Aaron Turon"]
description = "Our vision for Rust's next year"
aliases = ["2015/08/14/Next-year.html"]
+++

This week marks three months since Rust 1.0 was released. As we're starting to
hit our post-1.0 stride, we'd like to talk about **what 1.0 meant in hindsight,
and where we see Rust going in the next year**.

### What 1.0 was about

Rust 1.0 focused on stability, community, and clarity.

* **Stability**, we've discussed quite a bit in [previous posts][deliverable]
introducing our release channels and stabilization process.

* **Community** has always been one of Rust's greatest strengths. But in the year
leading up to 1.0, we introduced and refined the [RFC process][rfcs],
culminating with [subteams][subteams] to manage RFCs in each particular
area. Community-wide debate on RFCs was indispensable for delivering a quality
1.0 release.

* All of this refinement prior to 1.0 was in service of reaching **clarity** on
what Rust represents:

    - [Memory safety without garbage collection][fearless]
    - [Concurrency without data races][fearless]
    - [Abstraction without overhead][traits]
    - [Stability without stagnation][deliverable]

Altogether, **Rust is exciting because it is empowering: you can hack without
fear**. And you can do so in contexts you might not have before, dropping down
from languages like Ruby or Python, making your first foray into systems
programming.

That's Rust 1.0; but what comes next?

[deliverable]: https://blog.rust-lang.org/2014/10/30/Stability.html
[rfcs]: https://github.com/rust-lang/rfcs#when-you-need-to-follow-this-process
[subteams]: https://github.com/rust-lang/rfcs/pull/1068
[fearless]: https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html
[traits]: https://blog.rust-lang.org/2015/05/11/traits.html

### Where we go from here

After much discussion within the core team, early production users, and the
broader community, we've identified a number of improvements we'd like to make
over the course of the next year or so, falling into three categories:

- Doubling down on infrastructure;
- Zeroing in on gaps in key features;
- Branching out into new places to use Rust.

Let's look at some of the biggest plans in each of these categories.

### Doubling down: infrastructure investments

#### Crater

Our basic [stability promise][deliverable] for Rust is that upgrades
between versions are "hassle-free". To deliver on this promise, we need to
detect compiler bugs that cause code to stop working. Naturally, the compiler
has its own large test suite, but that is only a small fraction of the code
that's out there "in the wild". **[Crater] is a tool that aims to close that gap
by testing the compiler against all the packages found in [crates.io], giving us
a much better idea whether any code has stopped compiling on the latest nightly.**

Crater has quickly become an indispensable tool. We regularly compare the
nightly release against the latest stable build, and we use crater to check
in-progress branches and estimate the impact of a change.

Interestingly, we have often found that when code stops compiling, it's not
because of a bug in the compiler. Rather, it's because we *fixed* a bug, and
that code happened to be relying on the older behavior. Even in those cases,
using crater helps us improve the experience, by suggestion that we should phase
fixes in slowly with warnings.

Over the next year or so, we plan to improve crater in numerous ways:

- Extend the coverage to other platforms beyond Linux, and run test suites on
  covered libraries as well.
- Make it easier to use: leave an `@crater: test` comment to try out a PR.
- Produce a version of the tool that library authors can use to see effects of
  their changes on downstream code.
- Include code from other sources beyond [crates.io].

[Crater]: https://github.com/brson/taskcluster-crater

#### Incremental compilation

Rust has always had a "crate-wide" compilation
model. This means that the Rust compiler reads in all of the source files in
your crate at once. These are type-checked and then given to LLVM for
optimization. This approach is great for doing deep optimization, because it
gives LLVM full access to the entire set of code, allowing for more better
inlining, more precise analysis, and so forth. However, it can mean that
turnaround is slow: even if you only edit one function, we will recompile
everything. When projects get large, this can be a burden.

The incremental compilation project aims to change this by having the Rust
compiler save intermediate by-products and re-use them. This way, when you're
debugging a problem, or tweaking a code path, **you only have to recompile those
things that you have changed, which should make the "edit-compile-test" cycle
much faster**.

Part of this project is restructuring the compiler to introduce a new
intermediate representation, which we call [MIR][mir]. MIR is a simpler,
lower-level form of Rust code that boils down the more complex features, making
the rest of the compiler simpler. This is a crucial enabler for language changes
like non-lexical lifetimes (discussed in the next section).

[mir]: https://github.com/rust-lang/rfcs/pull/1211

#### IDE integration

Top-notch IDE support can help to make Rust even more
productive. Up until now, pioneering projects like [Racer][racer],
[Visual Rust][visualrust], and [Rust DT][rustdt] have been working largely
without compiler support. **We plan to extend the compiler to permit deeper
integration with IDEs and other tools**; the plan is to focus initially on two
IDEs, and then grow from there.

[syntax highlighting]: https://github.com/rust-lang/rust/blob/master/src/etc/CONFIGS.md
[racer]: https://github.com/phildawes/racer
[visualrust]: https://github.com/PistonDevelopers/VisualRust
[rustdt]: https://github.com/RustDT/RustDT

### Zeroing in: closing gaps in our key features

#### Specialization

The idea of zero-cost abstractions breaks down into two
separate goals, as identified by Stroustrup:

- What you don't use, you don't pay for.
- What you do use, you couldn't hand code any better.

Rust 1.0 has essentially achieved the first goal, both in terms of language
features and the standard library. But it doesn't quite manage to achieve the
second goal. Take the following trait, for example:

~~~~rust
pub trait Extend<A> {
    fn extend<T>(&mut self, iterable: T) where T: IntoIterator<Item=A>;
}
~~~~

The `Extend` trait provides a nice abstraction for inserting data from any kind of
iterator into a collection. But with traits today, that also means that each
collection can provide only one implementation that works for *all* iterator
types, which requires actually calling `.next()` repeatedly. In some cases, you
could hand code it better, e.g. by just calling `memcpy`.

To close this gap, we've proposed **[specialization][specialization], allowing
you to provide multiple, overlapping trait implementations as long as one is
clearly more specific than the other**. Aside from giving Rust a more complete
toolkit for zero-cost abstraction, specialization also improves its story for
code reuse. See [the RFC][specialization] for more details.

[specialization]: https://github.com/rust-lang/rfcs/pull/1210

#### Borrow checker improvements

The borrow checker is, in a way, the beating heart of Rust; it's the part of the
compiler that lets us achieve memory safety without garbage collection, by
catching use-after-free bugs and the like. But occasionally, the borrower
checker also "catches" non-bugs, like the following pattern:

~~~~rust
match map.find(&key) {
    Some(...) => { ... }
    None => {
        map.insert(key, new_value);
    }
}
~~~~

Code like the above snippet is perfectly fine, but the borrow checker struggles
with it today because the `map` variable is borrowed for the entire body of the
`match`, preventing it from being mutated by `insert`. We plan to address this
shortcoming soon by refactoring the borrow checker to view code in terms of
finer-grained ("non-lexical") regions -- a step made possible by the move to the
MIR mentioned above.

#### Plugins

There are some really neat things you can do in Rust today -- if you're willing
to use the Nightly channel. For example, the [regex crate][regex] comes with
macros that, at compile time, turn regular expressions directly into machine
code to match them. Or take the [rust-postgres-macros crate][postgres], which
checks strings for SQL syntax validity at compile time. Crates like these make
use of a highly-unstable compiler plugin system that currently exposes far too
many compiler internals. **We plan to propose a new plugin design that is more
robust and provides built-in support for hygienic macro expansion as well**.

[regex]: https://github.com/rust-lang/regex
[postgres]: https://github.com/sfackler/rust-postgres-macros

### Branching out: taking Rust to new places

#### Cross-compilation

While cross-compiling with Rust is possible today, it involves a lot of manual
configuration. **We're shooting for push-button cross-compiles**. The idea is
that compiling Rust code for another target should be easy:

1. Download a precompiled version of `libstd` for the target in question,
   if you don't already have it.
2. Execute `cargo build --target=foo`.
3. There is no step 3.

#### Cargo install

Cargo and [crates.io] is a really great tool for distributing
libraries, but it lacks any means to install executables.  [RFC 1200] describes a
simple addition to cargo, the `cargo install` command.  Much like the
conventional `make install`, **`cargo install` will place an executable in your
path so that you can run it**. This can serve as a simple distribution channel,
and is particularly useful for people writing tools that target Rust developers
(who are likely to be familiar with running cargo).

[RFC 1200]: https://github.com/rust-lang/rfcs/pull/1200

#### Tracing hooks

One of the most promising ways of using Rust is by "embedding" Rust code into
systems written in higher-level languages like [Ruby][skylight] or Python. This
embedding is usually done by giving the Rust code a C API, and works reasonably
well when the target sports a "C friendly" memory management scheme like
reference counting or conservative GC.

Integrating with an environment that uses a more advanced GC can be
quite challenging. Perhaps the most prominent examples are JavaScript engines
like V8 (used by [node.js]) and SpiderMonkey (used by [Firefox] and
[Servo]). Integrating with those engines requires very careful coding to ensure
that all objects are properly rooted; small mistakes can easily lead to
crashes. These are precisely the kind of memory management problems that Rust is
intended to eliminate.

**To bring Rust to environments with advanced GCs, we plan to extend the
compiler with the ability to generate "trace hooks"**. These hooks can be used
by a GC to sweep the stack and identify roots, making it possible to write code
that integrates with advanced VMs smoothly and easily. Naturally, the design
will respect Rust's "pay for what you use" policy, so that code which does not
integrate with a GC is unaffected.

[skylight]: https://blog.skylight.io/bending-the-curve-writing-safe-fast-native-gems-with-rust/
[crates.io]: https://crates.io
[node.js]: https://nodejs.org/
[Servo]: https://github.com/servo/servo
[Firefox]: https://firefox.com/

### Epilogue: RustCamp 2015, and Rust's community in 2016

We recently held the first-ever Rust conference, [RustCamp][rustcamp]
2015, which sold out with 160 attendees. It was amazing to see so much
of the Rust community in person, and to see the vibe of our online
spaces translate into a friendly and approachable in-person event. The
day opened with a keynote from Nicholas Matsakis and Aaron Turon
laying out the core team's view of where we are and where we're
headed. The
[slides are available online](https://rustcamp.com/schedule.html)
(along with several other talks), and the above serves as the missing
soundtrack. **Update**: now you can see
[the talks](https://confreaks.tv/events/rustcamp2015) as well!

There was a definite theme of the day: Rust's greatest potential is to unlock a
new generation of systems programmers. And that's not just because of the
language; it's just as much because of a community culture that says "Don't know
the difference between the stack and the heap? Don't worry, Rust is a great way
to learn about it, and I'd love to show you how."

The technical work we outlined above is important for our vision in 2016, but so
is the work of those on our moderation and community teams, and all of those who
tirelessly -- enthusiastically -- welcome people coming from all kinds of
backgrounds into the Rust community. So our greatest wish for the next year of
Rust is that, as its community grows, it continues to retain the welcoming
spirit that it has today.

[rustcamp]: https://rustcamp.com/
