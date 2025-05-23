+++
path = "2023/10/05/Rust-1.73.0"
title = "Announcing Rust 1.73.0"
authors = ["The Rust Release Team"]
aliases = [
    "2023/10/05/Rust-1.73.0.html",
    "releases/1.73.0",
]

[extra]
release = true
+++

The Rust team is happy to announce a new version of Rust, 1.73.0. Rust is a programming language empowering everyone to build reliable and efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.73.0 with:

```
$ rustup update stable
```

If you don't have it already, you can [get `rustup`](https://www.rust-lang.org/install.html) from the appropriate page on our website, and check out the [detailed release notes for 1.73.0](https://github.com/rust-lang/rust/releases/tag/1.73.0) on GitHub.

If you'd like to help us out by testing future releases, you might consider updating locally to use the beta channel (`rustup default beta`) or the nightly channel (`rustup default nightly`). Please [report](https://github.com/rust-lang/rust/issues/new/choose) any bugs you might come across!

## What's in 1.73.0 stable

## Cleaner panic messages

The output produced by the default panic handler has been changed
to put the panic message on its own line instead of wrapping it in quotes.
This can make panic messages easier to read, as shown in this example:

<div style="margin:1em"><pre><code>fn main() {
    let file = "ferris.txt";
    panic!("oh no! {file:?} not found!");
}</code></pre>
Output before Rust 1.73:
<pre style="margin-top:0"><code style="background:#000;color:#ccc" class="language-text">thread 'main' panicked at 'oh no! "ferris.txt" not found!', src/main.rs:3:5</code></pre>
Output starting in Rust 1.73:
<pre style="margin-top:0"><code style="background:#000;color:#ccc" class="language-text">thread 'main' panicked at src/main.rs:3:5:
oh no! "ferris.txt" not found!</code></pre></div>

This is especially useful when the message is long, contains nested quotes, or spans multiple lines.

Additionally, the panic messages produced by `assert_eq` and `assert_ne` have
been modified, moving the custom message (the third argument)
and removing some unnecessary punctuation, as shown below:

<div style="margin:1em"><pre><code>fn main() {
    assert_eq!("🦀", "🐟", "ferris is not a fish");
}</code></pre>
Output before Rust 1.73:
<pre style="margin-top:0"><code style="background:#000;color:#ccc" class="language-text">thread 'main' panicked at 'assertion failed: `(left == right)`
 left: `"🦀"`,
right: `"🐟"`: ferris is not a fish', src/main.rs:2:5</code></pre>
Output starting in Rust 1.73:
<pre style="margin-top:0"><code style="background:#000;color:#ccc" class="language-text">thread 'main' panicked at src/main.rs:2:5:
assertion `left == right` failed: ferris is not a fish
 left: "🦀"
right: "🐟"</code></pre></div>

### Thread local initialization

As proposed in [RFC 3184](https://github.com/rust-lang/rfcs/blob/master/text/3184-thread-local-cell-methods.md), `LocalKey<Cell<T>>` and `LocalKey<RefCell<T>>` can now be directly manipulated with `get()`, `set()`, `take()`, and `replace()` methods, rather than jumping through a `with(|inner| ...)` closure as needed for general `LocalKey` work. `LocalKey<T>` is the type of `thread_local!` statics.

The new methods make common code more concise and avoid running the extra initialization code for the default value specified in `thread_local!` for new threads.

```rust
thread_local! {
    static THINGS: Cell<Vec<i32>> = Cell::new(Vec::new());
}

fn f() {
    // before:
    THINGS.with(|i| i.set(vec![1, 2, 3]));
    // now:
    THINGS.set(vec![1, 2, 3]);

    // ...

    // before:
    let v = THINGS.with(|i| i.take());
    // now:
    let v: Vec<i32> = THINGS.take();
}
```

### Stabilized APIs

- [Unsigned `{integer}::div_ceil`](https://doc.rust-lang.org/stable/std/primitive.u32.html#method.div_ceil)
- [Unsigned `{integer}::next_multiple_of`](https://doc.rust-lang.org/stable/std/primitive.u32.html#method.next_multiple_of)
- [Unsigned `{integer}::checked_next_multiple_of`](https://doc.rust-lang.org/stable/std/primitive.u32.html#method.checked_next_multiple_of)
- [`std::ffi::FromBytesUntilNulError`](https://doc.rust-lang.org/stable/std/ffi/struct.FromBytesUntilNulError.html)
- [`std::os::unix::fs::chown`](https://doc.rust-lang.org/stable/std/os/unix/fs/fn.chown.html)
- [`std::os::unix::fs::fchown`](https://doc.rust-lang.org/stable/std/os/unix/fs/fn.fchown.html)
- [`std::os::unix::fs::lchown`](https://doc.rust-lang.org/stable/std/os/unix/fs/fn.lchown.html)
- [`LocalKey::<Cell<T>>::get`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.get)
- [`LocalKey::<Cell<T>>::set`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.set)
- [`LocalKey::<Cell<T>>::take`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.take)
- [`LocalKey::<Cell<T>>::replace`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.replace)
- [`LocalKey::<RefCell<T>>::with_borrow`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.with_borrow)
- [`LocalKey::<RefCell<T>>::with_borrow_mut`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.with_borrow_mut)
- [`LocalKey::<RefCell<T>>::set`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.set-1)
- [`LocalKey::<RefCell<T>>::take`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.take-1)
- [`LocalKey::<RefCell<T>>::replace`](https://doc.rust-lang.org/stable/std/thread/struct.LocalKey.html#method.replace-1)

These APIs are now stable in const contexts:

- [`rc::Weak::new`](https://doc.rust-lang.org/stable/alloc/rc/struct.Weak.html#method.new)
- [`sync::Weak::new`](https://doc.rust-lang.org/stable/alloc/sync/struct.Weak.html#method.new)
- [`NonNull::as_ref`](https://doc.rust-lang.org/stable/core/ptr/struct.NonNull.html#method.as_ref)

### Other changes

Check out everything that changed in [Rust](https://github.com/rust-lang/rust/releases/tag/1.73.0), [Cargo](https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-173-2023-10-05), and [Clippy](https://github.com/rust-lang/rust-clippy/blob/master/CHANGELOG.md#rust-173).

## Contributors to 1.73.0

Many people came together to create Rust 1.73.0. We couldn't have done it without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.73.0/)
