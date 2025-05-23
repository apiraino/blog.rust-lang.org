+++
path = "2020/07/06/Rustup-1.22.0"
title = "Announcing Rustup 1.22.0"
authors = ["The Rustup Working Group"]
aliases = ["2020/07/06/Rustup-1.22.0.html"]
+++

The rustup working group is happy to announce the release of rustup version 1.22.0. [Rustup][install] is the recommended tool to install [Rust][rust], a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of rustup installed, getting rustup 1.22.0 is as easy as closing your IDE and running:

```
rustup self update
```

Rustup will also automatically update itself at the end of a normal toolchain update:

```
rustup update
```

If you don't have it already, you can [get rustup][install] from the appropriate page on our website.

[rust]: https://www.rust-lang.org
[install]: https://rustup.rs

## What's new in rustup 1.22.0

This release is mostly related to internal rework and tweaks in UI messages.  It is effectively a quality-of-life update which includes things such as:

* Supporting the larger MIPS release files which now exceed 100MB in individual files
* Supporting running in a lower-memory mode on single-CPU systems, along with detecting any in-place soft-limits on memory consumption in an effort to reduce the chance you run out of RAM during an install on systems like Raspberry Pis
* When we skip a `nightly` for missing-component reasons we now tell you all the missing components
* We now tell you where overrides are coming from in `rustup show`
* Added `riscv64gc-unknown-linux-gnu` version of `rustup`
* You can now specify multiple components when installing a toolchain more easily.  For example, if you wanted to install nightly with the `default` profile, but add the IDE support all in one go, you can now run
  ```
  rustup toolchain install --profile default --component rls,rust-analysis,rust-src nightly
  ```

There are many more changes in 1.22.0, with around 90 PRs, though a large number of them are internal changes which you can look at in [Github](https://github.com/rust-lang/rustup/commits/master) if you want, and you can see a little more detail than the above in our [changelog](https://github.com/rust-lang/rustup/blob/stable/CHANGELOG.md#1220---2020-06-30).

## Thanks

Thanks to all the contributors who made rustup 1.22.0 possible!

- Alejandro Martinez Ruiz
- Alexander D'hoore
- Ben Chen
- Chris Denton
- Daniel Silverstone
- Evan Weiler
- Guillaume Gomez
- Harry Sarson
- Jacob Lifshay
- James Yang
- Joel Parker Henderson
- John Titor
- Jonas Platte
- Josh Stone
- Jubilee
- Kellda
- LeSeulArtichaut
- Linus Färnstrand
- LitoMore
- LIU An (劉安)
- Luciano Bestia
- Lzu Tao
- Manish Goregaokar
- Mingye Wang
- Montgomery Edwards
- Per Lundberg
- Pietro Albini
- Robert Collins
- Rudolf B.
- Solomon Ucko
- Stein Somers
- Tetsuharu Ohzeki
- Tom Eccles
- Trevor Arjeski
- Tshepang Lekhonkhobe
