---
layout: post
title: New Thoughts on Test Doubles
---

# New Thoughts on Test Doubles

[Richard Gabriel wrote this about generative advice](http://dreamsongs.net/Repetition.html):

> When I play squash and mis-hit or misdirect the ball, I
> chant, "follow through, follow through, you idiot." A
> mantra of simple things. [...] The law of follow-through
> is hard to forget - it’s a law, though, that makes little
> sense: where is its enchantment? How can advice about what
> to do after you hit a ball make any difference to what
> happens when you hit it, or before?
>
> When you follow through, your arm is aiming at a point
> well beyond where the racket will contact the ball, so
> there is no deceleration. "Don’t decelerate, don’t
> decelerate, you idiot!" It might make a mantra worth
> repeating, but it isn’t advice you can take.
>
> [The follow-through] mantra generates the effect we want -
> just as a seed generates the flower which eventually
> blossoms. Just as the wind in the sand generates dune
> designs and sidewinding sinews and ripples of grains of
> sand.

For the last, oh, year or so, I've been telling people that
my TDD strategy doesn't involve test doubles, and that
they'll write better tests if they try to eschew doubles
entirely. I still believe that's good generative advice, and
I still use it as a heuristic when testing, but it's based
on an informal, relatively unexamined view of the "physics"
of TDD, so it falls apart under close analysis. "Avoid test
doubles" is TDD's equivalent of the follow-through mantra.

Now I want to pull back the veil of generative advice and
discuss the physics of what's actually going on when I test
"without doubles". The scare quotes are there because,
really, I still use test doubles—just not the kind you may
be used to seeing.

## Definitions

Before we can begin to discuss the role of doubles in tests,
we need to agree on what a test double even *is*.

In this article, I'm going to define the term _test double_
very broadly, to include many entities that you probably
wouldn't normally think of as test doubles. Under this
definition, **a test double is any value controlled by the
test code that interacts with the test subject** (i.e. the
thing we're testing). The doubles used by a test are the
parameters the test holds constant so it can reliably
observe changes in the variable of interest—the behavior of
the subject. Thus, doubles are completely known quantities
to the test code. In fact, an understanding of the
properties of the doubles involved is crucial to
understanding what the test is even testing.

Based on this broad definition, almost any argument we pass
to a test subject is a double. Suppose I use the string
"hello, world" as data when testing a formatting module.
From an OO perspective, which considers the string as an
object with a set of behaviors, the string literal "hello,
world" is just a convenient way of creating a [*stub*](https://martinfowler.com/articles/mocksArentStubs.html) with
self-consistent behavior. This is because the observable
properties of the object—that it responds to `.concat("foo")`
with "hello, worldfoo" and to `.length()` with `12`—could just
as well be implemented by a stub, but it's convenient for
the person writing the test to just use a string literal
instead.

You might react to this idea, that a string can be a test
double, in one of two ways:

1. Oh, weird! I guess a string can be a test double!
2. That's crazy. Strings clearly aren't test doubles.

If you said (1)—great! You should keep reading the rest of
this article. If you said (2)—then I take back what I said
in the introduction, I actually don't use test doubles at
all, and you can take my "generative advice" literally.

## What's *Not* a Test Double?

At this point, you may be wondering "if a string can be a
test double, then what *isn't* a test double?" Really, what
I'd like you to take away from the previous section is not
"strings are test doubles" but that *a test double is a
role* that any type of object can play in a test.

To make this idea more concrete, here are some things that
are *not* test doubles, because they are not directly under
the control of the test code:

- the system clock
- the filesystem
- the test subject

And here is an example of a test that truly uses no test
doubles at all. The test subject, `foo`, is a procedure that
just sets a global variable.

```javascript
describe('foo', () => {
  it('sets a global variable `bar`', () => {
    foo()
    expect(bar).toEqual(3)
  })
})
```

This test controls essentially nothing in the environment;
it is at the mercy of `foo()` and its side effects.

Contrast that test with this one:

```javascript
describe('foo', () => {
  it('sets a global variable `bar`', () => {
    foo('blah')
    expect(bar).toEqual('blah')
  })
})
```

Here, we're telling `foo` what value to assign to the `bar`
global, and that value, the string `'blah'`, is a kind of
test double—a *dummy*.

So, do tests for code that takes parameters inevitably have
to use test doubles? No, they can use production data
instead!

```javascript
describe('foo', () => {
  it('sets a global variable `bar`', () => {
    foo(APP_CONFIG)
    expect(bar).toEqual('blah')
  })
})
```

This test uses a global constant `APP_CONFIG`, which is a
real production value that gets passed into `foo` when the
production system runs. There are no test doubles here,
because the test doesn't control what data `APP_CONFIG`
contains.

## You Can't Test (Well) Without Doubles

From these examples, it should be clear that test doubles
are not just useful; they're indispensable. You simply
cannot isolate your test subjects from external confounding
factors without using test doubles as I've defined them.

## Test Doubles I Have Loved

I enthusiastically use test doubles to the degree that the
following statements are true of the situation:

- the double implements an interface that is ubiquitous in
  the programming language. Often it's a built-in type like
  an iterable, string, stream, function, or channel.
- the double *is actually an instance* of a built-in type
  defined by the language itself.
- the double naturally has properties that are important to
  the contract between the system under test and the
  collaborator the double replaces. Examples of such
  properties include idempotency, nilpotency, commutativity,
  and associativity.
- the double is structured in a way that allows me to make
  assertions in my test using the full vocabulary of
  available matchers. For instance, it's more convenient and
  readable to assert that an array contains elements in a
  particular order than to assert that a mock received
  several method calls in a particular order.

## Examples

At this point, you probably want to see code. I'm happy to
oblige.

### 1. Observing state changes via a callback function (JavaScript)

```js
let changeEvent = 'never sent'

let appState = AppState(function(event) {
  changeEvent = event
})

appState.update({foo: 'bar'})

expect(changeEvent).toEqual({
  type: 'update',
  data: {foo: 'bar'}
})
```

Here, `AppState` and its callback function implement the
Observer Pattern: the contract between them is that the
function will be called whenever `AppState` is updated.
In the test, the function passed to AppState is a
double—specifically, a spy.

The object `{foo: bar}` is a dummy: its purpose is to pass
through `update`'s digestive tract unchanged and show up in
the `data` field of the event, where we can detect it. Its
apparent unrealism is actually a boon: it's clear to anyone
reading this test that `{foo: bar}` is not meaningful to the
application, so there's no illusion that `update` might have
special logic related to this value.

### 2. Reading data via a stream (Golang)

```go
input := strings.NewReader("{}")
dom, err := json.Parse(input)
Expect(err).NotTo(HaveOccurred())
Expect(dom.Type()).To(Equal(json.Object))
```

Here, we're testing a json.Parse function designed to read
input from a File by feeding it a string reader instead.
We could have designed Parse to simply take a string instead
of a Reader, but that would mean we could only parse JSON
files by reading the whole file into memory. That would be
fine for small files but might not be acceptable for very
large ones.

### 3. Processing data with a function (JavaScript)

In this example, we're testing a [curried](https://en.wikipedia.org/wiki/Currying) `map` function.
The concept of `map` comes from functional programming.
`map` takes a function that operates on a single value and
"upgrades" it to a function that performs that operation
on every element of a list, returning a new list of the
return values.

```javascript
let add1 = function(x) { return x + 1 }
let add1toEach = map(add1)
expect(add1toEach([1, 2, 3])).toEqual([2, 3, 4])
```

Here, the `add1` function is a test double. Our production
code probably wouldn't use such a trivial function with
`map`—there's rarely a need to increment every number in an
array. However, for the purposes of illustrating the
behavior of `map` in a test, a simple function like `add1`
works perfectly.

### 4. Collecting output in a string (Java)

Here, we're testing that a web server generates
the correct HTML to display a dashboard page.

```java
Writer htmlOutput = new StringWriter();
View view = new DashboardView(data);
view.renderTo(htmlOutput);
assertThat(htmlOutput.toString(), equalTo("..."));
```

The StringWriter is a test double which stands in for the
HttpResponseWriter that would be used in production to send
the response HTML to the client. The Writer interface implemented
by StringWriter specifies a large set of methods for appending
characters, strings, and lines to a buffer.

In this situation, we don't care about the exact sequence
of method calls received by the Writer—we just care that
the resulting HTML is correct. For example, the following
two snippets of production code should be equivalent from
the perspective of our tests, in the sense that we should
be free to refactor from one to the other without causing
any test failures:

```java
writer.write("<h1>");
writer.write("Dashboard");
writer.write("</h1>");
```

versus:

```java
writer.write("<h1>Dashboard</h1>");
```

Because we don't care about the specific sequence of calls
received, using a mock or a spy to test the View is
inappropriate.

Going a bit deeper: we can say that the calls to a
`writer` are
[*associative*](https://en.wikipedia.org/wiki/Associative_property).
That is:

```
write("ab"); write("c");
```

produces the same effect as

```
write("a"); write("bc");
```

We can use a string to collect the value on which we assert
precisely because string concatenation is associative,
just like the calls to `write`.

## These Tests Are Trivial; I Want Real Examples!

No they're not, your code is too complicated. These tests
are totally realistic examples of what the tests for your
actual production systems can and should look like if you
appropriately decouple your units of code and depend on
small, simple interfaces.

What "decoupled", "small", and "simple" mean is probably
a topic for a whole different post, but I'll sketch out
a few general points of design advice here:

- **Depend on few things.** If you can change your design so
  the interesting logic—the part you *really want to
  test*—depends on fewer objects and methods, you should do
  it. Your tests will get much simpler as a result.
- **Parameters are dependencies too.** They're more
  loosely-coupled than dependencies on global variables or
  classes, but *they are still dependencies*. Analyze your
  code's parameters carefully and try to prune and simplify
  them.
- **Depend on ubiquitous things.** Your code will be much
  more flexible—not to mention easier to test—if the
  interfaces it depends on have many, diverse implementors.

  For example, imagine we're writing some code to parse
  data from a file. Which of these (pseudo-Java) interfaces
  for getting the data would you rather implement a test
  double for?

  - A unique, snowflake-y class with a `getData` method
    that's not implemented by any other type in the system?

    ```java
    class MyData {
      public String getData() { /* ... */ }
    }
    ```

  - An application-specific `DataContainer` interface that
    has a few different implementors?

    ```java
    interface DataContainer {
      String getData();
    }
    ```

  - A `Readable` interface (defined by the standard library,
    and implemented by many different, widespread types)?

    ```java
    interface Reader {
      int read(CharBuffer cb);
    }
    ```

  - A string?

    ```java
    class String {
      // ...
    }
    ```

  - Something with a `toString` method?

    ```java
    interface CharSequence {
      String toString();
    }
    ```

  In case you missed it: these are ordered from least
  flexible to most flexible; tightly coupled to loosely
  coupled. The ranking is pragmatic rather than formal:
  perhaps `read(CharBuffer)` can be considered
  a more general interface than `toString()`, but it's far
  less common in a typical Java program.

## Conclusion

By now you can probably see how I arrived at the mantra
"don't use test doubles" based on an informal understanding
of these concepts. I was thinking only of the kind of test
doubles that are generated by mocking libraries. These
libraries are a symptom, not the cause, of our testing woes:
if you need a library to generate your doubles for you, your
test subject's collaborators are almost certainly of the
inflexible, tightly-coupled variety pictured earlier on the
list above.

Perhaps I should revise my advice from "don't use test
doubles" to "don't use mocking libraries". But I don't think
that would be adequate, because I received exactly that
advice from a [blog post by Bob
Martin](http://blog.cleancoder.com/uncle-bob/2014/05/10/WhenToMock.html)
years ago, but didn't understand it until recently.

These topics are complex, but I'll try to condense this
post into a few takeaways:

- A test double is any value controlled by the test.
- Thus defined, test doubles are indispensable for test
  isolation.
- The moment you create a test double, you get feedback
  about how flexible the design of your code is. Listen
  to that feedback.
- You know your code is flexible when your test doubles are
  types that are built into the language, or are part of the
  standard library: strings, streams, lists, functions, and
  channels.
