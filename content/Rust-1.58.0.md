+++
path = "2022/01/13/Rust-1.58.0"
title = "Announcing Rust 1.58.0"
authors = ["The Rust Release Team"]
aliases = ["2022/01/13/Rust-1.58.0.html"]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.58.0.
Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, getting Rust 1.58.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.58.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1580-2022-01-13

## What's in 1.58.0 stable

Rust 1.58 brings captured identifiers in format strings, a change to the
`Command` search path on Windows, more `#[must_use]` annotations in the
standard library, and some new library stabilizations.

### Captured identifiers in format strings

Format strings can now capture arguments simply by writing `{ident}` in the
string. Formats have long accepted positional arguments (optionally by index)
and named arguments, for example:

```rust
println!("Hello, {}!", get_person());                // implicit position
println!("Hello, {0}!", get_person());               // explicit index
println!("Hello, {person}!", person = get_person()); // named
```

Now named arguments can also be captured from the surrounding scope, like:

```rust
let person = get_person();
// ...
println!("Hello, {person}!"); // captures the local `person`
```

This may also be used in formatting parameters:

```rust
let (width, precision) = get_format();
for (name, score) in get_scores() {
  println!("{name}: {score:width$.precision$}");
}
```

Format strings can only capture plain identifiers, not arbitrary paths or
expressions. For more complicated arguments, either assign them to a local name
first, or use the older `name = expression` style of formatting arguments.

This feature works in all macros accepting format strings. However, one corner
case is the `panic!` macro in 2015 and 2018 editions, where `panic!("{ident}")`
is still treated as an unformatted string -- the compiler will warn about this
not having the intended effect. Due to the 2021 edition's update of panic
macros for [improved consistency], this works as expected in 2021 `panic!`.

[improved consistency]: https://doc.rust-lang.org/stable/edition-guide/rust-2021/panic-macro-consistency.html

### Reduced Windows `Command` search path

On Windows targets, `std::process::Command` will no longer search the current
directory for executables. That effect was owed to historical behavior of the
win32 [`CreateProcess`] API, so Rust was effectively searching in this order:

1. (Rust specific) The directories that are listed in the child's `PATH`
   environment variable, if it was explicitly changed from the parent.
2. The directory from which the application loaded.
3. The current directory for the parent process.
4. The 32-bit Windows system directory.
5. The 16-bit Windows system directory.
6. The Windows directory.
7. The directories that are listed in the `PATH` environment variable.

However, using the current directory can lead to surprising results, or even
malicious behavior when dealing with untrusted directories. For example,
`ripgrep` published [CVE-2021-3013] when they learned that their child
processes could be intercepted in this way. Even Microsoft's own PowerShell
[documents][PS] that they do not use the current directory for security.

Rust now performs its own search without the current directory, and the legacy
16-bit directory is also not included, as there is no API to discover its
location. So the new `Command` search order for Rust on Windows is:

1. The directories that are listed in the child's `PATH` environment variable.
2. The directory from which the application loaded.
3. The 32-bit Windows system directory.
4. The Windows directory.
5. The directories that are listed in the `PATH` environment variable.

Non-Windows targets continue to use their platform-specific behavior, most
often only considering the child or parent `PATH` environment variable.

[`CreateProcess`]: https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa
[CVE-2021-3013]: https://www.cve.org/CVERecord?id=CVE-2021-3013
[PS]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_command_precedence?view=powershell-7.2

### More `#[must_use]` in the standard library

The `#[must_use]` attribute can be applied to types or functions when failing
to explicitly consider them or their output is almost certainly a bug. This has
long been used in the standard library for types like `Result`, which should be
checked for error conditions. This also helps catch mistakes such as expecting
a function to mutate a value in-place, when it actually returns a new value.

Library [proposal 35] was approved in October 2021 to audit and expand the
application of `#[must_use]` throughout the standard library, covering many
more functions where the primary effect is the return value. This is similar
to the idea of function purity, but looser than a true language feature. Some
of these additions were present in release 1.57.0, and now in 1.58.0 the effort
has completed.

[proposal 35]: https://github.com/rust-lang/libs-team/issues/35

### Stabilized APIs

The following methods and trait implementations were stabilized.

- [`Metadata::is_symlink`]
- [`Path::is_symlink`]
- [`{integer}::saturating_div`]
- [`Option::unwrap_unchecked`]
- [`Result::unwrap_unchecked`]
- [`Result::unwrap_err_unchecked`]
- [`File::options`]

The following previously stable functions are now `const`.

- [`Duration::new`]
- [`Duration::checked_add`]
- [`Duration::saturating_add`]
- [`Duration::checked_sub`]
- [`Duration::saturating_sub`]
- [`Duration::checked_mul`]
- [`Duration::saturating_mul`]
- [`Duration::checked_div`]

[`Metadata::is_symlink`]: https://doc.rust-lang.org/stable/std/fs/struct.Metadata.html#method.is_symlink
[`Path::is_symlink`]: https://doc.rust-lang.org/stable/std/path/struct.Path.html#method.is_symlink
[`{integer}::saturating_div`]: https://doc.rust-lang.org/stable/std/primitive.i8.html#method.saturating_div
[`Option::unwrap_unchecked`]: https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.unwrap_unchecked
[`Result::unwrap_unchecked`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.unwrap_unchecked
[`Result::unwrap_err_unchecked`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.unwrap_err_unchecked
[`File::options`]: https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.options
[`Duration::new`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.new
[`Duration::checked_add`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.checked_add
[`Duration::saturating_add`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.saturating_add
[`Duration::checked_sub`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.checked_sub
[`Duration::saturating_sub`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.saturating_sub
[`Duration::checked_mul`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.checked_mul
[`Duration::saturating_mul`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.saturating_mul
[`Duration::checked_div`]: https://doc.rust-lang.org/stable/std/time/struct.Duration.html#method.checked_div

### Other changes

There are other changes in the Rust 1.58.0 release: check out what changed in
[Rust](https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1580-2022-01-13),
[Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-158-2022-01-13),
and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-158).

### Contributors to 1.58.0

Many people came together to create Rust 1.58.0.
We couldn't have done it without all of you.
[Thanks!](https://thanks.rust-lang.org/rust/1.58.0/)
