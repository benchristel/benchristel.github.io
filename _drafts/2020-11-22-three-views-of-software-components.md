---
layout: post
title: Three Views of Software Components
---

# Three Views of Software Components

We mentally organize code by analyzing it into components.
In this post, I'll talk about how I approach
this analysis, and the three main types of
components I've observed in software. These
correspond roughly to the three major programming paradigms
(procedural, object-oriented, and functional) so this
discussion will hopefully shed some light on the question of
when and how to apply each paradigm.

## What's a Component?

A software component...

- encapsulates some behavior
- has a boundary that separates it from other
  components.
- has a means of exchanging information with other
  components.


This definition is pretty abstract, so let's look at some
examples.

### A Simple Example

At the extreme of simplicity, a component can be as small as
an expression:

```
X + Y
```

This expression has behavior (addition), a means of taking
in information (the variables X and Y), and a means of
emitting information (the return value). There is a clear
boundary separating it from any neighbors it might have—the
syntactic beginning and end of the expression.

Note that the choice of boundary is indeed a _choice_. We
could have chosen to consider a somewhat larger unit of code
as our component of interest:

```
X = 0
for each Y in LIST:
  X = X + Y
```

When we draw a boundary around this whole loop, the variable
`LIST` is the input, and `Y` is an internal implementation
detail. `X` is an output, but is no longer an input. The
overall behavior is _summation_.

### A Complex Example

As an example of a more complex software system, consider a Git repository
and the `git` executable that manages it. This system takes
in input in the form of command line arguments and files in
the working tree of the repo, and outputs text (to standard
output) and updated files.

Note that here, too, we could have drawn the boundary
differently: we could consider just the `git` executable
itself as a software component, in which case the working tree
and the `.git`
repo directory would also be inputs. The choice of what is
inside or outside of the component boundary is ours to make.

## Recursive Structure of Software Components

Most software components are made up of other software
components. This should be more or less obvious from the
examples I've given above, but I wanted to call it out
explicitly.

## Components are an Interpretation, Not the Thing Itself

Because we are free to choose different divisions of a
system into components, it should be clear that components
don't exist as objective things that are "out there" in the
external world. Components are an
_interpretation_ of the system.
Often, we'll switch between different interpretations many
times during a programming task, sometimes so rapidly that we seem to
see two views of the system at once. I've found it very
useful to consider multiple different views when looking at
a complex system.

Additionally, some divisions into components may be "wrong",
but still useful. For example, in the `git` example above, I
failed to consider an important behavior of `git`:
interactions with other repositories, which might
communicate over the network! Still, my simplified view of
`git` is useful when thinking about commands that _don't_
need network access.

## But Some Things Are Objectively More "Componenty" Than Others

While we are free to divide software into components however
we want, some pieces of software systems are markedly easier
to think about in isolation than others. They may, for
instance, have a more clearly denoted boundary, perhaps even
taking advantage of special syntax:

```
function add(x, y) {
  return x + y
}
```

Various other factors contribute to the coherence of
components:

- Components must exchange information with their
  environments, but exchanging _too much_ information
  weakens coherence—imagine a function with 50 parameters!
  On the other hand, components that
  exchange _too little_ information are not very useful:
  a function that takes _no_ parameters and just does one
  hardcoded thing can't easily be reused for novel tasks.

So, in summary, we're "free" to choose how we analyze
the components of software in the same way we're "free" to
choose what videos we watch on YouTube: while any choice is
theoretically possible, the inherent shape of the system
will always nudge us toward particular interpretations.

## The Three Types

I titled this post "Three Types of Software Components", yet
we're 700 words in and I've yet to provide you with a list
of three things. Sorry about that.

The types are:

1. Stateless Functions
2. Mechanisms
3. Agents

In the following sections, I'll discuss each of these in
detail.

## Stateless Functions

![a diagram showing a half adder constructed from NAND gates](/assets/logic-gates.jpg)

<cite style="font-size:11px">"Half adder using NAND gates only" by Nitianabhigyan is licensed with CC BY-SA 4.0.</cite>

The simplest components are _stateless_: given the same
information as input, they always produce the same output.
Pure functions in software need no introduction; however
it's perhaps worth noting that the fundamental building
blocks of computer hardware are also stateless components:
AND, OR, and NOT gates all encode functions from input
voltages to output voltages.

## Mechanisms

A _mechanism_ takes the concept of a function and adds
state to it. A repeated interaction with a mechanism
may produce different results each time depending on
previous interactions.

![a gumball machine](/assets/gumball-machine.jpg)

An example of a mechanism is a gumball machine. You put in
a quarter, it spits out a gumball. In the process, its state
changes—it gains a quarter and loses a gumball. If you
repeat the interaction, you'll get a different color gumball
each time (or you may get nothing—eventually the machine
will run out of gumballs).

