+++
path = "inside-rust/2020/01/10/cargo-in-2020"
title = "Cargo in 2020"
authors = ["Eric Huss"]
description = "Roadmap for Cargo in 2020"
aliases = ["inside-rust/2020/01/10/cargo-in-2020.html"]

[extra]
team = "the Cargo team"
team_url = "https://www.rust-lang.org/governance/teams/dev-tools#cargo"
+++

This post is an overview of the major projects the Cargo team is interested in
tackling in 2020.

It can be difficult to plan and predict around a volunteer-based open-source
project with limited resources. Instead of trying to present a wish list,
these are projects that already have a solid effort planned to push them
forward. That doesn't mean that we are not interested in other projects. We
have compiled a more detailed wish list at
<https://github.com/rust-lang/cargo/projects/1> that gives an outline of
things we would like to see, but are unlikely to have significant progress
this year.

If you are interested in helping, please let us know! We may not have time to
shepherd additional projects, but we may have time to give some amount of
feedback and review, particularly for well-motivated people who can do the
legwork of design and gathering a consensus.

## Features

[Features] provide a way to express optional dependencies and conditional
compilation of code. Fixes and enhancements to Features are one of the
most requested things we hear. In the beginning of 2020, we plan to implement
a new feature resolver which will make it easier to make progress on
implementing and experimenting with new behavior. There is a wide variety of
different enhancements that we are looking at, which we hope to make
incremental progress on while retaining a full picture of the long-term
plan.

Initially we plan to address the issues of decoupling shared dependencies
built with different features. Currently, features are unified for all uses of
a dependency, even when it is not necessary. This causes problems when a
feature intended for one context is incompatible with another. This often
happens for packages which have conditional `no_std` support. This appears
with build-dependencies, dev-dependencies, target-specific dependencies, and
large workspaces, each of which have their unique challenges.

Beyond that, the following is a brief view of the other major enhancements we
are tracking for the future:

* Workspace feature selection and unification
* Automatic features
* Namespaced features
* Mutually exclusive features
* Private/unstable features
* Profile and target default features
* And working through some of the 50+ feature issues.

There are some significant challenges around retaining backwards
compatibility, and being sensitive to increased build times. We hope that we
can address some of the major pain points while balancing those concerns.

[features]: https://doc.rust-lang.org/cargo/reference/manifest.html#the-features-section

## std aware Cargo

The "std aware Cargo" project is to make Cargo aware of the Rust standard
library, and to build it from source instead of using the pre-built binaries
that ship with `rustc`. Some of the notable benefits are:

* Customizing the compile-time flags of the standard library, such as using
  different optimizations, target-cpu, debug settings, etc.
* Supporting cross-compiling to new targets which do not have official
  distributions.
* Paving the road for future enhancements, such as compiling with different
  features, and using custom sources.

A significant amount of work has already been finished in 2019 with the
[`-Zbuild-std`] feature available on the nightly channel. There is still a
long road to bring it to a state where it can be stabilized. Work is being
tracked in the [`wg-cargo-std-aware` repo], and anyone interested is
encouraged to leave feedback on the issue tracker.

[`-Zbuild-std`]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#build-std
[`wg-cargo-std-aware` repo]: https://github.com/rust-lang/wg-cargo-std-aware/

## Profiles

[Profiles] have received a significant amount of work in 2018 and 2019.
[Overrides] are now stable (shipping in Rust 1.41). [Custom named profiles]
are available on the nightly channel. In 2020 we hope to continue pushing
these enhancements forward. Some of the efforts we are working towards are:

* Stabilizing [config-based profiles].
* Stabilizing [custom-named profiles].
* Implementing the `build` profile which can make it easier to define build-script settings.

[Profiles]: https://doc.rust-lang.org/nightly/cargo/reference/profiles.html
[Overrides]: https://doc.rust-lang.org/nightly/cargo/reference/profiles.html#overrides
[Custom named profiles]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#custom-named-profiles
[config-based profiles]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#config-profiles
[custom-named profiles]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#custom-named-profiles

## Ongoing projects

Some ongoing efforts don't have an end, and we intend to continue making
progress with them. Several new chapters have been added to the documentation,
and there is more to come. The JSON APIs are continually expanding with new
information making it easier to integrate tools and extract information. And
of course, trying to stay on top of bugs and issues!
