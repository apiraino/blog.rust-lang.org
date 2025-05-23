+++
path = "2024/02/26/Windows-7"
title = "Updated baseline standards for Windows targets"
authors = ["Chris Denton on behalf of the Compiler Team"]
aliases = ["2024/02/26/Windows-7.html"]
+++

The minimum requirements for Tier 1 toolchains targeting Windows will increase with the 1.78 release (scheduled for May 02, 2024).
Windows 10 will now be the minimum supported version for the `*-pc-windows-*` targets.
These requirements apply both to the Rust toolchain itself and to binaries produced by Rust.

Two new targets have been added with Windows 7 as their baseline: `x86_64-win7-windows-msvc` and `i686-win7-windows-msvc`.
They are starting as Tier 3 targets, meaning that the Rust codebase has support for them but we don't build or test them automatically.
Once these targets reach Tier 2 status, they will be available to use via rustup.

## Affected targets

- `x86_64-pc-windows-msvc`
- `i686-pc-windows-msvc`
- `x86_64-pc-windows-gnu`
- `i686-pc-windows-gnu`
- `x86_64-pc-windows-gnullvm`
- `i686-pc-windows-gnullvm`

## Why are the requirements being changed?

Prior to now, Rust had Tier 1 support for Windows 7, 8, and 8.1 but these targets no longer meet our requirements.
In particular, these targets could no longer be tested in CI which is required by the [Target Tier Policy](https://doc.rust-lang.org/rustc/target-tier-policy.html#tier-1-target-policy) and are not supported by their vendor.
