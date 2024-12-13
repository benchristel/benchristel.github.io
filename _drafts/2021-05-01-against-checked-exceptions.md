---
layout: post
title: Against Checked Exceptions
---

> An error exit can be considered well-structured for precisely this reason -- it corresponds to a situation that is impossible according to the local invariant assertions; it is easiest to formulate assertions that assume nothing will go wrong, rather than to make the invariants cover all contingencies. When we jump out to an error exit we go to another level of abstraction having
different assumptions.
>
> —Donald Knuth, [Structured Programming With go to Statements](https://dl.acm.org/doi/pdf/10.1145/356635.356640)

UPDATE: Kevlin Henney thinks checked exceptions are bad too: https://youtu.be/-yHd-GUQpJE?t=274

A few people, all with many more years of Java experience than I, have remarked to me in passing that "checked exceptions in Java were a mistake" and that you should "only use runtime exceptions (i.e. those that aren't typechecked by the Java compiler)". This view always puzzled me, because to me, it seemed that exceptions that might be thrown were, semantically, part of the "return type" of a method and that if you *can* typecheck the return value of a language, you *should*.

This view is reinforced by languages like Go, Haskell, and Elm, where there are no exceptions and functions simply return information about errors they encounter. On the surface, this seems much more civilized, because the type system can both ensure the error is handled and document its chain of custody as it's returned up the call stack. This is how functional languages like Elm can claim they have "no runtime errors"—they force the programmer to handle all exceptional cases explicitly.

So, on the surface, checked exceptions seem like a great idea—or at least, a better idea than unchecked exceptions. However, in this post I'd like to take a closer look at the "no checked exceptions" philosophy, because I now believe not only that is it pragmatic, but that it's *necessary*. To get most of the benefits of object-oriented programming, you *must* use not only exceptions, but unchecked exceptions specifically. Furthermore, there is a way to use them that is safe and results in clear, simple code.

- "Checked exceptions were a mistake"
- "Don't use exceptions for control flow"
- ""

## What are checked exceptions?

## Why should I care about this if I don't write Java?

Java is the only mainstream language that has checked exceptions. If you're writing code in a language that doesn't have checked exceptions, it may seem like this issue doesn't affect you.

But, like it or not, Java is the dominant "object-oriented" language, and it has had a profound influence on the culture of object-oriented programming generally. Patterns used in Java tend to sneak into other languages—whether or not they're actually useful in those languages.

So it is with exceptions. Java may be the only mainstream language with checked exceptions, but it is _so_ mainstream that it seems Java's promulgation of checked exceptions has affected how programmers think about exceptions in other languages. So in languages that do not have checked exceptions, you find developers making up for this "lack" with workarounds, like documentation and tests that describe what exceptions a method might throw. However, as I said above, you can't get the benefits of object orientation if you're thinking about exceptions this way. The examples that follow will make it clear why this is.

So if you're doing object-oriented programming in *any* language, this matters to you. _Chances are, all of the exception-laden code you've seen is doing it wrong_—and it's Java's fault.

Perhaps your language of choice doesn't have exceptions at all—maybe you're writing Go, or you prefer functional languages. In that case, I hope this post can serve as a demonstration of how exceptions can make a language more expressive without sacrificing safety or readability. Once you see the tradeoffs, you can make more informed decisions about whether to use an OO or functional language for your next project. And if you _do_ choose to use an OO language, you'll be better equipped for success having read this post!

## What Exceptions Are

In order for me to grok the value of unchecked exceptions, my thinking about _what exceptions are_ had to undergo a paradigm shift.

Previously, I thought of exceptions as a kind of side dish to the return type of a method. For example, if you have a [`java.io.Writer`](https://docs.oracle.com/javase/7/docs/api/java/io/Writer.html), and you call `append("Hello, world!")` on it, you _might_ get a `Writer` instance in return, _or_ you might get an `IOException`. Two possibilities, that must be handled by different codepaths, and indeed by different syntax. For example:

```java
// This is an example of how NOT to do exception handling.
try {
  writer = writer.append("Hello, world!");
} catch (IOException e) {
  System.out.println("Oops, an error occurred");
}
```

But here's the thing:

**An exception is *not* a return value.**

**An exception is a jump into an error-handling routine.**

The difference? An error return hands control back to our
caller, saying "you figure out what to do with this". A
jump to an error handling routine deals with the problem of
not being able to return anything sensible by simply not
returning. Our caller never *gets* control back. Instead
of "failing back" to the caller, we "fail forward" to the
place where the error is handled.

Now, alarm bells may be going off in your head at this point:
this sounds dangerous!
And in the wrong environment, it *is* dangerous. If your
caller is in the middle of manipulating some fiddly global
state—say, in between updating two files that need to be
kept in sync—you don't want to suddenly jump somewhere else
in the program, because that will corrupt your data.

However, there is a simple rule you can follow to stay
safe. Code through which an exception might be
thrown *should not* update global state—or, if it
absolutely needs to, it must do so transactionally.
You probably want to adhere to this rule in most of your
code anyway, simply to make your system comprehensible,
so hopefully following this rule isn't a burden.

## The Core and the Shell

To understand a bit more deeply how this works, let's look
at a schematic of some (pseudo-javascript) code that uses
unchecked exceptions.

```

User Input

  |  ^
  v  |

try {
  doSomething(magic)
} catch {                   <-- exceptions are caught here
  // ...
}

  |
  v

function doSomething(magic) {
  // ...                       <-- this code knows nothing
  return magic(foo, bar)           about exceptions!
}

  |
  v

function magic(foo, bar) {
  // ...
  throw anError             <-- exceptions are thrown here
}
```

The first and third code chunks know all about exceptions
and errors. They are the parts that deal with the messy
outside world. The first code chunk takes user input,
validates it, figures out which objects to instantiate and
which procedures to invoke. The `magic` function might
talk to another process or something. If that process isn't
running, of course, it won't be able to do its job and will
throw an exception.

The second code chunk, the `doSomething` function, knows
nothing about exceptions.

<!--
Now, you may have gotten from Dijkstra that GOTOs are a very bad thing and we shouldn't use them. But Dijkstra made it clear in ["Go To Statement Considered Harmful"](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf) that his concern was with _deterministic, in-memory processes_. As soon as we introduce IO into the middle of a process, all of his careful reasoning about "independent coordinates in which to describe the progress of the process" flies out the window, because the process becomes nondeterministic and therefore its state cannot be represented as coordinates in the way that he suggests. However, for reasons that will become clear in a bit, when we have unchecked exceptions we can write nondeterministic, IO-involved programs and _pretend_ that the resulting processes are deterministic—and even test them in a deterministic way—without losing our ultimate grip on what the process is really doing.

Indeed, Dijkstra mentions "abortion clauses" (by which I think
he essentially means "catch statements") as a possible
extension to his vision of structured programming. So I think
this discussion fits within the spirit of "Go To Statement Considered Harmful"
-->

## A Code Example

This has all been pretty abstract so far. Let's switch gears
and look at some code.

Suppose we're writing a banking app and our users want to be able to view their entire transaction
history, as well as their current balance, on a webpage so they can print it out.
Here's the method to implement that feature (the class surrounding it has been omitted for brevity), which
uses checked exceptions to handle IO errors.

```java
void handle(UserId currentUser, Writer response) throws IOException {
  List<Transaction> txns = new TransactionRepository(currentUser).all();
  Dollars balance = new Balance(txns).value();
  new FullHistoryPage(txns, balance).renderHtml(response);
}
```

The `Balance` class sums the net amounts of the transactions passed to it:

```java
public class Balance extends Dollars {
  private Dollars amount;

  public Balance(List<Transaction> txns) {
    this.amount = new Dollars()
    for (Transaction t : txns) {
      this.amount = this.amount.add(t.netCredit());
    }
  }

  public long cents() {
    return this.amount.cents();
  }

  // ...
}
```

And the `FullHistoryPage` class renders the response HTML:

```
public class FullHistoryPage {
  // ...

  public void renderHtml(Writer dest) throws IOException {
    dest.write("<h1>Account History</h1>");
    dest.write("<h2>Balance</h2>");
    dest.write("<p>" + this.balance.toString() + "</p>");
    dest.write("<h2>Transactions</h2>");
    dest.write("<ul>");
    for (Transaction t : this.transactions) {
      dest.write("<li>" + t.toString() + "</li>");
    }
    dest.write("</ul>");
  }
}
```

This code has several nice properties. It delegates its behavior (loading data, computing the balance, and rendering the HTML) to three other classes that each have simple, clear responsibilites. Furthermore, it's clear how this method will deal with IO errors that occur when fetching the data or writing the response (it will throw an IOException).

Imagine we deploy this code to production and it works great! At least, it does for a while. Eventually, one of our users writes in to say the page isn't loading for them. Searching the logs, we find that our server's running out of memory.

The issue is this line:

```java
List<Transaction> txns = new TransactionRepository(currentUser).all();
```

We're loading all the user's transactions into memory at once. And some of our users have too many transactions to fit within the memory limit of our economy-sized JVM.

We could "solve" this problem by simply bumping up the memory limit and renting bigger servers. But that would increase our hosting costs substantially—all to fix an issue that really isn't very common, and will eventually reoccur no matter how high we set our memory limit.

Fortunately, someone on the team is versed in object-oriented programming and comes up with a clever solution. Rather than have the `all` method of the repository load all the transactions into memory at once and returning them in a `List`, it can return an `Iterable` that iterates over them lazily, a "page" at a time. Each page of transactions is loaded from the database as it's needed. Once we're done processing a page, it becomes unreferenced and can be garbage-collected, so we never have more than a page's worth of transactions in memory at once. The Balance class and the FullHistoryPage can easily be updated to take an Iterable rather than a List with no internal changes to their code, since they were just iterating over the list anyway.

This all would work great, except that there is still the possibility of an `IOException` to contend with. `IOException`s are checked exceptions, and the `Iterator` interface does not allow checked exceptions to be thrown! That means can't write a lazy iterator that implements `Iterator`.

## Checked Exceptions Limit Polymorphism

The difficulty we encountered in the previous section occurs because Java requires each caller of an IO-involved method to either handle `IOException`s or shepherd them up the call stack. That means that nothing in that call stack can implement an interface that isn't declared to throw that specific type of exception.

Specifically, our desire to perform IO inside an iterator means we can't use the [ubiquitous type](https://benchristel.github.io/blog/2019/08/10/new-thoughts-on-test-doubles/) defined by the Java standard library to represent iteration. We have to either define our own iterator type that throws an IOException, or catch the IOException inside our iterator implementation and throw an unchecked exception instead. As you might guess, I recommend the second option.

To some, the route of using unchecked exceptions feels like we're cheating. Like unsafe casting or reflection, unchecked exceptions seem to jump the guardrails the type system places around us. However, this feeling arises only if we're considering exceptions to be part of the return type of the function, and, as I stated earlier, it's actually more productive to think of them as something else. Not a return, but the avoidance of a return; not an answer, but a rejection of the assumption that an answer exists.

## Wishful thinking, and a doomsday device

It would be _really nice_, from the programmer's perspective, if all IO were completely reliable. However, in a networked or even multi-process environment, reliable IO is simply not possible. In the real world, machines go down, packets get lost, disks fail, and files get deleted out from under us.

So forget about the real world. What we'd like to do is create a sort of fantasy world in the core of our program, shielded from the messy realities of IO. If we could create such a virtual world, we could use it as a kind of higher-level control flow construct. That is, we could set a computational process running within the fantasy world, and it would appear to the process that all IO is completely reliable. The vast majority of the time, the IO _will_ succeed and the process will finish without mishap, producing a return value. Sometimes, of course, an IO operation will fail, in which case the illusion of reliable IO cannot be maintained. But rather than tell the process about the error, we simply pop the bubble of the fantasy world and the whole thing ceases to exist. We write its epitaph to our error logs, and the user gets a `500 Internal Server Error`.

One of the great legends of modern times (I'm not sure who came up with the idea first) is that of a computer that leverages quantum multiverse theory to solve any NP-hard problem in polynomial time. The way it works is this: the computer uses quantum randomness to generate a guess at a solution. If the guess happens to be correct (which can be verified in polynomial time), it's returned to the user. If it's wrong, then the computer simply destroys the universe. The only remaining universes are those in which the problem is solved.

Such a system is, of course, both risky and impractical.
There is the rather basic question of how one might go about destroying the whole universe, and also the risk that if one asks a question that has no answer, the entire multiverse might be destroyed. Fortunately, when programming in Java, the existential risks of this approach are limited to a few stack frames.

And this is exactly how unchecked exceptions work. We make a call to a method that, somewhere under the hood, might do IO, and we assume it will reliably return a value. And it does! Or at least, it does in all universes where our stack frame is around to see what happens. If the method can't return a value for whatever reason, our stack frame is obliterated, and control passes to some other code that we know nothing about. _In no case is an error ever returned_.

It pays to underscore this point: *An exception is not a return value. It is a jump into an error-handling routine*.

## Taking Responsibility

There is another, somewhat orthogonal, view of why unchecked
exceptions are good, that goes like this: Suppose we have
some co

We have not yet discussed the details of the error-handling routines that exceptions jump to. Who defines them? What kinds of stuff do they do? How do we verify that they're doing the right thing to recover or clean up after the error?

You may have guessed that when I say "error-handling routine," I am, for the purposes of Java, talking about a `catch` block. This implies that the error is going to be handled somewhere up the stack from where it occurs. (In other languages, error handlers might be implemented as continuations—but I'm trying not to get distracted here. Let's focus on Java).

Where: at the boundaries of the fantasy world
What: ideally, nothing except reporting the error. The process within the fantasy world should, ideally, just return a value and not do any non-nilpotent IO (i.e. writes). If this isn't possible, we should take care to avoid partial writes (e.g. by using a temp file and only writing to the

## The Limits of Fantasy

While the _unreliability_ of IO operations can be abstracted away in our fantasy world, other characteristics of IO, such as its slowness and the fact that it sometimes mutates global state, cannot be. This means that the abstraction often has to be allowed to leak a little bit; the code that runs in the fantasy world has to be aware of which operations might be slow. In general, any method on an injected dependency might be slow.

## When to return an explicit error

parsers

more generally, any in-memory operations where failure is predictable/reproducible.

## Tradeoffs between OO and FP

FP lets you precisely represent the possible states
of a program.

But this really only works well when all the data can
be processed in memory, or when the logic for chaining
together side-effects can be expressed in a straightforward
way using monads.

When there are complex interdependencies between functional
and side-effecting code, functional code becomes very convoluted.

The usual workaround is to load all conceivably necessary data upfront, handle any IO errors at that point, and process the data in memory. Depending on the application, this may have substantial, even prohibitive, performance costs.
