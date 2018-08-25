---
layout: post
title: The Bowling Kata in Verse
---

# The Bowling Kata in Verse

You may have heard of [code katas](http://codekata.com/),
but in case you haven't:
they're small exercises, designed to take perhaps half an
hour to an hour, that we software developers can use to
hone our craft through deliberate, focused practice.

You may have also heard of
[Verse](https://benchristel.github.io/verse/) (but let's be
real, you probably haven't). It's a browser-based development
environment for crafting beautifully simple programs. It
also happens to work very well for katas.

In this post, I describe how I used Verse to solve a
slightly modified version of the [Bowling Kata](http://codingdojo.org/kata/Bowling/).
The objective of the kata is simple: score a game of [ten-pin bowling](https://en.wikipedia.org/wiki/Ten-pin_bowling).
However, hidden in that simplicity are a lot of subtle
decisions.

The original kata doesn't have any UI requirement; the
solution can be as simple as a function that's invoked by
a suite of tests. I've added the
twist that the program must have some kind of UI (however
minimal) that a human can use to record a game.

This post is going to be a long one: I describe each tiny
change I make, along with my thought process at each
step. That's kind of the point of the kata: to be very
deliberate and reflect on each move you make. I use
[test-driven development](https://www.martinfowler.com/bliki/TestDrivenDevelopment.html)
throughout the kata, so I've added
**red**, **green**, and **refactor** headings to the
narrative to highlight where each step fits in the TDD cycle.

If you don't mind spoilers, you can see [the half-finished kata here](/assets/bowling-kata/finished.html) (click the `RUN`
button to see the program in action).

## The Stories

- My score starts at 0
- I can bowl and score one ball; my score is the number of pins knocked down
- I can bowl and score multiple times; my score is the total number of pins knocked down
- A game is ten frames long. In each frame I bowl twice.
- If I bowl a strike, there is no second ball in that frame.
- I can bowl a whole game of 10 frames (no bonuses for strikes or spares)
- I get bonuses for strikes and spares (except for the 10th frame)
- I can bowl 3 times in the 10th frame if I get a strike or spare,
  and my score for the 10th frame is the total of the scores
  for the 3 balls.

The stories follow the logical sequence of describing the
rules for the game: we start with the most general rules
and describe the boundaries and special cases as narrowings
and internal differentiations of those rules.

## The first failing test

```javascript
define({
  'test the score for a game starts at 0'() {
    simulate(run)
  }
})
```

The failure is: `ReferenceError: run is not defined`

We can make it pass by defining an empty `run` routine.

The name `run` is not arbitrary, by the way. By convention,
the routine called `run` is the entry point to a Verse
program, analogous to `main` in C or Java.

```javascript
define({
  'test the score for a game starts at 0'() {
    simulate(run)
  },

  *run() {}
})
```

Now the test passes.

Our test description doesn't match the test code, so we're
not done with this test yet.

```javascript
define({
  'test the score for a game starts at 0'() {
    simulate(run)
      .assertDisplay(contains, 'Total score: 0')
  },

  // ...
})
```

This test is more interesting. Now we have an actual
failing assertion. Verse provides the following error
message:

```
Error: Tried to assert that

contains
  Total score: 0
```

Now I wish I'd put quotes around string values in
failure messages—that blank line is a pretty awkward
representation of the empty string. Oh well, I'll fix it in
the next version.

To fix the test failure, we just need to `yield startDisplay` in
the `run` routine.

```javascript
define({
  // ...

  *run() {
    yield startDisplay(() => ['Total score: 0'])
  }
})
```

Hmm... the test is still failing with the same error. This
is actually due to a quirk of Verse: the screen is cleared
when a program exits, and right now our program is exiting
immediately after the `startDisplay` call. To fix it, we
just need to tell the `run` routine to wait a bit before
exiting.

```javascript
define({
  // ...

  *run() {
    yield startDisplay(() => ['Total score: 0'])
    yield wait(Infinity)
  }
})
```

Here I think waiting *forever* is the least surprising thing
for the program to do.

At this point we can see our program running (by clicking
the `RUN` button) and verify that it does in fact do what
the tests say it does.

## Bowling Once

Now we are ready to start the second user story: bowling
a single ball and recording the score.

Right now there is nothing on the screen prompting the
user to interact with the program in any way, so let's
nail that down first.

Our second test looks like this:

```javascript
define({
  // ...

  'test the user sees a prompt to enter their score'() {
    simulate(run)
      .assertDisplay(contains, 'Bowl once and enter your score.')
  },

  // ...
})
```

It fails, of course. We can make it pass by adding to the
lines we output in `startDisplay`:

```javascript
define({
  // ...

  *run() {
    yield startDisplay(() => [
      'Total score: 0',
      'Bowl once and enter your score.'
    ])
    yield wait(Infinity)
  }
})
```

Now we have to think about *how* the user will enter scores.
The possible scores for one ball are the integers 1 to 10.
Since a strike (10 points) can be represented by `X`, all
the possibilities can be made to fit in a single character.
This means the user will only have to press a single key
to enter their score, and validating the entered score
will be simple.

We'll have to communicate this scheme to the user, of course.
We can modify our existing test to describe that
communication.

```javascript
define({
  // ...

  'test the user sees a prompt to enter their score'() {
    simulate(run)
      .assertDisplay(contains, 'Bowl once and enter your score.')
      .assertDisplay(contains, 'Press 0-9, or X for a strike')
  },

  // ...
})
```

And the code to make it pass:

```javascript
define({
  // ...

  *run() {
    yield startDisplay(() => [
      'Total score: 0',
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
    yield wait(Infinity)
  }
})
```

Now let's add a test that demonstrates the user can actually
enter their score:

```javascript
define({
  // ...

  'test entering the score for one ball'() {
    simulate(run)
      .receive(keyDown('5'))
      .assertDisplay(contains, 'Total score: 5')
  },

  // ...
})
```

This test fails. We're ready to implement the first real
bit of functionality.

Here's the least code we can write to make the test pass:

```javascript
define({
  // ...

  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
    score = yield waitForChar()
    yield wait(Infinity)
  }
})
```

However, if you play around with this program in the `RUN`
view, you'll quickly notice a huge flaw: you can enter
characters that aren't numbers or Xs. Let's write a
regression test to demonstrate the bug:

```javascript
define({
  // ...

  'test entering an invalid score does nothing'() {
    simulate(run)
      .receive(keyDown('a'))
      .assertDisplay(contains, 'Total score: 0')
  },

  // ...
})
```

This test fails because the program actually displays
`Total score: a` when we expected `Total score: 0`.

At this point, I start feeling uncertain about how exactly
I'm going to fix this problem. Half-formed possible
solutions begin to flit through my brain in rapid succession.
However, I know that I don't have to solve the whole
problem all at once if I delegate the solution to a
function that I can unit-test. I take a deep breath and
start looking for places to insert a function call.

In this case, it's fairly easy to find the point at which
the call needs to happen. We can find it by looking for
the point where the code *starts to be wrong*. It's this
line:

```javascript
score = yield waitForChar()
```

The problem with this line is that its effects are too
immediate. There's no seam here, no opportunity for us to
insert intermediate processing between the keyboard input
and its effect. So let's create one.

```javascript
score = validateScore(yield waitForChar())
```

Boom! Now *all* our tests are failing.

Of course they are. Because, as the test output reminds us,
`validateScore is not defined`.

We can fix that.

```javascript
define({
  // ...

  validateScore(x) {
    return x
  }
})
```

And we're back to just the one failure.

Let's do the least we can to make the test pass. Then we
can drive out the remaining behavior of `validateScore` with
unit tests.

```javascript
define({
  // ...

  validateScore(x) {
    return x === '5' ? x : 0
  }
})
```

Note that I'm playing fast and loose with the types here.
Sometimes `validateScore` returns a string and sometimes it
returns a number. That's fine for now. We'll come back and
fix it later.

At this point, all the tests pass but `validateScore` is
obviously wrong. So let's write some unit tests.

```javascript
define({
  // ...

  'test validateScore'() {
    assert(validateScore('1'), is, '1')
  }

  // ...
})
```

This test gives us the following failure:

```
Error: Tried to assert that
  0
isExactly
  1
```

The code to make it pass:

```javascript
define({
  // ...

  validateScore(x) {
    return !isNaN(+x) ? x : 0
  }

  // ...
})
```

Here we're using unary `+` to convert `x` to a number,
which will result in `NaN` if `x` isn't numeric.

Now we need `validateScore` to return `10` for a strike.
Let's just add to our existing test.

```javascript
define({
  // ...

  'test validateScore'() {
    assert(validateScore('1'), is, '1')
    assert(validateScore('X'), is, 10)
  },

  // ...
})
```

Normally I'd limit myself to one assertion per test, but in
this case I feel it's okay to have more than one. I have
a couple reasons for this:

- The assertions are simple and self-explanatory. Breaking them out into
  separate tests would not make the test suite more
  readable or expressive. It would just add noise.
- The expected values of each assertion (the `'1'` and `10`)
  are different, so if one of the assertions fails I'll be
  able to tell from the error message which one it is.

This test fails. Let's make it pass.

```javascript
define({
  // ...

  validateScore(x) {
    if (x === 'X') return 10
    return !isNaN(+x) ? x : 0
  }

  // ...
})
```

## Refactoring `validateScore`

At this point I pause and ask myself if the code is as clean
as it could be. The first thing that stands out is the name `validateScore`.
I don't love it, because the function is not really doing validation. It's
converting input to a number. I change the name of the
function and update the callsites.

```javascript
define({
  // ...

  inputToNumber(x) {
    if (x === 'X') return 10
    return !isNaN(+x) ? x : 0
  }

  // ...
})
```

Next, I notice that the parameter name `x` says nothing
about its value. It doesn't even hint at what type it is. We
can fix that.

```javascript
define({
  // ...

  inputToNumber(c) {
    if (c === 'X') return 10
    return !isNaN(+c) ? c : 0
  }

  // ...
})
```

The letter `c` hints that the variable holds a
single-character string. In a longer function I might choose
a more descriptive name, but this function is short enough
that you can see all the usages of the parameter at a
glance, so I don't worry about it.

I do notice another problem, though: now that it's clear
that `c` is a character, our abuse of JavaScript's weak
typing stands out like a sore thumb. `c : 0`? Really?
Who wrote this code?

We just have to change one test to drive out the fix:
`inputToNumber('1')` should now return `1`.

```javascript
define({
  // ...

  'test inputToNumber'() {
    assert(inputToNumber('1'), is, 1)
    assert(inputToNumber('X'), is, 10)
  },

  // ...
})
```

And the fix:

```javascript
define({
  // ...

  inputToNumber(c) {
    if (c === 'X') return 10
    return !isNaN(+c) ? +c : 0
  }

  // ...
})
```

One more thing I'd like to change: we're using two different
constructs (one `if` statement and one ternary operator) for
very similar conditional logic. I prefer `if`  statements
here, but let's add some `else`s.

```javascript
define({
  // ...

  inputToNumber(c) {
    if (c === 'X') return 10
    else if (isNaN(+c)) return 0
    else return +c
  }

  // ...
})
```

Those `else`s aren't strictly necessary, but they save you
from having to look inside the `if` statement to see that
there are three mutually-exclusive branches.

## Case-insensitive strikes

After playing around a bit with the program in the `RUN`
view, it starts to bother me that I have to type a capital
`X` to record a strike. I imagine other users would find
this horribly unintuitive. So I decide to make
`inputToNumber` case-insensitive.

First, a failing test:

```javascript
define({
  // ...

  'test inputToNumber is case-insensitive'() {
    assert(inputToNumber('x'), is, 10)
  },

  // ...
})
```

I added a new test (instead of appending to the existing
`inputToNumber` test) because now we have two assertions
that expect `10`, and I'd like to be able to tell which one
is failing.

Here's the code that makes the test pass:

```javascript
define({
  // ...

  inputToNumber(c) {
    if (uppercase(c) === 'X') return 10
    else if (isNaN(+c)) return 0
    else return +c
  }

  // ...
})
```

## The code so far

At this point, we've done the first two user stories, and
the code looks like this:

```javascript
define({
  'test the score for a game starts at 0'() {
    simulate(run)
      .assertDisplay(contains, 'Total score: 0')
  },

  'test the user sees a prompt to enter their score'() {
    simulate(run)
      .assertDisplay(contains, 'Bowl once and enter your score.')
      .assertDisplay(contains, 'Press 0-9, or X for a strike')
  },

  'test entering the score for one ball'() {
    simulate(run)
      .receive(keyDown('5'))
      .assertDisplay(contains, 'Total score: 5')
  },

  'test entering an invalid score does nothing'() {
    simulate(run)
      .receive(keyDown('a'))
      .assertDisplay(contains, 'Total score: 0')
  },

  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
    score = inputToNumber(yield waitForChar())
    yield wait(Infinity)
  },

  'test inputToNumber'() {
    assert(inputToNumber('1'), is, 1)
    assert(inputToNumber('X'), is, 10)
  },

  'test inputToNumber is case-insensitive'() {
    assert(inputToNumber('x'), is, 10)
  },

  inputToNumber(c) {
    if (uppercase(c) === 'X') return 10
    else if (isNaN(+c)) return 0
    else return +c
  }
})
```

## Bowling Multiple Times

Now we come to the third user story: totaling the score
for multiple balls.

```javascript
define({
  // ...

  'test entering the score for multiple balls'() {
    simulate(run)
      .receive(keyDown('1'))
      .receive(keyDown('X'))
      .assertDisplay(contains, 'Total score: 11')
  },

  // ...
})
```

This fails because when the second `keyDown` happens,
our program is not listening for it. It's in the
`wait(Infinity)` call.

Here's the minimal code to make the test pass:

```javascript
define({
  // ...

  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
    score = inputToNumber(yield waitForChar())
    score += inputToNumber(yield waitForChar())
    yield wait(Infinity)
  },

  // ...
})
```

There's some obvious duplication here, but the duplicated
lines are not *quite* identical. Let's make them identical
so we can remove the duplication confidently.

```javascript
define({
  // ...

  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
    score += inputToNumber(yield waitForChar())
    score += inputToNumber(yield waitForChar())
    yield wait(Infinity)
  },

  // ...
})
```

Since the tests still pass, I feel good about refactoring
this to a loop.

```javascript
define({
  // ...

  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])

    for (__ of range(1, 1000)) {
      score += inputToNumber(yield waitForChar())
    }

    yield wait(Infinity)
  },

  // ...
})
```

Here I chose a large upper bound for the loop index because
there's no test constraining it.

Why a `for ... of` loop and not a `while` loop? Well, Verse
shuns `while` and `for(;;)` loops, because they tend to become
syntactically valid infinite loops at some point during their
creation. Since Verse runs the tests on every single code change,
these accidental infinite loops can
hang your browser at the most inopportune times.

Here's a safe way to create a truly unbounded loop in a
Verse routine:

```javascript
define({
  // ...

  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run())
  },

  // ...
})
```

Here, `retry` works like Clojure's `recur`. It
tail-recurses the routine, allowing repetition without
blowing up the stack.

Unfortunately, this code fails the tests. Upon closer
inspection, it's clear why: the `score` variable is initialized
to `0` at the top of the routine, so our `score +=` line
has no lasting effect.

Fortunately, `retry` lets you pass arguments to the next
invocation of the routine, so we can rephrase our code
thus:

```javascript
define({
  // ...

  *run(score = 0) {
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run(score))
  },

  // ...
})
```

This code is concise, and it's very idiomatic Verse :)

## Limiting the number of frames

Of course, a game of bowling doesn't go on forever. We need
to limit the `retry` loop to ten frames (or twenty balls).

Here's my first attempt at a failing test for that:

```javascript
define({
  // ...

  'test the game ends after 10 frames of 2 balls each'() {
    simulate(run)
      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .receive(keyDown('5'))
      .receive(keyDown('5'))

      .assertDisplay(contains, 'GAME OVER')
  },

  // ...
})
```

This test is a bit ridiculous. We can refactor the duplication
away:

```javascript
define({
  // ...

  'test the game ends after 10 frames of 2 balls each'() {
    const bowlFrames = numFrames => simulator => {
      for (__ of range(1, numFrames)) {
        simulator
          .receive(keyDown('5'))
          .receive(keyDown('5'))
      }
    }

    simulate(run)
      .do(bowlFrames(10))
      .assertDisplay(contains, 'GAME OVER')
  },

  // ...
})
```

Though I only refactor production code when all my tests are
green, I like to refactor the tests themselves when they're
red. It's all too easy to accidentally remove assertions or
crucial setup from a test during refactoring, so that the
test no longer tests what it's supposed to. By refactoring
tests only when they're failing, I avoid that.

We can make it pass by doing this:

```javascript
define({
  // ...

  *run(score = 0, balls = 0) {
    let gameOverMessage = balls > 5 ? 'GAME OVER' : ''

    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike',
      gameOverMessage
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run(score, balls + 1))
  },

  // ...
})
```

I've intentionally mistyped the number of balls needed to
end the game. The test passes despite this, so I think I
need a more specific test.

We can get the specificity we need by adding a negative
assertion just before the event that is supposed to display
the `GAME OVER` message:

```javascript
define({
  // ...

  'test the game ends after 10 frames of 2 balls each'() {
    const bowlFrames = numFrames => simulator => {
      for (__ of range(1, numFrames)) {
        simulator
          .receive(keyDown('5'))
          .receive(keyDown('5'))
      }
    }

    simulate(run)
      .do(bowlFrames(9))
      .receive(keyDown('5'))
      .assertDisplay(not(contains), 'GAME OVER')
      .receive(keyDown('5'))
      .assertDisplay(contains, 'GAME OVER')
  },

  // ...
})
```

Here's how we make it pass:

```javascript
define({
  // ...

  *run(score = 0, balls = 0) {
    let gameOverMessage = balls == 20 ? 'GAME OVER' : ''

    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike',
      gameOverMessage
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run(score, balls + 1))
  },

  // ...
})
```

The `== 20` is a bit sketchy. I think I'll be able to
keep bowling after the 20th ball, and indeed I can.

```javascript
define({
  // ...

  'test the game ends after 10 frames of 2 balls each'() {
    const bowlFrames = numFrames => simulator => {
      for (__ of range(1, numFrames)) {
        simulator
          .receive(keyDown('5'))
          .receive(keyDown('5'))
      }
    }

    simulate(run)
      .do(bowlFrames(9))
      .receive(keyDown('5'))
      .assertDisplay(not(contains), 'GAME OVER')
      .receive(keyDown('5'))
      .assertDisplay(contains, 'GAME OVER')
      .receive(keyDown('5'))
      .assertDisplay(contains, 'GAME OVER')
  },

  // ...
})
```

This fails. But I've violated my rule about keeping all the
assertions in a test looking different. Let's split out
a new test, and move the `bowlFrames` helper to the global
scope.

```javascript
define({
  // ...

  bowlFrames: numFrames => simulator => {
    for (__ of range(1, numFrames)) {
      simulator
        .receive(keyDown('5'))
        .receive(keyDown('5'))
    }
  },

  'test the game ends after 10 frames of 2 balls each'() {
    simulate(run)
      .do(bowlFrames(9))
      .receive(keyDown('5'))
      .assertDisplay(not(contains), 'GAME OVER')
      .receive(keyDown('5'))
      .assertDisplay(contains, 'GAME OVER')
  },

  'test I cannot bowl after the last frame'() {
    simulate(run)
      .do(bowlFrames(11))
      .assertDisplay(contains, 'GAME OVER')
  },

  // ...
})
```

And the implementation that makes it pass:

```javascript
  *run(score = 0, balls = 0) {
    let gameOverMessage = balls >= 20 ? 'GAME OVER' : ''

    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike',
      gameOverMessage
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run(score, balls + 1))
  },
```

In order to really assert that I can't bowl after the last
frame, it's not enough to check for the `GAME OVER` message.
I should also make sure my score didn't increase when I
tried to bowl that 11th frame.

```javascript
  'test I cannot bowl after the last frame'() {
    simulate(run)
      .do(bowlFrames(11))
      .assertDisplay(contains, 'GAME OVER')
      .assertDisplay(contains, 'Total score: 100')
  },
```

The test fails, and I can see from the message that the
program thinks my total score should be 110 after 11 frames.

Getting this test to pass using the existing constructs in
the `run` function is awkward enough to give me pause.
It's hard to express the *finality* of `GAME OVER` in the
same routine that does scorekeeping, so I decide to break
out a new one.

```javascript
  *run(score = 0, balls = 0) {
    if (balls >= 20) yield gameOver(score)

    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run(score, balls + 1))
  },

  *gameOver(finalScore) {
    yield startDisplay(() => [
      `Total score: ${score}`,
      'GAME OVER'
    ])
    yield wait(Infinity)
  },
```
