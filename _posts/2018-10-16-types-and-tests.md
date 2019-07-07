---
layout: post
title: Types and Tests
---

# Types and Tests

I recently watched Gary Bernhardt's talk ["Ideology"](https://www.destroyallsoftware.com/talks/ideology).
Broadly, it's about how unspoken assumptions can make it hard for people to reach agreement or even have a sensical debate. The specific example he uses is the perennial "types versus tests" flamewar.

> Some people claim that unit tests make type systems
> unnecessary: "types are just simple unit tests written for
> you, and simple unit tests aren't the important ones".
> Other people claim that type systems make unit tests
> unnecessary: "dynamic languages only need unit tests
> because they don't have type systems." What's going on
> here? These can't both be right.

As someone who likes both types and tests and has made an
effort to get both of them into
[Verse](https://benchristel.github.io/verse), I appreciated
Gary's analysis of the situation. You should go watch the
video. But it left me with more questions than I started
with. Here I am, this weirdo who's on *neither* pole of the
types–tests spectrum, who somehow likes *both at once*. Even
Gary admits to bouncing back and forth between the "type
ideology" and the "test ideology". What hope is there of
reconciling the two belief systems?

Spoiler alert: I'm going to reconcile them in this post.

## Orthogonal Approaches to Informal Reasoning

As Moseley and Marks point out in [Out of the Tar
Pit](https://github.com/papers-we-love/papers-we-love/blob/master/design/out-of-the-tar-pit.pdf),
most of our knowledge about the systems we build is gained
through informal reasoning. In other words, we read the code
to find out what it does. We have a mental model of what the
program should do, and if the code looks like it matches
that mental model, it increases our degree of trust in the
program.

Tests and types both provide a framework for informal reasoning, but they do so by making basically orthogonal statements about a program.

TYPES say: **"The program always does something."** In other
words, it won't crash. (Unfortunately, we often still have to worry about infinite loops thanks to the [halting
problem](https://en.wikipedia.org/wiki/Halting_problem), but [some typed languages can banish even that uncertainty](http://sblp2004.ic.uff.br/papers/turner.pdf)).

TESTS say: **"The program does the right thing some of the time."**

Clearly, neither of these statements, or even both together,
constitutes anything close to a proof of correctness. But
it's also clear to me that both are desirable. We want to
be able to say these things about our programs. Both
increase our degree of trust that the program is correct.
Together they provide a robust foundation to support
informal reasoning.

## Caveat: Type Systems

The statement above about types probably deserves clarification. Not all type systems allow us to definitively state that "the program always does something".
The type systems most of us use day-to-day—those of languages like C, Java, and Go—don't provide anything close to a guarantee of no runtime errors.

The main problem with these type systems is that they allow null references. Since `null` can masquerade as any type, but will crash your program if you do anything interesting with it, the type system generally can't guarantee your program won't crash.

The type systems I'm talking about above, the ones that say
definitively "the program always does something", are the
ones that forbid null references, arbitrary typecasting, and
other unsafe operations. These type systems are only found
in functional languages like Elm and Haskell (and that's not
accidental—they're impossible in languages that dynamically
load or evaluate code at runtime, so that rules out C, Java,
Ruby, Python, and basically every other OO language).

Because of these extreme limitations on the types of type
systems that can prove our programs are internally
consistent, I actually tend to rely a bit more on tests than
on types. But I still think types are valuable, and they
become more valuable the more you can avoid awful things
like null pointers and casting.
