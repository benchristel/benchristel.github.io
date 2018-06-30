---
layout: post
title: Principles of Testable Design
---

# Principles of Testable Design

I have to confess I don't like the term "TDD". First of
all because it's ambiguous: to most people it stands for
"test-driven development," but some insist that "test-driven
design" is a more fitting expansion. Second, I don't think
the word "driven" is well chosen. Tests do not *drive*
code. They do not *drive* the design. *Programmers do*.
Driving means making decisions, observing, reacting.
Tests are part of that loop, but they aren't the agent.
Saying that code or design is "test-driven" is like saying
that your car is "steering wheel-driven". It's bizarre,
bordering on nonsensical.

There is
[this persistent myth](./2017-01-15-don-t-listen-to-your-tests.md)
that well-designed code somehow emerges from a suite
of sufficiently magical unit tests, with no human creativity
required. This idea is—[to quote
veteran TDDer Bob Martin](http://blog.cleancoder.com/uncle-bob/2017/03/03/TDD-Harms-Architecture.html)—"horse shit." Yet I think the phrase
"test-driven" has contributed to this misunderstanding and
probably caused a lot of people to reject TDD as a practice
because it failed to live up to the myth.

*&lt;/rant&gt;*

## Programmer-Driven Design

So, here's how TDD *actually* fits into the design process.
TDD is a
[behavior-shaping constraint](https://en.wikipedia.org/wiki/Behavior-shaping_constraint)
that makes tightly-coupled code
more difficult to write. As a result, you write
loosely-coupled code (if you understand design), which is more
general, reusable, and comprehensible. Or, if you don't
understand design, you continue writing tightly-coupled code
with complicated, brittle tests, and you end up tearing your
hair out and then writing blog posts about how [TDD is
dead and Capybara is the future](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html).

To shield you from that second fate, I present the
heuristics I have used over the last few years to design
testable code.

## 1. Separate Data Transformation from Side Effects

I learned this one from [Gary Bernhardt's video "Boundaries"](https://www.destroyallsoftware.com/talks/boundaries).
Once you learn this pattern, you'll start to notice its
absence *everywhere*. It's so easy to design code this way,
yet humans seem to instinctively do the opposite and combine
data transformation and side effects at every opportunity.
The mistake is so pervasive that I refer to it in my head as
*The Mistake*.

Here is what The Mistake looks like:

```java
class ShipmentTracking {
  /* Do not write code that looks like this. It's bad. */
  private EmailService service;
  public ShipmentTracking(EmailService service) {
    this.service = service;
  }

  public void sendEmail(User user, Shipment shipment, Order order) {
    Email email = new Email();
    email.setHeaders(/* some complicated logic */);
    email.setSubject(/* more complicated logic */);
    email.setBody(/* logic logic logic */);
    service.send(email);
  }
}
```

This probably looks a lot like code you've seen before.
We're in the guts of an e-commerce application, and we need
to notify a user that their shipment is on the way and give
them their package tracking info. To that end, we have
a class that will send the email given the user,
shipment, and order. Sending the email is a **side effect**,
and we don't want it to happen when we're
testing. The logic for creating the email object based on
the user, shipment, and order is **data transformation**.

To test this, we need to construct a `ShipmentTracking`
instance, passing a mock `EmailService` to the constructor.
Then we call `sendEmail(...)`, and assert that our mock
`EmailService` received the `send` call with the right data.

```java
EmailService service = mock(EmailService.class);
doNothing().when(service.send(any(Email.class)));
ShipmentTracking subject = new ShipmentTracking(service);

subject.sendEmail(user, shipment, order);

ArgumentCaptor<Email> emailArg = ArgumentCaptor.forClass(Email.class);
verify(service).send(emailArg.capture());
Email sent = emailArg.getValue();
// and now, at long last, we can make assertions about the
// email that was sent.
assertEqual(sent.getBody(), "...")
```

The problem with this is that verifying calls on a mock is
tedious and error-prone compared to just checking a return
value from a function. For example, if you use Mockito to
create your mocks, you'll probably need an `ArgumentCaptor`
to make assertions about the `email` passed to `service.send()`.
Mock objects require both "training" (setup that tells them
how to respond to method calls) and more convoluted assertions.
This adds noise to the test and makes it harder to follow.
I mean, just look at those extra six lines of rigamarole we have to go
through before we can assert anything about the email that
was sent!

To be sure, this test is not bad *just* because it's verbose.
Sometimes verbosity is required. But here the verbosity
is not necessary; it does nothing but obscure the
interesting behavior we
want to test. The core logic of the `sendEmail` method—the creation
of the `Email` instance—is the interesting part of this code,
the part that really deserves to be unit-tested. Yet in order to test
it, we have to slog through this haze of mock objects and
`ArgumentCaptor`s.

Let me ask you this: how *interesting* is the fact that we call
`send()` on the email service? *How many times* do we need
to test that line to be sure that it's correct? If we use
the mocking approach, that line will be invoked in *every
single test* for `sendEmail`. There may be dozens of such
tests that cover different combinations of `user`,
`shipment`, and `order` values. Do we really need to assert,
in every one of those tests, that `send()` was called, with
all of the boilerplate that entails? Using
mocks *forces* us to make that assertion.

Here's a better test:

```java
assertEqual(
    new ShipmentTrackingEmail(user, shipment, order).getBody(),
    "...");
```

Whoa! Where did all the code go?

It may be clear what we've done here, but in case it's not:
the "interesting logic" of constructing the email's subject
line, body, etc. is now encapsulated in the email class
itself. Sending the email happens elsewhere. We
don't need to care about it for this test.

## 2. Pass Output Receivers as Arguments

Which of these is better?

```java
exception.printStackTrace();
exception.printStackTrace(System.error);
```

I like the second one, personally. It's more explicit: you
don't have to look up the docs for `printStackTrace()` to
find out that it prints to `STDERR` and not `STDOUT`. The
code *says* where it's printing to.

I also like it because it's testable. You can pass in a
mock PrintStream in your tests and assert that the error
gets logged. This also has the side benefit of keeping
the stacktraces from making a mess of your test output.

When you have some object that's producing a stream of
output or some other sequence of messages, *pass the
receiver of those messages as an argument* either to the
object's constructor or to the method that produces the
messages. This applies not only to
file and network IO, but in-process constructs like
the Observer pattern and callbacks for asynchronous
operations.

This pattern is called **Dependency Inversion**—not to be
confused with dependency *injection*, which is a slightly
[different beast](https://lostechies.com/derickbailey/2011/09/22/dependency-injection-is-not-the-same-as-the-dependency-inversion-principle/) that I'll cover later.

You don't always have to use dependency inversion.
If the object is only going to produce one message, or a
small number of messages, it can often be cleaner to just
return a value:

```java
String stack = exception.getStackTrace();
System.error.println(stack);
```

## 3. Depend on Roles, Not Specific Objects

I once told a coworker that I thought dependency injection
was a code smell. He was speechless for a few seconds, a
look of mingled pity and horror on his face. "What are you
talking about?" he finally sputtered. "It's the D of SOLID!
It's a foundational principle of object-oriented design!"
"No," I replied. "The D is dependency *inversion*."

A bit of context for this anecdote: this coworker and I were
writing a web server in Spring Boot. Spring Boot *loves*
dependency injection, and goes out of its way to make it
easy. You annotate classes with `@Service` to make them
available to the dependency injector. Then, when you want
to use one of those classes somewhere else, you add it as
a field to the class that's using it and annotate that field
as `@Autowired`. At runtime, the dependency injector creates
an instance of every `@Service` and sets all the `@Autowired`
fields to point to the appropriate service instances. It's
delightfully magical.

But it's also *not dependency inversion*, and it gives you
*none of the benefits* of dependency inversion. Dependency
inversion says you should depend on abstractions rather
than coupling your code to specific other classes. Spring Boot's
dependency injection is literally the opposite of that: each
class specifies *exactly* which `@Service`s it wants; no
other `@Service` will do. In my experience this produces
architectures where you have services calling services calling
services; each layer doing something *slightly* lower-level
than the one above it—but no real decoupling.

It can also lead to some horrific-looking tests. If you're
going to test a Spring Boot class that has ten different
`@Service`s injected into it, you have to mock each one.
*And* you have to assert that the right methods were called
on each of those mocks. The result is tests that break
whenever one of the things you're mocking changes its
interface, and that rarely inspire confidence that the code
actually works. Since all the dependencies are mocked,
the tests are only as correct as your assumptions about
how those dependencies will behave when everything is wired
together in production. And since the tests
don't have any way to express those assumptions and verify
that they're correct, it's easy to get them wrong and only
find your mistake when you run the integration tests, or
worse, deploy to prod.

The fix is to keep the interfaces you depend on small, simple, and abstract (the [Interface Segregation principle](https://en.wikipedia.org/wiki/Interface_segregation_principle)).
For example, if you're writing a class that adds up some line items and generates an
invoice, don't inject a `DatabaseService` to get the line items; accept an `ILineItemCollection` as a parameter. One production implementation of `ILineItemCollection` could lazily fetch values from the database as you iterate over it. The one you use in tests might be a simple wrapper around `List`. To be flexible, an interface should be very specific about what it does but vague about how it does it.

It may help to design interfaces with at least one (ideally two) other
implementations in mind. Ask yourself questions like "what if we were getting data from STDIN or the network instead of the database?"

## 3. Embrace Destruction

The nice thing about objects is that you can throw them away
and make a new one when you want to wipe the slate clean.
This is especially convenient in tests. Each test should
create a new instance of the class it's testing, to ensure
that it's starting from a known state and can't be "polluted"
by changes made in other tests. Test pollution can cause
hard-to-debug failures that only appear when all the tests
run together.

Unfortunately, there are some OO patterns (and antipatterns)
that make test pollution harder to avoid.

- Static (i.e. global) variables
- Singletons

## 4. Assert on Values, Not Method Calls

A while ago, a coworker and I were writing tests for an HTTP
server implemented using Jetty. In Jetty, a simple request
handler looks approximately like this:

```java
public void handle(String url, HttpServletResponse response) {
  response.setContentType("text/html;charset=utf-8");
  response.setStatus(200);
  response.getWriter().println("<h1>Hello World</h1>");
}
```

Note the last line, where we output the HTML by sending it
to the `response`'s `PrintWriter`.

The code we were trying to test was much more complicated,
of course. Specifically, there was not just one call to
`response.getWriter().println()`, but many. There were
even some calls to `print()` and `printf()`.

Our first approach was to mock the `PrintWriter` and assert
that it got the right method calls to generate the response
body, but that didn't feel right. We didn't want to specify
the exact sequence of method calls; all we cared about was
that the resulting HTML was correct.

What I wished we'd done was to use a real `PrintWriter`
wrapping a `StringWriter`. If we'd done that, the
arguments to the various `print*` calls would have been
concatenated into a string which we could easily compare
to the expected output.
