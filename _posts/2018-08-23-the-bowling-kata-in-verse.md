---
layout: post
title: The Bowling Kata in Verse
---

# The Bowling Kata in Verse

You may have heard of [code katas](http://codekata.com/),
but in case you haven't:
they're small coding exercises that programmers use to
practice and hone their craft.

Katas aren't just about solving problems and
they're not just about getting code to work. Rather, katas
challenge us to think deeply and deliberately about the
hundreds of tiny decisions we make while coding. By
reflecting on our choices, we can learn to make better ones.

In a few of my previous posts I've written about my side-project
[Verse](https://benchristel.github.io/verse/).
Verse is a browser-based development
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
button to see the program in action). You can also edit
the code on that page, so you can try your own solution or
follow along with mine.

## The Problem

You can read [how ten-pin bowling is scored here](https://en.wikipedia.org/wiki/Ten-pin_bowling#Rules_of_play),
but to summarize:

- The game consists of ten *frames*. In each frame the
  player has
  two throws of the ball to knock down as many of the ten
  pins as possible. The pins are reset after every frame.
- A player's score for one frame is the total number of pins
  knocked down in that frame, plus any bonuses (see below).
- Knocking down all ten pins with a single ball is termed
  a *strike*. If a player scores a strike, they get a bonus:
  the number of pins knocked down by the next two balls is
  added to the score for that frame.
- Knocking down all ten pins with two balls is called a
  *spare*. The bonus for a spare is the number of pins
  knocked down by the next ball.
- If a player scores a strike or spare in the tenth frame,
  the pins are reset and the player is
  awarded bonus throws (two throws for a strike and
  one for a spare). The number of pins knocked down by the
  bonus throws is added to the score for the tenth frame.

Modern bowling alleys keep score automatically, using sensors
that detect when pins have been knocked down. Of course,
there's no reason the players can't keep score themselves
(aside from the tedium of doing so). The goal of the kata
is to write a program that makes manual scoring painless
(and perhaps even delightfully satisfying),
eliminating the need for expensive pin-detection systems.

The kata imposes no hard requirements on what the UI or UX
must be. Any program that satisfies the needs of the
bowler/scorekeeper will do.

## The User Stories

Rather than try to implement the entire scoring system all
at once, I'm going to take an incremental approach. The
program will at first be able to score only a small subset
of possible games correctly. We will gradually expand this
subset, until the program correctly scores all possible
games of bowling.

At each step we will have a working piece of software that
we can test and even show to potential users. This will help
us discover bugs and usability issues early.

We can express each increment of functionality as a *user story*:
a statement about what the player should be able to achieve
with the program after that functionality is added.

- When I start a game, my score is 0
  (Once this is implemented, we will correctly score games
  where the player does not knock down any pins)
- I can bowl and score one ball; my score is the number of pins knocked down.
  (Once this is implemented, we will correctly score games where the
  player only knocks down pins with one ball)
- I can bowl and score multiple times; my score is the total number of pins knocked down.
  (Once this is implemented, we will correctly score games with
  no strikes or spares)
- A game is ten frames long. In each frame I bowl twice.
  (Once this is implemented, we will halt the game after ten
  frames rather than relying on the player to know when the
  game is over)
- If I bowl a strike, there is no second ball in that frame.
  (Once this is implemented, we will count frames correctly
  if the player bowls a strike, though the player will have
  to add the bonus points for the strike themselves.)
- I get bonuses for strikes and spares (except for the 10th frame).
  (Once this is implemented, strikes and spares in all frames but
  the tenth will be scored correctly.)
- I can bowl 3 times in the 10th frame if I get a strike or spare,
  and my score for the 10th frame is the total of the scores
  for the 3 balls.
  (Once this is implemented, we will correctly score all possible games.)

The stories follow the logical sequence of describing the
rules for the game: we start with the most general rules
and describe the boundaries and special cases as narrowings
and internal differentiations of those rules.

## The first user story

> When I start a game, my score is 0.

I open a development build of [Verse](https://github.com/benchristel/verse)
in my browser, and I am presented with a blank editor.
I click the `RUN` button, which tells Verse that it's okay
to `eval` any code I type. Then I switch over to the `TEST`
tab, which is currently telling me that `All 0 tests passed`.

Time to write our first failing test.

### Red

Verse comes with a built-in test framework that makes it
easy to simulate interactions with a program's UI. The
first failing test I'll write will just try
to `simulate` `run`ning our program, which of course doesn't
exist yet.

```javascript
define({
  'test the score for a game starts at 0'() {
    simulate(run)
  }
})
```

Verse parses the code and runs the tests every time I
type a character in the editor, so as soon as I'm done
typing the code above I can see that the test is failing
with `ReferenceError: run is not defined`.

### Green

We can make the test pass by defining a `run` routine.
Verse makes a strong distinction between *routines*, which
may interact with the outside world via *effects*, and
*functions*, which may only interact with other program
components through their parameters and return values.
Verse implements routines using JavaScript [generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*).

We want to create a routine here because we'll soon need to
interact with the keyboard and screen, and we can't do that
from a function. The asterisk in `*run` makes it a routine.

```javascript
define({
  'test the score for a game starts at 0'() {
    simulate(run)
  },

  *run() {}
})
```

The name `run` is not arbitrary, by the way. By convention,
the routine called `run` is the entry point to a Verse
program, analogous to `main` in C or Java.

### Refactor

There is no duplication or ugly code yet, so the refactor
step is a no-op.

### Red

Our test description doesn't match the test code, so we're
not done with this test yet. We'll add an assertion:

```javascript
  'test the score for a game starts at 0'() {
    simulate(run)
      .assertDisplay(contains, 'Total score: 0')
  },
```

Verse provides the following failure message:

```
Error: Tried to assert that

contains
  Total score: 0
```

Now I wish I'd put quotes around string values in
assertion failures—that blank line is a pretty awkward
way to represent the empty string. Oh well, I'll fix it in
the next version.

### Green

To fix the test failure, we can `yield startDisplay(...)`
from the `run` routine. The `yield` keyword is needed for
the code to affect the outside world.

```javascript
  *run() {
    yield startDisplay(() => ['Total score: 0'])
  }
```

At this point I notice that the test is still failing with
the same error. This
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
to do.

At this point I click the `RUN` button to see the program
in action. I can quickly verify that it does in fact do what
the tests say it does.

## The second user story

> I can bowl and score one ball; my score is the number of pins knocked down.

### Red

Right now there is nothing on the screen prompting the
user to record their score or interact with the program in
any way. Let's expose that problem with a test.

```javascript
  'test the user sees a prompt to enter their score'() {
    simulate(run)
      .assertDisplay(contains, 'Bowl once and enter your score.')
  },
```

### Green

```diff
  *run() {
    yield startDisplay(() => [
      'Total score: 0',
+     'Bowl once and enter your score.'
    ])
    yield wait(Infinity)
  }
```

### Reflect

Now we have to think about *how* the user will enter scores.
The possible scores for one ball are the integers 1 to 10.
Since a strike (10 points) can be represented by `X`, all
the possibilities can be made to fit in a single character.
This means the user will only have to press a single key
to enter their score, and validating the entered score
will be simple.

### Red

We'll have to communicate this scheme to the user, of course.
We can modify our existing test to describe that
communication.

```diff
  'test the user sees a prompt to enter their score'() {
    simulate(run)
      .assertDisplay(contains, 'Bowl once and enter your score.')
+     .assertDisplay(contains, 'Press 0-9, or X for a strike')
  },
```

### Green

```diff
  *run() {
    yield startDisplay(() => [
      'Total score: 0',
      'Bowl once and enter your score.',
+     'Press 0-9, or X for a strike'
    ])
    yield wait(Infinity)
  }
```

### Red

Now we need a test that demonstrates the user can actually
enter their score. We can use the `receive` method on the
simulated program to send it a keypress.

```javascript
  'test entering the score for one ball'() {
    simulate(run)
      .receive(keyDown('5'))
      .assertDisplay(contains, 'Total score: 5')
  },
```

This test fails. We're ready to implement the first real
bit of functionality.

### Green

Here's the least code we can write to make the test pass.
The `yield waitForChar` statement reads a single character
from the keyboard and returns it.

```diff
  *run() {
+   let score = 0
    yield startDisplay(() => [
-     'Total score: 0',
+     `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
+   score = yield waitForChar()
    yield wait(Infinity)
  }
```

However, if you play around with this program in the `RUN`
view, you'll quickly notice a huge flaw: you can enter
characters that aren't numbers or Xs. Let's write a
test to demonstrate the bug.

### Red

```javascript
  'test entering an invalid score does nothing'() {
    simulate(run)
      .receive(keyDown('a'))
      .assertDisplay(contains, 'Total score: 0')
  },
```

This test fails because the program actually displays
`Total score: a` when we expected `Total score: 0`.

### Refactor

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
  validateScore(x) {
    return x
  }
```

And we're back to just the one failure.

### Green

Let's do the least we can to make the test pass. Then we
can drive out the remaining behavior of `validateScore` with
unit tests.

```javascript
  validateScore(x) {
    return x === '5' ? x : 0
  }
```

Note that I'm playing fast and loose with the types here.
Sometimes `validateScore` returns a string and sometimes it
returns a number. That's fine for now. We'll come back and
fix it later.

### Red

At this point, all the tests pass but `validateScore` is
obviously wrong. So let's write some unit tests.

```javascript
  'test validateScore'() {
    assert(validateScore('1'), is, '1')
  }
```

The failure:

```
Error: Tried to assert that
  0
isExactly
  1
```

### Green

```diff
  validateScore(x) {
-   return x === '5' ? x : 0
+   return !isNaN(+x) ? x : 0
  }
```

Here we're using unary `+` to convert `x` to a number,
which will result in `NaN` if `x` isn't numeric.

### Red

Now we need `validateScore` to return `10` for a strike.
Let's just add to our existing test.

```diff
  'test validateScore'() {
    assert(validateScore('1'), is, '1')
+   assert(validateScore('X'), is, 10)
  },
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

### Green

```diff
  validateScore(x) {
+   if (x === 'X') return 10
    return !isNaN(+x) ? x : 0
  }
```

### Refactor

At this point I pause and ask myself if the code is as clean
as it could be. The first thing that stands out is the name `validateScore`.
I don't love it, because the function is not really doing validation. It's
converting input to a number. I change the name of the
function and update the callsites.

```javascript
  inputToNumber(x) {
    if (x === 'X') return 10
    return !isNaN(+x) ? x : 0
  }
```

Next, I notice that the parameter name `x` says nothing
about its value. It doesn't even hint at what type it is. We
can fix that.

```javascript
  inputToNumber(c) {
    if (c === 'X') return 10
    return !isNaN(+c) ? c : 0
  }
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

### Red

We just have to change one test to drive out the fix:
`inputToNumber('1')` should now return `1`.

```diff
  'test inputToNumber'() {
-   assert(inputToNumber('1'), is, '1')
+   assert(inputToNumber('1'), is, 1)
    assert(inputToNumber('X'), is, 10)
  },
```

### Green

```diff
  inputToNumber(c) {
    if (c === 'X') return 10
-   return !isNaN(+c) ? c : 0
+   return !isNaN(+c) ? +c : 0
  }
```

### Refactor

One more thing I'd like to change: we're using two different
constructs (one `if` statement and one ternary operator) for
very similar conditional logic. I prefer `if`  statements
here, but let's add some `else`s.

```javascript
  inputToNumber(c) {
    if (c === 'X') return 10
    else if (isNaN(+c)) return 0
    else return +c
  }
```

Those `else`s aren't strictly necessary, but they save you
from having to look inside the `if` statement to see that
there are three mutually-exclusive branches.

## Case-insensitive strikes

After playing around a bit with the program in the `RUN`
view, it starts to bother me that I have to hold `shift`
and type a capital
`X` to record a strike. I imagine other users would find
this horribly unintuitive. So I decide to make
`inputToNumber` case-insensitive.

### Red

```javascript
  'test inputToNumber is case-insensitive'() {
    assert(inputToNumber('x'), is, 10)
  },
```

I added a new test (instead of appending to the existing
`inputToNumber` test) because now we have two assertions
that expect `10`, and I'd like to be able to tell which one
is failing.

### Green

```diff
  inputToNumber(c) {
-   if (c === 'X') return 10
+   if (uppercase(c) === 'X') return 10
    else if (isNaN(+c)) return 0
    else return +c
  }
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

## The third user story

> I can bowl and score multiple times; my score is the total number of pins knocked down.

### Red

```javascript
  'test entering the score for multiple balls'() {
    simulate(run)
      .receive(keyDown('1'))
      .receive(keyDown('X'))
      .assertDisplay(contains, 'Total score: 11')
  },
```

This fails because when the second `keyDown` happens,
our program is not listening for it. It's in the
`wait(Infinity)` call.

### Green

```diff
  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
    score = inputToNumber(yield waitForChar())
+   score += inputToNumber(yield waitForChar())
    yield wait(Infinity)
  },
```

### Refactor

There's some obvious duplication here, but the duplicated
lines are not *quite* identical. Let's make them identical
so we can remove the duplication confidently.

```diff
  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])
