---
layout: post
title: Stress-testing Verse
---

# Stress-testing Verse

I was concerned about Verse's ability to handle lots and lots of code,
so I did some informal stress testing on my dual-core 2014
Mac Mini.

![screenshot of the Verse editor with over 100,000 lines of code](/assets/stress-testing-verse.png)

At around 60,000 lines, the latency of starting an
application becomes noticeable. At 100,000 lines, the
experience is sufficiently degraded that I would call it
"not great". The latency of operations (typing in the
editor, starting the app, syntax highlighting)
is still well in the sub-second range, but there's a
noticeable delay between when you hit a key and when the
character shows up on screen.

I'm actually kind of astonished that it performs this well.
The largest human-maintained files I've encountered are
in the 10-20 kiloline range, and some IDEs written in Java
start to choke on files of that size. Verse is able
to handle 20,000 lines, no problem.

Kudos to the fine maintainers of [Ace](https://ace.c9.io/)
for making an awesome editor that can handle my unbelievably
shitty code.

How will Verse scale to multiple-file projects? Well, the
editor only shows one file at a time, and only
the current file is syntax-highlighted and hot-swapped.
Additional files will, of course, increase the burden on
memory, but not on the CPU. I see no reason Verse wouldn't
work for projects with megabytes of JavaScript, as long as
it's appropriately decomposed into multiple files.
