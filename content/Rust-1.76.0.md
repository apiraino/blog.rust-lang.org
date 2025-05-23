+++
path = "2024/02/08/Rust-1.76.0"
title = "Announcing Rust 1.76.0"
authors = ["The Rust Release Team"]
aliases = [
    "2024/02/08/Rust-1.76.0.html",
    "releases/1.76.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.76.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.76.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.76.0](https://doc.rust-lang.org/nightly/releases.html#version-1760-2024-02-08).

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.76.0 stable

This release is relatively minor, but as always, even incremental improvements lead to a greater whole. A few of those changes are highlighted in this post, and others may yet fill more niche needs.

### ABI compatibility updates

A new [ABI Compatibility](https://doc.rust-lang.org/stable/std/primitive.fn.html#abi-compatibility) section in the function pointer documentation describes what it means for function signatures to be ABI-compatible. A large part of that is the compatibility of argument types and return types, with a list of those that are currently considered compatible in Rust. For the most part, this documentation is not adding any new guarantees, only describing the existing state of compatibility.

The one new addition is that it is now guaranteed that `char` and `u32` are ABI compatible. They have always had the same size and alignment, but now they are considered equivalent even in function call ABI, consistent with the documentation above.

### Type names from references

For debugging purposes, [`any::type_name::<T>()`](https://doc.rust-lang.org/stable/std/any/fn.type_name.html) has been available since Rust 1.38 to return a string description of the type `T`, but that requires an explicit type parameter. It is not always easy to specify that type, especially for unnameable types like closures or for opaque return types. The new [`any::type_name_of_val(&T)`](https://doc.rust-lang.org/stable/std/any/fn.type_name_of_val.html) offers a way to get a descriptive name from any reference to a type.

```rust
fn get_iter() -> impl Iterator<Item = i32> {
    [1, 2, 3].into_iter()
}

fn main() {
    let iter = get_iter();
    let iter_name = std::any::type_name_of_val(&iter);
    let sum: i32 = iter.sum();
    println!("The sum of the `{iter_name}` is {sum}.");
}
```

This currently prints:

```
The sum of the `core::array::iter::IntoIter<i32, 3>` is 6.
```

### Stabilized APIs

- [`Arc::unwrap_or_clone`](https://doc.rust-lang.org/stable/std/sync/struct.Arc.html#method.unwrap_or_clone)
- [`Rc::unwrap_or_clone`](https://doc.rust-lang.org/stable/std/rc/struct.Rc.html#method.unwrap_or_clone)
- [`Result::inspect`](https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.inspect)
- [`Result::inspect_err`](https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.inspect_err)
- [`Option::inspect`](https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.inspect)
- [`type_name_of_val`](https://doc.rust-lang.org/stable/std/any/fn.type_name_of_val.html)
- [`std::hash::{DefaultHasher, RandomState}`](https://doc.rust-lang.org/stable/std/hash/index.html#structs)
  These were previously available only through `std::collections::hash_map`.
- [`ptr::{from_ref, from_mut}`](https://doc.rust-lang.org/stable/std/ptr/fn.from_ref.html)
- [`ptr::addr_eq`](https://doc.rust-lang.org/stable/std/ptr/fn.addr_eq.html)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.76.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-176-2024-02-08), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-176).

## Contributors to 1.76.0

Many people came together to create Rust 1.76.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.76.0/)