-   score = inputToNumber(yield waitForChar())
+   score += inputToNumber(yield waitForChar())
    score += inputToNumber(yield waitForChar())
    yield wait(Infinity)
  },
```

Since the tests still pass, I feel good about refactoring
this to a loop.

```diff
  *run() {
    let score = 0
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])

-   score += inputToNumber(yield waitForChar())
-   score += inputToNumber(yield waitForChar())
+   for (__ of range(1, 1000)) {
+     score += inputToNumber(yield waitForChar())
+   }

    yield wait(Infinity)
  },
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
```

The `retry` function works like Clojure's `recur`. It
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
  *run(score = 0) {
    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike'
    ])

    score += inputToNumber(yield waitForChar())
    yield retry(run(score))
  },
```

This code is concise, and it's very idiomatic Verse :)

## The fourth user story

> A game is ten frames long. In each frame I bowl twice.

Of course, a game of bowling doesn't go on forever. We need
to limit the `retry` loop to ten frames (or twenty balls).

### Red

Here's my first attempt at a failing test:

```javascript
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
```

I normally don't like to use loops in my test code because
I start feeling like I should have tests for my tests, but
in this case the repetition is just too ridiculous.
We can refactor the duplication away:

```javascript
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
```

I try to refactor production code only when all my tests are
green, but I like to refactor the tests themselves when
they're red. This helps me avoid mistakes. It's all too easy
to accidentally remove assertions or
crucial setup from a passing test during refactoring, so that the
test no longer tests what it's supposed to. By refactoring
tests only when they're failing, I can check that the test
fails in the same way before and after the refactor. This
gives me confidence that I didn't remove anything
important.

