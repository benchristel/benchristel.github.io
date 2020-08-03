---
layout: post
title: Tech Talks That Changed How I Think
---

# Tech Talks That Changed How I Think

(Disclaimer: I mention my employer,
[Pivotal Software](https://pivotal.io/), in this post, but
the views I present are my own. Pivotal has not endorsed
this content.)

I facilitate tech video lunches every Monday at Pivotal's
Palo Alto office (basically a bunch of people congregate in
a conference room, the company buys us food, and we watch
someone talk about software development for 30–45 minutes.
It's good fun!) Over the last four years this has caused me
to watch hundreds of talks from many different conferences.

In no particular order, here are the technical videos that
have had the most influence on me. These are the talks that
I recommend to my friends and colleagues—the talks I come
back to over and over again. Many are so dense and so good
that you'll probably want to watch them many times over the
course of your career, and you'll learn new things every
time. Though most of these videos use examples from specific
languages and technologies, the ideas presented generalize
so well that I expect they will remain relevant for decades
to come.

## "Boundaries" — Gary Bernhardt

When I first saw this talk, in 2014, it blew my mind. Gary
dissects one of the core problems facing software
developers: how to test software in a way that gives you
confidence in its correctness *and* the ability
to change the code safely and easily. He backs up the theory
with concrete examples and even a live demo.

"Boundaries" greatly influenced [Verse's](https://benchristel.github.io/verse)
use of generator coroutines to corral side effects.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/yTkzNHF6rMs" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "Ease at Work" — Kent Beck

[Google discovered](https://www.nytimes.com/2016/02/28/magazine/what-google-learned-from-its-quest-to-build-the-perfect-team.html)
that psychological safety is *the* key factor
that predicts a team's performance. Kent talks about his own
journey toward feeling good about what he does, and muses
on some common thought patterns and software-development
tropes that detract from psychological safety.

<iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?list=PLE9763518A2765373" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "Functional Principles for Object-Oriented Developers" — Jessica Kerr

In this talk, Jessica dismantles the myth that functional
programming and object orientation are somehow mutually
exclusive disciplines. She provides examples of
object-oriented code that could benefit from the application
of functional patterns. The talk is aimed at developers who
are used to object-oriented programming.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/GpXsQ-NIKXY" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "Making Badass Developers" — Kathy Sierra

The one thing software developers can count on is that
*things will change*. The languages and frameworks we use
today will evolve, or be replaced. Adapting to change means
learning new things, but this can seem daunting, [even
overwhelming](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f).
Kathy starts off by normalizing this feeling
of *overwhelmedness*—we're all human, and we all have
limited attention and cognitive resources to spend on
learning new stuff. She describes a few simple tricks for
learning new skills quickly and effectively—and being
compassionate toward the people who will have to learn to
use the tools *you* create.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/FKTxC9pl-WM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "Fast Test, Slow Test" — Gary Bernhardt

This talk convinced me to get serious about the size and
speed of my tests. Small, fast unit tests enable a virtuosic
development flow that makes integrated testing look more
like punchcard coding than a modern development practice.
Once you start writing and relying on fast tests, you'll
never want to write another slow one.

"Fast Test, Slow Test" was a major influence on
[Verse's](https://benchristel.github.io/verse)
design goal of allowing programmers to write tests that
are fast enough to run on every keystroke.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/RAxiiRPHS9k" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "Your Brain At Work" — David Rock

David argues that programmers often overvalue rational
thought and therefore over-strain their prefrontal
cortex—one of the weakest and least efficient parts of
the brain. He provides some concrete tips on how to make the
most of our limited attention and rational processing
ability while giving the other parts of our brain the
respect and care they deserve.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/XeJSXfXep4M" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "You can be a kernel hacker!" — Julia Evans

Julia's talk opened my eyes to a fact about OS design that, in
retrospect, should have been obvious: operating systems are
designed to help programmers, not to intimidate them, and
once you've grokked that fact a whole world of amazingly
helpful tools and techniques opens up to you.

Also, contrary to what I'd previously believed, unrestricted side
effects are not built into the nature of imperative
programming. As early as the 1970s, programmers understood
that unrestricted side effects were confusing and
potentially dangerous. Thus, in Unix, all side effects must
take place via the kernel's system call apparatus, a channel
which is easily observable and readily intelligible.

Julia's talk inspired the design of [Verse's](https://benchristel.github.io/verse)
syscall-equivalent: side effects in Verse are
represented as data yielded from a coroutine.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/0IQlpFWTFbM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## "Making Impossible States Impossible" — Richard Feldman

When I started programming, I found static typing annoying:
writing out types was just boring paperwork that I had to do
so the compiler could optimize my program. Richard's talk
flipped that belief on its head: types are for humans! Not
only can they help you understand your code, they let you
precisely encode the system's possible states, preventing
runtime errors and bugs.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/IcgmSRJHu_8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## 6502 Computer Series — Ben Eater

It's a truism that "computers are fundamentally imperative,
stateful systems". But underlying that statefulness are
analog, _functional_ elec&shy;tronics. To get imperative,
stateful systems
you have use hacks like [SR latches](https://www.youtube.com/watch?v=KM0DdEaY5sY) which feed the output
of a functional system back into its input. You can even
take advantage of the functional nature of computer systems
to [encode complicated logic in ROM](https://www.youtube.com/watch?v=BA12Z7gQ4P0).

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/yl8vPW5hydQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
