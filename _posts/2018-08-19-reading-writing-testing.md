---
layout: post
title: Reading, Writing, Testing
---

# Reading, Writing, Testing

When I was in grade school, my teachers would often write
remarks on my papers like "What about X?" or "Can you give
an example of this?"—even though I'd addressed their
concerns in the
very next sentence or paragraph. This frustrated me to no
end. If they would only *read* what I'd written! What was
so hard about deferring judgement until they'd at least
read a couple sentences past the point where they started
to feel critical?

Now that I've been writing (and reading) code for a while,
I'm much more sympathetic to my teachers' position. Every
unresolved question or what-if scenario, whether in a paper
or a chunk of code, takes
[precious mental resources](http://seriouspony.com/blog/2013/7/24/your-app-makes-me-fat)
to hold in mind until it is resolved. My teachers, no doubt
weary from many late nights of grading, opted to spare
their working memories the burden by writing their concerns
on my papers. In this way, the act of writing comments
helped them understand what I'd written. The seemingly
premature remarks that so frustrated my younger self were
not a sign that my teachers misunderstood my writing;
rather, they may have been necessary to the process of
understanding.

By the same token, I often start refactoring code before
I've entirely read and understood it. I use refactoring as
*a tool for understanding* because it allows me
to separate in my mind the invariant properties of the code
from the surface manifestation of the algorithm, and it's
usually the properties, not the algorithm, that I want
to understand.

The original authors
of the code might wring their hands about this, arguing that
I should seek to understand before I criticize, but
refactoring is not about criticism. "It's not you, it's me",
I'd say to them. I undo many of my refactors soon after
making them, because changing the code isn't always the
point. For me, refactoring is part of the process of
reading.

## The role of tests

I don't recommend refactoring without a comprehensive suite of
tests that are fast enough to run after every small change. The
reason should be obvious: messing around with existing code
without some way to quickly and accurately check whether
you've broken something is just asking for trouble.

But tests also help the understanding process in another
way: by reading the tests, you can very quickly answer
"what-if" questions about the code.

- [What if a keypress event is received during a `sleep`?](https://github.com/benchristel/verse/blob/fedd915e02ad399f54d1f7a1ce3597dd0b993af0/src/core/Process.spec.js#L104)
- What if both inputs are empty?
- What if the flag is `false` and the iterator never terminates?
- [What if the passed-in length doesn't match the content?](https://xkcd.com/1354/)

In a sense, the tests are an FAQ for the code. They are a
way for the author of the code to preempt our questions and
criticisms, by saying "yep, already thought of that".

If our goal is not to change the code under test, but simply
to understand it, being able to answer these questions
quickly and accurately can make us more confident in using
the code.

## Choosing the black box

When teaching TDD and unit-testing, I often say that unit
tests function as executable developer documentation. That
is, they serve the same purpose as, say, JavaDoc, but with
the advantage that the test runner continually verifies that
they're up to date.

However, I've noticed that my own behavior doesn't really
support this statement. I rarely use the tests to
understand unfamiliar code. I typically just dig into the
implementation, using refactoring as I describe above to
quickly get my bearings.

Object-oriented programmers love to talk about
encapsulation, and the separation of implementation from
interface. However, when I have the choice between looking
at the interface of an object (as described by the tests)
and looking at its implementation, I generally opt for the
implementation.

If we were really creating easy-to-understand interfaces,
I'd expect to *choose* to look at the interface and the
behavior rather than the implementation. If black boxes are
best, I should *choose* black-box thinking over white-box
thinking. The fact that I don't is an indication that the
interfaces I encounter do not provide good abstractions;
they either hide essential aspects of behavior or reveal
accidental details.

Black-box thinking fails even harder when tests use a
lot of test doubles. I've seen heavily-mocked tests
where [the tests exactly
mirror the structure of the code](https://github.com/danfinnie/code-driven-development),
only with more boilerplate and noise. If the code and tests
are saying the exact same thing, but the tests are harder
to read, why should anyone ever look at the tests?

I believe we can assess the quality of our designs by paying
attention to how much we read tests and how much we read
code. When we can gain all the understanding we need by
reading tests, it means two things:

- We have created an interface that can be understood independently
  of its implementation, meaning we don't have to hold that
  implementation in our heads when we use
  the interface. This prevents us from getting bogged down
  in details as the codebase grows.
- We have encapsulated a non-trivial piece of behavior—something
  we really don't want to re-implement all the time. It's only
  when the behavior is non-trivial that the tests can become
  easier to understand than the code.

Here are some examples of interfaces I love to depend on:

- Higher-order functions like `map`, `reduce`, and `filter`.
  It's especially lovely when these work on many different
  collections, like arrays, strings, and lazy sequences.
- Parsers. I'd much rather look at the tests for a parser than
  the code, and I imagine you would too :)
- String and array manipulation functions where off-by-one
  errors are a risk. The code for such functions is often
  a tiny bit anxiety-inducing, but the tests are joyfully
  simple and expressive.
- Objects with complex internal state but simple invariants
  that must be maintained. Examples include collection
  types like lists, sets, and queues. I'd say [the `Process`
  class](https://github.com/benchristel/verse/blob/fedd915e02ad399f54d1f7a1ce3597dd0b993af0/src/core/Process.js) at the heart of [Verse](https://benchristel.github.io/verse)
  also falls into this category.

## Paths to understanding

So, we've got two ways to understand a piece of code:

- Read the code, refactoring as needed to separate the
  essence from the details in our minds.
- Read the tests and the interface.

Of these, I think the first is often the most useful, but
only because the design of the code we work with is so
often flawed. Code that must be read (and reworked) to be
understood imposes a tax on development. The more code
there is in the codebase that can only be understood in
terms of its implementation, the slower development will be.
If, on the other hand, we write code that is easiest to
think about in terms of its interface and behavior, we can
go fast forever.
