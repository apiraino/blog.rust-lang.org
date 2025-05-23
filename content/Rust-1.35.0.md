+++
path = "2019/05/23/Rust-1.35.0"
title = "Announcing Rust 1.35.0"
authors = ["The Rust Release Team"]
aliases = [
    "2019/05/23/Rust-1.35.0.html",
    "releases/1.35.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.35.0. Rust is a
programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.35.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the appropriate page on our website,
and check out the [detailed release notes for 1.35.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1350-2019-05-23

## What's in 1.35.0 stable

The highlight of this release is the implementation of the `FnOnce`, `FnMut`,
and `Fn` closure traits for `Box<dyn FnOnce>`, `Box<dyn FnMut>`, and `Box<dyn Fn>` respectively.
Additionally, closures may now be coerced to unsafe function pointers.
The `dbg!` macro introduced in [Rust 1.32.0] can now also be called without arguments.
Moreover, there were a number of standard library stabilizations.
Read on for a few highlights, or see the [detailed release notes][notes] for additional information.

### `Fn*` closure traits implemented for `Box<dyn Fn*>`

[fn-pr]: https://github.com/rust-lang/rust/pull/55431
[`FnBox`]: https://doc.rust-lang.org/1.34.0/std/boxed/trait.FnBox.html
[unsized locals]: https://doc.rust-lang.org/nightly/unstable-book/language-features/unsized-locals.html

In Rust 1.35.0, the `FnOnce`, `FnMut`, and the `Fn` traits [are now implemented][fn-pr] for `Box<dyn FnOnce>`,
`Box<dyn FnMut>`, and `Box<dyn Fn>` respectively.

Previously, if you wanted to call the function stored in a boxed closure, you had to use [`FnBox`].
This was because instances of `Box<dyn FnOnce>` and friends did not implement the respective `Fn*` traits.
This also meant that it was not possible to pass boxed functions to code expecting an implementor of a `Fn` trait,
and you had to create temporary closures to pass them down.

This was ultimately due to a limitation in the compiler's ability to reason about such implementations,
which has since been fixed with the introduction of [unsized locals].

With this release, you can now use boxed functions in places that expect items implementing a function trait.

The following code now works:

```rust
fn foo(x: Box<dyn Fn(u8) -> u8>) -> Vec<u8> {
    vec![1, 2, 3, 4].into_iter().map(x).collect()
}
```

Furthermore, you can now directly call `Box<dyn FnOnce>` objects:

```rust
fn foo(x: Box<dyn FnOnce()>) {
    x()
}
```

### Coercing closures to `unsafe fn` pointers

[Rust 1.19.0]: https://blog.rust-lang.org/2017/07/20/Rust-1.19.html
[`RawWakerVTable`]: https://doc.rust-lang.org/beta/std/task/struct.RawWakerVTable.html

Since [Rust 1.19.0], it has been possible to coerce closures that do not capture from their environment into function pointers.
For example, you may write:

```rust
fn twice(x: u8, f: fn(u8) -> u8) -> u8 {
    f(f(x))
}

fn main() {
    assert_eq!(42, twice(0, |x| x + 21));
}
```

This has however not extended to `unsafe` function pointers.
With this release of Rust, you may now do so. For example:

```rust
/// The safety invariants are those of the `unsafe fn` pointer passed.
unsafe fn call_unsafe_fn_ptr(f: unsafe fn()) {
    f()
}

fn main() {
    // SAFETY: There are no invariants.
    // The closure is statically prevented from doing unsafe things.
    unsafe {
        call_unsafe_fn_ptr(|| {
            dbg!();
        });
    }
}
```

### Calling `dbg!()` with no argument

[Rust 1.32.0]: https://blog.rust-lang.org/2019/01/17/Rust-1.32.0.html#the-dbg-macro
[the `dbg!` macro]: https://doc.rust-lang.org/std/macro.dbg.html

For the benefit of all the occasional and frequent "print debuggers" out there,
[Rust 1.32.0] saw the release of [the `dbg!` macro].
To recap, the macro allows you to quickly inspect the value of some expression with context.
For example, when running:

```rust
fn main() {
    let mut x = 0;

    if dbg!(x == 1) {
        x += 1;
    }

    dbg!(x);
}
```

...you would see:

```
[src/main.rs:4] x == 1 = false
[src/main.rs:8] x = 0
```


As seen in the previous section, where the higher order function `call_unsafe_fn_ptr` is called,
you may now also call `dbg!` without passing any arguments.
This is useful when tracing what branches your application takes.
For example, with:

```rust
fn main() {
    let condition = true;

    if condition {
        dbg!();
    }
}
```

...you would see:

```
[src/main.rs:5]
```

### Library stabilizations

[`f32::copysign`]: https://doc.rust-lang.org/std/primitive.f32.html#method.copysign
[`f64::copysign`]: https://doc.rust-lang.org/std/primitive.f64.html#method.copysign
[`RefCell::replace_with`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.replace_with
[`Ref::map_split`]: https://doc.rust-lang.org/std/cell/struct.Ref.html#method.map_split
[`RefMut::map_split`]: https://doc.rust-lang.org/std/cell/struct.RefMut.html#method.map_split
[`ptr::hash`]: https://doc.rust-lang.org/std/ptr/fn.hash.html
[`Range::contains`]: https://doc.rust-lang.org/std/ops/struct.Range.html#method.contains
[`RangeFrom::contains`]: https://doc.rust-lang.org/std/ops/struct.RangeFrom.html#method.contains
[`RangeTo::contains`]: https://doc.rust-lang.org/std/ops/struct.RangeTo.html#method.contains
[`RangeInclusive::contains`]: https://doc.rust-lang.org/std/ops/struct.RangeInclusive.html#method.contains
[`RangeToInclusive::contains`]: https://doc.rust-lang.org/std/ops/struct.RangeToInclusive.html#method.contains
[`Option::copied`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.copied

In 1.35.0, a number of APIs have become stable.

In addition, some implementations were added and other changes occurred as well.
See the [detailed release notes][notes] for more details.

#### Copy the sign of a floating point number onto another

[`f32`]: https://doc.rust-lang.org/std/primitive.f32.html
[`f64`]: https://doc.rust-lang.org/std/primitive.f64.html

With this release, new methods `copysign` have been added to the floating point primitive types [`f32`] and [`f64`]:

- [`f32::copysign`]
- [`f64::copysign`]

As the name suggests, you can use these to copy the sign of one number onto another. For example:

```rust
fn main() {
    assert_eq!(3.5_f32.copysign(-0.42), -3.5);
}
```

#### Check whether a `Range` `contains` a value

Rust 1.35.0 contains a few freshly minted methods on the `Range` types:

- [`Range::contains`]
- [`RangeFrom::contains`]
- [`RangeTo::contains`]
- [`RangeInclusive::contains`]
- [`RangeToInclusive::contains`]

With these, you can easily check whether a given value exists in a range. For example, you may write:

```rust
fn main() {
    if (0..=10).contains(&5) {
        println!("Five is included in zero to ten.");
    }
}
```

#### Map and split a borrowed `RefCell` value in two

With Rust 1.35.0, you can now map and split the borrowed value of a `RefCell` into multiple borrows for different components of the borrowed data:

- [`Ref::map_split`]
- [`RefMut::map_split`]

#### Replace the value of a `RefCell` through a closure

This release introduces a convenience method `replace_with` on `RefCell`:

- [`RefCell::replace_with`]

With it, you can more ergonomically map and replace the current value of the cell and get back the old value as a result.

#### Hash a pointer or reference by address, not value

In this release, we have introduced:

- [`ptr::hash`]

This function takes a raw pointer and hashes it. Using `ptr::hash`,
you can avoid hashing the pointed-to value of a reference and instead hash the address.

#### Copy the contents of an `Option<&T>`

From the very beginning with Rust 1.0.0,
the methods `Option::cloned` for `Option<&T>` and `Option<&mut T>` have allowed you to clone the contents in case of `Some(_)`.
However, cloning can sometimes be an expensive operation and the methods `opt.cloned()` provided no hints to that effect.

With this release of Rust, we introduced:

- [`Option::copied`] for both `Option<&T>` and `Option<&mut T>`

The functionality of `opt.copied()` is the same as for `opt.cloned()`.
However, calling the method requires that `T: Copy`.
Using this method, you can make sure that code stops compiling should `T` no longer implements `Copy`.

### Changes in Clippy

[relnotes-clippy]:
https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-135-beta

[`drop_bounds`]: https://rust-lang.github.io/rust-clippy/master/index.html#drop_bounds

[split_lint]: https://github.com/rust-lang/rust-clippy/pull/4101

In this release of Rust,
Clippy (a collection of lints to catch common mistakes and improve your Rust code) added a new lint [`drop_bounds`].
This lint triggers when you add a bound `T: Drop` to a generic function. For example:

```rust
fn foo<T: Drop>(x: T) {}
```

Having a bound `T: Drop` is almost always a mistake as it excludes types,
such as `u8`, which have trivial drop-glues.
Moreover, `T: Drop` does not account for types like `String` not having interesting destructor behavior directly but rather as a result of embedding types,
such as `Vec<u8>`, that do.

In addition to [`drop_bounds`],
this release of Clippy [split][split_lint] the lint`redundant_closure` into `redundant_closure` and `redundant_closure_for_method_calls`.

See the [detailed release notes for Clippy][relnotes-clippy] for more details.

### Changes in Cargo

[relnotes-cargo]:
https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-135-2019-05-23

See the [detailed release notes for Cargo][relnotes-cargo] for more details.

## Contributors to 1.35.0

Many people came together to create Rust 1.35.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.35.0/)
