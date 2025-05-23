+++
path = "2018/07/27/what-is-rust-2018"
title = "What is Rust 2018?"
authors = ["The Rust Core Team"]
aliases = ["2018/07/27/what-is-rust-2018.html"]
+++

Back in March, [we announced](https://blog.rust-lang.org/2018/03/12/roadmap.html) something new:

> This year, we will deliver Rust 2018, marking the first major new edition
> of Rust since 1.0 (aka Rust 2015).
> 
> We will continue to publish releases every six weeks as usual. But we will
> designate a release in the latter third of the year (Rust 1.29 - 1.31) as Rust 2018.
> This new 'edition' of Rust will be the culmination of feature
> stabilization throughout the year, and will ship with polished documentation,
> tooling, and libraries that tie in to those features.

Now that some time has passed, we wanted to share more about what this actually
*means* for Rust and Rust developers.

## Language change over time

One of the key questions facing language developers is "how do you manage
change over time"? How does that work for your users?  We believe quite
strongly that language stability is of utmost importance.  A language is the
foundation that you build your application on top of, and you cannot build
reliable, long-living things on a foundation of sand.  The very second post on
our blog, way back in October of 2014, was "[Stability as
a](https://blog.rust-lang.org/2014/10/30/Stability.html)
[Deliverable](https://blog.rust-lang.org/2014/10/30/Stability.html)". This laid
out our plans for the six week release schedule that we still follow to this
day. It also described how stability was important:

> It’s important to be clear about what we mean by stable. We don’t mean that
> Rust will stop evolving. We will release new versions of Rust on a regular,
> frequent basis, and we hope that people will upgrade just as regularly. But
> for that to happen, those upgrades need to be painless.

We put in a lot of work to make upgrades painless; for example, we run a tool
(called "crater") before each Rust release that downloads every package on
crates.io and attempts to build their code and run their tests. We have a
strong culture of testing, and we use tooling to ensure that every single
pull request is tested on every platform.
While we still believe that the six-week process is a fantastic *engineering*
strategy, it has some flaws.

### Losing the big picture

Increasing the number of releases means that each release is smaller. That's
the point! From an engineering perspective, this is great. But from a
user-facing perspective, it's harder to keep track of what's going on in Rust
unless you pay close attention every six weeks. And for those of us who do pay
such attention, it's easy to lose sight of the big picture. Rust has come a
long way in the last three years! Finally, for people who have tried Rust and
stopped using it for whatever reason, it's hard to know if the concerns have
been addressed: they'd have to pay attention every six weeks, which is not
something that is likely to happen.

### Tiny but necessary changes

Especially in a language with static types, almost any release can contain
something that breaks someone's code. Rust's
[RFC](https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md)
[1105](https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md)
lays out what kinds of changes we can make when incrementing a major, minor, or
patch version of the language.  However, the concept of "2.0" is extremely
overloaded in the minds of developers. 2.0 is *major* breaking change. Time to
throw everything out and start again. As such, we are very wary of releasing a
Rust 2.0.  There are some small changes that would be nice to make without
needing to bump to 2.0, however. For example, the addition of a new keyword is
a breaking change.  But sometimes, new features require a new keyword to work
properly.  In many ways, Rust is about taking tradeoffs and bending the curve.
Can we have our cake and eat it too?

## What is Rust 2018?

The release of Rust 1.31.0 on December 6th will be the first release of "Rust
2018." This marks a culmination of the last three years of Rust's development,
and brings it together in one neat package. For example, there will be a 2018
edition of the book that incorporates features stabilized since the print
edition was considered finalized.

You'll be able to put `edition = '2018'` into your `Cargo.toml`, and `cargo
new` will add it by default for new projects. At first, this will unlock
some new features that are not possible without it, and eventually, it will
enable some new lints that nudge you towards new idioms. You can also choose
`'2015'`, and if you don't have an `edition` key at all, it will default to
this value. These projects will continue on as before.  We'll be shipping a
tool, that helps you automatically upgrade your code to use these new features
and idioms. Running `cargo fix` will get your code ready in an automated
fashion.

From one perspective, editions are mostly about that cohesive package: they're
about celebrating what we've accomplished, and telling the world about it. From
another, editions are a way for us to make "breaking" changes without breaking
your code. For example, `try` will become a keyword in Rust 2018.  We can't
make that change in Rust 2015, as it may break code that uses it as a variable
name. But since you opt-in to Rust 2018, we can. We can also turn some warnings
into hard errors. But these changes are extremely limited; without getting too
deep into the technical details, editions can only change surface-level
features; the core of Rust is still the same.

## Managing compatibility

It goes even further than that: these two universes are compatible with one
another. We are quite sensitive to the issues in other language ecosystems,
where new code and old code can't interoperate. Making sure that this worked
well was a key aspect of the design of editions. In some sense, editions are
following in the steps of Java and C++, two languages that are known for their
stability stories.

In short, the Rust compiler will know how to compile both editions of code.
This is similar to how `javac` can compile both Java 9 and Java 10, or how
`gcc` and `clang` support both C++14 and C++17. Additionally, each compiler
will understand how to link both kinds of code together. This means that if
you're using Rust 2018, you can use dependencies that use Rust 2015 with zero
problems. If you're sticking with Rust 2015, you can use libraries that use
Rust 2018. It all works together.  This lets people who want to use new things
start immediately, while others that want to take it slower can upgrade on
their own time schedule.

Beyond that, it’s also important to mention that this release will be the
*initial* release of Rust 2018; in some sense, it’s the start, not the end.
We haven’t formally committed to a schedule for editions, but it’s likely
that the next one will be Rust 2021. We’ll continue to add features to Rust
2018 after its release, just like we continued to add features to Rust after
the Rust 1.0 release.

It’s also important to note that Rust 2015 is not frozen. Anything that does
not *require* being a part of Rust 2018 will work on Rust 2015 as well. This is
due to the way editions work; given the small nature of possible changes, the
compiler uses the same internal representation for all editions.

## A few words on long-term support

The Rust project currently only supports the most recent version of the stable
compiler. Some have wondered if the concept of editions ties into some form of
longer support. It does not, however, we've been talking about introducing some
sort of LTS policy, and may do so in the future.

## Giving it a try

We'll be doing several preview releases of Rust 2018. The most adventurous Rust
users are already giving it a try on nightly; once we get feedback from them
and do some polishing, we'll announce a beta that’s ready for more wide usage
for you to try here on the blog.
