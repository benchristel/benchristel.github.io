---
layout: post
title: Test-free Development
---

# Test-free Development

When I started the [Grove II](https://druidic.github.io/grove-ii/) project, I decided to try an experiment:
what would happen if I didn't write any tests? It had been
years since I wrote any significant chunk of code without
doing TDD, and I'd started to wonder: if I didn't have
automated tests, at what point would I start to feel the
pain, and how would it manifest?

Perhaps unsurprisingly, a lot of what I learned is
specific to browser-based JavaScript. Here are the major
takeaways:

- 700 lines of single-file JavaScript is about my pain threshold.
- TDD isn't that important, but *getting immediate feedback* is.
  Feedback can be from poking around the UI, or the JavaScript console.
  If you can find out in a few seconds whether your code works, you'll reap many of the same
  benefits that you'd get from TDD.
- It's a waste of time to write tests if you aren't sure yet what
  problem you're solving. First figure out what you want to build,
  then build it right.
- Third-party libraries can help
  you push the boundary of how long you can go without writing tests,
  since you'll write less code.
- If you design for testability, you can delay writing tests
  until you want to refactor.
- Don't write any procedural code at the top level. It's too easy for this
  to get scattered all over the codebase and make a huge mess. Write a function called `main()`
  and call it as the very last thing in your top-level script, or use OOP and dependency injection to take care
  of initializing your objects.
- Global variables are great for debugging things from the console.
  Don't encapsulate unless you feel a pressing need to do it.
- When you start hating your code, you'll stop taking care of it.

## Don't try this at work

First things first: I absolutely don't advise that you try
writing code without tests in a professional situation. Work
projects are different from personal projects for a few
reasons:

- If the code is going to ship to production, bugs will
  affect people who can't fix them.
- You don't have control over, and can't really predict,
  the product direction. You will
  eventually run into a situation where you have to try to
  refactor snarled code to accommodate a new feature...
  without tests. Yeah, have fun with that.
- You can't throw the whole thing away and start over.
- Other people will have to read and understand your code.

As a corollary: never, ever, ever skip writing tests on a
team project. If you're working with other people, and don't
want them breaking features you wrote, you need automated
tests that run on every commit.

## The first 700 lines

For the first ~700 lines of JavaScript, everything is smooth
sailing. You can keep everything in one HTML file and it's
still pretty easy to remember where you put a chunk of code.
The app is likely simple enough that you can manually test
whenever you make a change, and it's new enough that you
still find this exciting.

At this stage, you'll probably have a lot of global
variables. This makes it convenient to debug from the JavaScript console:
you don't even have to set breakpoints. Just by poking
around the UI and checking the values of your globals, and you
can often find the source of a bug.

That being said, I was amazed by how few bugs I actually
found in this stage of writing the Grove II. With one exception
(which turned out to be due to a quirk of the `<input>` element that I didn't know about)
everything just worked.

## The entire rest of time and space

I don't know why 700 lines is the critical mass at which
untested code starts to get confusing, but it is, at least
for me. The threshold seems to be unaffected by age or
experience; I noticed that my programs tended to max out at 700 lines
when I was in college, at which point I'd feel like I had to
extract the ugly bits into a library and start the
whole thing over again.

Your own threshold may vary, but once you pass it,
your project will start to slip slowly
but inexorably downhill. The first variety of discomfort
I felt was, unexpectedly, not confusion but **distraction**.
I'd be scrolling through code looking for the place I
needed to change. A few other parts of
the code would catch my eye—duplication I
might want to refactor, maybe future change
points—in any case, they're not what I'm looking for
right now. I'd keep scrolling. Eventually, I'd find the
function I wanted to change. There's a bunch of stuff in
there. Should I put the new code before that line, or
after? Does it make a difference? Why is that other line
there twice? Can I pull it outside the conditional or would
that create a bug?

```javascript
cliInput.addEventListener('keydown', function(event) {
  let input = cliInput.value
  if (event.key === 'Enter') {
    event.preventDefault()

    if (input.charAt(0) === ';') {
      // if the line starts with the magic character ';',
      // eval it as JavaScript
      evalAndPrintResult(input.slice(1))
    } else if (programState === 'waitingForInput') {
      programState = 'running'
      program.next(input)
    }

    cliInput.value = ''
  } else if (event.key === 'Escape') {
    event.preventDefault()
    if (programState !== 'running') {
      programState = 'running'
      program.throw(CANCELED_MSG)
      cliInput.value = ''
    }
  }
})
```

This is a far cry from the panic-inducing chaos of enterprise-quality
legacy code, of course, but it creates a noticeable amount of
mental drag. The feeling of working with the code is qualitatively
different. I'm no longer hammering out line after line
of carefree, greenfield code. Now I have to pick my way carefully around the
debris of existing features.

## Feedback fails

Eventually, I ran into a situation where I could no longer
get useful feedback on my code by manually testing in the
browser. The first time this happened was when I started
implementing the "load from file" feature of the Grove II.

The idea behind this feature is that, although the IDE's
state is always saved in local storage, there are times
when you want to export your work so you can back it up,
or move it to another computer, or whatever. Thus, the Grove
II needed to be able to save and load files from the host
filesystem.

It turns out that when you load a project from a file into
the IDE, a *lot* of stuff needs to happen. The editor has
to be updated to display a file from the new project. Any
program state left over from the old project must be wiped
clean. The local storage module has to be informed of the
new files. All of these things must be done in the right
order, or subtle bugs will result.

Because the feature involved interacting with so many
subsystems, I felt that manually testing each little piece
wouldn't be helpful. The hard part wasn't updating the editor
or swapping objects in local storage; it was managing the
interaction between them.

Having tests in place probably wouldn't have made thinking
about this feature any easier. But it might have made me
think twice before diving into implementation.

## Things fall apart; the center cannot hold

In retrospect, the "load from file" feature was fairly
complex. But when I started implementing it, it seemed like
it should have been simple. I started optimistically hacking,
confident that things would work out.

Things did not work out. After coding for a couple hours
(without ever testing my work), I thought I was close to done. When
I ran my code in the browser for the first time, I
discovered a veritable swarm of bugs.

At this point, I should have realized that I was in over my
head. I should have thought about getting some tests in
place and refactoring my ad-hoc mess toward consistent
abstractions. I did not do that. I refused to admit that
the problem was harder than I'd anticipated, so I kept
hacking. The next line of code was sure to be the fix.
No, maybe the one after that. Or the one after that.

When things refused to work, I became angry at my code. This
is a dangerous frame of mind to be programming in, because
when you start to see the code as an antagonist to be
overcome, you stop taking care of it. Somewhere deep down,
you just wish this code would die. Maybe if you put in
enough hacky changes, it will just collapse under its own
weight and you'll be able to justify throwing it all away.
You certainly don't want to spend more time with it than you
have to, which means you're never going to refactor it.

I think a lot of real-world projects run into this trap; the
developers hate the code and are subversively trying to get
it killed. This can happen without any overt collusion among
the developers; they simply need to frame any motion to
write tests or refactor the code as a waste of time, which is an easy case
to make to management since these activities are
expensive when the code is messy. The moment one developer
makes an argument against refactoring and the other developers
nod in agreement, the codebase is doomed to slip inexorably
toward oblivion.

(As a corollary, management will pretty much never tell
developers to refactor more; they don't know what situation
the code is in, and refactoring may not even be on their
radar as a applicable technique. The initiative to keep the
code clean has to come from engineers.)

## The good parts

My experiment wasn't all doom and gloom. I reaped some
significant benefits by not writing unit tests right off the
bat.

One of the main factors that made my lack of testing an
asset was the fact that when I started the Grove II, I
wasn't at all sure how it should work. Details like how
the shell should behave, how programs should be structured,
and how the user might want to execute one-off JavaScript
were completely up in the air. If I'd written painstaking
unit tests for each of these facets of the design, I'd have
been wasting my time. Each one changed radically
between iterations, so any tests I'd written would have
had to be thrown away at each step.

The key here is that I was intentionally not writing
polished software. I didn't know what I wanted to build,
so my first order of business was to throw stuff at the
wall as fast as possible and find out what made the UX
better and what made it worse. It didn't matter if the
code was ugly or buggy or brittle, because chances were
good it wouldn't outlive the hour. The only thing that
code needed to do was tell me whether or not to pursue a
particular course of implementation.

Somewhat counterintuitively, not writing automated tests
made my code easier to test manually. Because I wasn't concerned
about writing isolated unit tests, my first pass at the
code had global variables all over the place. This meant
that when I wanted to inspect what my program was doing or
test some condition, I could just go into the JavaScript
console and get or set the appropriate variable.

All that said, I think I'm going to go back over my
code and write some tests now.
