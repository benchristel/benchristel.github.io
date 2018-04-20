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
