+++
path = "2023/12/28/Rust-1.75.0"
title = "Announcing Rust 1.75.0"
authors = ["The Rust Release Team"]
aliases = [
    "2023/12/28/Rust-1.75.0.html",
    "releases/1.75.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.75.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.75.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.75.0](https://doc.rust-lang.org/nightly/releases.html#version-1750-2023-12-28).

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.75.0 stable

### `async fn` and return-position `impl Trait` in traits

As [announced](https://blog.rust-lang.org/2023/12/21/async-fn-rpit-in-traits.html)
last week, Rust 1.75 supports use of `async fn` and `-> impl Trait` in traits.
However, this initial release comes with some limitations that are described in
the [announcement post](https://blog.rust-lang.org/2023/12/21/async-fn-rpit-in-traits.html#where-the-gaps-lie).

It's expected that these limitations will be lifted in future releases.

### Pointer byte offset APIs

Raw pointers (`*const T` and `*mut T`) used to primarily support operations
operating in units of `T`. For example, `<*const T>::add(1)` would add
`size_of::<T>()` bytes to the pointer's address. In some cases, working with
byte offsets is more convenient, and these new APIs avoid requiring callers to
cast to `*const u8`/`*mut u8` first.

- [`pointer::byte_add`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_add)
- [`pointer::byte_offset`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_offset)
- [`pointer::byte_offset_from`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_offset_from)
- [`pointer::byte_sub`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_sub)
- [`pointer::wrapping_byte_add`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.wrapping_byte_add)
- [`pointer::wrapping_byte_offset`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.wrapping_byte_offset)
- [`pointer::wrapping_byte_sub`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.wrapping_byte_sub)

### Code layout optimizations for rustc

The Rust compiler continues to get faster, with this release including the
application of
[BOLT](https://github.com/llvm/llvm-project/blob/main/bolt/README.md) to our
binary releases, bringing a 2% mean wall time improvements on our
benchmarks. This tool optimizes the layout of the `librustc_driver.so` library
containing most of the rustc code, allowing for better cache utilization.

We are also now building rustc with `-Ccodegen-units=1`, which provides more
opportunity for optimizations in LLVM. This optimization brought a separate
1.5% wall time mean win to our benchmarks.

In this release these optimizations are limited to `x86_64-unknown-linux-gnu`
compilers, but we expect to expand that over time to include more platforms.

### Stabilized APIs

- [`Atomic*::from_ptr`](https://doc.rust-lang.org/stable/core/sync/atomic/struct.AtomicUsize.html#method.from_ptr)
- [`FileTimes`](https://doc.rust-lang.org/stable/std/fs/struct.FileTimes.html)
- [`FileTimesExt`](https://doc.rust-lang.org/stable/std/os/windows/fs/trait.FileTimesExt.html)
- [`File::set_modified`](https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.set_modified)
- [`File::set_times`](https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.set_times)
- [`IpAddr::to_canonical`](https://doc.rust-lang.org/stable/core/net/enum.IpAddr.html#method.to_canonical)
- [`Ipv6Addr::to_canonical`](https://doc.rust-lang.org/stable/core/net/struct.Ipv6Addr.html#method.to_canonical)
- [`Option::as_slice`](https://doc.rust-lang.org/stable/core/option/enum.Option.html#method.as_slice)
- [`Option::as_mut_slice`](https://doc.rust-lang.org/stable/core/option/enum.Option.html#method.as_mut_slice)
- [`pointer::byte_add`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_add)
- [`pointer::byte_offset`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_offset)
- [`pointer::byte_offset_from`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_offset_from)
- [`pointer::byte_sub`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.byte_sub)
- [`pointer::wrapping_byte_add`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.wrapping_byte_add)
- [`pointer::wrapping_byte_offset`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.wrapping_byte_offset)
- [`pointer::wrapping_byte_sub`](https://doc.rust-lang.org/stable/core/primitive.pointer.html#method.wrapping_byte_sub)

These APIs are now stable in const contexts:

- [`Ipv6Addr::to_ipv4_mapped`](https://doc.rust-lang.org/stable/core/net/struct.Ipv6Addr.html#method.to_ipv4_mapped)
- [`MaybeUninit::assume_init_read`](https://doc.rust-lang.org/stable/core/mem/union.MaybeUninit.html#method.assume_init_read)
- [`MaybeUninit::zeroed`](https://doc.rust-lang.org/stable/core/mem/union.MaybeUninit.html#method.zeroed)
- [`mem::discriminant`](https://doc.rust-lang.org/stable/core/mem/fn.discriminant.html)
- [`mem::zeroed`](https://doc.rust-lang.org/stable/core/mem/fn.zeroed.html)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.75.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-175-2023-12-28), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-175).

## Contributors to 1.75.0

Many people came together to create Rust 1.75.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.75.0/)
