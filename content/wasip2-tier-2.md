+++
path = "2024/11/26/wasip2-tier-2"
title = "The wasm32-wasip2 Target Has Reached Tier 2 Support"
authors = ["Yosh Wuyts"]
aliases = ["2024/11/26/wasip2-tier-2.html"]
+++

## Introduction

In April of this year we posted an update about [Rust's WASI
targets](https://blog.rust-lang.org/2024/04/09/updates-to-rusts-wasi-targets.html)
to the main Rust blog. In it we covered the rename of the `wasm32-wasi` target
to `wasm32-wasip1`, and the introduction of the new `wasm32-wasip2` target as a
"tier 3" target. This meant that while the target was available as part of
`rust-lang/rustc`, it was not guaranteed to build. We're pleased to announce
that this has changed in Rust 1.82.

For those unfamiliar with WebAssembly (Wasm) components and WASI 0.2, here is a quick, simplified primer:

- **Wasm** is a (virtual) instruction format for programs to be compiled into (think: x86).
- **Wasm Components** are a container format and type system that wrap Core Wasm instructions into typed, hermetic binaries and libraries (think: ELF).
- **WASI** is a reserved namespace for a collection of standardized Wasm component interfaces (think: POSIX header files).

For a more detailed explanation see the [WASI 0.2 announcement post](https://bytecodealliance.org/articles/WASI-0.2) on the Bytecode Alliance blog.

## What's new?

Starting Rust 1.82 (2024-10-17) the `wasm32-wasip2` (WASI 0.2) target has
reached tier-2 platform support in the Rust compiler. Among other things this
now means it is guaranteed to build, and is now available to install via Rustup
using the following command:

```bash
rustup target add wasm32-wasip2
```

Up until now Rust users writing [Wasm
Components](https://component-model.bytecodealliance.org) would always have to rely on tools (such as
[cargo-component]) which target the WASI 0.1 target (`wasm32-wasip1`) and
package it into a WASI 0.2 Component via a post-processing step invoked. Now
that `wasm32-wasip2` is available to everyone via Rustup, tooling can
begin to directly target WASI 0.2 without the need for additional post-processing.

What this also means is that ecosystem crates can begin targeting WASI 0.2
directly for platform-specific code. WASI 0.1 did not have support for sockets.
Now that we have a stable tier 2 platform available, crate authors should be
able to finally start writing WASI-compatible network code. To target WASI 0.2
from Rust, authors can use the following `cfg` attribute:

[cargo-component]: https://github.com/bytecodealliance/cargo-component

```rust
#[cfg(all(target_os = "wasi", target_env = "p2"))]
mod wasip2 {
    // items go here
}
```

To target the older WASI 0.1 target, Rust also accepts `target_env = "p1"`.

## Standard Library Support

The WASI 0.2 Rust target reaching tier 2 platform support is in a way just the
beginning. means it's supported and stable. While the platform itself is now
stable, support in the stdlib for WASI 0.2 APIs is still limited. While the WASI
0.2 specification specifies APIs for example for timers, files, and sockets - if
you try and use the stdlib APIs for these today, you'll find they don't yet
work.

We expect to gradually extend the Rust stdlib with support for WASI 0.2 APIs
throughout the remainder of this year into the next. That work has already
started, with
[rust-lang/rust#129638](https://github.com/rust-lang/rust/pull/129638) adding
native support for `std::net` in Rust 1.83. We expect more of these PRs to land
through the remainder of the year.

Though this doesn't need to stop users from using WASI 0.2 today. The stdlib is
great because it provides *portable* abstractions, usually built on top of an
operating system's `libc` or equivalent. If you want to use WASI 0.2 APIs
directly today, you can either use the
[wasi](https://docs.rs/wasi/latest/wasi/) crate directly. Or generate your own
WASI bindings from the [WASI
specification's](https://github.com/WebAssembly/WASI/tree/main/wasip2) interface
types using [wit-bindgen](https://github.com/bytecodealliance/wit-bindgen/).

## Conclusion

The `wasm32-wasip2` target is now installable via Rustup. This makes it possible
for the Rust compiler to directly compile to the Wasm Components format
targeting the WASI 0.2 interfaces. There is now also a way for crates to compile
add WASI 0.2 platform support by writing:

```rust
#[cfg(all(target_os = "wasi", target_env = "p2"))]
mod wasip2 {}
```

We're excited for Wasm Components and WASI 0.2 to have reached this milestone
within the Rust project, and are excited to see what folks in the community will
be building with it!
