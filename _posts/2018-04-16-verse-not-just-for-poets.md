---
layout: post
title: 'Verse: Not Just for Poets'
---

# Verse: Not Just for Poets

I just released the [first tech
demo](https://benchristel.github.io/pico-fermi-bagel) of my
browser-based programming environment
[Verse](https://github.com/druidic/verse).

![A screenshot of the Verse editor running a simple game](/assets/verse-demo.png)

# Why?

Verse's goal is to *allow people to program without being
programmers*. There are tons of tutorials and tools out there
for people without CS degrees who are trying to become
professional software developers. That's awesome! I'm really
glad those things exist. But despite all of the effort that
people have dedicated to giving aspiring software engineers
a leg up, the learning curve for becoming a
JavaScript dev is still pretty frightening, and the mountain
of Things To Learn is only getting higher.

But here's the thing: when I started programming, I had no
desire to be a professional programmer. I didn't even know
what that *meant*. But I still felt the thrill of making
things work: of creating something I could see and interact
with just by describing it to a computer in sufficiently
precise terms. Coding is the act of thinking
*so hard* and *so deeply* about something that it literally
pops into existence. The first time you experience this, it
is nothing short of astonishing. If you code, you learn in
your gut that
rational thought is a useful tool, that "true" and "false"
are just as unambiguous and concrete as "working" and "broken",
because when you think things that make sense and are
correct, your code works, and when you think things that are
wrong or nonsense, it doesn't.

That rational instinct shouldn't be the sole
property of professional computer programmers. It can
and should be available to anyone with a computer in their
house or their pocket. I also think that many people, if not
most, have at least some small problems they could solve, or
delights they could pursue, if only they had a platform for
writing code and sharing it with others.

# What is it?

At first glance, Verse seems reminiscent of something like
[JSFiddle](https://jsfiddle.net), but it has a few important
differences:

- It's intended to be a "real" development environment for
  building fairly complex applications. E.g. it facilitates
  TDD and lets you do futuristic things like organize your
  code into multiple files. Soon, you'll even be able to
  download your code as a standalone HTML file which can be
  easily checked into Git and loaded back into the editor.

- It's designed with offline use in mind; the editor's code
  is cached locally, so once you've visited
  <druidic.github.io> once, you can access it from anywhere,
  even without a network connection. Your app's code is saved
  locally as well.

- It comes with an opinionated framework that facilitates
  I/O, imperative programming using simulated threads, and
  Redux-like state manage&shy;ment with runtime typechecking
  and automatic persistence.

- It doesn't let you waste time messing with HTML and CSS.
  If you want a terminal-style app that takes textual input
  and logs textual output, it makes that really easy. If you
  want graphics, you have to use Canvas. But you'll never
  have to debug an improperly-scoped CSS selector, or
  scratch your head over some weird HTML layout thing that
  only happens in Safari.

I'll probably write more about its philosophy and
implementation in the coming weeks. Stay tuned.
