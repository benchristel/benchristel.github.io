---
layout: post
title: Musings on Statefulness
---

# Musings on Statefulness

One of the central problems of UX is aligning the user's
mental model of the system with the actual state of the system.
This involves either informing the user that state has changed,
or changing the state of the system to what we think the
user thinks it is.

Despite the efforts of programmers and UX designers, our
mental model of a system's
state can get out of sync with its actual state. Often, this
happens simply because the computer has a perfect memory,
and we don't. When a computer system does something we don't
understand, it's often because we've simply forgotten about
some long-ago state change that is now affecting its behavior.

Here's an example: Say I ran a program last week and I
remember that it worked fine. Today I try to run it and it
crashes with a cryptic error. The problem is that at some
point in the interim I changed `LD_LIBRARY_PATH` in my
environment and now my program can't find a library it
needs.

You might say that I should just be more cognizant of the
environment in which I am running programs. However,
I maintain that this is really a UX issue. Given that
computer systems can be "up" and accumulating state for
arbitrary amounts of time, the user can't really be expected
to remember or look up every state change that could affect
the operation of a program.

I call this type of UX issue a
"mental state synchronization error".

## Avoiding Mental State Synchronization Errors

Fortunately, there are a few high-level techniques that you,
the programmer and/or UX designer, can apply to mitigate
mental state synchronization errors. I describe them below.

### Convention Over Configuration

We can sometimes eliminate state by using a "convention over
configuration" approach. If programs always looked for
libraries in `/usr/local/lib`, and that path were not
configurable, I couldn't have run into the `LD_LIBRARY_PATH`
issue described earlier. I might, of course, run into
difficulties if I wanted to have multiple versions of
a library installed on my system and be able to switch
between them at will. I'd have to move files into and out
of `/usr/local/lib`. However, there would be one less
thing I'd have to check if I wanted to know which libraries
my program was loading.

### Eliminate Dependencies

If a program's operation doesn't depend on any state, we don't have
to worry about mental state synchronization. Continuing the
`LD_LIBRARY_PATH` example: if my program were written in
Go, it couldn't depend on external libraries, so it wouldn't
matter what `LD_LIBRARY_PATH` was set to or what libraries
were present on my system; the program would execute the
same code on every run, no matter what.

Eliminating dependencies is often quite simple, though it
comes at the cost of duplicating code or data that could
be gotten from some external source. However, that
duplication is a small price to pay for a more
comprehensible system.

### Make Inputs Explicit

One way to make sure mental state and system state are
synchronized is to force the user to tell the system what
their mental state is. If I had to explicitly specify an
`LD_LIBRARY_PATH` whenever I ran a program, I probably
would be more aware that the value of `LD_LIBRARY_PATH`
might affect how my program runs.

Unfortunately, forcing the user to specify inputs that
rarely change can make a system very tedious to use. You
run the risk that the input will just become noise, a
chunk of meaningless boilerplate that gets thoughtlessly
copy-pasted. Even so, the user is probably more likely to
think about that particular input as a potential source
of any issues they encounter than if the input is
left implicit.

### Make State Visible

This is state synchronization in the opposite direction from
"Make Inputs Explicit". Instead of the user telling the
system what their mental state is, the system tells the
user what its state is.

This approach creates less tedium and mental burden than
"Make Inputs Explicit", since the user just has to visually
process the state instead of typing it out. Unfortunately,
the flip side of this is that the user can easily tune out
the display of state, especially if it changes infrequently.
(How often have you SSH'd somewhere and forgotten that your
commands were being sent to another machine, even though the
hostname was in the command prompt?)

### Make State Comprehensible

The final technique is to make it easier for users to debug
state-related issues by showing them all the state that is
affecting their current operation *when they request to see
it*. For extra credit, you could even show the earlier
operations that changed the state to its present values.

It would be nice if, when I encountered my `LD_LIBRARY_PATH`
issue, the operating system could remind me that my program
implicitly depends on `LD_LIBRARY_PATH`. With that kind of UX,
debugging state-related issues becomes a rote search through
a (hopefully short) list of implicit inputs.

While extremely powerful, this technique carries the highest
implemen&shy;tation cost. To be done well, the system would have
to be reflective; i.e. it would need to be able to inspect
its own code and find out what its inputs are. If you are
designing a new language, programming environment, or
operating system, it may be worthwhile to keep this kind of
feature in mind :)