Mechanism-like components tend to be harder to reason about than
stateless functions because their internal state adds
information to the system—and is often invisible. Still,
they can be an intuitive way of organizing systems in a
variety of situations.

For example, suppose we want to write a program that
lists the 50 most frequent words in a stream of text. The
text stream may be too big to fit in memory all at once, so
the program has to be able to process it a page or so at a
time.

At the heart of this program, I can imagine a mechanism-like
component with the following interface:

```
{
  method receive(String)
  method mostFrequentWords() => List<String>
}
```

When this component receives a chunk of text, it extracts the words
and updates its table of running counts for each word. Once
the whole stream has been processed, we can ask the
mechanism for the top 50 words.

### Mechanisms vs. Stateless Functions

There is a rather simple mapping from mechanisms to pure
functions. If we represent the state of the mechanism as
an immutable data type `State`, we can take the mechanism apart
and expose its innards:

```
function update(State, Message) => (State, Response)
```

That is, we can write an `update` function that takes the
current state and a message and returns the new state along
with the mechanism's response (i.e. the gumball or
whatever). This can be made to behave just like the old
mechanism, _as long as we make sure to pass the new state
into the next invocation of `update`_. If we mess up and
pass an old version of the state instead, the program won't
behave correctly.

And there (to quote Hamlet) 's the rub.
It is now _the caller's_ responsibility to ensure
that the state is shepherded from one invocation of `update`
to the next. The burden of that responsibility may not be
worth the flexibility we gain by taking the mechanism apart.

