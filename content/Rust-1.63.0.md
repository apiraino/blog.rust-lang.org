+++
path = "2022/08/11/Rust-1.63.0"
title = "Announcing Rust 1.63.0"
authors = ["The Rust Release Team"]
aliases = [
    "2022/08/11/Rust-1.63.0.html",
    "releases/1.63.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.63.0. Rust is a programming language
empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.63.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.63.0][notes] on GitHub.

If you'd like to help us out by testing future releases, you might consider updating locally to use
the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`).
Please [report] any bugs you might come across!

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1630-2022-08-11
[report]: https://github.com/rust-lang/rust/issues/new/choose

## What's in 1.63.0 stable

### Scoped threads

Rust code could launch new threads with `std::thread::spawn` since 1.0, but this
function bounds its closure with `'static`. Roughly, this means that threads
currently must have ownership of any arguments passed into their closure; you
can't pass borrowed data into a thread. In cases where the threads are expected
to exit by the end of the function (by being `join()`'d), this isn't strictly
necessary and can require workarounds like placing the data in an [`Arc`].

Now, with 1.63.0, the standard library is adding *scoped* threads, which allow
spawning a thread borrowing from the local stack frame. The
[`std::thread::scope`] API provides the necessary guarantee that any spawned threads
will have exited prior to itself returning, which allows for safely borrowing
data. Here's an example:

```rust
let mut a = vec![1, 2, 3];
let mut x = 0;

std::thread::scope(|s| {
    s.spawn(|| {
        println!("hello from the first scoped thread");
        // We can borrow `a` here.
        dbg!(&a);
    });
    s.spawn(|| {
        println!("hello from the second scoped thread");
        // We can even mutably borrow `x` here,
        // because no other threads are using it.
        x += a[0] + a[2];
    });
    println!("hello from the main thread");
});

// After the scope, we can modify and access our variables again:
a.push(4);
assert_eq!(x, a.len());
```

[`std::thread::scope`]: https://doc.rust-lang.org/stable/std/thread/fn.scope.html
[`std::thread::spawn`]: https://doc.rust-lang.org/stable/std/thread/fn.spawn.html
[`Arc`]: https://doc.rust-lang.org/stable/std/sync/struct.Arc.html


### Rust ownership for raw file descriptors/handles (I/O Safety)

Previously, Rust code working with platform APIs taking raw file descriptors (on
unix-style platforms) or handles (on Windows) would typically work directly with
a platform-specific representation of the descriptor (for example, a `c_int`, or the alias `RawFd`).
For Rust bindings to such native APIs, the type system then failed to encode
whether the API would take ownership of the file descriptor (e.g., `close`) or
merely borrow it (e.g., `dup`).

Now, Rust provides wrapper types such as [`BorrowedFd`] and [`OwnedFd`], which are marked as
`#[repr(transparent)]`, meaning that `extern "C"` bindings can directly take
these types to encode the ownership semantics. See the stabilized APIs section
for the full list of wrapper types stabilized in 1.63, currently, they are
available on cfg(unix) platforms, Windows, and WASI.

We recommend that new APIs use these types instead of the previous type aliases
(like [`RawFd`]).

[`RawFd`]: https://doc.rust-lang.org/stable/std/os/unix/io/type.RawFd.html
[`BorrowedFd`]: https://doc.rust-lang.org/stable/std/os/unix/io/struct.BorrowedFd.html
[`OwnedFd`]: https://doc.rust-lang.org/stable/std/os/unix/io/struct.OwnedFd.html

### `const` Mutex, RwLock, Condvar initialization

The [`Condvar::new`], [`Mutex::new`], and [`RwLock::new`] functions are now
callable in `const` contexts, which allows avoiding the use of crates like
`lazy_static` for creating global statics with `Mutex`, `RwLock`, or `Condvar`
values. This builds on the [work in 1.62] to enable thinner and faster mutexes
on Linux.

[work in 1.62]: https://blog.rust-lang.org/2022/06/30/Rust-1.62.0.html#thinner-faster-mutexes-on-linux

### Turbofish for generics in functions with `impl Trait`

For a function signature like `fn foo<T>(value: T, f: impl Copy)`, it was an
error to specify the concrete type of `T` via turbofish: `foo::<u32>(3, 3)`
would fail with:

```
error[E0632]: cannot provide explicit generic arguments when `impl Trait` is used in argument position
 --> src/lib.rs:4:11
  |
4 |     foo::<u32>(3, 3);
  |           ^^^ explicit generic argument not allowed
  |
  = note: see issue #83701 <https://github.com/rust-lang/rust/issues/83701> for more information
```

In 1.63, this restriction is relaxed, and the explicit type of the generic can be specified.
However, the `impl Trait` parameter, despite desugaring to a generic, remains
opaque and cannot be specified via turbofish.

### Non-lexical lifetimes migration complete

As detailed in [this blog post], we've fully removed the previous lexical borrow checker
from rustc across all editions, fully enabling the non-lexical, new, version of the borrow
checker. Since the borrow checker doesn't affect the output of rustc, this won't change
the behavior of any programs, but it completes a long-running migration (started in the
initial stabilization of NLL for the 2018 edition) to deliver the full benefits of the new
borrow checker across all editions of Rust. For most users, this change will bring
slightly better diagnostics for some borrow checking errors, but will not otherwise impact
which code they can write.

You can read more about non-lexical lifetimes in [this section of the 2018 edition announcement][nll].

[this blog post]: https://blog.rust-lang.org/2022/08/05/nll-by-default.html
[nll]: https://blog.rust-lang.org/2018/12/06/Rust-1.31-and-rust-2018.html#non-lexical-lifetimes

### Stabilized APIs

The following methods and trait implementations are now stabilized:

- [`array::from_fn`]
- [`Box::into_pin`]
- [`BinaryHeap::try_reserve`]
- [`BinaryHeap::try_reserve_exact`]
- [`OsString::try_reserve`]
- [`OsString::try_reserve_exact`]
- [`PathBuf::try_reserve`]
- [`PathBuf::try_reserve_exact`]
- [`Path::try_exists`]
- [`Ref::filter_map`]
- [`RefMut::filter_map`]
- [`NonNull::<[T]>::len`][`NonNull::<slice>::len`]
- [`ToOwned::clone_into`]
- [`Ipv6Addr::to_ipv4_mapped`]
- [`unix::io::AsFd`]
- [`unix::io::BorrowedFd<'fd>`]
- [`unix::io::OwnedFd`]
- [`windows::io::AsHandle`]
- [`windows::io::BorrowedHandle<'handle>`]
- [`windows::io::OwnedHandle`]
- [`windows::io::HandleOrInvalid`]
- [`windows::io::HandleOrNull`]
- [`windows::io::InvalidHandleError`]
- [`windows::io::NullHandleError`]
- [`windows::io::AsSocket`]
- [`windows::io::BorrowedSocket<'handle>`]
- [`windows::io::OwnedSocket`]
- [`thread::scope`]
- [`thread::Scope`]
- [`thread::ScopedJoinHandle`]

These APIs are now usable in const contexts:

- [`array::from_ref`]
- [`slice::from_ref`]
- [`intrinsics::copy`]
- [`intrinsics::copy_nonoverlapping`]
- [`<*const T>::copy_to`]
- [`<*const T>::copy_to_nonoverlapping`]
- [`<*mut T>::copy_to`]
- [`<*mut T>::copy_to_nonoverlapping`]
- [`<*mut T>::copy_from`]
- [`<*mut T>::copy_from_nonoverlapping`]
- [`str::from_utf8`]
- [`Utf8Error::error_len`]
- [`Utf8Error::valid_up_to`]
- [`Condvar::new`]
- [`Mutex::new`]
- [`RwLock::new`]

[`array::from_fn`]: https://doc.rust-lang.org/stable/std/array/fn.from_fn.html
[`Box::into_pin`]: https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.into_pin
[`BinaryHeap::try_reserve_exact`]: https://doc.rust-lang.org/stable/alloc/collections/binary_heap/struct.BinaryHeap.html#method.try_reserve_exact
[`BinaryHeap::try_reserve`]: https://doc.rust-lang.org/stable/std/collections/struct.BinaryHeap.html#method.try_reserve
[`OsString::try_reserve`]: https://doc.rust-lang.org/stable/std/ffi/struct.OsString.html#method.try_reserve
[`OsString::try_reserve_exact`]: https://doc.rust-lang.org/stable/std/ffi/struct.OsString.html#method.try_reserve_exact
[`PathBuf::try_reserve`]: https://doc.rust-lang.org/stable/std/path/struct.PathBuf.html#method.try_reserve
[`PathBuf::try_reserve_exact`]: https://doc.rust-lang.org/stable/std/path/struct.PathBuf.html#method.try_reserve_exact
[`Path::try_exists`]: https://doc.rust-lang.org/stable/std/path/struct.Path.html#method.try_exists
[`Ref::filter_map`]: https://doc.rust-lang.org/stable/std/cell/struct.Ref.html#method.filter_map
[`RefMut::filter_map`]: https://doc.rust-lang.org/stable/std/cell/struct.RefMut.html#method.filter_map
[`NonNull::<slice>::len`]: https://doc.rust-lang.org/stable/std/ptr/struct.NonNull.html#method.len
[`ToOwned::clone_into`]: https://doc.rust-lang.org/stable/std/borrow/trait.ToOwned.html#method.clone_into
[`Ipv6Addr::to_ipv4_mapped`]: https://doc.rust-lang.org/stable/std/net/struct.Ipv6Addr.html#method.to_ipv4_mapped
[`unix::io::AsFd`]: https://doc.rust-lang.org/stable/std/os/unix/io/trait.AsFd.html
[`unix::io::BorrowedFd<'fd>`]: https://doc.rust-lang.org/stable/std/os/unix/io/struct.BorrowedFd.html
[`unix::io::OwnedFd`]: https://doc.rust-lang.org/stable/std/os/unix/io/struct.OwnedFd.html
[`windows::io::AsHandle`]: https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsHandle.html
[`windows::io::BorrowedHandle<'handle>`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.BorrowedHandle.html
[`windows::io::OwnedHandle`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.OwnedHandle.html
[`windows::io::HandleOrInvalid`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.HandleOrInvalid.html
[`windows::io::HandleOrNull`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.HandleOrNull.html
[`windows::io::InvalidHandleError`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.InvalidHandleError.html
[`windows::io::NullHandleError`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.NullHandleError.html
[`windows::io::AsSocket`]: https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsSocket.html
[`windows::io::BorrowedSocket<'handle>`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.BorrowedSocket.html
[`windows::io::OwnedSocket`]: https://doc.rust-lang.org/stable/std/os/windows/io/struct.OwnedSocket.html
[`thread::scope`]: https://doc.rust-lang.org/stable/std/thread/fn.scope.html
[`thread::Scope`]: https://doc.rust-lang.org/stable/std/thread/struct.Scope.html
[`thread::ScopedJoinHandle`]: https://doc.rust-lang.org/stable/std/thread/struct.ScopedJoinHandle.html

[`array::from_ref`]: https://doc.rust-lang.org/stable/std/array/fn.from_ref.html
[`slice::from_ref`]: https://doc.rust-lang.org/stable/std/slice/fn.from_ref.html
[`intrinsics::copy`]: https://doc.rust-lang.org/stable/std/intrinsics/fn.copy.html
[`intrinsics::copy_nonoverlapping`]: https://doc.rust-lang.org/stable/std/intrinsics/fn.copy_nonoverlapping.html
[`<*const T>::copy_to`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.copy_to
[`<*const T>::copy_to_nonoverlapping`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.copy_to_nonoverlapping
[`<*mut T>::copy_to`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.copy_to-1
[`<*mut T>::copy_to_nonoverlapping`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.copy_to_nonoverlapping-1
[`<*mut T>::copy_from`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.copy_from
[`<*mut T>::copy_from_nonoverlapping`]: https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.copy_from_nonoverlapping
[`str::from_utf8`]: https://doc.rust-lang.org/stable/std/str/fn.from_utf8.html
[`Utf8Error::error_len`]: https://doc.rust-lang.org/stable/std/str/struct.Utf8Error.html#method.error_len
[`Utf8Error::valid_up_to`]: https://doc.rust-lang.org/stable/std/str/struct.Utf8Error.html#method.valid_up_to
[`Condvar::new`]: https://doc.rust-lang.org/stable/std/sync/struct.Condvar.html#method.new
[`Mutex::new`]: https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html#method.new
[`RwLock::new`]: https://doc.rust-lang.org/stable/std/sync/struct.RwLock.html#method.new


### Other changes

There are other changes in the Rust 1.63.0 release. Check out what changed in
[Rust](https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1630-2022-08-11),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-163-2022-08-11),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-163).

### Contributors to 1.63.0

Many people came together to create Rust 1.63.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.63.0/)
