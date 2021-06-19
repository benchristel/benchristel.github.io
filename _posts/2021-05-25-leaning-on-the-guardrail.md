---
layout: post
title: Leaning on the Guardrail
---

# Leaning on the Guardrail

I recently re-read Sandi Metz and Katrina Owen's book [_99 Bottles of OOP_](https://sandimetz.com/99bottles).
It's a fantastic and detailed look at TDD and refactoring
and I highly recommend it.

However, there are some ideas in it that don't quite mesh
with my TDD practice. Notably: the book opines that
your tests must always stay green during refactoring, and
if they fail at any point you should undo your last change.
I disagree with this approach. During refactoring, I'll often
intentionally put my code into a failing state before completing
the change and seeing the tests go green again. In this post
I want to talk about the benefits of working this way.

## Testing the Tests

It's a fact of life that test coverage isn't perfect. Coverage
is certainly going to be _better_ if you do TDD, but covering
every possible set of inputs is impractical, so it is totally
possible to introduce bugs during refactoring that evade
all the tests.

By verifying that the tests go red when our refactor is partially
complete, and green when it's fully complete, we can continually
verify our test coverage.

## Leaning on the Guardrail

Some people use the metaphor of tests as a "safety net" that
catches mistakes. My opinion is that the safety net metaphor
is misguided. A safety net is only useful when there's a
catastrophe, but using tests only to prevent catastrophic
mistakes wastes much of their power. We aren't tightrope
performers, paid to intentionally do something dangerous for
the sake of spectacle. We're just trying to get from A to B
in the safest, most expedient way possible. So the metaphor
I prefer is "tests are a guardrail".

Imagine you're walking along a trail at the edge of a cliff.
There's a handrail to keep you from falling off, but you're
not sure how secure it really is.

Do you:

- lean on the rail only when you slip and start to fall,
  putting all your weight on it at once?
- put a little bit of weight on the rail all the time, so
  if you discover a place where it's wobbly and can't actually
  support your weight, you take extra precautions (fixing the rail?)
  before going on.

IMO, the second option is clearly preferable. The benefit of
tests is not just that they provide some abstract, remote sense
of safety, but that you can actually verify just how much
safety they're giving you and adjust that dial up or down depending
on the needs of your project.
