+++
path = "2022/02/24/Rust-1.59.0"
title = "Announcing Rust 1.59.0"
authors = ["The Rust Team"]
aliases = [
    "2022/02/24/Rust-1.59.0.html",
    "releases/1.59.0",
]

[extra]
release = true
+++

The Rust team has published a new version of Rust, 1.59.0. Rust is a programming
language that is empowering everyone to build reliable and efficient software.

---

Today's release falls on the day in which the world's attention is captured by
the sudden invasion of Ukraine by Putin's forces. Before going into the details
of the new Rust release, we'd like to state that we stand in solidarity with the
people of Ukraine and express our support for all people affected by this
conflict.

----

If you have a previous version of Rust installed via rustup, you can get 1.59.0
with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.59.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1590-2022-02-24

## What's in 1.59.0 stable

### Inline assembly

The Rust language now supports inline assembly. This enables many applications
that need very low-level control over their execution, or access to
specialized machine instructions.

When compiling for x86-64 targets, for instance, you can now write:

```rust
use std::arch::asm;

// Multiply x by 6 using shifts and adds
let mut x: u64 = 4;
unsafe {
    asm!(
        "mov {tmp}, {x}",
        "shl {tmp}, 1",
        "shl {x}, 2",
        "add {x}, {tmp}",
        x = inout(reg) x,
        tmp = out(reg) _,
    );
}
assert_eq!(x, 4 * 6);
```

The format string syntax used to name registers in the `asm!` and `global_asm!`
macros is the same used in Rust [format strings], so it should feel quite familiar
to Rust programmers.

The assembly language and instructions available with inline assembly vary
according to the target architecture. Today, the stable Rust compiler supports
inline assembly on the following architectures:

* x86 and x86-64
* ARM
* AArch64
* RISC-V

You can see more examples of inline assembly in [Rust By Example][asm-example],
and find more detailed documentation in the [reference][asm-reference].

[asm-example]: https://doc.rust-lang.org/nightly/rust-by-example/unsafe/asm.html
[asm-reference]: https://doc.rust-lang.org/nightly/reference/inline-assembly.html
[format strings]: https://doc.rust-lang.org/stable/std/fmt/

### Destructuring assignments

You can now use tuple, slice, and struct patterns as the left-hand side of an
assignment.

```rust
let (a, b, c, d, e);

(a, b) = (1, 2);
[c, .., d, _] = [1, 2, 3, 4, 5];
Struct { e, .. } = Struct { e: 5, f: 3 };

assert_eq!([1, 2, 1, 4, 5], [a, b, c, d, e]);
```

This makes assignment more consistent with `let` bindings, which have long
supported the same thing. Note that destructuring assignments with operators
such as `+=` are not allowed.

### Const generics defaults and interleaving

Generic types can now specify default values for their const generics. For
example, you can now write the following:

```rust
struct ArrayStorage<T, const N: usize = 2> {
    arr: [T; N],
}

impl<T> ArrayStorage<T> {
    fn new(a: T, b: T) -> ArrayStorage<T> {
        ArrayStorage {
            arr: [a, b],
        }
    }
}
```

Previously, type parameters were required to come before all const parameters.
That restriction has been relaxed and you can now interleave them.

```rust
fn cartesian_product<
    T, const N: usize,
    U, const M: usize,
    V, F
>(a: [T; N], b: [U; M], f: F) -> [[V; N]; M]
where
    F: FnMut(&T, &U) -> V
{
    // ...
}
```

### Future incompatibility warnings

Sometimes bugs in the Rust compiler cause it to accept code that should not
have been accepted. An example of this was [borrows of packed struct
fields][packed_borrows] being allowed in safe code.

[packed_borrows]: https://github.com/rust-lang/rust/issues/46043

While this happens very rarely, it can be quite disruptive when a crate used by
your project has code that will no longer be allowed. In fact, you might not
notice until your project inexplicably stops building!

Cargo now shows you warnings when a dependency will be rejected by a future
version of Rust. After running `cargo build` or `cargo check`, you might see:

```
warning: the following packages contain code that will be rejected by a future version of Rust: old_dep v0.1.0
note: to see what the problems were, use the option `--future-incompat-report`, or run `cargo report future-incompatibilities --id 1`
```

You can run the `cargo report` command mentioned in the warning to see a full
report of the code that will be rejected. This gives you time to upgrade your
dependency before it breaks your build.

### Creating stripped binaries

It's often useful to strip unnecessary information like debuginfo from binaries
you distribute, making them smaller.

While it has always been possible to do this manually after the binary is
created, cargo and rustc now support stripping when the binary is linked. To
enable this, add the following to your `Cargo.toml`:

```toml
[profile.release]
strip = "debuginfo"
```

This causes debuginfo to be stripped from release binaries. You can also supply
`"symbols"` or just `true` to strip all symbol information where supported.

The standard library typically ships with debug symbols and line-level
debuginfo, so Rust binaries built without debug symbols enabled still include
the debug information from the standard library by default. Using the `strip`
option allows you to remove this extra information, producing smaller Rust
binaries.

