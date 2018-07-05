---
layout: post
title: TDD Is About Minimizing Testing
---

# TDD Is About Minimizing Testing

If you've read much about test-driven development, you'll
likely have heard many of the
[criticisms](http://david.heinemeierhansson.com/2014/test-induced-design-damage.html)
and the
[rebuttals](http://blog.cleancoder.com/uncle-bob/2017/12/18/Excuses.html).
To wit: TDD is a force for design (good or bad), or a matter of
professionalism, or a sign of weakness, or religious
dogma that serves no end but Kent Beck's book sales,
and on and on the arguments go.
In this post, I want to advocate for TDD from a slightly
different angle—one you probably haven't heard before.

TDD is about *saving your company money*.

**The purpose of TDD is to *minimize the amount of testing
you do while maximizing the value of each test*.**
Test less, test better. The ability to write and run fewer tests
means that
TDD, [correctly applied](/blog/2018/06/08/principles-of-testable-design/),
will let your development team minimize the amount time and
money they spend testing, without impacting quality.

To see why TDD is effective at doing this, let's look at
some of the alternatives.

## Quality Assurance

In college, I did an internship at a company that was trying
to automate more of their testing. Their main product was a
physical device controlled by a GUI written in Java.

Their problem was that their GUI had bugs. Many bugs. They
had three QA people who couldn't keep up with the number of
bugs the developers were putting in the code. The developers,
in turn, couldn't keep up with the number of bugs the QA
folks were reporting. And these were not subtle,
hard-to-spot bugs. I myself, as an untrained intern who was
not working in QA, found
several. Things like "when I go to this screen and go back
none of the buttons work," or "when I press OK it actually
cancels", or "when I reach this screen by such-and-such a path,
the theme color changes from green to blue." I never saw the
code for the UI, but given the types of bugs that came out
of it, I imagine it must have been truly nightmarish.

Despite their best efforts, the QA team was merely a
band-aid on a huge, festering problem. The problem was that
the code was crap, and shoveling crappy bugfixes onto it
did nothing but produce more bugs.

If something isn't working, the obvious fix is to
automate it; that way, it can fail to provide value
faster.
Thus, we arrive at the solution to the QA
bottleneck: automated integrated testing.

## Post-Development Integrated Testing

A common way of doing test automation is to write tests
in Capybara or some other framework that pokes and prods the UI
the way a user would. Basically, this approach automates
what would otherwise be the job of a really bored QA person.

The problems with integrated tests are many.

- They are time-consuming to write.
- They are difficult to read.
- They take a really long time to run.
- When they fail, it takes a long time to figure out what's
  actually broken.
- They often depend on external services or precise timings
  and therefore can fail "flakily", for reasons unrelated to
  the code they're supposed to be testing.
- They often can't even run on developers' workstations,
  meaning your continuous integration build is always red.
- They often require expensive infrastructure (e.g. cloud
  VMs, or even dedicated physical hardware).
- They're unlikely to cover every behavior of
  every module of your code.
- Because of this, you still need QA.
- They *definitely* can't cover all paths through *multiple*
  modules of your code, because the number of such paths grows
  exponentially with the size of the codebase.
- Because of this, you'll find clever ways to slice and dice
  the combinatoric explosion of possibilities into scenarios
  that cover *most* of the important codepaths. Unfortunately,
  all such slicings are arbitrary, and no one else will be
  able to understand how your tests are organized or why.
- Did I mention they take a *really long time to run*?

All of these problems are bad, but the slowness is the
real killer. It means developers will either be idle for
hours while the tests run (unlikely, because most good
programmers *actually like writing code*) or they'll start
some other task and then get interrupted hours later by the
test failure. This in turn means those developers will have
to juggle multiple versions of the codebase in their heads: the version
they're currently working on, and the version where they're debugging
the test failure. This is not, in general, possible. Understanding *one* version of
a complex codebase is hard enough for most humans. When
people try to fit multiple versions of the code in their
heads, they go slower, and they make more mistakes.

Slow tests also mean that developers work in longer cycles.
If the tests take hours to run, I'm pretty much only going
to run them when I think I'm "done". When tests run
infrequently, risk piles up. The more work I do in between
test runs, the more I'll have to throw away if the tests
reveal a conceptual flaw in my design.

All of this means that integrated testing is expensive.
It's not quite as expensive as relying on manual QA,
nor as potentially disastrous as forgoing testing entirely,
but it is expensive. And, just like QA, it does nothing to
discourage developers from writing horrific, bug-prone code
under time pressure. Over a product's lifespan, this can
add up to a lot of money lost to maintenance headaches and
customer churn as people ditch the product for one with
fewer bugs.

## Post-Development Unit Testing

Isolated unit tests fix many of the problems associated with
integrated tests:

- They run quickly. [It is not unreasonable to expect a
  suite of unit tests to complete in under a second](https://www.youtube.com/watch?v=RAxiiRPHS9k), though
  personally, I aim for 100 milliseconds so I can run my
  tests on every editor keystroke.
- They test a relatively small amount of code, so finding
  the cause of a failure is usually straightforward.
- They are deterministic and do not fail flakily.
- They can run on developers' workstations.
- 100% code coverage is actually achievable. When you don't
  have to deal with the combinatoric explosion of codepaths
  precipitated by integrated testing, you end up writing
  fewer tests, and getting more value out of each one.

However, when unit tests are written after the code is
complete, they have their own set of pitfalls. The main
one is the difficulty of isolating the units of code to
be tested. When modules are not designed
explicitly for testability, it's all too easy to let them
interact with a large number of other systems. A module that
makes network calls, gets the current time, generates random
data, and writes to a database is going to be difficult to
test, because all of those external interactions must be
controlled in some way, or the test won't be repeatable.

Fortunately (or perhaps unfortunately) for people wishing
to unit-test their code, introducing a level of indirection
allows one to ignore almost any design problem. The relevant
techniques are dependency injection and the use of test
doubles, which I won't describe in detail here. Suffice it
to say that I think these techniques have some glaring
downsides:

- They complicate code, making it harder to understand.
- They reduce the readability of tests. This is not an idle
  concern. Tests are just code, and code that is hard to
  read is more likely to have bugs. What
  happens when your tests have as many bugs as the thing
  they're supposed to be testing?
- They impede refactoring across module boundaries. One of
  the few advantages of integrated testing is
  the freedom it gives you to redesign the internals of your
  code and (eventually) have confidence that the functionality
  was relatively unscathed. Unit tests with
  test doubles lock you
  in to particular module boundaries unless you rewrite all
  the tests for those modules. When you have
  to rewrite your tests to refactor the code, how do you know
  your refactoring didn't change behavior? Answer: you don't.
- Because unit tests cannot (by their nature) check that the
units interoperate properly, a minimal number of *integration
tests* (note the distinction from *integrat-**ed** tests*)
is usually required to verify that the program as a whole
actually works. It's important to note that integration
tests *only* target the interactions between architectural
layers; they are not meant to provide comprehensive testing
of the system's behavior. However, integration tests retain
some of the downsides of integrated testing. Specifically,
they are slow and flaky, and often can't run on
developers' workstations.

While extremely common in unit-tested codebases, these
problems are not universal. They are, rather, symptomatic
of tests written after the code's design was fixed.
It is possible to avoid these problems and their associated
costs by designing the code with testability in mind.

## Test-Driven Development Done Right

TDD is not *just* the discipline of writing tests first. [In fact, it's
not really about writing tests first at all](http://blog.cleancoder.com/uncle-bob/2016/11/10/TDD-Doesnt-work.html).
It's simply a mental trick that gets you into the habit of
designing testable code.

Testable code is valuable
because testability is a proxy for reusability—in a sense,
testability *is* reusability because code
with tests always has at least two callers: the tests and
some other part of the production system. When code is
reusable, the amount of code (and therefore the number of
tests) that developers will need to write is reduced. This
in turn reduces the overall cost of development and
maintenance.

Done well, TDD solves the readability problems that come
from proliferating test doubles. It's *really
hard* to write a test with complicated test doubles before
the production code exists. So if you do disciplined TDD,
and force yourself to write the tests first,
you won't use many test doubles. You'll find [a way to design
your code so they're unnecessary](/blog/2018/06/08/principles-of-testable-design/).

If you do that, many of the problems discussed earlier go away:

- The code is simple. Dependency injection and other design
  convolutions are minimized.
- The tests are simple. Often, each test is only a few lines
  long.
- It is easier to refactor across module boundaries because
  there are no test doubles that need to be updated.
- Groups of modules that have no external dependencies can
  be tested together. This provides the benefits of
  integration tests, but with lower costs—the tests run
  quickly and reliably, and they can run on developer
  workstations. Depending on your application, you may or
  may not feel the need for integration tests.
- Because each unit is tested in isolation, you won't
  have to untangle the Gordian knot of possible paths through your code.
  You can instead slice it into little pieces and deal with
  each one separately. This means that the number of tests
  you need to write to cover all the functionality grows
  only linearly with the size of your codebase.

Sounds great, right?

Now, I should stress that you can't just tell developers
about red-green-refactor, call them TDD experts, and let
them at the codebase. If you do, they're going to make a
mess. Why? Because, as I said above, TDD isn't about test-first,
it's about design. Developers who are not used to designing
testable code aren't magically going to be able to do it
just because they're writing tests first. And adding TDD
to a legacy codebase isn't going to magically rejuvenate it;
once code has calcified into an untestable shape, you can't
easily back out those design decisions no matter what
discipline you use.

It takes experience and practice to do TDD well and reap
its benefits. Fortunately, there are many resources you
can point people to if they express interest in learning
TDD:

- [Exercism](http://exercism.io/) provides coding
  exercises in over 30 different languages—tests included.
  It's a great way to start thinking test-first.
- The [Hexagonal TDD in Ruby](http://moonmaster9000.github.io/hexagonal_tdd_in_ruby/)
  series may be interesting to Rails developers—it shows
  how you can separate the logic of a web application
  from external dependencies like the network.
- Consultants like [Pivotal Labs](https://pivotal.io/labs)
  can help incorporate TDD into your workflow.
  (Full disclosure: I work for Pivotal, but
  they have not endorsed or funded this post.)

## Coda: How Much Testing Is Enough?

When you write tests after the code is "done", you have
to ask yourself the question: "how do I know when I'm done testing?"
More testing is safer, of course, but it's also more expensive.
A lot of developers will (rightly) err on the side of safety
and spend time writing tests to cover every case they can
think of.

TDD provides an unambiguous answer to this question: you're
done when you can't think of another test that would fail.
Seeing the tests fail before making them pass proves that
each test is adding value. It means you'll never waste
time and money on unnecessary testing.

Of course, sometimes you *think* you can't write another
failing test but the code still has a bug. That means
"extra" testing may actually be a good investment if
you can't easily fix code that's deployed. But that doesn't
stop you from doing TDD—it just means that the last few
tests you write probably won't fail.

## Summary

> "[TDD] is the worst form of [testing], except for all the
> others." —Winston Churchill

TDD will save you money, for all of these reasons:

- Automated testing means you don't have to pay QA engineers
  to do stuff a robot could do. Instead, they can do
  [exploratory testing](https://en.wikipedia.org/wiki/Exploratory_testing) to discover usability issues and subtle bugs.
- Unit testing lets you write fewer tests, because the number of tests needed to cover all cases grows linearly with
the size of the codebase. This contrasts sharply with integrated testing,
where the number of tests you need grows exponentially.
- Unit testing minimizes the time developers spend idle waiting
  for tests to run.
- TDD ensures that your tests are small and simple, so
  developers will spend less time figuring out what
  each test does.
- TDD ensures that code is reusable, allowing you to write
  less code overall.
- TDD makes it less risky to change your code—whether
  to add a feature, fix a bug, or improve the design.
- TDD tells you when you don't need any more tests.

TDD is difficult to do well. But applied correctly, it can
be a powerful tool for keeping bugs at bay and ensuring
the long-term health of the codebase. And who knows? It
just might give your company the edge it needs to succeed.
