+++
path = "2018/05/10/Rust-1.26"
title = "Announcing Rust 1.26"
authors = ["The Rust Core Team"]
aliases = [
    "2018/05/10/Rust-1.26.html",
    "releases/1.26.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.26.0. Rust is a
systems programming language focused on safety, speed, and concurrency.

If you have a previous version of Rust installed via rustup, getting Rust
1.26.0 is as easy as:

```bash
$ rustup update stable
```

If you don't have it already, you can [get `rustup`][install] from the
appropriate page on our website, and check out the [detailed release notes for
1.26.0][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/master/RELEASES.md#version-1260-2018-05-10

## What's in 1.26.0 stable

The past few releases have had a steady stream of relatively minor additions. We've
been working on a lot of stuff, however, and it's all starting to land in stable. 1.26 is
possibly the most feature-packed release since Rust 1.0. Let's dig in!

#### "The Rust Programming Language" Second Edition

For almost 18 months, Carol, Steve, and others have been working on a
complete re-write of "The Rust Programming Language." We've learned a lot
about how people learn Rust since the first book was written, and this
version is an improvement in every way.

We've shipped the draft of the second edition on the website for a while now,
but with a disclaimer that it was a work in progress. At this point, the book
is undergoing some final, minor copy-edits, and being prepared for print. As
such, with this release, we are recommending the second edition over the
first. You can [read it on
doc.rust-lang.org](https://doc.rust-lang.org/book/second-edition/) or
locally via `rustup doc --book`.

Speaking of print, you can pre-order a dead tree version of the book [from
NoStarch Press](https://www.nostarch.com/Rust). The contents are identical,
but you get a nice physical book to put on a shelf, or a beautifully typeset
PDF. Proceeds are going to charity.

#### `impl Trait`

At long last, `impl Trait` is here! This feature has been highly desired for
quite a while, and provides a feature known as "existential types." It's
simpler than that sounds, however. The core of it is this idea:

```rust
fn foo() -> impl Trait {
    // ...
}
```

This type signature says "`foo` is a function that takes no arguments but
returns a type that implements the `Trait` trait." That is, we're not
going to tell you what the return type of `foo` actually is, only that
it implements a particular trait. You may wonder how this differs from
a trait object:

```rust
fn foo() -> Box<Trait> {
    // ...
}
```

While it's true that you could have written this code today, it's not
ideal in all situations. Let's say we have a trait `Trait` that
is implemented for both `i32` and `f32`:

```rust
trait Trait {
    fn method(&self);
}

impl Trait for i32 {
    // implementation goes here
}

impl Trait for f32 {
    // implementation goes here
}
```

Consider this function:

```rust
fn foo() -> ? {
    5
}
```

We want to fill in the return type with something. Previously, only the trait
object version was possible:

```rust
fn foo() -> Box<Trait> {
    Box::new(5) as Box<Trait>
}
```

But this introduces a `Box`, which means allocation. We're not actually
returning some kind of dynamic data here either, so the dynamic dispatch of
the trait object hurts too. So instead, as of Rust 1.26, you can write this:

```rust
fn foo() -> impl Trait {
    5
}
```

This doesn't create a trait object, it's like we had written `-> i32`, but
instead, we're only mentioning the part about `Trait`. We get static
dispatch, but we can hide the real type like this.

Why is this useful? One good use is closures. Remember that closures in
Rust all have a unique, un-writable type, yet implement the `Fn` trait.
This means that if your function returns a closure, you can do this:

```rust
// before
fn foo() -> Box<Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}

// after
fn foo() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

No boxing, no dynamic dispatch. A related scenario happens when returning
iterators. Not only do iterators often include closures, but since they
nest, you get quite deeply nested types. For example:

```rust
fn foo() {
    vec![1, 2, 3]
        .into_iter()
        .map(|x| x + 1)
        .filter(|x| x % 2 == 0)
}
```

when compiled, gives this error:

```
error[E0308]: mismatched types
 --> src/main.rs:5:5
  |
5 | /     vec![1, 2, 3]
6 | |         .into_iter()
7 | |         .map(|x| x + 1)
8 | |         .filter(|x| x % 2 == 0)
  | |_______________________________^ expected (), found struct `std::iter::Filter`
  |
  = note: expected type `()`
             found type `std::iter::Filter<std::iter::Map<std::vec::IntoIter<{integer}>, [closure@src/main.rs:7:14: 7:23]>, [closure@src/main.rs:8:17: 8:31]>`
```

That's a huge 'found type'. Each adapter in the chain adds a new type.
Additionally, we have that closure in there. Previously, we'd have had
to use a trait object here, but now we can simply do

```rust
fn foo() -> impl Iterator<Item = i32> {
    vec![1, 2, 3]
        .into_iter()
        .map(|x| x + 1)
        .filter(|x| x % 2 == 0)
}
```

and be done with it. Working with [futures] is very similar.

[futures]: https://crates.io/crates/futures

It's important to note that sometimes trait objects are still
what you need. You can only use `impl Trait` if your function returns
a single type; if you want to return multiple, you need dynamic dispatch.
For example:

```rust
fn foo(x: i32) -> Box<Iterator<Item = i32>> {
    let iter = vec![1, 2, 3]
        .into_iter()
        .map(|x| x + 1);

    if x % 2 == 0 {
        Box::new(iter.filter(|x| x % 2 == 0))
    } else {
        Box::new(iter)
    }
}
```

Here, we may return a filtered iterator, or maybe not. There's two different
types that can be returned, and so we must use a trait object.

Oh, and one last thing: to make the syntax a bit more symmetrical, you can
use `impl Trait` in argument position too. That is:

```rust
// before
fn foo<T: Trait>(x: T) {

// after
fn foo(x: impl Trait) {
```

which can look a bit nicer for short signatures.

> Side note for you type theorists out there: this isn't an existential, still
> a universal. In other words, `impl Trait` is universal in an input position, but
> existential in an output position.

#### Nicer `match` bindings

Have you ever had a reference to an `Option`, and tried to use `match`? For
example, code like this:

```rust
fn hello(arg: &Option<String>) {
    match arg {
        Some(name) => println!("Hello {}!", name),
        None => println!("I don't know who you are."),
    }
}
```

If you tried to compile this in Rust 1.25, you'd get this error:

```
error[E0658]: non-reference pattern used to match a reference (see issue #42640)
 --> src/main.rs:6:9
  |
6 |         Some(name) => println!("Hello {}!", name),
  |         ^^^^^^^^^^ help: consider using a reference: `&Some(name)`

error[E0658]: non-reference pattern used to match a reference (see issue #42640)
 --> src/main.rs:7:9
  |
7 |         None => println!("I don't know who you are."),
  |         ^^^^ help: consider using a reference: `&None`
```

Okay, sure. Let's modify the code:

```rust
fn hello(arg: &Option<String>) {
    match arg {
        &Some(name) => println!("Hello {}!", name),
        &None => println!("I don't know who you are."),
    }
}
```

We added the `&`s the compiler complained about. Let's try to compile again:

```
error[E0507]: cannot move out of borrowed content
 --> src/main.rs:6:9
  |
6 |         &Some(name) => println!("Hello {}!", name),
  |         ^^^^^^----^
  |         |     |
  |         |     hint: to prevent move, use `ref name` or `ref mut name`
  |         cannot move out of borrowed content
```

Okay, sure. Let's make the compiler happy again by taking its advice:

```rust
fn hello(arg: &Option<String>) {
    match arg {
        &Some(ref name) => println!("Hello {}!", name),
        &None => println!("I don't know who you are."),
    }
}
```

This will finally compile. We had to add two `&`s, and a `ref`. But more
importantly, none of this was really *helpful* to us as programmers. Sure,
we forgot a `&` at first, but does that matter? We had to add `ref` to
get a reference to the inside of the option, but we couldn't do anything *but*
get a reference, as we can't move out of a `&T`.

So, as of Rust 1.26, the initial code, without the `&`s and `ref`, will just
compile and do exactly what you'd expect. In short, the compiler will automatically
reference or de-reference in `match` statements. So when we say

```rust
    match arg {
        Some(name) => println!("Hello {}!", name),
```

the compiler automatically references the `Some`, and since we're borrowing,
`name` is bound as `ref name` automatically as well. If we were mutating:

```rust
fn hello(arg: &mut Option<String>) {
    match arg {
        Some(name) => name.push_str(", world"),
        None => (),
    }
}
```

the compiler will automatically borrow by mutable reference, and `name` will
be bound as `ref mut` too.

We think this will remove a significant papercut for new and old Rustaceans
alike. The compiler will just do the right thing more often without the need
for boilerplate.

#### `main` can return a `Result`

Speaking of papercuts, since Rust uses the `Result` type for returning
errors, and `?` to make handling them easy, a common pain-point of
new Rustaceans is to try and use `?` in `main`:

```rust
use std::fs::File;

fn main() {
    let f = File::open("bar.txt")?;
}
```

This will give an error like "error[E0277]: the `?` operator can only be used
in a function that returns `Result`". This leads to a pattern where many
people write code that [looks like this](https://doc.rust-lang.org/book/second-edition/ch12-03-improving-error-handling-and-modularity.html#extracting-logic-from-main):

```rust
fn run(config: Config) -> Result<(), Box<Error>> {
    // --snip--
}

fn main() {
    // --snip--

    if let Err(e) = run(config) {
        println!("Application error: {}", e);

        process::exit(1);
    }
}
```

Our `run` function has all of the real logic, and `main`
calls `run`, only checking to see if there was an error
and exiting. We need to make this second function because
`main` can't return a `Result`, but we'd like to use `?`
in that logic.

In Rust 1.26, you can now declare `main` that returns `Result`:

```rust
use std::fs::File;

fn main() -> Result<(), std::io::Error> {
    let f = File::open("bar.txt")?;

    Ok(())
}
```

This now works just fine! If `main` returns an error, this will
exit with an error code, and print out a debug representation
of the error.

#### Inclusive ranges with `..=`

Since well before Rust 1.0, you've been able to create exclusive ranges with `..`
like this:

```rust
for i in 1..3 {
    println!("i: {}", i);
}
```

This will print `i: 1` and then `i: 2`. In Rust 1.26, you can now create an
inclusive range, like this:

```rust
for i in 1..=3 {
    println!("i: {}", i);
}
```

This will print `i: 1` and then `i: 2` like before, but also `i: 3`; the
three is included in the range. Inclusive ranges are especially useful
if you want to iterate over every possible value in a range. For example,
this is a surprising Rust program:

```rust
fn takes_u8(x: u8) {
    // ...
}

fn main() {
    for i in 0..256 {
        println!("i: {}", i);
        takes_u8(i);
    }
}
```

What does this program do? The answer: nothing. The warning we get when
compiling has a hint:

```
warning: literal out of range for u8
 --> src/main.rs:6:17
  |
6 |     for i in 0..256 {
  |                 ^^^
  |
  = note: #[warn(overflowing_literals)] on by default
```

That's right, since `i` is a `u8`, this overflows, and is the same as writing
`for i in 0..0`, so the loop executes zero times.

We can do this with inclusive ranges, however:

```rust
fn takes_u8(x: u8) {
    // ...
}

fn main() {
    for i in 0..=255 {
        println!("i: {}", i);
        takes_u8(i);
    }
}
```

This will produce those 256 lines of output you might have been expecting.

#### Basic slice patterns

Another long-awaited feature is "slice patterns." These let you match on
slices similar to how you match on other data types. For example:

```rust
let arr = [1, 2, 3];

match arr {
    [1, _, _] => "starts with one",
    [a, b, c] => "starts with something else",
}
```

In this case, we know `arr` has a length of three, and so we need three entries
inside the `[]`s. We can also match when we don't know the length:

```rust
fn foo(s: &[u8]) {
    match s {
        [a, b] => (),
        [a, b, c] => (),
        _ => (),
    }
}
```

Here, we don't know how long `s` is, so we can write the first two arms, each with
different lengths. This also means we need a `_` term, since we aren't covering
every possible length, nor could we!

#### Speed improvements

We continue to work on the speed of the compiler. We discovered that deeply
nesting types was non-linear in some cases, and [a fix was
implemented](https://github.com/rust-lang/rust/pull/48296). We're seeing up
to a 12% reduction in compile times from this change, but many other smaller
fixes landed as well. More to come in the future!

#### 128 bit integers

Finally, a very simple feature: Rust now has 128 bit integers!

```rust
let x: i128 = 0;
let y: u128 = 0;
```

These are twice the size of `u64`, and so can hold more values. More specifically,

* `u128`: 0 - 340,282,366,920,938,463,463,374,607,431,768,211,455
* `i128`: −170,141,183,460,469,231,731,687,303,715,884,105,728 - 170,141,183,460,469,231,731,687,303,715,884,105,727

Whew!

See the [detailed release notes][notes] for more.

### Library stabilizations

We stabilized [`fs::read_to_string`](https://doc.rust-lang.org/std/fs/fn.read_to_string.html),
a convenience over `File::open` and `io::Read::read_to_string` for easily reading an entire
file into memory at once:

```rust
use std::fs;
use std::net::SocketAddr;

let foo: SocketAddr = fs::read_to_string("address.txt")?.parse()?;
```

You can now [format numbers as hexadecimal with `Debug`
formatting](https://github.com/rust-lang/rust/pull/48978):

```rust
assert!(format!("{:02x?}", b"Foo\0") == "[46, 6f, 6f, 00]")
```

Trailing commas [are now supported by all macros in the standard
library](https://github.com/rust-lang/rust/pull/48056).

See the [detailed release notes][notes] for more.

### Cargo features

Cargo didn't receive many big new features this release but rather saw a steady
stream of stability and performance improvements. Cargo should now resolve lock
files even faster, backtrack more intelligently, and require manual `cargo
update` invocations less. Cargo's binary [now also shares the same version as
`rustc`](https://github.com/rust-lang/cargo/pull/5083).

See the [detailed release notes][notes] for more.

## Contributors to 1.26.0

Many people came together to create Rust 1.26. We couldn't have done it
without all of you.

[Thanks!](https://thanks.rust-lang.org/rust/1.26.0)
