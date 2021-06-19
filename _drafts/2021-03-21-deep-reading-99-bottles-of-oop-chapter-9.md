---
layout: post
title: Deep Reading: 99 Bottles of OOP, Chapter 9
---

Intro: extoll the virtues of the book--thoroughly reshaped
how I thought about OO design before I was halfway through.

A funny consequence: I don't think I read the last chapter
on my first read-through. I recently went back and read this
chapter, and let's just say that I have some bones to pick.

> Your general approach to testing should be to create a unit test for every class, and to test every method in that class’s public API.

Testing individual methods only makes sense if objects represent
immutable values—c.f. faux-O.

> It cannot be emphasized strongly enough that most classes deserve their own explicit unit tests.

> If you subscribe to the principle that applications should have 100% test coverage, you might need to reexamine your definition of that rule. Perhaps it means "100% of the code should be exercised during unit tests," rather than "100% of the public methods should have their own personal tests."

Huh? I thought this was already the commonly-accepted meaning of test coverage. Every test coverage tool I've used does not care whether methods
are called directly by a test or indirectly—they just count lines executed.

> Be extremely biased towards creating a unit test for every method in every class’s public API.

> 1.Tell everyone to implement the correct API in their role players.
>
> ...
>
> 4. Programmatically verify that all players of a role respond to its API, define the correct number of parameters, receive arguments of the correct types, and return the expected type.
>
> 5. Switch to a language with compiler guaranteed interfaces.

I think this undersells the value of contract tests by quite a bit.
Static typing can't do everything contract tests can do: for
instance, it can't enforce the guarantee that a stack must not
be empty after pushing an item. It can't verify that a repository
object retrieves values that were previously stored.

If you don't have stateful objects then indeed contract
tests are less valuable. Metz and Owen do indicate that
they're leaning toward immutability in 99 bottles, and while
I do think immutability is a good default, there are cases
where a mutable object or two can simplify a program
immensely.

One shortcoming of the book is that it limits itself to a
purely functional domain without being explicit about the
fact that it's doing so. IMO this sets people up for an
unpleasant surprise when they try to apply the book's wisdom
on real projects that interact with state. It's not that TDD
and refactoring are inapplicable to stateful systems, but
that stateful systems require additional, non-obvious
techniques.
