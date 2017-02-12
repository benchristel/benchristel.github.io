---
layout: post
title: TDD is neither necessary nor sufficient for quality code
---

# TDD is neither necessary nor sufficient for quality code

Define "Code Quality"
- Can someone use the software for its intended task?
- How many bugs or unspecified behaviors does it have? How serious are they?
- When I need to add a feature or fix a bug, how quickly can I identify the part of the code that needs to change?
- How long does it take me to actually *make* the change?
- When I'm done changing stuff, does the code have any new bugs?
- How quickly can new developers grasp the relationship between the code and its behavior?

These are, of course, merely heuristics. I don't know how to formally define code quality. However, when I talk about code quality, these are the kinds of metrics I have in mind.

Automated tests—especially when deployed as part of a test-driven development regime—enhance code quality.

Matt Parker is of the opinion that TDD helps increase your confidence that the code is correct.

Let's look at the quality metrics I've defined and see how TDD drives each of them.

> Can someone use the software for its intended task?

If you have tests that exercise the behaviors of your code, you have some level of assurance that users will be able to exercise those behaviors too. If you keep those tests passing, even the code on your development branch is likely to meet most or all of your users' requirements.

> How many bugs or unspecified behaviors does it have? How serious are they?

If you write tests, you'll have a better chance of catching unhandled edge cases that could turn into bugs.

> When I need to add a feature or fix a bug, how quickly can I identify the part of the code that needs to change?

The best tests serve as living documentation for the code, allowing developers to quickly match behaviors to parts of the codebase.

> How long does it take me to actually *make* the change?

If you have good test coverage, you can change code without worrying whether you've broken something. If you're not sure if something is covered by tests, change it and see if any tests fail.

> When I'm done changing stuff, does the code have any new bugs?

Tests help prevent regressions.

> How quickly can new developers grasp the relationship between the code and its behavior?

If you're not sure what a piece of code is for, change it and see what tests break.

## Tests usually aren't good documentation

On most of the projects I've worked on, the tests are messy. No matter how readable and well-factored we can make the code, the tests always lag one step behind.

You can refactor your tests to be more readable, but woe betide the next person who has to change those tests. They'll end up running `git blame` and cursing your name as they hack their way through the wilderness of your fixtures and mocks and bespoke DSLs.

Not to mention that it's irresponsible to refactor without test coverage, and your tests probably don't have tests, so... how is it ever safe to refactor them? The ideal is to make your tests so simple that they will never need refactoring to be readable, but in my experience few teams approach this ideal.

Anecdotally, people who have tried Elm don't feel they need to write unit tests or do TDD to the same degree that they used to when writing object-oriented code. Why?

- Elm functions cannot have side-effects. A major area of uncertainty is eliminated in functional languages.
- Elm's type system prevents many edge-case bugs that can occur in other languages. Elm literally *will not let you* forget to handle a `null` or an empty list. As a result, there's no reason to write tests for those cases. If the code compiles, chances are good that it handles edge cases just fine.

Why test stuff with side effects?

Heavily-mocked tests are hard to understand

If you separate data transformations from side effects, you get tests that have no test doubles, and tests that have nothing but test doubles.

100% line coverage does not imply 100% confidence. You can have 100% of your code covered by tests and still have bugs. There could be edge cases you didn't think about (and therefore didn't test), or integration problems that your unit tests won't catch.

Testing is still important. You want to test your data transformations, especially the edge cases. You want to test that subsystems integrate correctly. You probably still want to do TDD for your data transformations.

## But you're not *really* doing TDD!

TDD, correctly applied, *does* lead to higher code quality than test-last development. But note that weasel word, "correctly". It is not enough to simply write a test, make it pass, refactor, and repeat. The amount of lore that you have to internalize to do TDD effectively is immense. It spills out into every other area of software development: the SOLID design principles, the law of Demeter, design patterns... this is why I hate it when TDD gurus say ["listen to your tests"](http://chimera.labs.oreilly.com/books/1234000000754/ch19.html#_listen_to_your_tests_ugly_tests_signal_a_need_to_refactor). If you say ["listen to your tests"](https://grysz.com/2015/11/05/listen-to-your-tests-a-real-example/) to a junior developer, what you're actually telling them is "go rederive the last 50 years of advances in software design methodologies, and then fix your shitty code." Of course, what they *hear* is not that; they hear "the solution to this problem would be obvious if you were smarter".

Defining "TDD" in such a way pretty much makes it synonymous with "software development skills", thereby rendering the term void of informational content. If we instead define TDD as the simple practice of "red, green, refactor"—and no more than that—then TDD is neither necessary nor sufficient for quality software.
