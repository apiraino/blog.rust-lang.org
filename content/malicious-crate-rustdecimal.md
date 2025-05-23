+++
path = "2022/05/10/malicious-crate-rustdecimal"
title = "Security advisory: malicious crate rustdecimal"
authors = ["The Rust Security Response WG"]
aliases = ["2022/05/10/malicious-crate-rustdecimal.html"]
+++

> This is a cross-post of [the official security advisory][advisory]. The
> official advisory contains a signed version with our PGP key, as well.

[advisory]: https://groups.google.com/g/rustlang-security-announcements/c/5DVtC8pgJLw

The Rust Security Response WG and the crates.io team [were notified][1] on
2022-05-02 of the existence of the malicious crate `rustdecimal`, which
contained malware. The crate name was intentionally similar to the name of the
popular [`rust_decimal`][2] crate, hoping that potential victims would misspell
its name (an attack called "typosquatting").

To protect the security of the ecosystem, the crates.io team permanently
removed the crate from the registry as soon as it was made aware of the
malware. An analysis of all the crates on crates.io was also performed, and no
other crate with similar code patterns was found.

Keep in mind that the [`rust_decimal`][2] crate was **not** compromised, and it
is still safe to use.

## Analysis of the crate

The crate had less than 500 downloads since its first release on 2022-03-25,
and no crates on the crates.io registry depended on it.

The crate contained identical source code and functionality as the legit
`rust_decimal` crate, except for the `Decimal::new` function.

When the function was called, it checked whether the `GITLAB_CI` environment
variable was set, and if so it downloaded a binary payload into
`/tmp/git-updater.bin` and executed it. The binary payload supported both Linux
and macOS, but not Windows.

An analysis of the binary payload was not possible, as the download URL didn't
work anymore when the analysis was performed.

## Recommendations

If your project or organization is running GitLab CI, we strongly recommend
checking whether your project or one of its dependencies depended on the
`rustdecimal` crate, starting from 2022-03-25. If you notice a dependency on
that crate, you should consider your CI environment to be compromised.

In general, we recommend regularly auditing your dependencies, and only
depending on crates whose author you trust. If you notice any suspicious
behavior in a crate's source code please follow [the Rust security
policy][3] and report it to the Rust Security Response WG.

## Acknowledgements

We want to thank GitHub user [`@safinaskar`][4] for identifying the
malicious crate in [this GitHub issue][1].

[1]: https://github.com/paupino/rust-decimal/issues/514#issuecomment-1115408888
[2]: https://crates.io/crates/rust_decimal
[3]: https://www.rust-lang.org/policies/security
[4]: https://github.com/safinaskar
