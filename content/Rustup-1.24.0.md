+++
layout = "post"
date = 2021-04-27
title = "Announcing Rustup 1.24.0"
author = "The Rustup Working Group"
+++

> Shortly after publishing the release we got reports of [a regression][2737]
> preventing users from running `rustfmt` and `cargo fmt` after upgrading to
> Rustup 1.24.0. To limit the damage we **reverted** the release to version
> 1.23.1.
>
> If you have been affected by this issue you can revert to version 1.23.1 by
> running the following command:
>
> ```
> rustup self update
> ```

[2737]: https://github.com/rust-lang/rustup/issues/2737

The rustup working group is happy to announce the release of rustup version 1.24.0. [Rustup][install] is the recommended tool to install [Rust][rust], a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of rustup installed, getting rustup 1.24.0 is as easy as closing your IDE and running:

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

## What's new in rustup 1.24.0

### Support of `rust-toolchain.toml` as a filename for specifying toolchains.

Last year we released a new `toml` format for the `rust-toolchain` file. In order to bring Rustup closer into line with Cargo's behaviour around `.cargo/config` we now support the `.toml` extension for that file. If you call the toolchain file `rust-toolchain.toml` then you _must_ use the `toml` format, rather than the legacy one-line format.

If both `rust-toolchain` and `rust-toolchain.toml` are present, then the former will win out over the latter to ensure compatibility between Rustup versions.

### Better support for low-memory systems

Rustup's component unpacker has been changed to have a smaller memory footprint when unpacking large components. This should permit users of memory-constrained systems such as some Raspberry Pi systems to install newer Rust toolchains which contain particularly large files.

### Better support for Windows Add/Remove programs

Fresh installations of Rustup on Windows will now install themselves into the program list so that you can trigger the uninstallation of Rustup via the Add/Remove programs dialogs similar to any other Windows program.

_This will only take effect on installation, so you will need to rerun `rustup-init.exe` if you want this on your PC._

### Other changes

There are more changes in rustup 1.24.0: check them out in the [changelog]!

Rustup's documentation is also available in [the rustup book][book].

[changelog]: https://github.com/rust-lang/rustup/blob/stable/CHANGELOG.md
[book]: https://rust-lang.github.io/rustup/

## Thanks

Thanks to all the contributors who made rustup 1.24.0 possible!

- Alex Chan
- Aloïs Micard
- Andrew Norton
- Avery Harnish
- chansuke
- Daniel Alley
- Daniel Silverstone
- Eduard Miller
- Eric Huss
- est31
- Gareth Hubball
- Gurkenglas
- Jakub Stasiak
- Jynn Nelson
- Jubilee (workingjubilee)
- kellda
- Michael Cooper
- Philipp Oppermann
- Robert Collins
- SHA Miao
- skim (sl4m)
- Tudor Brindus
- Vasili (3point2)
- наб (nabijaczleweli)
- 二手掉包工程师 (hi-rustin)
