+++
path = "2020/11/27/Rustup-1.23.0"
title = "Announcing Rustup 1.23.0"
authors = ["The Rustup Working Group"]
aliases = ["2020/11/27/Rustup-1.23.0.html"]
+++

The rustup working group is happy to announce the release of rustup version 1.23.0. [Rustup][install] is the recommended tool to install [Rust][rust], a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of rustup installed, getting rustup 1.23.0 is as easy as closing your IDE and running:

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

## What's new in rustup 1.23.0

### Support for Apple M1 devices

Rustup is now natively available for the new Apple M1 devices, allowing you to install it on the new Macs the same way you'd install it on other platforms!

Note that at the time of writing this blog post the `aarch64-apple-darwin` compiler is at [Tier 2 target][tiers]: precompiled binaries are available starting from Rust 1.49 (currently in the beta channel), but no automated tests are executed on them.

You can follow [issue #73908][rust/73908] to track the work needed to bring Apple Silicon support to Tier 1.

[tiers]: https://doc.rust-lang.org/rustc/platform-support.html#tier-2
[rust/73908]: https://github.com/rust-lang/rust/issues/73908

### Support for installing minor releases

The Rust team releases a new version every six weeks, bringing new features and bugfixes on a regular basis. Sometimes a regression slips into a stable release, and the team releases a "point release" containing fixes for that regression. For example, [1.45.1] and [1.45.2] were point releases of [Rust 1.45.0][1.45.0], while [1.46.0] and [1.47.0] both had no point releases.

With rustup 1.22.1 or earlier if you wanted to use a stable release you were able to either install `stable` (which automatically updates to the latest one) or a specific version number, such as `1.48.0`, `1.45.0` or `1.45.2`. Starting from this release of rustup (1.23.0) you can also install a minor version without specifying the patch version, like `1.48` or `1.45`. These "virtual" releases will always point to the latest patch release of that cycle, so `rustup toolchain install 1.45` will get you a [1.45.2] toolchain.

[1.45.0]: https://blog.rust-lang.org/2020/07/16/Rust-1.45.0.html
[1.45.1]: https://blog.rust-lang.org/2020/07/30/Rust-1.45.1.html
[1.45.2]: https://blog.rust-lang.org/2020/08/03/Rust-1.45.2.html
[1.46.0]: https://blog.rust-lang.org/2020/08/27/Rust-1.46.0.html
[1.47.0]: https://blog.rust-lang.org/2020/10/08/Rust-1.47.html

### New format for `rust-toolchain`

The rustup 1.5.0 release introduced the `rust-toolchain` file, allowing you to choose the default toolchain for a project. When the file is present rustup ensures the toolchain specified in it is installed on the local system, and it will use that version when calling `rustc` or `cargo`:

```
$ cat rust-toolchain
nightly-2020-07-10
$ cargo --version
cargo 1.46.0-nightly (fede83ccf 2020-07-02)
```

The file works great for projects wanting to use a specific nightly version, but didn't allow to add extra components (like `clippy`) or compilation targets. Rustup 1.23.0 introduces a new, optional TOML syntax for the file, with support for specifying components and targets:

```toml
[toolchain]
channel = "nightly-2020-07-10"
components = ["rustfmt", "clippy"]
targets = ["wasm32-unknown-unknown"]
```

The new syntax doesn't replace the old one, and both will continue to work. You can learn more about overriding the default toolchain in the ["Overrides" chapter of the rustup book][overrides].

[overrides]: https://rust-lang.github.io/rustup/overrides.html#the-toolchain-file

### Other changes

There are more changes in rustup 1.23.0: check them out in the [changelog]! Rustup's documentation is also available in [the rustup book][book] starting from this release.

[changelog]: https://github.com/rust-lang/rustup/blob/stable/CHANGELOG.md
[book]: https://rust-lang.github.io/rustup/

## Thanks

Thanks to all the contributors who made rustup 1.23.0 possible!

- Aaron Loucks
- Aleksey Kladov
- Aurelia Dolo
- Camelid
- Chansuke
- Carol (Nichols || Goulding)
- Daniel Silverstone
- Dany Marcoux
- Eduard Miller
- Eduardo Broto
- Eric Huss
- Francesco Zardi
- FR Bimo
- Ivan Nejgebauer
- Ivan Tham
- Jake Goulding
- Jens Reidel
- Joshua M. Clulow
- Jynn Nelson
- Jubilee Young
- Leigh McCulloch
- Lzu Tao
- Matthias Krüger
- Matt Kraai
- Matt McKay
- Nick Ashley
- Pascal Hertleif
- Paul Lange
- Pietro Albini
- Robert Collins
- Stephen Muss
- Tom Eccles