See [Cargo's documentation][cargo-docs] for more details.

[cargo-docs]: https://doc.rust-lang.org/beta/cargo/reference/profiles.html#strip

### Incremental compilation off by default

The 1.59.0 release disables incremental by default (unless explicitly asked for
by via an environment variable: `RUSTC_FORCE_INCREMENTAL=1`). This mitigates
the effects of a known bug, [#94124], which can cause deserialization errors (and panics) during compilation
with incremental compilation turned on.

The specific fix for [#94124] has landed and is currently in the 1.60 beta,
which will ship in six weeks. We are not presently aware of other issues that
would encourage a decision to disable incremental in 1.60 stable, and if none
arise it is likely that 1.60 stable will re-enable incremental compilation
again. Incremental compilation remains on by default in the beta and nightly
channels.

As always, we encourage users to test on the nightly and beta channels and
report issues you find: particularly for incremental bugs, this is the best way
to ensure the Rust team can judge whether there is breakage and the number of
users it affects.

[#94124]: https://github.com/rust-lang/rust/issues/94124

### Stabilized APIs

The following methods and trait implementations are now stabilized:

- [`std::thread::available_parallelism`][available_parallelism]
- [`Result::copied`][result-copied]
- [`Result::cloned`][result-cloned]
- [`arch::asm!`][asm]
- [`arch::global_asm!`][global_asm]
- [`ops::ControlFlow::is_break`][is_break]
- [`ops::ControlFlow::is_continue`][is_continue]
- [`TryFrom<char> for u8`][try_from_char_u8]
- [`char::TryFromCharError`][try_from_char_err]
  implementing `Clone`, `Debug`, `Display`, `PartialEq`, `Copy`, `Eq`, `Error`
- [`iter::zip`][zip]
- [`NonZeroU8::is_power_of_two`][is_power_of_two8]
- [`NonZeroU16::is_power_of_two`][is_power_of_two16]
- [`NonZeroU32::is_power_of_two`][is_power_of_two32]
- [`NonZeroU64::is_power_of_two`][is_power_of_two64]
- [`NonZeroU128::is_power_of_two`][is_power_of_two128]
- [`DoubleEndedIterator for ToLowercase`][lowercase]
- [`DoubleEndedIterator for ToUppercase`][uppercase]
- [`TryFrom<&mut [T]> for [T; N]`][tryfrom_ref_arr]
- [`UnwindSafe for Once`][unwindsafe_once]
- [`RefUnwindSafe for Once`][refunwindsafe_once]
- [armv8 neon intrinsics for aarch64][stdarch/1266]

The following previously stable functions are now `const`:

- [`mem::MaybeUninit::as_ptr`][muninit_ptr]
- [`mem::MaybeUninit::assume_init`][muninit_init]
- [`mem::MaybeUninit::assume_init_ref`][muninit_init_ref]
- [`ffi::CStr::from_bytes_with_nul_unchecked`][cstr_from_bytes]

### Other changes

There are other changes in the Rust 1.59.0 release. Check out what changed in
[Rust](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1590-2022-02-24),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-159-2022-02-24),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-159).

### Contributors to 1.59.0

Many people came together to create Rust 1.59.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.59.0/)

[cstr_from_bytes]: https://doc.rust-lang.org/stable/std/ffi/struct.CStr.html#method.from_bytes_with_nul_unchecked
[muninit_ptr]: https://doc.rust-lang.org/stable/std/mem/union.MaybeUninit.html#method.as_ptr
[muninit_init]: https://doc.rust-lang.org/stable/std/mem/union.MaybeUninit.html#method.assume_init
[muninit_init_ref]: https://doc.rust-lang.org/stable/std/mem/union.MaybeUninit.html#method.assume_init_ref
[unwindsafe_once]: https://doc.rust-lang.org/stable/std/sync/struct.Once.html#impl-UnwindSafe
[refunwindsafe_once]: https://doc.rust-lang.org/stable/std/sync/struct.Once.html#impl-RefUnwindSafe
[tryfrom_ref_arr]: https://doc.rust-lang.org/stable/std/convert/trait.TryFrom.html#impl-TryFrom%3C%26%27_%20mut%20%5BT%5D%3E
[lowercase]: https://doc.rust-lang.org/stable/std/char/struct.ToLowercase.html#impl-DoubleEndedIterator
[uppercase]: https://doc.rust-lang.org/stable/std/char/struct.ToUppercase.html#impl-DoubleEndedIterator
[try_from_char_err]: https://doc.rust-lang.org/stable/std/char/struct.TryFromCharError.html
[available_parallelism]: https://doc.rust-lang.org/stable/std/thread/fn.available_parallelism.html
[result-copied]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.copied
[result-cloned]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.cloned
[option-copied]: https://doc.rust-lang.org/stable/std/result/enum.Option.html#method.copied
[option-cloned]: https://doc.rust-lang.org/stable/std/result/enum.Option.html#method.cloned
[asm]: https://doc.rust-lang.org/stable/core/arch/macro.asm.html
[global_asm]: https://doc.rust-lang.org/stable/core/arch/macro.global_asm.html
[is_break]: https://doc.rust-lang.org/stable/std/ops/enum.ControlFlow.html#method.is_break
[is_continue]: https://doc.rust-lang.org/stable/std/ops/enum.ControlFlow.html#method.is_continue
[try_from_char_u8]: https://doc.rust-lang.org/stable/std/primitive.char.html#impl-TryFrom%3Cchar%3E
[zip]: https://doc.rust-lang.org/stable/std/iter/fn.zip.html
[is_power_of_two8]: https://doc.rust-lang.org/stable/core/num/struct.NonZeroU8.html#method.is_power_of_two
[is_power_of_two16]: https://doc.rust-lang.org/stable/core/num/struct.NonZeroU16.html#method.is_power_of_two
[is_power_of_two32]: https://doc.rust-lang.org/stable/core/num/struct.NonZeroU32.html#method.is_power_of_two
[is_power_of_two64]: https://doc.rust-lang.org/stable/core/num/struct.NonZeroU64.html#method.is_power_of_two
[is_power_of_two128]: https://doc.rust-lang.org/stable/core/num/struct.NonZeroU128.html#method.is_power_of_two
[stdarch/1266]: https://github.com/rust-lang/stdarch/pull/1266