In making the choice between functions and mechanisms, we
need to consider the coupling between state and behavior
that mechanisms add and ask ourselves if it helps or hurts
the comprehensibility of the system as a whole. Yes, you
read that correctly—it may
come as a surprise, but coupling can actually be a good
thing! Well-designed coupling can help us enforce guarantees
that keep the system consistent and secure. See
[Michael Nygard's talk "Uncoupling"](https://www.youtube.com/watch?v=esm-1QXtA2Q) for more.

## Agents

An agent is an entity that _does stuff_. It reaches out to
other components, it modifies state external to itself, and
it (often) keeps track of time and does things on a
schedule. It is a mover and/or a shaker. While mechanisms
are passive, and don't do anything until you send them a
message, agents are active and do stuff whenever they feel
like it.

A simple example of an agent is an alarm clock. You tell it
a number of hours and minutes, and at some point in the
future it screams you into wakefulness. In the interim, you
don't have to do anything to it: it independently
counts down the seconds until your next relapse into
consciousness.

The behavior of agents is, in general, only describable in
code as a step-by-step algorithm: do this, then do that, then do
the other thing. Agents march to the beat of their own
drummer, and, like music, they are inherently sequential.
Since their interactions with other components might change
the state of those components, the order and timing of the
interactions matter.

The obvious advantage of agent-like components is that they
are powerful: they can do things the other types of components
can't, like take the initiative to talk to other components
and react to the passage of time. But more to the point, agents are
_necessary_. It is simply not possible to construct a system
that functions as an alarm clock without creating an
agent-like component.

That said, agents come at a cost: their
sequential nature and promiscuous interaction with other
components make their behavior hard to describe. If you've
ever tried to unit-test a bunch of procedural code, you'll
know what I'm talking about. And if you have _multiple_
agents, running in parallel? You can pretty much forget
about deriving your confidence in the system from unit
tests!

## The Main Difference Between the Three Types

By now it should be clear that the three types of
software components differ mainly in _what they are allowed
to do_—that is, their _capability_. Stateless functions are the
most limited in what they're allowed to do. Mechanisms add
the ability to remember and update state. And agents add the
ability to proactively communicate with other components.

## The Principle of Least Power

The [Principle of Least Power](https://en.wikipedia.org/wiki/Rule_of_least_power) states that

## Mapping to Programming Paradigms

Perhaps this all sounds familiar, and it should—this whole
time we've basically been talking about the three major
programming paradigms (functional, OO, and procedural) by
different names.

Note, however, that the implementation techniques preferred
by each paradigm
that are abstracted away by the component-based view. We
haven't touched at all on functional programming's
enthusiasm for functions as first-class values, nor
procedural programming's wealth of control-flow structures.
And saying that object-oriented programming is about writing
passive, mechanism-like components might well seem strange to many
Java programmers, who dwell in a twilight land of
"active objects" that spawn parallel threads and tangle
with Time.

But that's okay, because this post isn't about programming
paradigms. It's about analyzing programs into components
and categorizing them according to the most convenient way
of thinking about them.

In an object-oriented program, some objects will be agent-like
components and some will be function-like components, and
that's fine. That's part of how the Object-Oriented paradigm
works.

Similarly, most Haskell programs contain functions that use
IO monads with `do` notation and look a lot like agents.
It arguably goes against the paradigm of functional programming
to think of them as agents, but it is often _useful_ to
think of them that way.

## A Practical Example

alarm clock:

```
hourInput.addEventListener("change", setAlarm)
minuteInput.addEventListener("change", setAlarm)

snoozeButton.addEventListener("click", () => {
  if (alarmIsSounding) {
    silenceAlarm()
    setTimeout(soundAlarm, 9 * MINUTE)
  }
})

offButton.addEventListener("click", () => {
  if (alarmIsSounding) {
    silenceAlarm()
  }
})
```

```
function AlarmClock(now) {
  let alarmHour
  let alarmMinute

  return {
    tick,
    setAlarm,
    isAlarmSounding,
  }

  function tick(currentTime) {
    now = currentTime
  }

  function setAlarm(hour, minute) {
    alarmHour   = hour
    alarmMinute = minute
  }

  function isAlarmSounding() {
    return (
      now.getHours() === alarmHour
      && now.getMinutes() === alarmMinute
    )
  }
}
```

```
function Buzzer() {
  const audio = new AudioContext()

  // create a sine waveform
  const oscillator = audio.createOscillator()
  oscillator.frequency.setValueAtTime(1760, audio.currentTime)

  // create a volume control
  const volume = audio.createGain()
  volume.gain.setValueAtTime(0, audio.currentTime)

  // wire up the waveform -> volume -> speakers
  oscillator.connect(volume)
  volume.connect(audio.destination)

  // start the oscillator (silent initially, since volume
  // is zero).
  oscillator.start();

  return {
    setVolume,
  }

  function setVolume(v) {
    volume.gain.setValueAtTime(v, audio.currentTime)
  }
}

const buzzer = Buzzer()
const clock = AlarmClock(new Date())

setInterval(() => {
  clock.tick(new Date())
  buzzer.setVolume(clock.isAlarmSounding() ? 1 : 0)
}, 1000)

clock.setAlarm(7, 45)
```

agent (hard to test)

agents for buttons, clock, and alarm. Mechanism for logic.
Pure functions for view.

## But What Does It All Mean?

If I had to boil down the main points of this post into
practical advice, I'd say: get good at writing mechanisms.



To be honest, I _would_ have written a post about
programming paradigms, except I don't think I could have written
anything cogent about Object-Oriented Programming, because
no one seems to agree on what Object-Oriented Programming
_is_.

Indeed, Object-Oriented Programming stands out as the
paradigm whose conceptualization in the zeitgeist is
scarcely recognizable from what I've said about components.
But I think this
mismatch is due to the fact that Object-Oriented Programming
has, since its conception, decayed—from a simple, practical,
and ingenious set of ideas for how to think about certain
parts of programs, to an incomprehensible mishmash
that is frequently (and deservedly) the, er, [_object_ of
ridicule](https://www.youtube.com/watch?v=QM1iUe6IofM).

To me, though, it seems that both the critics and supporters
of OOP have largely missed the point. The rest of this post
will attempt to present, and hopefully
restore, what I see as the vital core of Object-Oriented
Programming as distinct from procedural and functional
programming.

## Object-Oriented vs. Functional Programming

### Collecting Values From Across Time and Space

If you're building up a list in an OO language, you don't
have to reference all the elements of the list in the same
place. Different parts of the program can all put things
in the list.

Test discovery example.

### Decentralization

generalizing from previous point...

### Judicious Coupling

## Object-Oriented vs. Procedural Programming

### Islands of State

### The Conquest of Time

### Self-Similarity and Emulation

## Red Flags

active objects

"collaborators", dependency injection

<!--
## Where OOP Went Wrong

I mentioned earlier that agent-like components—and the
procedural, sequential style of code that necessarily powers
them—are essential to building certain types of systems.
Any general-purpose programming language, then, must provide
_some_ way of describing procedures. Since OO languages
typically have imperative semantics, representing procedures
is straightforward

The only widely-used
language I'm aware of that attempts to do away with
procedures entirely is Elm, which represents all side effects
as data returned from the program to the language runtime.
(The runtime executes the side effects on the program's
behalf). However, Elm's approach doesn't scale well to
situations where different side effects must be chained
together in complex ways, interleaved with conditionals,
error handling, and data transformation. In such situations,
the program's `State` datatype must assume the cumbersome
task of representing the state of an ongoing computational
process, complete with variables, instruction pointer, and
stack frames.

on mechanism-like components that accept messages and return
responses.

## Mechanisms are Not Agents

Mechanisms are self-contained. This point is crucial.


-->
## Centers

I'd be remiss if I didn't mention Christopher Alexander's
concept of centers in this post. His definition of a center
in architectural design maps very closely to my definition
of a component in software, and was one of the inspirations
for this post. I highly recommend reading the first volume
of _The Nature of Order_ or Richard Gabriel's excellent
summaries to learn more about this concept.
