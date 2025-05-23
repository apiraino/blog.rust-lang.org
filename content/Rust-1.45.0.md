+++
path = "2020/07/16/Rust-1.45.0"
title = "Announcing Rust 1.45.0"
authors = ["The Rust Release Team"]
aliases = [
    "2020/07/16/Rust-1.45.0.html",
    "releases/1.45.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.45.0. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, getting Rust
1.45.0 is as easy as:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.45.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1450-2020-07-16

## What's in 1.45.0 stable

There are two big changes to be aware of in Rust 1.45.0: a fix for some
long-standing unsoundness when casting between integers and floats, and the
stabilization of the final feature needed for one of the more popular web
frameworks to work on stable Rust.

## Fixing unsoundness in casts

[Issue 10184](https://github.com/rust-lang/rust/issues/10184) was originally
opened back in October of 2013, a year and a half before Rust 1.0. As you may
know, `rustc` uses [LLVM](http://llvm.org/) as a compiler backend. When you
write code like this:

```rust
pub fn cast(x: f32) -> u8 {
    x as u8
}
```

The Rust compiler in Rust 1.44.0 and before would produce LLVM-IR that looks
like this:

```
define i8 @_ZN10playground4cast17h1bdf307357423fcfE(float %x) unnamed_addr #0 {
start:
  %0 = fptoui float %x to i8
  ret i8 %0
}
```

That `fptoui` implements the cast, it is short for "floating point to
unsigned integer."

But there's a problem here. From [the
docs](https://llvm.org/docs/LangRef.html#fptoui-to-instruction):

> The ‘fptoui’ instruction converts its floating-point operand into the
> nearest (rounding towards zero) unsigned integer value. If the value cannot
> fit in ty2, the result is a poison value.

Now, unless you happen to dig into the depths of compilers regularly, you may
not understand what that means. It's full of jargon, but there's a simpler
explanation: if you cast a floating point number that's large to an integer
that's small, you get undefined behavior.

That means that this, for example, was not well-defined:

```rust
fn cast(x: f32) -> u8 {
    x as u8
}

fn main() {
    let f = 300.0;

    let x = cast(f);

    println!("x: {}", x);
}
```

On Rust 1.44.0, this happens to print "x: 0" on my machine. But it could
print anything, or do anything: this is undefined behavior. But the `unsafe`
keyword is not used within this block of code. This is what we call a
"soundness" bug, that is, it is a bug where the compiler does the wrong thing.
We tag these bugs as
[I-unsound](https://github.com/rust-lang/rust/issues?q=is%3Aissue+is%3Aopen+label%3A%22I-unsound+%F0%9F%92%A5%22)
on our issue tracker, and take them very seriously.

This bug took a long time to resolve, though. The reason is that it was very
unclear what the correct path forward was.

In the end, the decision was made to do this:

* `as` would perform a "saturating cast".
* A new `unsafe` cast would be added if you wanted to skip the checks.

This is very similar to array access, for example:

* `array[i]` will check to make sure that `array` has at least `i + 1` elements.
* You can use `unsafe { array.get_unchecked(i) }` to skip the check.

So, what's a saturating cast? Let's look at a slightly modified example:

```rust
fn cast(x: f32) -> u8 {
    x as u8
}

fn main() {
    let too_big = 300.0;
    let too_small = -100.0;
    let nan = f32::NAN;

    println!("too_big_casted = {}", cast(too_big));
    println!("too_small_casted = {}", cast(too_small));
    println!("not_a_number_casted = {}", cast(nan));
}
```

This will print:

```
too_big_casted = 255
too_small_casted = 0
not_a_number_casted = 0
```

That is, numbers that are too big turn into the largest possible value.
Numbers that are too small produce the smallest possible value (which is
zero). NaN produces zero.

The new API to cast in an unsafe manner is:

```rust
let x: f32 = 1.0;
let y: u8 = unsafe { x.to_int_unchecked() };
```

But as always, you should only use this method as a last resort. Just like
with array access, the compiler can often optimize the checks away, making
the safe and unsafe versions equivalent when the compiler can prove it.

## Stabilizing function-like procedural macros in expressions, patterns, and statements

In [Rust 1.30.0](https://blog.rust-lang.org/2018/10/25/Rust-1.30.0.html), we stabilized
"function-like procedural macros in item position." For example, [the
`gnome-class` crate](https://gitlab.gnome.org/federico/gnome-class):

> Gnome-class is a procedural macro for Rust.  Within the macro, we
> define a mini-language which looks as Rust-y as possible, and that has
> extensions to let you define GObject subclasses, their properties,
> signals, interface implementations, and the rest of GObject's
> features.  The goal is to require no unsafe code on your part.

This looks like this:

```rust
gobject_gen! {
    class MyClass: GObject {
        foo: Cell<i32>,
        bar: RefCell<String>,
    }

    impl MyClass {
        virtual fn my_virtual_method(&self, x: i32) {
            ... do something with x ...
        }
    }
}
```

The "in item position" bit is some jargon, but basically what this means is that
you could only invoke `gobject_gen!` in certain places in your code.

Rust 1.45.0 adds the ability to invoke procedural macros in three new places:

```rust
// imagine we have a procedural macro named "mac"

mac!(); // item position, this was what was stable before

// but these three are new:
fn main() {
  let expr = mac!(); // expression position

  match expr {
      mac!() => {} // pattern position
  }

  mac!(); // statement position
}
```

Being able to use macros in more places is interesting, but there's another
reason why many Rustaceans have been waiting for this feature for a long time:
[Rocket](https://rocket.rs). Initially released in December of 2016, Rocket is
a popular web framework for Rust often described as one of the best things the
Rust ecosystem has to offer. Here's the "hello world" example from its upcoming
release:

```rust
#[macro_use] extern crate rocket;

#[get("/<name>/<age>")]
fn hello(name: String, age: u8) -> String {
    format!("Hello, {} year old named {}!", age, name)
}

#[launch]
fn rocket() -> rocket::Rocket {
    rocket::ignite().mount("/hello", routes![hello])
}
```

Until today, Rocket depended on nightly-only features to deliver on its promise
of flexibility and ergonomics. In fact, as can be seen on the [project's
homepage](https://rocket.rs/v0.4), the same example above in the current version
of Rocket requires the `proc_macro_hygiene` feature to compile. However, as you
may guess from the feature's name, today it ships in stable! [This
issue](https://github.com/SergioBenitez/Rocket/issues/19) tracked the history of
nightly-only features in Rocket. Now, they're all checked off!

This next version of Rocket is still in the works, but when released, many folks
will be very happy :)

### Library changes

In Rust 1.45.0, the following APIs were stabilized:

- [`Arc::as_ptr`]
- [`BTreeMap::remove_entry`]
- [`Rc::as_ptr`]
- [`rc::Weak::as_ptr`]
- [`rc::Weak::from_raw`]
- [`rc::Weak::into_raw`]
- [`str::strip_prefix`]
- [`str::strip_suffix`]
- [`sync::Weak::as_ptr`]
- [`sync::Weak::from_raw`]
- [`sync::Weak::into_raw`]
- [`char::UNICODE_VERSION`]
- [`Span::resolved_at`]
- [`Span::located_at`]
- [`Span::mixed_site`]
- [`unix::process::CommandExt::arg0`]

Additionally, you can [use `char` with
ranges](https://github.com/rust-lang/rust/pull/72413/), to iterate over
codepoints:

```rust
for ch in 'a'..='z' {
    print!("{}", ch);
}
println!();
// Prints "abcdefghijklmnopqrstuvwxyz"
```

For a full list of changes, see [the full release notes][notes].

### Other changes

There are other changes in the Rust 1.45.0 release: check out what changed in
[Rust][notes], [Cargo][relnotes-cargo], and [Clippy][relnotes-clippy].

## Contributors to 1.45.0

Many people came together to create Rust 1.45.0. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.45.0/)

[relnotes-cargo]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-145-2020-07-16
[relnotes-clippy]: https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-145

[`Arc::as_ptr`]: https://doc.rust-lang.org/stable/std/sync/struct.Arc.html#method.as_ptr
[`BTreeMap::remove_entry`]: https://doc.rust-lang.org/stable/std/collections/struct.BTreeMap.html#method.remove_entry
[`Rc::as_ptr`]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html#method.as_ptr
[`rc::Weak::as_ptr`]: https://doc.rust-lang.org/stable/std/rc/struct.Weak.html#method.as_ptr
[`rc::Weak::from_raw`]: https://doc.rust-lang.org/stable/std/rc/struct.Weak.html#method.from_raw
[`rc::Weak::into_raw`]: https://doc.rust-lang.org/stable/std/rc/struct.Weak.html#method.into_raw
[`sync::Weak::as_ptr`]: https://doc.rust-lang.org/stable/std/sync/struct.Weak.html#method.as_ptr
[`sync::Weak::from_raw`]: https://doc.rust-lang.org/stable/std/sync/struct.Weak.html#method.from_raw
[`sync::Weak::into_raw`]: https://doc.rust-lang.org/stable/std/sync/struct.Weak.html#method.into_raw
[`str::strip_prefix`]: https://doc.rust-lang.org/stable/std/primitive.str.html#method.strip_prefix
[`str::strip_suffix`]: https://doc.rust-lang.org/stable/std/primitive.str.html#method.strip_suffix
[`char::UNICODE_VERSION`]: https://doc.rust-lang.org/stable/std/char/constant.UNICODE_VERSION.html
[`Span::resolved_at`]: https://doc.rust-lang.org/stable/proc_macro/struct.Span.html#method.resolved_at
[`Span::located_at`]: https://doc.rust-lang.org/stable/proc_macro/struct.Span.html#method.located_at
[`Span::mixed_site`]: https://doc.rust-lang.org/stable/proc_macro/struct.Span.html#method.mixed_site
[`unix::process::CommandExt::arg0`]: https://doc.rust-lang.org/std/os/unix/process/trait.CommandExt.html#tymethod.arg0
