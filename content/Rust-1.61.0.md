+++
path = "2022/05/19/Rust-1.61.0"
title = "Announcing Rust 1.61.0"
authors = ["The Rust Release Team"]
aliases = [
    "2022/05/19/Rust-1.61.0.html",
    "releases/1.61.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.61.0. Rust is a programming language
empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.61.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.61.0][notes] on GitHub.

If you'd like to help us out by testing future releases, you might consider updating locally to use
the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`).
Please [report] any bugs you might come across!

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1610-2022-05-19
[report]: https://github.com/rust-lang/rust/issues/new/choose

## What's in 1.61.0 stable

### Custom exit codes from `main`

In the beginning, Rust `main` functions could only return the unit type `()` (either implicitly or explicitly), always indicating success
in the exit status, and if you wanted otherwise you had to call `process::exit(code)`. Since Rust
1.26, `main` has been allowed to return a `Result`, where `Ok` translated to a C `EXIT_SUCCESS` and
`Err` to `EXIT_FAILURE` (also debug-printing the error). Under the hood, these alternate return
types were unified by an unstable `Termination` trait.

In this release, that `Termination` trait is finally stable, along with a more general `ExitCode`
type that wraps platform-specific return types. That has `SUCCESS` and `FAILURE` constants, and also
implements `From<u8>` for more arbitrary values. The `Termination` trait can also be implemented for
your own types, allowing you to customize any kind of reporting before converting to an `ExitCode`.

For example, here's a type-safe way to write exit codes for a [`git bisect run`] script:

```rust
use std::process::{ExitCode, Termination};

#[repr(u8)]
pub enum GitBisectResult {
    Good = 0,
    Bad = 1,
    Skip = 125,
    Abort = 255,
}

impl Termination for GitBisectResult {
    fn report(self) -> ExitCode {
        // Maybe print a message here
        ExitCode::from(self as u8)
    }
}

fn main() -> GitBisectResult {
    std::panic::catch_unwind(|| {
        todo!("test the commit")
    }).unwrap_or(GitBisectResult::Abort)
}
```

[`git bisect run`]: https://git-scm.com/docs/git-bisect#_bisect_run

### More capabilities for `const fn`

Several incremental features have been stabilized in this release to enable more functionality in
`const` functions:

* **Basic handling of `fn` pointers**: You can now create, pass, and cast function pointers in a
  `const fn`. For example, this could be useful to build compile-time function tables for an
  interpreter. However, it is still not permitted to call `fn` pointers.

* **Trait bounds**: You can now write trait bounds on generic parameters to `const fn`, such as
  `T: Copy`, where previously only `Sized` was allowed.

* **`dyn Trait` types**: Similarly, `const fn` can now deal with trait objects, `dyn Trait`.

* **`impl Trait` types**: Arguments and return values for `const fn` can now be opaque `impl Trait`
  types.

Note that the trait features do not yet support calling methods from those traits in a `const fn`.

See the [Constant Evaluation](https://doc.rust-lang.org/stable/reference/const_eval.html) section of
the reference book to learn more about the current capabilities of `const` contexts, and future
capabilities can be tracked in [rust#57563](https://github.com/rust-lang/rust/issues/57563).

### Static handles for locked stdio

The three standard I/O streams -- `Stdin`, `Stdout`, and `Stderr` -- each have a `lock(&self)` to
allow more control over synchronizing read and writes. However, they returned lock guards with a
lifetime borrowed from `&self`, so they were limited to the scope of the original handle. This was
determined to be an unnecessary limitation, since the underlying locks were actually in static
storage, so now the guards are returned with a `'static` lifetime, disconnected from the handle.

For example, a common error came from trying to get a handle and lock it in one statement:

```rust
// error[E0716]: temporary value dropped while borrowed
let out = std::io::stdout().lock();
//        ^^^^^^^^^^^^^^^^^       - temporary value is freed at the end of this statement
//        |
//        creates a temporary which is freed while still in use
```

Now the lock guard is `'static`, not borrowing from that temporary, so this works!

### Stabilized APIs

The following methods and trait implementations are now stabilized:

- [`Pin::static_mut`](https://doc.rust-lang.org/1.61.0/std/pin/struct.Pin.html#method.static_mut)
- [`Pin::static_ref`](https://doc.rust-lang.org/1.61.0/std/pin/struct.Pin.html#method.static_ref)
- [`Vec::retain_mut`](https://doc.rust-lang.org/1.61.0/std/vec/struct.Vec.html#method.retain_mut)
- [`VecDeque::retain_mut`](https://doc.rust-lang.org/1.61.0/std/collections/struct.VecDeque.html#method.retain_mut)
- [`Write` for `Cursor<[u8; N]>`](https://doc.rust-lang.org/1.61.0/std/io/struct.Cursor.html#impl-Write-4)
- [`std::os::unix::net::SocketAddr::from_pathname`](https://doc.rust-lang.org/1.61.0/std/os/unix/net/struct.SocketAddr.html#method.from_pathname)
- [`std::process::ExitCode`](https://doc.rust-lang.org/1.61.0/std/process/struct.ExitCode.html)
- [`std::process::Termination`](https://doc.rust-lang.org/1.61.0/std/process/trait.Termination.html)
- [`std::thread::JoinHandle::is_finished`](https://doc.rust-lang.org/1.61.0/std/thread/struct.JoinHandle.html#method.is_finished)

The following previously stable functions are now `const`:

- [`<*const T>::offset`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.offset)
  and [`<*mut T>::offset`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.offset-1)
- [`<*const T>::wrapping_offset`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.wrapping_offset)
  and [`<*mut T>::wrapping_offset`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.wrapping_offset-1)
- [`<*const T>::add`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.add)
  and [`<*mut T>::add`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.add-1)
- [`<*const T>::sub`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.sub)
  and [`<*mut T>::sub`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.sub-1)
- [`<*const T>::wrapping_add`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.wrapping_add)
  and [`<*mut T>::wrapping_add`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.wrapping_add-1)
- [`<*const T>::wrapping_sub`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.wrapping_sub)
  and [`<*mut T>::wrapping_sub`](https://doc.rust-lang.org/1.61.0/std/primitive.pointer.html#method.wrapping_sub-1)
- [`<[T]>::as_mut_ptr`](https://doc.rust-lang.org/1.61.0/std/primitive.slice.html#method.as_mut_ptr)
- [`<[T]>::as_ptr_range`](https://doc.rust-lang.org/1.61.0/std/primitive.slice.html#method.as_ptr_range)
- [`<[T]>::as_mut_ptr_range`](https://doc.rust-lang.org/1.61.0/std/primitive.slice.html#method.as_mut_ptr_range)

### Other changes

There are other changes in the Rust 1.61.0 release. Check out what changed in
[Rust](https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1610-2022-05-19),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-161-2022-05-19),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-161).

In a future release we're planning to increase the baseline requirements for
the Linux kernel to version 3.2, and for glibc to version 2.17. We'd love
your feedback in [rust#95026](https://github.com/rust-lang/rust/pull/95026).

### Contributors to 1.61.0

Many people came together to create Rust 1.61.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.61.0/)
