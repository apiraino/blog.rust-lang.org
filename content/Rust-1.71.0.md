+++
path = "2023/07/13/Rust-1.71.0"
title = "Announcing Rust 1.71.0"
authors = ["The Rust Release Team"]
aliases = [
    "2023/07/13/Rust-1.71.0.html",
    "releases/1.71.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.71.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.71.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.71.0](https://github.com/rust-lang/rust/releases/tag/1.71.0) on GitHub.

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.71.0 stable

### C-unwind ABI

1.71.0 stabilizes `C-unwind` (and other `-unwind` suffixed ABI variants[^1]).

The behavior for unforced unwinding (the typical case) is specified in [this
table from the RFC which proposed this feature][rfc-table]. To summarize:

Each ABI is mostly equivalent to the same ABI without `-unwind`, except that
with `-unwind` the behavior is defined to be safe when an unwinding operation
(`panic` or C++ style exception) crosses the ABI boundary. For `panic=unwind`,
this is a valid way to let exceptions from one language unwind the stack in
another language without terminating the process (as long as the exception is
caught in the same language from which it originated); for `panic=abort`, this
will typically abort the process immediately.

For this initial stabilization, *no change* is made to the existing ABIs (e.g.
`"C"`), and unwinding across them remains undefined behavior. A future Rust
release will amend these ABIs to match the behavior specified in the RFC as the
final part in stabilizing this feature (usually aborting at the boundary).
Users are encouraged to start using the new unwind ABI variants in their code
to remain future proof if they need to unwind across the ABI boundary.

### Debugger visualization attributes

1.71.0 stabilizes support for a new attribute, `#[debug_visualizer(natvis_file
= "...")]` and `#[debug_visualizer(gdb_script_file = "...")]`, which allows
embedding Natvis descriptions and GDB scripts into Rust libraries to
improve debugger output when inspecting data structures created by those
libraries. Rust itself has packaged similar scripts for some time for the
standard library, but this feature makes it possible for library authors to
provide a similar experience to end users.

See the [reference](https://doc.rust-lang.org/nightly/reference/attributes/debugger.html#the-debugger_visualizer-attribute)
for details on usage.

### raw-dylib linking

On Windows platforms, Rust now supports using functions from dynamic libraries without requiring those libraries to be available at build time, using the new `kind="raw-dylib”` option for `#[link]`.

This avoids requiring users to install those libraries (particularly difficult for cross-compilation), and avoids having to ship stub versions of libraries in crates to link against. This simplifies crates providing bindings to Windows libraries.

Rust also supports binding to symbols provided by DLLs by ordinal rather than named symbol, using the new `#[link_ordinal]` attribute.

### Upgrade to musl 1.2

As [previously announced](https://blog.rust-lang.org/2023/05/09/Updating-musl-targets.html),
Rust 1.71 updates the musl version to 1.2.3. Most users should not be affected by this change.

### Const-initialized thread locals

Rust 1.59.0 stabilized `const` initialized thread local support in the standard
library, which allows for more optimal code generation. However, until now this
feature was missed in release notes and
[documentation](https://doc.rust-lang.org/stable/std/macro.thread_local.html).
Note that this stabilization does not make `const { ... }` a valid expression
or syntax in other contexts; that is a separate and currently unstable
[feature](https://github.com/rust-lang/rust/issues/76001).

```rust
use std::cell::Cell;

thread_local! {
    pub static FOO: Cell<u32> = const { Cell::new(1) };
}
```

### Stabilized APIs

- [`CStr::is_empty`](https://doc.rust-lang.org/stable/std/ffi/struct.CStr.html#method.is_empty)
- [`BuildHasher::hash_one`](https://doc.rust-lang.org/stable/std/hash/trait.BuildHasher.html#method.hash_one)
- [`NonZeroI*::is_positive`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#method.is_positive)
- [`NonZeroI*::is_negative`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#method.is_negative)
- [`NonZeroI*::checked_neg`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#method.checked_neg)
- [`NonZeroI*::overflowing_neg`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#method.overflowing_neg)
- [`NonZeroI*::saturating_neg`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#method.saturating_neg)
- [`NonZeroI*::wrapping_neg`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#method.wrapping_neg)
- [`Neg for NonZeroI*`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#impl-Neg-for-NonZeroI32)
- [`Neg for &NonZeroI*`](https://doc.rust-lang.org/stable/std/num/struct.NonZeroI32.html#impl-Neg-for-%26NonZeroI32)
- [`From<[T; N]> for (T...)`](https://doc.rust-lang.org/stable/std/primitive.array.html#impl-From%3C%5BT;+1%5D%3E-for-(T,))
  (array to N-tuple for N in 1..=12)
- [`From<(T...)> for [T; N]`](https://doc.rust-lang.org/stable/std/primitive.array.html#impl-From%3C(T,)%3E-for-%5BT;+1%5D)
  (N-tuple to array for N in 1..=12)
- [`windows::io::AsHandle for Box<T>`](https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsHandle.html#impl-AsHandle-for-Box%3CT%3E)
- [`windows::io::AsHandle for Rc<T>`](https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsHandle.html#impl-AsHandle-for-Rc%3CT%3E)
- [`windows::io::AsHandle for Arc<T>`](https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsHandle.html#impl-AsHandle-for-Arc%3CT%3E)
- [`windows::io::AsSocket for Box<T>`](https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsSocket.html#impl-AsSocket-for-Box%3CT%3E)
- [`windows::io::AsSocket for Rc<T>`](https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsSocket.html#impl-AsSocket-for-Rc%3CT%3E)
- [`windows::io::AsSocket for Arc<T>`](https://doc.rust-lang.org/stable/std/os/windows/io/trait.AsSocket.html#impl-AsSocket-for-Arc%3CT%3E)

These APIs are now stable in const contexts:

- [`<*const T>::read`](https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.read)
- [`<*const T>::read_unaligned`](https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.read_unaligned)
- [`<*mut T>::read`](https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.read-1)
- [`<*mut T>::read_unaligned`](https://doc.rust-lang.org/stable/std/primitive.pointer.html#method.read_unaligned-1)
- [`ptr::read`](https://doc.rust-lang.org/stable/std/ptr/fn.read.html)
- [`ptr::read_unaligned`](https://doc.rust-lang.org/stable/std/ptr/fn.read_unaligned.html)
- [`<[T]>::split_at`](https://doc.rust-lang.org/stable/std/primitive.slice.html#method.split_at)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.71.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-171-2023-07-13), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-171).

## Contributors to 1.71.0

Many people came together to create Rust 1.71.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.71.0/)

[^1]: List of stabilized ABIs can be found in [the stabilization report](https://github.com/rust-lang/rust/issues/74990#issuecomment-1363473645) 

[rfc-table]: https://github.com/rust-lang/rfcs/blob/master/text/2945-c-unwind-abi.md#abi-boundaries-and-unforced-unwinding
