+++
path = "2024/05/02/Rust-1.78.0"
title = "Announcing Rust 1.78.0"
authors = ["The Rust Release Team"]
aliases = [
    "2024/05/02/Rust-1.78.0.html",
    "releases/1.78.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.78.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via `rustup`, you can get 1.78.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.78.0](https://doc.rust-lang.org/nightly/releases.html#version-1780-2024-05-02).

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.78.0 stable

### Diagnostic attributes

Rust now supports a `#[diagnostic]` attribute namespace to influence compiler error messages. These are treated as hints which the compiler is not _required_ to use, and it is also not an error to provide a diagnostic that the compiler doesn't recognize. This flexibility allows source code to provide diagnostics even when they're not supported by all compilers, whether those are different versions or entirely different implementations.

With this namespace comes the first supported attribute, `#[diagnostic::on_unimplemented]`, which can be placed on a trait to customize the message when that trait is required but hasn't been implemented on a type. Consider the example given in the [stabilization pull request](https://github.com/rust-lang/rust/pull/119888/):

```rust
#[diagnostic::on_unimplemented(
    message = "My Message for `ImportantTrait<{A}>` is not implemented for `{Self}`",
    label = "My Label",
    note = "Note 1",
    note = "Note 2"
)]
trait ImportantTrait<A> {}

fn use_my_trait(_: impl ImportantTrait<i32>) {}

fn main() {
    use_my_trait(String::new());
}
```

Previously, the compiler would give a builtin error like this:

```
error[E0277]: the trait bound `String: ImportantTrait<i32>` is not satisfied
  --> src/main.rs:12:18
   |
12 |     use_my_trait(String::new());
   |     ------------ ^^^^^^^^^^^^^ the trait `ImportantTrait<i32>` is not implemented for `String`
   |     |
   |     required by a bound introduced by this call
   |
```

With `#[diagnostic::on_unimplemented]`, its custom message fills the primary error line, and its custom label is placed on the source output. The original label is still written as help output, and any custom notes are written as well. (These exact details are subject to change.)

```
error[E0277]: My Message for `ImportantTrait<i32>` is not implemented for `String`
  --> src/main.rs:12:18
   |
12 |     use_my_trait(String::new());
   |     ------------ ^^^^^^^^^^^^^ My Label
   |     |
   |     required by a bound introduced by this call
   |
   = help: the trait `ImportantTrait<i32>` is not implemented for `String`
   = note: Note 1
   = note: Note 2
```

For trait authors, this kind of diagnostic is more useful if you can provide a better hint than just talking about the missing implementation itself. For example, this is an abridged sample from the standard library:

```rust
#[diagnostic::on_unimplemented(
    message = "the size for values of type `{Self}` cannot be known at compilation time",
    label = "doesn't have a size known at compile-time"
)]
pub trait Sized {}
```

For more information, see the reference section on [the `diagnostic` tool attribute namespace](https://doc.rust-lang.org/nightly/reference/attributes/diagnostics.html#the-diagnostic-tool-attribute-namespace).

### Asserting `unsafe` preconditions

The Rust standard library has a number of assertions for the preconditions of `unsafe` functions, but historically they have only been enabled in `#[cfg(debug_assertions)]` builds of the standard library to avoid affecting release performance. However, since the standard library is usually compiled and distributed in release mode, most Rust developers weren't ever executing these checks at all.

Now, the condition for these assertions is delayed until code generation, so they will be checked depending on the user's own setting for debug assertions -- enabled by default in debug and test builds. This change helps users catch undefined behavior in their code, though the details of how much is checked are generally not stable.

For example, [`slice::from_raw_parts`](https://doc.rust-lang.org/std/slice/fn.from_raw_parts.html) requires an aligned non-null pointer. The following use of a purposely-misaligned pointer has undefined behavior, and while if you were unlucky it may have *appeared* to "work" in the past, the debug assertion can now catch it:

```rust
fn main() {
    let slice: &[u8] = &[1, 2, 3, 4, 5];
    let ptr = slice.as_ptr();

    // Create an offset from `ptr` that will always be one off from `u16`'s correct alignment
    let i = usize::from(ptr as usize & 1 == 0);
    
    let slice16: &[u16] = unsafe { std::slice::from_raw_parts(ptr.add(i).cast::<u16>(), 2) };
    dbg!(slice16);
}
```

```
thread 'main' panicked at library/core/src/panicking.rs:220:5:
unsafe precondition(s) violated: slice::from_raw_parts requires the pointer to be aligned and non-null, and the total size of the slice not to exceed `isize::MAX`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
thread caused non-unwinding panic. aborting.
```

### Deterministic realignment

The standard library has a few functions that change the alignment of pointers and slices, but they previously had caveats that made them difficult to rely on in practice, if you followed their documentation precisely. Those caveats primarily existed as a hedge against `const` evaluation, but they're only stable for non-`const` use anyway. They are now promised to have consistent runtime behavior according to their actual inputs.

- [`pointer::align_offset`](https://doc.rust-lang.org/std/primitive.pointer.html#method.align_offset) computes the offset needed to change a pointer to the given alignment. It returns `usize::MAX` if that is not possible, but it was previously permitted to _always_ return `usize::MAX`, and now that behavior is removed.

- [`slice::align_to`](https://doc.rust-lang.org/std/primitive.slice.html#method.align_to) and [`slice::align_to_mut`](https://doc.rust-lang.org/std/primitive.slice.html#method.align_to_mut) both transmute slices to an aligned middle slice and the remaining unaligned head and tail slices. These methods now promise to return the largest possible middle part, rather than allowing the implementation to return something less optimal like returning everything as the head slice.

### Stabilized APIs

- [`impl Read for &Stdin`](https://doc.rust-lang.org/stable/std/io/struct.Stdin.html#impl-Read-for-%26Stdin)
- [Accept non `'static` lifetimes for several `std::error::Error` related implementations](https://github.com/rust-lang/rust/pull/113833/)
- [Make `impl<Fd: AsFd>` impl take `?Sized`](https://github.com/rust-lang/rust/pull/114655/)
- [`impl From<TryReserveError> for io::Error`](https://doc.rust-lang.org/stable/std/io/struct.Error.html#impl-From%3CTryReserveError%3E-for-Error)

These APIs are now stable in const contexts:

- [`Barrier::new()`](https://doc.rust-lang.org/stable/std/sync/struct.Barrier.html#method.new)

### Compatibility notes

* As [previously announced](https://blog.rust-lang.org/2024/02/26/Windows-7.html), Rust 1.78 has increased its minimum requirement to Windows 10 for the following targets:
  - `x86_64-pc-windows-msvc`
  - `i686-pc-windows-msvc`
  - `x86_64-pc-windows-gnu`
  - `i686-pc-windows-gnu`
  - `x86_64-pc-windows-gnullvm`
  - `i686-pc-windows-gnullvm`
* Rust 1.78 has upgraded its bundled LLVM to version 18, completing the announced [`u128`/`i128` ABI change](https://blog.rust-lang.org/2024/03/30/i128-layout-update.html) for x86-32 and x86-64 targets. Distributors that use their own LLVM older than 18 may still face the calling convention bugs mentioned in that post.

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.78.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-178-2024-05-02), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-178).

## Contributors to 1.78.0

Many people came together to create Rust 1.78.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.78.0/)
