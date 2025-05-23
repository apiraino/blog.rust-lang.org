+++
path = "2018/12/17/Rust-2018-dev-tools"
title = "Tools in the 2018 edition"
authors = ["The Dev-tools team"]
aliases = ["2018/12/17/Rust-2018-dev-tools.html"]
+++

Tooling is an important part of what makes a programming language practical and
productive. Rust has always had some great tools (Cargo in particular has a
well-deserved reputation as a best-in-class package manager and build tool), and
the 2018 edition includes more tools which we hope further improve Rust users'
experience.

In this blog post I'll cover Clippy and Rustfmt – two tools that have been
around for a few years and are now stable and ready for general use. I'll also
cover IDE support – a key workflow for many users which is now much better
supported. I'll start by talking about Rustfix, a new tool which was central to
our edition migration plans.

## Rustfix

Rustfix is a tool for automatically making changes to Rust code. It is a key
part of our migration story for the 2018 edition, making the transition from
2015 to 2018 editions much easier, and in many cases completely automatic. This
is essential, since without such a tool we'd be much more limited in the kinds
of breaking changes users would accept.

A simple example:

```rust
trait Foo {
    fn foo(&self, i32);
}
```

The above is legal in Rust 2015, but not in Rust 2018 (method arguments must be
made explicit). Rustfix changes the above code to:

```rust
trait Foo {
    fn foo(&self, _: i32);
}
```

