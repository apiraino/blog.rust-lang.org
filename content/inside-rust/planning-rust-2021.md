+++
path = "inside-rust/2021/03/04/planning-rust-2021"
title = "Planning the Rust 2021 Edition"
authors = ["Ryan Levick"]
aliases = ["inside-rust/2021/03/04/planning-rust-2021.html"]

[extra]
team = "The Rust 2021 Edition Working Group"
team_url = "https://www.rust-lang.org/governance/teams/core#project-edition-2021"
+++

The Rust 2021 Edition working group is happy to announce that the next edition of Rust, Rust 2021, is scheduled for release later this year. While the [RFC](https://github.com/rust-lang/rfcs/pull/3085) formally introducing this edition is still open, we expect it to be merged soon. Planning and preparation have already begun, and we're on schedule!

If you're curious what features Rust 2021 will introduce or what the timeline for getting the edition released on stable is, keep reading! 

## What's included in this edition?

The final list of features for inclusion in Rust 2021 is still being decided. Overall, we aim for Rust 2021 to be a much smaller release than Rust 2018. This is for several reasons: 
* establishing a regular cadence for edition releases means we get many of the benefits of Rust's "train" release model at the edition level.
* Rust 2018 worked directly against the Rust model of "low stress" releases. 
* there's simply fewer breaking changes needed to continue to evolve the language. 

You can read more about the evolution of the concept of editions [in the RFC](https://github.com/rust-lang/rfcs/pull/3085).

Whether a feature will be included in Rust 2021 is a part of the RFC process, so the short list of possible features can and will change between now and the edition being released. That being said, here are some of the possible features that may be a part of the edition:

### Prelude changes 

While types and free functions can be added to the prelude independent of edition boundaries, the same is not true for traits. Adding a trait to the prelude can cause compatibility issues because calls to methods named the same as methods of the newly in-scope traits can become ambiguous. 

Currently the following traits are being proposed for inclusion in the Rust 2021 edition:
* `TryFrom`/`TryInto`
* `FromIterator`

The RFC for this change can be found [here](https://github.com/rust-lang/rfcs/pull/3090). Please note that the RFC is not yet merged, and the contents for a new prelude are still under active discussion.

### New closure capture rules 

[RFC 2229](https://github.com/rust-lang/rfcs/pull/2229) proposed that closures capture individual fields and not the whole struct when possible. This RFC has been accepted. In some circumstances this change would cause destructors to run at different times than they currently do, so the change must be tied to an edition. Migration lints will be provided to avoid changing the semantics of existing code.

### New default feature resolver in Cargo 

In Rust 1.51, Cargo will stabilize a new [feature resolver](https://github.com/rust-lang/cargo/issues/8088) which allows a crate's dependencies to use different features in different contexts. For example, a `#[no_std]` crate might want to use a particular dependency both as a build dependency (with `std` enabled) and as a regular dependency (with `std` disabled). Currently, this leads to `std` being enabled in both cases since features belong to a global namespace. 

In Rust 2021 this new resolver will become the default, but older editions can still use the new resolver by opting into it.

### Other 

Other proposed changes include [unifying how `panic` in `std` and `core` work](https://github.com/rust-lang/rust/issues/80162) and upgrading several lints from [warnings to errors](https://github.com/rust-lang/rust/issues/80165).

You can find a full list of features that are under consideration [here](https://docs.google.com/spreadsheets/d/1chZ2SL9T444nvU9al1kQ7TJMwC3IVQQV2xIv1HWGQ_k/edit?usp=sharing). 

If you're aware of a feature that has already been under discussion for inclusion in the next edition of Rust but is not listed here, [please let us know](https://rust-lang.zulipchat.com/#narrow/stream/268952-edition-2021). While we are excited to hear additional features that have not yet been discussed for inclusion in a Rust edition, we are unlikely to have the bandwidth to discuss such features until after the Rust 2021 edition is ready for release. 

## Rough timeline 

So how do we plan on shipping the new edition? Here's a timeline of milestones we're aiming for: 

* 01 April: All relevant RFCs either merged or in a good state (i.e., all major decisions reached and merging will happen in the following weeks). 
* 01 May: All features for inclusion in Rust 2021 are on nightly under feature flags.
* 01 June: All lints are implemented on nightly.
* 01 September: The edition is stabilized on nightly.
* 21 October: The edition hits stable.

As we approach these deadlines, we'll be narrowing down the list of proposed changes to those items that have made active progress.

## Call for participation

If you're interested in helping with the 2021 edition release, please [get in touch](https://rust-lang.zulipchat.com/#narrow/stream/268952-edition-2021). Besides feature work and edition management planning, there will be plenty of work to do. Some of the additional work items that will need to happen for the edition release include:
* `rustfix` migrations for all relevant features 
* testing all features and their migration paths
* blog posts and other marketing material
