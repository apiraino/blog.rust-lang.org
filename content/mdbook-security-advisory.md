+++
path = "2021/01/04/mdbook-security-advisory"
title = "mdBook security advisory"
authors = ["Rust Security Response WG"]
aliases = ["2021/01/04/mdbook-security-advisory.html"]
+++

> This is a cross-post of [the official security advisory][ml]. The official post
> contains a signed version with our PGP key, as well.

[ml]: https://groups.google.com/g/rustlang-security-announcements/c/3-sO6of29O0

The Rust Security Response Working Group was recently notified of a security
issue affecting the search feature of mdBook, which could allow an attacker to
execute arbitrary JavaScript code on the page.

The CVE for this vulnerability is [CVE-2020-26297][1].

## Overview

The search feature of mdBook (introduced in version 0.1.4) was affected by a
cross site scripting vulnerability that allowed an attacker to execute
arbitrary JavaScript code on an user's browser by tricking the user into typing
a malicious search query, or tricking the user into clicking a link to the
search page with the malicious search query prefilled.

mdBook 0.4.5 fixes the vulnerability by properly escaping the search query.

## Mitigations

Owners of websites built with mdBook have to upgrade to mdBook 0.4.5 or greater
and rebuild their website contents with it. It's possible to install mdBook
0.4.5 on the local system with:

```
cargo install mdbook --version 0.4.5 --force
```

## Acknowledgements

Thanks to Kamil Vavra for responsibly disclosing the vulnerability to us
according to [our security policy][2].

## Timeline of events

All times are listed in UTC.

* 2020-12-30 20:14 - The issue is reported to the Rust Security Response WG
* 2020-12-30 20:32 - The issue is acknowledged and the investigation began
* 2020-12-30 21:21 - Found the cause of the vulnerability and prepared the patch
* 2021-01-04 15:00 - Patched version released and vulnerability disclosed

[1]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-26297
[2]: https://www.rust-lang.org/policies/security
