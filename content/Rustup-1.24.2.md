+++
path = "2021/05/17/Rustup-1.24.2"
title = "Announcing Rustup 1.24.2"
authors = ["The Rustup Working Group"]
aliases = ["2021/05/17/Rustup-1.24.2.html"]
+++

The rustup working group is happy to announce the release of rustup version 1.24.2. [Rustup][install] is the recommended tool to install [Rust][rust], a programming language that is empowering everyone to build reliable and efficient software.

If you have a previous version of rustup installed, getting rustup 1.24.2 is as easy as closing your IDE and running:

```
rustup self update
```

Rustup will also automatically update itself at the end of a normal toolchain update:

```
rustup update
```

If you don't have it already, you can [get rustup][install] from the appropriate page on our website.

[rust]: https://www.rust-lang.org
[install]: https://rustup.rs

## What's new in rustup 1.24.2

1.24.2 introduces pooled allocations to prevent memory fragmentation issues on
some platforms with 1.24.x. We're not entirely sure what aspect of the streamed
unpacking logic caused allocator fragmentation, but memory pools are a well
known fix that should solve this for all platforms.

Those who were encountering CI issues with 1.24.1 should find them resolved.

### Other changes

You can check out all the changes to Rustup for 1.24.2 in the [changelog]!

Rustup's documentation is also available in [the rustup book][book].

[changelog]: https://github.com/rust-lang/rustup/blob/stable/CHANGELOG.md
[book]: https://rust-lang.github.io/rustup/

Finally, the Rustup working group are pleased to welcome a new member. Between
1.24.1 and 1.24.2 二手掉包工程师 (hi-rustin) has joined, having already made some
excellent contributions.

## Thanks

Thanks again to all the contributors who made rustup 1.24.2 possible!

- Carol (Nichols || Goulding)
- Daniel Silverstone
- João Marcos Bezerra
- Josh Rotenberg
- Jynn Nelson
- Martijn Gribnau
- pierwill
- Robert Collins
- 二手掉包工程师 (hi-rustin)
