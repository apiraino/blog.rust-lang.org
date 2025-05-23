+++
path = "2017/07/20/Rust-1.19"
title = "Announcing Rust 1.19"
authors = ["The Rust Core Team"]
aliases = [
    "2017/07/20/Rust-1.19.html",
    "releases/1.19.0",
]

[extra]
release = true
+++

The Rust team is happy to announce the latest version of Rust, 1.19.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed, getting Rust 1.19 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.19.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]:  https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1190-2017-07-20

### What's in 1.19.0 stable

Rust 1.19.0 has some long-awaited features, but first, a note for our Windows
users. On Windows, Rust relies on `link.exe` for linking, which you can get via
the "Microsoft Visual C++ Build Tools." With the recent release of Visual
Studio 2017, the directory structure for these tools has changed. As such, to
use Rust, you had to stick with the 2015 tools or use a workaround (such as
running `vcvars.bat`). In 1.19.0, `rustc` now knows how to find the 2017 tools,
and so they work without a workaround.

On to new features! Rust 1.19.0 is the first release that supports [`union`s]:

```rust
union MyUnion {
    f1: u32,
    f2: f32,
}
```

Unions are kind of like `enum`s, but they are "untagged". Enums have a "tag"
that stores which variant is the correct one at runtime; unions elide this tag.

Since we can interpret the data held in the union using the wrong variant and
Rust can't check this for us, that means reading or writing a union's field is
unsafe:

```rust
let u = MyUnion { f1: 1 };

unsafe { u.f1 = 5 };

let value = unsafe { u.f1 };
```

Pattern matching works too:

```rust
fn f(u: MyUnion) {
    unsafe {
        match u {
            MyUnion { f1: 10 } => { println!("ten"); }
            MyUnion { f2 } => { println!("{}", f2); }
        }
    }
}
```

When are unions useful? One major use-case is interoperability with C. C APIs
can (and depending on the area, often do) expose unions, and so this makes writing
API wrappers for those libraries significantly easier. Additionally, from [its RFC]:

> A native union mechanism would also simplify Rust implementations of
> space-efficient or cache-efficient structures relying on value
> representation, such as machine-word-sized unions using the least-significant
> bits of aligned pointers to distinguish cases.

This feature has been long awaited, and there's still more improvements to come.
For now, `union`s can only include `Copy` types and may not implement `Drop`.
We expect to lift these restrictions in the future.

[`union`s]: https://github.com/rust-lang/rust/pull/42068
[its RFC]: https://github.com/rust-lang/rfcs/blob/master/text/1444-union.md#motivation

