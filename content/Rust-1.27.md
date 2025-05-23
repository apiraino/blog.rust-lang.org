+++
path = "2018/06/21/Rust-1.27"
title = "Announcing Rust 1.27"
authors = ["The Rust Core Team"]
aliases = [
    "2018/06/21/Rust-1.27.html",
    "releases/1.27.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.27.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.27.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.27.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1270-2018-06-21

Additionally, we would like to draw attention to something: just before the
release of 1.27.0, we found [a
bug](https://github.com/rust-lang/rust/pull/51686) in the 'default match
bindings' feature introduced in 1.26.0 that can possibly introduce unsoundness.
Since it was discovered very late in the release process, and has been present
since 1.26.0, we decided to stick to our release train model. We expect to put
out a 1.27.1 with that fix applied soon, and if there's demand, possibly a
1.26.3 as well. More information about the specifics here will come in that
release announcement.

## What's in 1.27.0 stable

This release has two big language features that people have been waiting for.
But first, a small comment on documentation: All books in [the Rust
Bookshelf] are [now searchable]! For example, here's [a search of "The Rust
Programming Language" for
'borrow'](https://doc.rust-lang.org/book/second-edition/?search=borrow).
This will hopefully make it much easier to find what you're looking for.
Additionally, there's one new book: [the `rustc` Book]. This book explains
how to use `rustc` directly, as well as some other useful information, like a
list of all lints.

[the Rust Bookshelf]: https://doc.rust-lang.org/
[now searchable]: https://github.com/rust-lang/rust/pull/49623/
[the `rustc` Book]: https://github.com/rust-lang/rust/pull/49707/

### SIMD

Okay, now for the big news: the [basics of SIMD] are now available! SIMD
stands for "single instruction, multiple data." Consider a function
like this:

```rust
pub fn foo(a: &[u8], b: &[u8], c: &mut [u8]) {
    for ((a, b), c) in a.iter().zip(b).zip(c) {
        *c = *a + *b;
    }
}
```

[basics of SIMD]: https://github.com/rust-lang/rust/pull/49664/

Here, we're taking two slices, and adding the numbers together, placing the
result in a third slice. The simplest possible way to do this would be to do
exactly what the code does, and loop through each set of elements, add them
together, and store it in the result. However, compilers can often do better.
LLVM will often "autovectorize" code like this, which is a fancy term for
"use SIMD." Imagine that `a` and `b` were both 16 elements long. Each element
is a `u8`, and so that means that each slice would be 128 bits of data. Using
SIMD, we could put *both* `a` and `b` into 128 bit registers, add them
together in a `*single*` instruction, and then copy the resulting 128 bits
into `c`. That'd be much faster!

While stable Rust has always been able to take advantage of
autovectorization, sometimes, the compiler just isn't smart enough to realize
that we can do something like this. Additionally, not every CPU has these
features, and so LLVM may not use them so your program can be used on a wide
variety of hardware. So, in Rust 1.27, the addition of [the `std::arch`
module] allows us to use these kinds of instructions *directly*, which
means we don't need to rely on a smart compiler. Additionally, it includes
some features that allow us to choose a particular implementation based
on various criteria. For example:

[the `std::arch` module]: https://doc.rust-lang.org/stable/std/arch/

```rust
#[cfg(all(any(target_arch = "x86", target_arch = "x86_64"),
      target_feature = "avx2"))]
fn foo() {
    #[cfg(target_arch = "x86")]
    use std::arch::x86::_mm256_add_epi64;
    #[cfg(target_arch = "x86_64")]
    use std::arch::x86_64::_mm256_add_epi64;

    unsafe {
        _mm256_add_epi64(...);
    }
}
```

Here, we use `cfg` flags to choose the correct version based on the machine
we're targeting; on `x86` we use that version, and on `x86_64` we use
its version. We can also choose at runtime:

```rust
fn foo() {
    #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
    {
        if is_x86_feature_detected!("avx2") {
            return unsafe { foo_avx2() };
        }
    }

    foo_fallback();
}
```

Here, we have two versions of the function: one which uses `AVX2`, a specific
kind of SIMD feature that lets you do 256-bit operations. The
`is_x86_feature_detected!` macro will generate code that detects if your CPU
supports AVX2, and if so, calls the `foo_avx2` function. If not, then we fall
back to a non-AVX implementation, `foo_fallback`. This means that our code
will run super fast on CPUs that support AVX2, but still work on ones that
don't, albeit slower.

If all of this seems a bit low-level and fiddly, well, it is! `std::arch` is
specifically *primitives* for building these kinds of things. We hope to
eventually stabilize a `std::simd` module with higher-level stuff in the
future. But landing the basics now lets the ecosystem experiment with higher
level libraries starting today. For example, check out the
[faster](https://github.com/AdamNiederer/faster) crate. Here's a code
snippet with no SIMD:

```rust
let lots_of_3s = (&[-123.456f32; 128][..]).iter()
    .map(|v| {
        9.0 * v.abs().sqrt().sqrt().recip().ceil().sqrt() - 4.0 - 2.0
    })
    .collect::<Vec<f32>>();
```

To use SIMD with this code via `faster`, you'd change it to this:

```rust
let lots_of_3s = (&[-123.456f32; 128][..]).simd_iter()
    .simd_map(f32s(0.0), |v| {
        f32s(9.0) * v.abs().sqrt().rsqrt().ceil().sqrt() - f32s(4.0) - f32s(2.0)
    })
    .scalar_collect();
```

It looks almost the same: `simd_iter` instead of `iter`, `simd_map` instead
of `map`, `f32s(2.0)` instead of `2.0`. But you get a SIMD-ified version
generated for you.

Beyond *that*, you may never write any of this yourself, but as always, the
libraries you depend on may. For example, the [regex crate has already added
support](https://github.com/rust-lang/regex/pull/456), and a new release
will contain these SIMD speedups without you needing to do anything at all!

### `dyn Trait`

Rust's trait object syntax is one that we ultimately regret. If you'll recall,
given a trait `Foo`, this is a trait object:

```rust
Box<Foo>
```

However, if `Foo` were a struct, it'd just be a normal struct placed inside a
`Box<T>`. When designing the language, we thought that the similarity here was
a good thing, but experience has demonstrated that it is confusing. And it's
not just for the `Box<Trait>` case; `impl SomeTrait for SomeOtherTrait` is
also technically valid syntax, but you almost always want to write `impl<T>
SomeTrait for T where T: SomeOtherTrait` instead. Same with `impl SomeTrait`,
which looks like it would add methods or possibly default implementations
but in fact adds inherent methods to a trait object. Finally, with the recent
addition of `impl Trait` syntax, it's `impl Trait` vs `Trait` when explaining
things, and so that feels like `Trait` is what you should use, given that it's
shorter, but in reality, that's not always true.

As such, in Rust 1.27, we have stabilized a new syntax, [`dyn Trait`]. A
trait object now looks like this:

```rust
// old => new
Box<Foo> => Box<dyn Foo>
&Foo => &dyn Foo
&mut Foo => &mut dyn Foo
```

And similarly for other pointer types, `Arc<Foo>` is now `Arc<dyn Foo>`, etc.
Due to backwards compatibility, we cannot remove the old syntax, but we have
included a lint, which is set to allow by default, called [`bare-trait-object`].
If you want to lint against the older syntax, you can turn it on. We thought that
it would throw far too many warnings to turn on by default at present.

> Incidentally, we're working on a tool called `rustfix` that can automatically
> upgrade your code to newer idioms. It uses these sorts of lints to do so.
> Expect to hear more about `rustfix` in a future announcement.

[`dyn Trait`]: https://github.com/rust-lang/rfcs/blob/master/text/2113-dyn-trait-syntax.md
[`bare-trait-object`]: https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#bare-trait-object

### `#[must_use]` on functions

Finally, the `#[must_use]` attribute is getting an upgrade: [it can now be
used on functions](https://github.com/rust-lang/rust/pull/48925/).

Previously, it only applied to types, like `Result<T, E>`. But now, you can
do this:

```rust
#[must_use]
fn double(x: i32) -> i32 {
    2 * x
}

fn main() {
    double(4); // warning: unused return value of `double` which must be used

    let _ = double(4); // (no warning)
}
```

We've also [enhanced several bits of the standard
library](https://github.com/rust-lang/rust/pull/49533/) to make use of this;
`Clone::clone`, `Iterator::collect`, and `ToOwned::to_owned` will all start
warning if you don't use their results, helping you notice expensive operations
you may be throwing away by accident.

See the [detailed release notes][notes] for more.

### Library stabilizations

Several new APIs were stabilized this release:

- [`DoubleEndedIterator::rfind`]
- [`DoubleEndedIterator::rfold`]
- [`DoubleEndedIterator::try_rfold`]
- [`Duration::from_micros`]
- [`Duration::from_nanos`]
- [`Duration::subsec_micros`]
- [`Duration::subsec_millis`]
- [`HashMap::remove_entry`]
- [`Iterator::try_fold`]
- [`Iterator::try_for_each`]
- [`NonNull::cast`]
- [`Option::filter`]
- [`String::replace_range`]
- [`Take::set_limit`]
- [`hint::unreachable_unchecked`]
- [`os::unix::process::parent_id`]
- [`process::id`]
- [`ptr::swap_nonoverlapping`]
- [`slice::rsplit_mut`]
- [`slice::rsplit`]
- [`slice::swap_with_slice`]

[`DoubleEndedIterator::rfind`]: https://doc.rust-lang.org/std/iter/trait.DoubleEndedIterator.html#method.rfind
[`DoubleEndedIterator::rfold`]: https://doc.rust-lang.org/std/iter/trait.DoubleEndedIterator.html#method.rfold
[`DoubleEndedIterator::try_rfold`]: https://doc.rust-lang.org/std/iter/trait.DoubleEndedIterator.html#method.try_rfold
[`Duration::from_micros`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.from_micros
[`Duration::from_nanos`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.from_nanos
[`Duration::subsec_micros`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.subsec_micros
[`Duration::subsec_millis`]: https://doc.rust-lang.org/std/time/struct.Duration.html#method.subsec_millis
[`HashMap::remove_entry`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.remove_entry
[`Iterator::try_fold`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.try_fold
[`Iterator::try_for_each`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.try_for_each
[`NonNull::cast`]: https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.cast
[`Option::filter`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.filter
[`String::replace_range`]: https://doc.rust-lang.org/std/string/struct.String.html#method.replace_range
[`Take::set_limit`]: https://doc.rust-lang.org/std/io/struct.Take.html#method.set_limit
[`slice::rsplit_mut`]: https://doc.rust-lang.org/std/primitive.slice.html#method.rsplit_mut
[`slice::rsplit`]: https://doc.rust-lang.org/std/primitive.slice.html#method.rsplit
[`slice::swap_with_slice`]: https://doc.rust-lang.org/std/primitive.slice.html#method.swap_with_slice
[`hint::unreachable_unchecked`]: https://doc.rust-lang.org/std/hint/fn.unreachable_unchecked.html
[`os::unix::process::parent_id`]: https://doc.rust-lang.org/std/os/unix/process/fn.parent_id.html
[`ptr::swap_nonoverlapping`]: https://doc.rust-lang.org/std/ptr/fn.swap_nonoverlapping.html
[`process::id`]: https://doc.rust-lang.org/std/process/fn.id.html

See the [detailed release notes][notes] for more.

### Cargo features

Cargo has two small upgrades this release. First, it now [takes a
`--target-dir` flag](https://github.com/rust-lang/cargo/pull/5393/) if you'd
like to change the target directory for a given invocation.

Additionally, a tweak to the way Cargo deals with targets has landed. Cargo
will attempt to automatically discover tests, examples, and binaries within
your project. However, sometimes explicit configuration is needed. But the
initial implementation had a problem: let's say that you have two examples,
and Cargo is discovering them both. You want to tweak one of them, and so
you add a `[[example]]` to your `Cargo.toml` to configure its settings.
Cargo currently sees that you've set one explicitly, and therefore, doesn't
attempt to do any autodetection for the others. That's quite surprising.

As such, we've [added several 'auto' keys to
`Cargo.toml`](https://github.com/rust-lang/cargo/pull/5335/) We can't fix
this behavior without possibly breaking projects that may have inadvertently
been relying on it, and so, if you'd like to configure some targets, but not
others, you can set the `autoexamples` key to `true` in the `[package]`
section.

See the [detailed release notes][notes] for more.

## Contributors to 1.27.0

Many people came together to create Rust 1.27. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.27.0)