For detailed information on how to use Rustfix, see [these instructions](https://doc.rust-lang.org/stable/edition-guide/editions/transitioning-an-existing-project-to-a-new-edition.html).
To transition your code from the 2015 to 2018 edition, run `cargo fix --edition`.

Rustfix can do a lot, but it is not perfect. When it can't fix your code, it
will emit a warning informing you that you need to fix it manually. We're
continuing to work to improve things.

Rustfix works by automatically applying suggestions from the compiler. When we
add or improve the compiler's suggestion for fixing an error or warning, then
that improves Rustfix. We use the same information in an IDE to give quick fixes
(such as automatically adding imports).

Thank you to Pascal Hertleif (killercup), Oliver Scherer (oli-obk), Alex
Crichton, Zack Davis, and Eric Huss for developing Rustfix and the compiler
lints which it uses.


## Clippy

Clippy is a linter for Rust. It has numerous (currently 290!) lints to help
improve the correctness, performance and style of your programs. Each lint can
be turned on or off (`allow`), and configured as either an error (`deny`) or
warning (`warn`).

An example: the [`iter_next_loop`](https://rust-lang.github.io/rust-clippy/master/index.html#iter_next_loop)
lint checks that you haven't made an error by iterating on the result of `next`
rather than the object you're calling `next` on (this is an easy mistake to make
when changing a `while let` loop to a `for` loop).

```rust
for x in y.next() {
    // ...
}
```

will give the error

```
error: you are iterating over `Iterator::next()` which is an Option; this will compile but is probably not what you want
 --> src/main.rs:4:14
  |
4 |     for x in y.next() {
  |              ^^^^^^^^
  |
  = note: #[deny(clippy::iter_next_loop)] on by default
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#iter_next_loop
```

Clippy works by extending the Rust compiler. The compiler has support for a few
built-in lints, Clippy uses the same mechanisms but with lots more lints. That
means Clippy's error/warning format should be familiar, you should be able to
apply Clippy's suggestions in your IDE (or using Rustfix), and that the lints
are reliable and accurate.

With Rust 1.31 and the 2018 edition, Clippy is available on stable Rust and has
backwards compatibility guarantees (if it had a version number, it would be
1.0). Clippy has the same stability guarantees as rustc: new lints may be added,
and lints may be modified to add more functionality, however lints may never be
removed (only deprecated). This means that code that compiles with Clippy will
continue to compile with Clippy (provided there are no lints set to error via
`deny`), but may throw new warnings.

Clippy can be installed using `rustup component add clippy`, then use it with
`cargo clippy`. For more information, including how to run it in your CI, see
[the repo readme](https://github.com/rust-lang/rust-clippy/).

Thank you Clippy team (Pascal Hertleif (killercup), Oliver Scherer (oli-obk),
Manish Goregaokar (manishearth), and Andre Bogus (llogiq))!


## Rustfmt

Rustfmt is a tool for formatting your source code. It takes arbitrary, messy
code and turns it into neat, beautifully styled code.

Automatically formatting saves you time and mental energy. You don't need to
worry about style as you code. If you use Rustfmt in your CI (`cargo fmt
--check`), then you don't need to worry about code style in review. By using a
standard style you make your project feel more familiar for new contributors and
spare yourself arguments about code style. Rust's [standard code
style](https://github.com/rust-lang/rfcs/blob/master/style-guide/README.md) is
the Rustfmt default, but if you must, then you can customize Rustfmt
extensively.

Rustfmt 1.0 is part of the 2018 edition release. It should work on all code and
will be backwards compatible until the 2.0 release. By backwards compatible we
mean that if your code is formatted (i.e., excluding bugs which prevent any
formatting or code which does not compile), it will always be formatted in the
same way. This guarantee only applies if you use the default formatting options.

Rustfmt is not done. Formatting is not perfect, in particular we don't touch
comments and string literals and we are pretty limited with macro definitions
and some macro uses. We're likely to improve formatting here, but you will need
to opt-in to these changes until there is a 2.0 release. We *are* planning on
having a 2.0 release. Unlike Rust itself, we think its a good idea to have a
breaking release of Rustfmt and expect that to happen some time in late 2019.

To install Rustfmt, use `rustup component add rustfmt`. To format your project,
use `cargo fmt`. You can also format individual files using `rustfmt` (though
note that by default rustfmt will format nested modules). You can also use
Rustfmt in your editor or IDE using the RLS (see below; no need to install
rustfmt for this, it comes as part of the RLS). We recommend configuring your
editor to run rustfmt on save. Not having to think about formatting at all as
you type is a pleasant change.

Thank you Seiichi Uchida (topecongiro), Marcus Klaas, and all the Rustfmt
contributors!

## IDE support

For many users, their IDE is the most important tool. Rust IDE support has been
in the works for a while and is a highly demanded feature. Rust is now supported
in many IDEs and editors:
[IntelliJ](https://plugins.jetbrains.com/plugin/8182-rust), [Visual Studio
Code](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust),
[Atom](https://github.com/rust-lang-nursery/atom-ide-rust), [Sublime
Text](https://github.com/rust-lang/rust-enhanced),
[Eclipse](https://www.eclipse.org/downloads/packages/release/photon/r/eclipse-ide-rust-developers-includes-incubating-components)
 (and more...). Follow each link for installation instructions.

Editor support is powered in two different ways: IntelliJ uses its own compiler,
the other editors use the Rust compiler via the Rust Language Server (RLS). Both
approaches give a good but imperfect IDE experience. You should probably choose
based on which editor you prefer (although if your project does not use Cargo,
then you won't be able to use the RLS).

All these editors come with support for standard IDE functions including 'go to
definition', 'find all references', code completion, renaming, and reformatting.

The RLS has been developed by the Rust dev tools team, it is a bid to bring Rust
support to as many IDEs and editors as possible. It directly uses Cargo and the
Rust compiler to provide accurate information about a program. Due to
performance constraints, code completion is not yet powered by the compiler and
therefore can be a bit more hit and miss than other features.

Thanks to the IDEs and editors team for work on the RLS and the various IDEs and
extensions (alexheretic, autozimu, jasonwilliams, LucasBullen, matklad,
vlad20012, Xanewok), Jonathan Turner for helping start off the RLS, and
phildawes, kngwyu, jwilm, and the other Racer contributors for their work on
Racer (the code completion component of the RLS)!

## The future

We're not done yet! There's lots more we think we can do in the tools domain
over the next year or so.

We've been improving rust debugging support in LLDB and GDB and there is more in
the works. We're experimenting with distributing our own versions with Rustup
and making debugging from your IDE easier and more powerful.

We hope to make the RLS faster, more stable, and more accurate; including using
the compiler for code completion.

We want to make Cargo a lot more powerful: Cargo will handle compiled binaries
as well as source code, which will make building and installing crates faster.
We will support better integration with other build systems (which in turn will
enable using the RLS with more projects). We'll add commands for adding and
upgrading dependencies, and to help with security audits.

Rustdoc will see improvements to its source view (powered by the RLS) and links
between documentation for different crates.

There's always lots of interesting things to work on. If you'd like to help chat
to us on GitHub or [Discord](https://discordapp.com/invite/rust-lang).
