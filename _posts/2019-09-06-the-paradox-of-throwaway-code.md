---
layout: post
title: The Paradox of Throwaway Code
---

# The Paradox of Throwaway Code

I want to write code that can be thrown away.

By that, I don't mean that I want to write useless code, nor
that I want to write quick, sloppy code. Almost the
opposite.

I believe that all code is temporary. So when I find that a
method or a class has outlived its usefulness, I want to be
able to discard it without worry.

There is a trap that people fall into when they try to get
software built in a hurry. They think "this code's not going
to stick around; it's just a quick hack. Since it will be
deleted soon, why spend time on design or tests?"

The problem is, when the time comes and the code can finally
be deleted, often *you can't find it.* It's buried under a
mess of semi-related logic, experiments, and quick fixes.
The code is so entangled that even when you think you've
found an unused bit, *you can't really be confident that
it's actually unused*. Therefore, the safest thing to do
is to just leave the code as it is.

Over time, such code accretes more and more entangled hacks,
until finally it collapses and the whole thing must be
rewritten from scratch.

By contrast, when you design well and test thoughtfully,
code that's no longer needed stands out. Since it has a
single clearly defined purpose and is decoupled from its
surroundings, it can be pruned from the codebase as easily
as a dead leaf can be plucked from a tree.

Design and TDD are largely about decoupling. Decoupling is
the inverse of attachment. Attachment is what makes it hard
to let things go.
