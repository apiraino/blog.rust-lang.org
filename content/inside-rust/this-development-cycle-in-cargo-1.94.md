+++
path = "inside-rust/2026/02/18/this-development-cycle-in-cargo-1.94"
title = "This Development-cycle in Cargo: 1.94"
authors = ["Ed Page"]

[extra]
team = "The Cargo Team"
team_url = "https://www.rust-lang.org/governance/teams/dev-tools#cargo"
+++

# This Development-cycle in Cargo: 1.94

This is a summary of what has been happening around Cargo development for the last 6 weeks which is approximately the merge window for Rust 1.94.

<!-- time period: 2025-12-12 through 2026-01-21-->

- [Plugin of the cycle](#plugin-of-the-cycle)
- [Implementation](#implementation)
  - [Build dir layout](#build-dir-layout)
  - [Target dir locking](#target-dir-locking)
  - [Structured logging](#structured-logging)
  - [TOML 1.1](#toml-1-1)
  - [cargo-cargofmt](#cargo-cargofmt)
  - [lockfile-path](#lockfile-path)
- [Design discussions](#design-discussions)
  - [Workspace and configuration discovery](#workspace-and-configuration-discovery)
- [Misc](#misc)
- [Focus areas without progress](#focus-areas-without-progress)

## Plugin of the cycle

Cargo can't be everything to everyone,
if for no other reason than the compatibility guarantees it must uphold.
Plugins play an important part of the Cargo ecosystem and we want to celebrate them.

Our plugin for this cycle is [cargo-edit](https://crates.io/crates/cargo-edit) which provides commands for editing `Cargo.toml` files.  `cargo add` and `cargo rm` have already been merged into `cargo`.  This also provides `cargo upgrade` for changing version requirements (support in Cargo tracked at [#12425](https://github.com/rust-lang/cargo/issues/12425)) and `cargo set-version` for changing `package.version` (no request exists for merging into `cargo`).

Thanks to [kpreid](https://github.com/kpreid) for the suggestion!

[Please submit your suggestions for the next post.](https://rust-lang.zulipchat.com/#narrow/stream/246057-t-cargo/topic/Plugin.20of.20the.20Dev.20Cycle/near/420703211)

## Implementation

### Build dir layout

*Update from [1.93](https://blog.rust-lang.org/inside-rust/2026/01/07/this-development-cycle-in-cargo-1.93/#build-dir-layout)*

[ranger-ross](https://github.com/ranger-ross)
posted [#16502](https://github.com/rust-lang/cargo/pull/16502)
to update Cargo's internal documentation on the build dir layout.
The documentation provided another angle for reviewing this change which led to further refinements like
[#16514](https://github.com/rust-lang/cargo/pull/16514),
[#16515](https://github.com/rust-lang/cargo/pull/16515),
and [#16519](https://github.com/rust-lang/cargo/pull/16519).

For an unrelated change,
[epage](https://github.com/epage) had previously proposed making
`CARGO_BIN_EXE_*` available at runtime, and not just compile time
([Zulip](https://rust-lang.zulipchat.com/#narrow/channel/246057-t-cargo/topic/cargo_bin_exe.20and.20tests/near/564776712))
but abandoned it after finding it wasn't needed.
After examining the results of the [first crater run](https://github.com/rust-lang/rust/pull/149852#issuecomment-3664993475),
they decided to move forward with this to reduce the ecosystem impact of this change and possible offer other benefits and posted
[#16421](https://github.com/rust-lang/cargo/pull/16421).

[epage](https://github.com/epage) also experimented with running all of Cargo's test suite on the new build dir layout
([#16375](https://github.com/rust-lang/cargo/pull/16375))
which led to [#16348](https://github.com/rust-lang/cargo/pull/16348).

A [new crater run](https://github.com/rust-lang/rust/pull/149852#issuecomment-3764244130) was kicked off and analysis of the results is in going.

### Target dir locking

*Update from [1.93](https://blog.rust-lang.org/inside-rust/2026/01/07/this-development-cycle-in-cargo-1.93/#target-dir-locking)*

<!--
https://rust-lang.zulipchat.com/#narrow/channel/246057-t-cargo/topic/Build.20cache.20and.20locking.20design/near/563673412
https://github.com/rust-lang/cargo/pull/16155#discussion_r2635734239
-->
In addition to the challenges since the last update,
another is that a lock needs to be held while reading the fingerprint to decide if we want to mutate the cache entry or not.
Lock upgrading and downgrading has the risk of deadlocking

The last update ended with the idea of the top-level build operation owning all of the locks and them being grabbed exclusively.
That can help solve the fingerprint problem,
we just grab the lock before reading the fingerprint and we know it is good.
This does mean two of the same build will contend for the locks.
At least `cargo check` from rust-analyzer and `cargo test` (wrapping a `cargo build`) or `cargo clippy` won't contend.
Except they will in some easy to overlook but significant cases.
For `cargo check` and `cargo clippy`, `cargo clippy` only gets unique cache entries for workspace members,
so non-workspace members will contend for the locks.
For `cargo check` and `cargo test`, the cache entries are unique
(at least for now, see [#3501](https://github.com/rust-lang/cargo/issues/3501))
except when it comes to proc-macros and build scripts.
We decided to punt on this for now to get a minimal design merged that we can iterate on further.

We also punted on
[dynamic rlimits](https://github.com/rust-lang/cargo/pull/16155#discussion_r2632376448),
[Blocking messages for the user](https://github.com/rust-lang/cargo/pull/16155#discussion_r2648691107),
and reusing a build thread with another build unit rather than sitting idle when blocked.
At that point,
we were able to merge [#16155](https://github.com/rust-lang/cargo/pull/16155).
We are tracking the progress on these deferred items in [#4282](https://github.com/rust-lang/cargo/issues/4282).

### Structured logging

*Update from [1.93](https://blog.rust-lang.org/inside-rust/2026/01/07/this-development-cycle-in-cargo-1.93/#structured-logging)*

[weihanglo](https://github.com/weihanglo) continued making progress on this, including
- refactors ([#16485](https://github.com/rust-lang/cargo/pull/16485))
- documentation ([#16476](https://github.com/rust-lang/cargo/pull/16476))
- adding missing features to `cargo report timings` ([#16414](https://github.com/rust-lang/cargo/pull/16414), [#16441](https://github.com/rust-lang/cargo/pull/16441))
- adding `cargo report rebuild` ([#16456](https://github.com/rust-lang/cargo/pull/16456), [#16408](https://github.com/rust-lang/cargo/pull/16408), [#16448](https://github.com/rust-lang/cargo/pull/16448)) to see why rebuilds happens
- adding `cargo report sessions` ([#16428](https://github.com/rust-lang/cargo/pull/16428)) to find the ID needed for use in `cargo report timings` and `cargo report rebuild`
- providing man pages for `cargo report *` commands ([#16432](https://github.com/rust-lang/cargo/pull/16432), [#16430](https://github.com/rust-lang/cargo/pull/16430))
- removing unstable `--timings=FMT` as it is redundant with `cargo report timings` ([#16420](https://github.com/rust-lang/cargo/pull/16420))

On the project goal tracking issue,
[weihanglo](https://github.com/weihanglo)
[posted a summary](https://github.com/rust-lang/rust-project-goals/issues/398#issuecomment-3725163795),
remaining steps towards stabilization,
and how people can help.

<!--
https://rust-lang.zulipchat.com/#narrow/channel/246057-t-cargo/topic/build.20analysis.20log.20format/near/563511292
https://rust-lang.zulipchat.com/#narrow/channel/246057-t-cargo/topic/logging.20unit.20fingerprint/near/565825913
-->

### TOML 1.1

On December 18th, [TOML v1.1 spec](https://github.com/toml-lang/toml/releases/tag/1.1.0) was released.
The major change with this version is that newlines are now allowed inside of inline-tables.
The same day, [`toml` v0.9.10](https://docs.rs/toml/latest/toml/) was released to 
[support parsing TOML 1.1 files](https://github.com/toml-rs/toml/blob/main/crates/toml/CHANGELOG.md#0910---2025-12-18).

On [Zulip](https://rust-lang.zulipchat.com/#narrow/channel/246057-t-cargo/topic/TOML.201.2E1/near/564132825),
we discussed the transition for Cargo.
Users could inadvertently use a TOML v1.1 feature and bump the required version of Cargo to parse their manifest.
This is one of many reasons why [we encourage those keeping a `rust-version` to verify it in CI](https://doc.rust-lang.org/cargo/reference/rust-version.html#support-expectations).
However, the impact will be limited because `cargo package` rewrites the published `Cargo.toml`,
including using only TOML v0.5 or earlier features.
The impact for this will mostly be felt when using a
[git patch](https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html#the-patch-section)
to the original repo.

There is one caveat in this:
`toml` does not currently preserve whether [seconds or nanoseconds in a time](https://docs.rs/toml_datetime/0.7.5+spec-1.1.0/toml_datetime/struct.Time.html) were omitted or `0`,
assuming that seconds is never omitted and that 0 nanoseconds is always omitted.
If `toml` starts to preserve this information *and* a `Cargo.toml` field uses a time (likely only in a `[*.metadata]` field) *and* the user formats their time using the new syntax,
then `cargo package` will generate a rewritten `Cargo.toml` that requires a new version of Cargo to parse.

Cargo could detect that a TOML v1.1 feature is used and warn if the `package.rust-version` field is too old but we didn't view this as blocking because this is the same situation as any other field we have today that you can use that will bump your MSRV.

Cargo support for TOML v1.1 was merged on December 28th
([#16415](https://github.com/rust-lang/cargo/pull/16415)).

### `cargo-cargofmt`

There has long been a desire for `cargo fmt` to also format `Cargo.toml` files
([rustfmt#4091](https://github.com/rust-lang/rustfmt/issues/4091)).
One major blocker for this work is that the [official style guide for `Cargo.toml` files](https://doc.rust-lang.org/nightly/style-guide/cargo.html) does not align with existing or expected uses of `Cargo.toml` files.
Proposed ideas for the style guide had been discussed on
[Zulip](https://rust-lang.zulipchat.com/#narrow/channel/246057-t-cargo/topic/.60Cargo.2Etoml.60.20style.20guide/near/380796244)
but the conversation stalled out.

[epage](https://github.com/epage) created [`cargo-cargofmt`](https://github.com/crate-ci/cargo-cargofmt) as a test bed for style guide ideas as well as how to implement them.
This included
[summarizing past conversations](https://github.com/crate-ci/cargo-cargofmt/discussions/9)
as well as
[comparing existing formatters](https://github.com/crate-ci/cargo-cargofmt/discussions/3).

[iepathos](https://github.com/iepathos) stepped in and expanded the formatting rules,
including some gnarly work to adjust between single and multi-line arrays
([cargo-cargofmt#37](https://github.com/crate-ci/cargo-cargofmt/pull/37)).
Formatting inline tables to multi-line was deferred out as it would likely require a new [Style Edition](https://doc.rust-lang.org/nightly/style-guide/editions.html) to ensure the package's MSRV is high enough to support it.

See [cargo-cargofmt#25](https://github.com/crate-ci/cargo-cargofmt/discussions/25)
for what is supported today
and [the issues](https://github.com/crate-ci/cargo-cargofmt/issues) for what is being considered.

### lockfile-path

*Update from [1.82](https://blog.rust-lang.org/inside-rust/2024/10/01/this-development-cycle-in-cargo-1.82/#misc)*

Previously, unstable support for `--lockfile-path ../Cargo.lock` had been added ([#14326](https://github.com/rust-lang/cargo/pull/14326)).
In [#15510](https://github.com/rust-lang/cargo/issues/15510),
we got a request to also support setting it via an environment variable.
In discussing this, we felt we should shift the implementation away from a CLI flag to a config field as that would support the environment variables and CLI (through `--config`).
In particular, something we try to keep in mind is how easily someone can look at `cargo <cmd> --help` and find what they are looking for.
The more flags that exist, the more likely it is that a user won't find the flag they are looking for, the less value users get out of all flags as people instead work around what they presume to be the lack of support for a feature.
This came up before in the discussion of `--out-dir` / `--artifact-dir` ([#6100](https://github.com/rust-lang/cargo/issues/6100)).
In weighing the scope of this feature,
"hiding" it away in config seems the best course of action.

[weihanglo](https://github.com/weihanglo) added `resolver.lockfile-path` in [#16510](https://github.com/rust-lang/cargo/pull/16510).
We'll remove `--lockfile-path` in another development cycle to allow callers time to transition.

## Design discussions

### Workspace and configuration discovery

<!--
https://github.com/rust-lang/cargo-team/blob/main/meetings/sync-meeting/2026-01-13.md#backlog-nightly-features-in-a-parent-cargotoml-causes-build-in-unrelated-sub-crate-to-fail-when-building-with-beta-or-stable-cargo

https://github.com/rust-lang/cargo-team/blob/main/meetings/sync-meeting/2026-01-20.md#backlog-cargo-confused-about-broken-cargotoml-in-home
-->

If you accidentally copy a `Cargo.toml` file to your home directory,
it will fail the build of all of your packages without an explicit `[workspace]`.
This is true for any broken or nightly-only `Cargo.toml` or `.cargo/config.toml` file that happens to be in a parent directory
(e.g. [#6646](https://github.com/rust-lang/cargo/issues/6646),
[#6706](https://github.com/rust-lang/cargo/issues/6706)).
For `Cargo.toml`, Cargo is checking if the current manifest is part of a workspace.

We can at least improve the error message which we are tracking in [#6706](https://github.com/rust-lang/cargo/issues/6706).
It would also help if we discouraged new users from accidentally creating packages in their home directory
([#16562](https://github.com/rust-lang/cargo/issues/16562)).

For the nightly manifest case,
Cargo could check if the parent `Cargo.toml` has a `[workspace]` table and skip it by delaying the nightly feature check.
However, a nightly feature could impact workspace discovery.

For manifests, a workaround is to add an empty `[workspace]` to your package.
However, if you run `cargo new` in a subdirectory, it will automatically be added as a member.
We could extend [`package.workspace = "<path>"`](https://doc.rust-lang.org/cargo/reference/manifest.html#the-workspace-field)
with `package.workspace = <bool>` for opting in or out of auto-discover of the workspace.
For this case, you could insert `package.workspace = false` to avoid walking ip the directory tree.
Cargo script is starting with workspace auto-discovery disabled and this could be how we allow enabling it.
This idea is being tracked in [#16563](https://github.com/rust-lang/cargo/issues/16563).

We would like to more broadly improve the workspace and config discovery behavior which we are tracking in [#7871](https://github.com/rust-lang/cargo/issues/7871).

## Misc

- [osiewicz](https://github.com/osiewicz) sped up `cargo clean -p` and `cargo clean --workspace` in [#16264](https://github.com/rust-lang/cargo/pull/16264).
- *(Update from [1.93](https://blog.rust-lang.org/inside-rust/2026/01/07/this-development-cycle-in-cargo-1.93/#custom-final-artifacts))* [ranger-ross](https://github.com/ranger-ross) added unstable support for build scripts to use `cargo::metadata` without `package.links` manifest key ([#16436](https://github.com/rust-lang/cargo/pull/16436))

## Focus areas without progress

These are areas of interest for Cargo team members with no reportable progress for this development-cycle.

Project goals in need of owners
- [Stabilize public/private dependencies](https://rust-lang.github.io/rust-project-goals/2025h2/pub-priv.html)
- [Prototype a new set of Cargo "plumbing" commands](https://rust-lang.github.io/rust-project-goals/2025h2/cargo-plumbing.html)
- [Finish the libtest json output experiment](https://rust-lang.github.io/rust-project-goals/2025h2/libtest-json.html)
<!--
- [Prototype Cargo build analysis](https://rust-lang.github.io/rust-project-goals/2025h2/cargo-build-analysis.html)
- [Stabilize cargo-script](https://rust-lang.github.io/rust-project-goals/2025h2/cargo-script.html)
- [Rework Cargo Build Dir Layout](https://rust-lang.github.io/rust-project-goals/2025h2/cargo-build-dir-layout.html)
-->

Ready-to-develop:
- [Open namespaces](https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#open-namespaces)
- [Auto-generate completions](https://github.com/rust-lang/cargo/issues/14520)
  - See [clap-rs/clap#3166](https://github.com/clap-rs/clap/issues/3166)

<!--
Needs design and/or experimentation:
-->

Planning:
- [Disabling of default features](https://github.com/rust-lang/cargo/issues/3126)
- [RFC #3416: `features` metadata](https://github.com/rust-lang/rfcs/pull/3416)
  - [RFC #3487: visibility](https://github.com/rust-lang/rfcs/pull/3487) (visibility)
  - [RFC #3486: deprecation](https://github.com/rust-lang/rfcs/pull/3486)
  - [Unstable features](https://doc.rust-lang.org/cargo/reference/unstable.html#list-of-unstable-features)
- [Pre-RFC: Global, mutually exclusive features](https://internals.rust-lang.org/t/pre-rfc-mutually-excusive-global-features/19618)
- [RFC #3553: Cargo SBOM Fragment](https://github.com/rust-lang/rfcs/pull/3553)
- [OS-native config/cache directories (ie XDG support)](https://github.com/rust-lang/cargo/issues/1734)

## How you can help

If you have ideas for improving cargo,
we recommend first checking [our backlog](https://github.com/rust-lang/cargo/issues/)
and then exploring the idea on [Internals](https://internals.rust-lang.org/c/tools-and-infrastructure/cargo/15).

If there is a particular issue that you are wanting resolved that wasn't discussed here,
some steps you can take to help move it along include:
- Summarizing the existing conversation (example:
  [Better support for docker layer caching](https://github.com/rust-lang/cargo/issues/2644#issuecomment-1489371226),
  [Change in `Cargo.lock` policy](https://github.com/rust-lang/cargo/issues/8728#issuecomment-1610265047),
  [MSRV-aware resolver](https://github.com/rust-lang/cargo/issues/9930#issuecomment-1489089277)
  )
- Document prior art from other ecosystems so we can build on the work others have done and make something familiar to users, where it makes sense
- Document related problems and solutions within Cargo so we see if we are solving to the right layer of abstraction
- Building on those posts, propose a solution that takes into account the above information and cargo's compatibility requirements ([example](https://github.com/rust-lang/cargo/issues/9930#issuecomment-1489269471))

We are available to help mentor people for
[S-accepted issues](https://doc.crates.io/contrib/issues.html#issue-status-labels)
on
[zulip](https://rust-lang.zulipchat.com/#narrow/stream/246057-t-cargo)
and you can talk to us in real-time during
[Contributor Office Hours](https://github.com/rust-lang/cargo/wiki/Office-Hours).
If you are looking to help with one of the bigger projects mentioned here and are just starting out,
[fixing some issues](https://doc.crates.io/contrib/process/index.html#working-on-issues)
will help familiarize yourself with the process and expectations,
making things go more smoothly.
If you'd like to tackle something
[without a mentor](https://doc.crates.io/contrib/issues.html#issue-status-labels),
the expectations will be higher on what you'll need to do on your own.