> As a side note, have you ever wondered how new features get added to Rust? This
> feature was suggested by Josh Triplett, and he [gave a talk at RustConf
> 2016](https://youtu.be/U8Gl3RTXf88?list=PLE7tQUdRKcybLShxegjn0xyTTDJeYwEkI)
> about the process of getting `union`s into Rust. You should check it out!

In other news, [`loop`s can now `break` with a value](https://github.com/rust-lang/rust/pull/42016):

```rust
// old code
let x;

loop {
    x = 7;
    break;
}

// new code
let x = loop { break 7; };
```

Rust has traditionally positioned itself as an "expression oriented language", that is,
most things are expressions that evaluate to a value, rather than statements. `loop` stuck
out as strange in this way, as it was previously a statement.

What about other forms of loops? It's not yet clear. See [its
RFC](https://github.com/rust-lang/rfcs/blob/master/text/1624-loop-break-value.md#extension-to-for-while-while-let)
for some discussion around the open questions here.

A smaller feature, closures that do not capture an environment [can now be coerced
to a function pointer](https://github.com/rust-lang/rust/pull/42162):

```rust
let f: fn(i32) -> i32 = |x| x + 1;
```

We now produce [xz compressed tarballs](https://github.com/rust-lang/rust-installer/pull/57) and prefer them by default,
making the data transfer smaller and faster. `gzip`'d tarballs are still produced
in case you can't use `xz` for some reason.

The compiler can now [bootstrap on
Android](https://github.com/rust-lang/rust/pull/41370). We've long supported Android
in various ways, and this continues to improve our support.

Finally, a compatibility note. Way back when we were running up to Rust 1.0, we did
a huge push to verify everything that was being marked as stable and as unstable.
We overlooked one thing, however: `-Z` flags. The `-Z` flag to the compiler enables
unstable flags. Unlike the rest of our stability story, you could still use `-Z` on
stable Rust. Back in April of 2016, in Rust 1.8, we made the use of `-Z` on stable
or beta produce a warning. Over a year later, we're fixing this hole in our
stability story by [disallowing `-Z` on stable and beta](https://github.com/rust-lang/rust/pull/41751).

See the [detailed release notes][notes] for more.

#### Library stabilizations

The largest new library feature is the [`eprint!` and `eprintln!` macros].
These work exactly the same as `print!` and `println!` but instead write
to standard error, as opposed to standard output.

[`eprint!` and `eprintln!` macros]: https://github.com/rust-lang/rust/pull/41192

Other new features:

- [`String` now implements `FromIterator<Cow<'a, str>>` and
  `Extend<Cow<'a, str>>`][41449]
- [`Vec` now implements `From<&mut [T]>`][41530]
- [`Box<[u8]>` now implements `From<Box<str>>`][41258]
- [`SplitWhitespace` now implements `Clone`][41659]
- [
     `[u8]::reverse` is now 5x faster and
     `[u16]::reverse` is now 1.5x faster][41764]

[41449]: https://github.com/rust-lang/rust/pull/41449
[41530]: https://github.com/rust-lang/rust/pull/41530
[41258]: https://github.com/rust-lang/rust/pull/41258
[41659]: https://github.com/rust-lang/rust/pull/41659
[41764]: https://github.com/rust-lang/rust/pull/41764

And some freshly-stabilized APIs:

- [`OsString::shrink_to_fit`]
- [`cmp::Reverse`]
- [`Command::envs`]
- [`thread::ThreadId`]

[`OsString::shrink_to_fit`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html#method.shrink_to_fit
[`cmp::Reverse`]: https://doc.rust-lang.org/std/cmp/struct.Reverse.html
[`Command::envs`]: https://doc.rust-lang.org/std/process/struct.Command.html#method.envs
[`thread::ThreadId`]: https://doc.rust-lang.org/std/thread/struct.ThreadId.html

See the [detailed release notes][notes] for more.

#### Cargo features

Cargo mostly received small but valuable improvements in this release. The
largest is possibly that [Cargo no longer checks out a local working
directory for the crates.io index][cargo/4026]. This should provide smaller
file size for the registry and improve cloning times, especially on Windows
machines.

Other improvements:

- [Build scripts can now add environment variables to the environment
  the crate is being compiled in.
  Example: `println!("cargo:rustc-env=FOO=bar");`][cargo/3929]
- [Workspace members can now accept glob file patterns][cargo/3979]
- [Added `--all` flag to the `cargo bench` subcommand to run benchmarks of all
  the members in a given workspace.][cargo/3988]
- [Added an `--exclude` option for excluding certain packages when using the
  `--all` option][cargo/4031]
- [The `--features` option now accepts multiple comma or space
  delimited values.][cargo/4084]
- [Added support for custom target specific runners][cargo/3954]

[cargo/3929]: https://github.com/rust-lang/cargo/pull/3929
[cargo/3954]: https://github.com/rust-lang/cargo/pull/3954
[cargo/3979]: https://github.com/rust-lang/cargo/pull/3979
[cargo/3988]: https://github.com/rust-lang/cargo/pull/3988
[cargo/4026]: https://github.com/rust-lang/cargo/pull/4026
[cargo/4031]: https://github.com/rust-lang/cargo/pull/4031
[cargo/4084]: https://github.com/rust-lang/cargo/pull/4084

See the [detailed release notes][notes] for more.

### Contributors to 1.19.0

Many people came together to create Rust 1.19. We couldn't have done it without
all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.19.0)