### Green

```diff
- *run(score = 0) {
+ *run(score = 0, balls = 0) {
+   let gameOverMessage = balls > 5 ? 'GAME OVER' : ''

    yield startDisplay(() => [
      `Total score: ${score}`,
      'Bowl once and enter your score.',
      'Press 0-9, or X for a strike',
+     gameOverMessage
    ])

    score += inputToNumber(yield waitForChar())
-   yield retry(run(score))
+   yield retry(run(score, balls + 1))
  },
```

I've intentionally mistyped the number of balls needed to
end the game. The test passes despite this, so I think I
need a more specific test.

### Red

We can get the specificity we need by adding a negative
assertion just before the event that is supposed to display
the `GAME OVER` message:

```diff
  'test the game ends after 10 frames of 2 balls each'() {
    const bowlFrames = numFrames => simulator => {
      for (__ of range(1, numFrames)) {
        simulator
          .receive(keyDown('5'))
          .receive(keyDown('5'))
      }
    }

    simulate(run)
-     .do(bowlFrames(10))
+     .do(bowlFrames(9))
+     .receive(keyDown('5'))
+     .assertDisplay(not(contains), 'GAME OVER')
+     .receive(keyDown('5'))
      .assertDisplay(contains, 'GAME OVER')
  },
```

### Green

