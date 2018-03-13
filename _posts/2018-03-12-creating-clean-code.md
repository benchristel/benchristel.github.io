---
layout: post
title: Creating Clean Code
---

# Creating Clean Code

[Ward Cunningham](c2.com), creator of the Wiki, once said,

> You know you are working on clean code when each routine you read turns out to be pretty much what you expected.

Joel Spolsky (of [joelonsoftware.com](https://www.joelonsoftware.com)) [observed](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/) the following "cardinal, fundamental law of programming":

> It's harder to read code than to write it.

And these two observations are related! They're both about
reading code, obviously, but there's something deeper going on here.
I want to take a moment to unpack Joel's statement.

> It's harder to read code than to write it.

If that's actually, literally true, then we should *never*
read code. If we find ourselves not understanding a piece of
code, we should just throw it away
and rewrite it rather than trying to figure it out. Clearly
that's absurd... or is it?

## Reading by Writing

Reading anything, code or prose, involves reconstructing the
author's intentions. I don't feel like I truly understand a
piece of code until I know not only what it does, but *why*,
and to know *why* I have to understand not only the immediate
effects of the bytes under my nose, but the original programmer's overall
strategy. In order to *really* understand code, I have to reconstruct
that programmer's entire thought process (or glean it from the comments, tests, and names). The thought process of understanding the code
is, in a sense, a replaying of the thought process used to write it.

Joel's law turns out to be true. Passively "reading" code is
just too hard, so instead we *rewrite code in our heads* to
understand what it's doing.

The normal process of understanding a piece of code goes
something like this:

1. Understand the problem it's trying to solve.
2. Imagine plausible solutions—in other words, figure out
  how *you* might solve the problem.
3. Based on a cursory reading of the code, come up with a
  hypothesis about which solution was chosen.
4. Read the code more closely to find flaws in your hypothesis—i.e.
  places where it differs from the actual code.

With this process in mind, we can return to Ward's statement.

> You know you are working on clean code when each routine you read turns out to be pretty much what you expected.

"Clean code" means code that's easy to understand. "Easy to understand"
is of course dependent on the background knowledge possessed by the person
reading the code. In order to write clean code, you have to anticipate
the thought process of the person reading it. You have to know
which hypothetical solutions they will consider and which they
will reject out of hand. You have to know which possible solutions
they will consider most likely. You should anticipate the flaws
in their hypothesis—edge cases they might not consider at a first
pass, for instance—and be sure to document and test those liberally.

All of this is not too hard when the solution space is small.
If there are only a couple ways to solve a problem, it's relatively
easy to communicate to another person which of the solutions you
chose.

Unfortunately, this is *programming* we're talking about. There's
*never* just a couple ways to solve a problem. The space of
possible solutions to any nontrivial problem is *vast*.
When the solution space grows, the probability of two people
finding the same solution shrinks. The probability that some future maintainer of our code will open up our file and see
"pretty much what they expected" becomes minuscule if we place
no constraints on what solutions that person *can expect*.

And if "seeing what you expect" is the essence of code cleanliness,
that means that "clean code" actually has very little to do with
code _per se_. "Clean code" isn't about the code, it's about
*the experience someone has when reading it*.
Your functions can be short and well-documented,
your names clear and consistent, your formatting calligraphic—it
does not matter. If the person reading your code doesn't see
the concepts they expect when they first open the file, your
code will be confusing to them. It won't appear clean.

In order to achieve clean code, there needs to be a contract
between the writer and the reader of that code. That contract
takes the form of a subset of the *possible solutions* that
are considered *likely solutions*. If the writer and reader
of the code can agree on what *types of solutions* are likely
to appear in the code, then the probability that they will
hit on the same solution—i.e. the probability that the reader
will experience "clean code"—rises.

And that's why...

***Everyone developing JavaScript for the browser should use
Redux***

Okay, I didn't *really* mean for this post to be a plug for
Redux. But Redux largely solves the problem of how to
narrow the range of likely solutions—at least for the domain
of frontend web code. It does this by encouraging programmers
to shoehorn their solutions into a particular architecture,
wherein all the state of the application lives in a single
object, and all updates to that object are performed by "reducers".
Reducers listen for events from the UI and update the part
of the state that they own in response. Then your view code
can subscribe to those state changes and update the DOM in
response. It's a wonderfully simple framework that unambiguously
tells you where to put any given piece of code. There's no
hand-wringing over "should this go in my model or my controller, I dunno, bloo blee bloo". If it updates state, it's a reducer.
If it has other side effects, it's an action
creator. Otherwise, it's probably part of the view.

By following these simple rules, two developers from
completely different backgrounds can work on a Redux
app and have a common language for talking about the
structure of their code.

A common complaint leveled at Redux is that it forces you
to express verbosely what would
minimally require only a few lines of JavaScript. And that's
true! It's a seemingly unavoidable side-effect of forcing
all problems to be solved by the repeated application of a
single pattern: sometimes, you're going to see a really
easy way to do something, and Redux is going to be a butt and
not let you do that. However, it's all in service of
crafting code that looks pretty much like what other developers
on your team are going to expect: in other words, clean code.
