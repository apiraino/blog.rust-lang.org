+++
path = "2023/03/28/Rust-1.68.2"
title = "Announcing Rust 1.68.2"
authors = ["The Rust Release Team"]
aliases = [
    "2023/03/28/Rust-1.68.2.html",
    "releases/1.68.2",
]

[extra]
release = true
+++

The Rust team has published a new point release of Rust, 1.68.2. Rust is a
programming language that is empowering everyone to build reliable and
efficient software.

If you have a previous version of Rust installed via rustup, you can get 1.68.2 with:

```
rustup update stable
```

If you don't have it already, you can [get `rustup`][install]
from the appropriate page on our website, and check out the
[detailed release notes for 1.68.2][notes] on GitHub.

[install]: https://www.rust-lang.org/install.html
[notes]: https://github.com/rust-lang/rust/blob/stable/RELEASES.md#version-1682-2023-03-28

## What's in 1.68.2 stable

Rust 1.68.2 addresses [GitHub's recent rotation of their RSA SSH host
key](https://github.blog/2023-03-23-we-updated-our-rsa-ssh-host-key/), which
happened on March 24th 2023 after their previous key accidentally leaked:

* [GitHub's RSA key bundled in Cargo has been
  updated](https://github.com/rust-lang/cargo/pull/11883), to ensure systems
  that haven't interacted with GitHub yet won't connect trusting the leaked
  key.

* [The leaked key has been hardcoded as revoked in
  Cargo](https://github.com/rust-lang/cargo/pull/11889), to ensure the key
  won't be used by Cargo even on systems that still trust the key.

[Support for `@revoked` entries in
`.ssh/known_hosts`](https://github.com/rust-lang/cargo/pull/11635) (along with
a better error message when the unsupported `@cert-authority` entries are used)
is also included in Rust 1.68.2, as that change was a pre-requisite for
backporting the hardcoded revocation.

If you cannot upgrade to Rust 1.68.2, we recommend [following GitHub's
instructions](https://github.blog/2023-03-23-we-updated-our-rsa-ssh-host-key/#what-you-can-do)
on updating the trusted keys in your system. Note that the keys bundled in
Cargo are only used if no trusted key for `github.com` is found on the system.

### Contributors to 1.68.2

Many people came together to create Rust 1.68.2. We couldn't have done it
without all of you. [Thanks!](https://thanks.rust-lang.org/rust/1.68.2/)