```diff
  *run(score = 0, balls = 0) {
-   let gameOverMessage = balls > 5 ? 'GAME OVER' : ''
+   let gameOverMessage = balls == 20 ? 'GAME OVER' : ''

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

The `== 20` is a bit sketchy. I think I'll be able to
keep bowling after the 20th ball—and indeed I can.

### Red

```diff
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
+     .receive(keyDown('5'))
+     .assertDisplay(contains, 'GAME OVER')
  },
```

This fails. But I've violated my rule about keeping all the
expected values in a test different. Let's split out
a new test, and move the `bowlFrames` helper to the global
scope.

```javascript
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
```

### Green

```diff
  *run(score = 0, balls = 0) {
-   let gameOverMessage = balls == 20 ? 'GAME OVER' : ''
+   let gameOverMessage = balls >= 20 ? 'GAME OVER' : ''

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

### Red

In order to really assert that I can't bowl after the last
frame, it's not enough to check for the `GAME OVER` message.
I should also make sure my score didn't increase when I
tried to bowl that 11th frame.

```diff
  'test I cannot bowl after the last frame'() {
    simulate(run)
      .do(bowlFrames(11))
      .assertDisplay(contains, 'GAME OVER')
+     .assertDisplay(contains, 'Total score: 100')
  },
```

The test fails: the program thinks the final score should be
110.

### Green

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
      `Total score: ${finalScore}`,
      'GAME OVER'
    ])
    yield wait(Infinity)
  },
