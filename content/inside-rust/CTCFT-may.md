+++
path = "inside-rust/2022/05/10/CTCFT-may"
title = "CTCFT 2022-05-16 Agenda"
authors = ["Rust CTCFT Team"]
aliases = ["inside-rust/2022/05/10/CTCFT-may.html"]
+++

The next ["Cross Team Collaboration Fun Times" (CTCFT)][CTCFT] meeting will take
place on Monday, 2022-05-16 at **11am US Eastern Time** ([click to see in your
time zone][timezone]). You’ll find the full details (along with a calendar
event, zoom details, etc) [on the CTCFT website][CTCFT-meeting].

[CTCFT]: https://rust-lang.github.io/ctcft/
[timezone]: https://everytimezone.com/s/6c2a0d08
[CTCFT-meeting]: https://rust-lang.github.io/ctcft/meetings/2022-05-16.html

## Agenda

The theme for this month's CTCFT is **Embedded Rust**. We'll hear from some
members of the Rust Embedded Working Group and community about the state of the ecosystem, as
well as how async Rust is working for embedded systems. We also have some people
coming in from the automotive industry to talk about how Rust use is starting to
progress.

- (5 min) Opening remarks 👋 ([angelonfira])
- (15 min) A whirlwind tour of Embedded Rust ([jamesmunns])
  - A brief history of the embedded-wg and use of Rust for embedded
  - A look at how developing embedded Rust looks like today
  - A sample of patterns that are special to embedded Rust, or differences from
    "desktop" Rust
- (15 min) Async Rust for Embedded Systems ([Dirbaio])
  - We'll explore how concurrency is traditionally handled in embedded, and how
    Rust's async makes it significantly easier while still requiring no runtime,
    no OS, and no allocation, and what Rust improvements could make it even more
    awesome.
- (15 min) Rust in Automotive ([cpetig], [skade])
  - We'll look at Rust from a Functional Safety perspective, and continuing to
    the AUTOSAR architecture. We'll also look a bit at what Ferrocene's role is
    in all this, and look at the AUTOSAR Rust bindings. Finally, we'll see
    what's next for this space.
- (5 min) Closing ([angelonfira])

[angelonfira]: https://github.com/angelonfira
[jamesmunns]: https://github.com/jamesmunns
[Dirbaio]: https://github.com/Dirbaio
[cpetig]: https://github.com/cpetig
[skade]: https://github.com/skade

## Afterwards: Social Hour

Like always, we'll be running a social hour after the CTCFT. The idea is really
simple: for the hour after the meeting, we will create breakout rooms in Zoom
with different themes. You can join any breakout room you like and hangout.
