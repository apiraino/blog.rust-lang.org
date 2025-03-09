+++
layout = "post"
date = 2020-12-31
title = "Announcing Rust 1.49.0"
author = "The Rust Release Team"
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.49.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.49.0 is as easy as:

```console
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.49.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1490-2020-12-31

## What's in 1.49.0 stable

For this release, we have some new targets and an improvement to the test
framework. See the [detailed release notes][notes] to learn about other
changes not covered by this post.

### 64-bit ARM Linux reaches Tier 1

The Rust compiler supports [a wide variety of targets][platform-support], but
the Rust Team can't provide the same level of support for all of them. To
clearly mark how supported each target is, we use a tiering system:

* Tier 3 targets are technically supported by the compiler, but we don't check
  whether their code build or passes the tests, and we don't provide any
  prebuilt binaries as part of our releases.
* Tier 2 targets are guaranteed to build and we provide prebuilt binaries, but
  we don't execute the test suite on those platforms: the produced binaries
  might not work or might have bugs.
* Tier 1 targets provide the highest support guarantee, and we run the full
  suite on those platforms for every change merged in the compiler. Prebuilt
  binaries are also available.

Rust 1.49.0 promotes the `aarch64-unknown-linux-gnu` target to Tier 1 support,
bringing our highest guarantees to users of 64-bit ARM systems running Linux!
We expect this change to benefit workloads spanning from embedded to desktops
and servers.

This is an important milestone for the project, since it's the first time a
non-x86 target has reached Tier 1 support: we hope this will pave the way for
more targets to reach our highest tier in the future.

Note that Android is not affected by this change as it uses a different Tier 2
target.

[platform-support]: https://doc.rust-lang.org/stable/rustc/platform-support.html

### 64-bit ARM macOS and Windows reach Tier 2

Rust 1.49.0 also features two targets reaching Tier 2 support:

* The `aarch64-apple-darwin` target brings support for Rust on Apple M1 systems.
* The `aarch64-pc-windows-msvc` target brings support for Rust on 64-bit ARM
  devices running Windows on ARM.

Developers can expect both of those targets to have prebuilt binaries
installable with `rustup` from now on! The Rust Team is not running the test
suite on those platforms though, so there might be bugs or instabilities.

### Test framework captures output in threads

Rust's built-in testing framework doesn't have a ton of features, but that
doesn't mean it can't be improved! Consider a test that looks like this:

```rust
#[test]
fn thready_pass() {
    println!("fee");
    std::thread::spawn(|| {
        println!("fie");
        println!("foe");
    })
    .join()
    .unwrap();
    println!("fum");
}
```

Here's what running this test looks like before Rust 1.49.0:

```text
❯ cargo +1.48.0 test
   Compiling threadtest v0.1.0 (C:\threadtest)
    Finished test [unoptimized + debuginfo] target(s) in 0.38s
     Running target\debug\deps\threadtest-02f42ffd9836cae5.exe

running 1 test
fie
foe
test thready_pass ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests threadtest

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

You can see that the output from the thread is printed, which intermixes
from the output of the test framework itself. Wouldn't it be nice
if every `println!` worked like that one that prints "`fum`?" Well, [that's
the behavior in Rust 1.49.0](https://github.com/rust-lang/rust/pull/78227):

```text
❯ cargo test
   Compiling threadtest v0.1.0 (C:\threadtest)
    Finished test [unoptimized + debuginfo] target(s) in 0.52s
     Running target\debug\deps\threadtest-40aabfaa345584be.exe

running 1 test
test thready_pass ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests threadtest

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

But don't worry; if the test were to fail, you'll still see all of the
output. By adding a `panic!` to the end of the test, we can see what failure
looks like:

```text
❯ cargo test
   Compiling threadtest v0.1.0 (C:\threadtest)
    Finished test [unoptimized + debuginfo] target(s) in 0.52s
     Running target\debug\deps\threadtest-40aabfaa345584be.exe

running 1 test
test thready_pass ... FAILED

failures:

---- thready_pass stdout ----
fee
fie
foe
fum
thread 'thready_pass' panicked at 'explicit panic', src\lib.rs:11:5
```

Specifically, the test runner makes sure to capture the output, and saves it
in case the test fails.

### Library changes

In Rust 1.49.0, there are three new stable functions:

- [`slice::select_nth_unstable`]
- [`slice::select_nth_unstable_by`]
- [`slice::select_nth_unstable_by_key`]

And two functions were made `const`:

- [`Poll::is_ready`]
- [`Poll::is_pending`]

See the [detailed release notes][notes] to learn about other changes.

[`slice::select_nth_unstable`]: https://doc.rust-lang.org/nightly/std/primitive.slice.html#method.select_nth_unstable
[`slice::select_nth_unstable_by`]: https://doc.rust-lang.org/nightly/std/primitive.slice.html#method.select_nth_unstable_by
[`slice::select_nth_unstable_by_key`]: https://doc.rust-lang.org/nightly/std/primitive.slice.html#method.select_nth_unstable_by_key
[`Poll::is_ready`]: https://doc.rust-lang.org/stable/std/task/enum.Poll.html#method.is_ready
[`Poll::is_pending`]: https://doc.rust-lang.org/stable/std/task/enum.Poll.html#method.is_pending

### Other changes

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-149-2020-12-31
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-149

There are other changes in the Rust 1.49.0 release: check out what changed in
[Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.49.0

Many people came together to create Rust 1.49.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.49.0/)
