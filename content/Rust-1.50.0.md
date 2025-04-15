+++
path = "2021/02/11/Rust-1.50.0"
title = "Announcing Rust 1.50.0"
authors = ["The Rust Release Team"]
aliases = [
    "2021/02/11/Rust-1.50.0.html",
    "releases/1.50.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.50.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.50.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.50.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1500-2021-02-11

## What's in 1.50.0 stable

For this release, we have improved array indexing, expanded safe access to union fields, and added to the standard library.
See the [detailed release notes][notes] to learn about other changes
not covered by this post.

### Const-generic array indexing

Continuing the march toward stable `const` generics, this release adds
implementations of `ops::Index` and `IndexMut` for arrays `[T; N]` for
_any_ length of `const N`. The indexing operator `[]` already worked on
arrays through built-in compiler magic, but at the type level, arrays
didn't actually implement the library traits until now.

```rust
fn second<C>(container: &C) -> &C::Output
where
    C: std::ops::Index<usize> + ?Sized,
{
    &container[1]
}

fn main() {
    let array: [i32; 3] = [1, 2, 3];
    assert_eq!(second(&array[..]), &2); // slices worked before
    assert_eq!(second(&array), &2); // now it also works directly
}
```

### `const` value repetition for arrays

Arrays in Rust can be written either as a list `[a, b, c]` or a repetition `[x; N]`.
For lengths `N` greater than one, repetition has only been allowed for `x`s that are `Copy`,
and [RFC 2203] sought to allow any `const` expression there. However,
while that feature was unstable for arbitrary expressions, its implementation
since Rust 1.38 accidentally allowed stable use of `const` _values_ in array
repetition.

```rust
fn main() {
    // This is not allowed, because `Option<Vec<i32>>` does not implement `Copy`.
    let array: [Option<Vec<i32>>; 10] = [None; 10];

    const NONE: Option<Vec<i32>> = None;
    const EMPTY: Option<Vec<i32>> = Some(Vec::new());

    // However, repeating a `const` value is allowed!
    let nones = [NONE; 10];
    let empties = [EMPTY; 10];
}
```

In Rust 1.50, that stabilization is formally acknowledged. In the future, to avoid such "temporary" named
constants, you can look forward to inline `const` expressions per [RFC 2920].

[RFC 2203]: https://rust-lang.github.io/rfcs/2203-const-repeat-expr.html
[RFC 2920]: https://rust-lang.github.io/rfcs/2920-inline-const.html

### Safe assignments to `ManuallyDrop<T>` union fields

Rust 1.49 made it possible to add `ManuallyDrop<T>` fields to a `union` as part
of allowing `Drop` for unions at all. However, unions don't drop old values
when a field is assigned, since they don't know which variant was formerly
valid, so safe Rust previously limited this to `Copy` types only, which never `Drop`.
Of course, `ManuallyDrop<T>` also doesn't need to `Drop`, so now Rust 1.50
allows safe assignments to these fields as well.

### A niche for `File` on Unix platforms

Some types in Rust have specific limitations on what is considered a
valid value, which may not cover the entire range of possible memory
values. We call any remaining invalid value a [niche], and this space
may be used for type layout optimizations. For example, in Rust 1.28
we introduced `NonZero` integer types (like `NonZeroU8`) where `0` is a niche, and this allowed
`Option<NonZero>` to use `0` to represent `None` with no extra memory.

On Unix platforms, Rust's `File` is simply made of the system's integer
file descriptor, and this happens to have a possible niche
as well because it can never be `-1`! System calls which return a file
descriptor use `-1` to indicate that an error occurred (check `errno`)
so it's never possible for `-1` to be a real file descriptor. Starting
in Rust 1.50 this niche is added to the type's definition so it can be
used in layout optimizations too. It follows that `Option<File>` will
now have the same size as `File` itself!

[niche]: https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#niche

### Library changes

In Rust 1.50.0, there are nine new stable functions:

- [`bool::then`]
- [`btree_map::Entry::or_insert_with_key`]
- [`f32::clamp`]
- [`f64::clamp`]
- [`hash_map::Entry::or_insert_with_key`]
- [`Ord::clamp`]
- [`RefCell::take`]
- [`slice::fill`]
- [`UnsafeCell::get_mut`]

And quite a few existing functions were made `const`:

- [`IpAddr::is_ipv4`]
- [`IpAddr::is_ipv6`]
- [`Layout::size`]
- [`Layout::align`]
- [`Layout::from_size_align`]
- `pow` for all integer types.
- `checked_pow` for all integer types.
- `saturating_pow` for all integer types.
- `wrapping_pow` for all integer types.
- `next_power_of_two` for all unsigned integer types.
- `checked_power_of_two` for all unsigned integer types.

See the [detailed release notes][notes] to learn about other changes.

[`IpAddr::is_ipv4`]: https://doc.rust-lang.org/stable/std/net/enum.IpAddr.html#method.is_ipv4
[`IpAddr::is_ipv6`]: https://doc.rust-lang.org/stable/std/net/enum.IpAddr.html#method.is_ipv6
[`Layout::align`]: https://doc.rust-lang.org/stable/std/alloc/struct.Layout.html#method.align
[`Layout::from_size_align`]: https://doc.rust-lang.org/stable/std/alloc/struct.Layout.html#method.from_size_align
[`Layout::size`]: https://doc.rust-lang.org/stable/std/alloc/struct.Layout.html#method.size
[`Ord::clamp`]: https://doc.rust-lang.org/stable/std/cmp/trait.Ord.html#method.clamp
[`RefCell::take`]: https://doc.rust-lang.org/stable/std/cell/struct.RefCell.html#method.take
[`UnsafeCell::get_mut`]: https://doc.rust-lang.org/stable/std/cell/struct.UnsafeCell.html#method.get_mut
[`bool::then`]: https://doc.rust-lang.org/stable/std/primitive.bool.html#method.then
[`btree_map::Entry::or_insert_with_key`]: https://doc.rust-lang.org/stable/std/collections/btree_map/enum.Entry.html#method.or_insert_with_key
[`f32::clamp`]: https://doc.rust-lang.org/stable/std/primitive.f32.html#method.clamp
[`f64::clamp`]: https://doc.rust-lang.org/stable/std/primitive.f64.html#method.clamp
[`hash_map::Entry::or_insert_with_key`]: https://doc.rust-lang.org/stable/std/collections/hash_map/enum.Entry.html#method.or_insert_with_key
[`slice::fill`]: https://doc.rust-lang.org/stable/std/primitive.slice.html#method.fill

### Other changes

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-150-2021-02-11
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-150

There are other changes in the Rust 1.50.0 release: check out what changed in
[Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.50.0

Many people came together to create Rust 1.50.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.50.0/)