```

## Takeaways

Though this isn't the whole kata, it's probably enough for
one post. I've already learned a lot from it, and I hope
it's been at least somewhat entertaining to see my thought
process as I test-drive code.

I honestly didn't expect to be *surprised* by this kata
in any way. I started with a pretty clear picture in my
head of what I thought the program would
end up looking like. But it didn't turn out the way I
planned at all, and I think that's a very good thing.
The code that grew naturally from the user stories and tests
is simpler than anything I would have planned.

The idea to record the score with a single keypress
surprised me in the midst of the kata. I went in
thinking I would have to implement a general-purpose
text-input/number-parsing/validation routine, but of course
that's not necessary for such a simple program.

I like that
the single-keypress solution exactly fits the problem it's trying to solve.
It is simple, casual, almost naïve—but at the same time
very exact. It reminds me of Christopher Alexander's
[quality without a name](https://news.ycombinator.com/item?id=17489023).
You can decide for yourself if the code actually possesses
this quality :)

I also started off thinking I'd have to use some of the more advanced
features of Verse—the [Redux](https://redux.js.org/)-style state container, for one.
But now I see that I can grow the program quite adequately
without it.

Doing the kata also helped me clarify my thinking about
some of the subtle points of TDD. I learned:

- you can start writing a test with just a little bit
  of test code, and keep adding to it until the test's
  implementation matches its description.
- it's only okay to have multiple assertions per test if
  it's easy to tell which one is failing.

And I reaffirmed some things I'd already figured out:

- It's best to refactor a test when it's failing, and
  to refactor production code when all the tests are
  passing.

I also made some mistakes:

- At one point I refactored some production code while a
  test was failing. The danger of doing this is that it's
  harder to notice when you've broken something.
  I could have backed out the test change,
  done the refactor, and re-added the test.
- While extracting the `bowlFrames` test helper, I accidentally
  hardcoded the upper bound of the loop to `10` instead of
  using the `numFrames` parameter. I only realized my
  mistake later, when my tests started acting strangely.
  The change history was convoluted enough that I decided to
  edit it out of this post, but it's a great
  example of how complex test code can cause problems.
