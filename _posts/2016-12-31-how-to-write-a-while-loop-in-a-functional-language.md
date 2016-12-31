---
layout: post
title: How to Write a While Loop in a Functional Language
---

# How to write a while loop in a functional language

I've been on an [Elm](http://elm-lang.org/) kick recently. I'm trying to learn the language by writing a multiplayer Go game, which, in retrospect, might have been slightly too ambitious. Ah, well.

Learning a functional language is a mind-bending experience if you're used to imperative code. Functional languages have no loops, because changing values is verboten—instead of assigning a new value to a local variable you have to make a recursive call and pass the new value as an argument. In exchange for putting up with such restrictions, functional programs gain a wonderful property, called _referential transparency_, which can make them easier to reason about and test than equivalent imperative programs. However, my brain is still a citizen of imperative-land, and so when thinking about algorithms for my Go game I kept feeling the need for a `while` loop.

Although Elm doesn't have built-in support for `while` loops, it's not very hard to come up with our own `while` function that does the job very nicely. We can analyze the concept of a `while` loop in imperative languages and see how each of its component concepts maps onto a functional language.

In an imperative language like Java, a `while` loop has three main components.

- Some **state**, in the form of variables defined outside the loop body. The loop will update the value of these variables as it runs—that's the only way a loop can have any effect on the rest of the program. Although imperative programs sometimes use empty while loops whose only job is to eat up CPU time, we won't consider that use case here since you'd pretty much never want to spin the CPU in a functional language. Imperative programs can also make use of while loops that create externally-visible side effects instead of changing variables. For example, the following loop (in C) prints "hello world" forever.

```c
while (1) {
  printf("hello world\n");
}
```

Since functional functions cannot have side effects like the `printf` call above, we don't have to think about this use case when translating `while` loops to a functional language.

- A **condition** which determines when the loop is finished. Note that the condition must reference some part of the loop's **state**—if it didn't, the body of the loop could never cause the value of the condition to change, and the loop would run forever. In imperative languages, a while loop's condition might not reference any of the state variables—it might be watching for changes or events triggered by some other process, like a file being written to the filesystem. However, that use case doesn't apply to functional languages because in that universe the value of a function is determined by its arguments, and can't depend on external factors like the filesystem or other processes.
- A **body** which performs some computation and updates the **state**.

# Translating while loops to Elm

Now we can take a stab at writing a `while` function in Elm.

We can represent the body of the loop as a function `state -> state`. That is, the function takes a `state` value as an argument and returns the new state to use for the next iteration of the loop.

We can represent the condition as a function `state -> Bool`. Again, this has a very close parallel in the imperative world.

So what does the concept of state translate to? Well, the state of a loop is just a collection of values, so in a functional type system it could be any type the programmer wants. The only restriction is that the condition and body must agree on what the type of `state` is. In Elm, we can represent this by using a type variable, denoted by a lowercase `state` in the type signature.

```haskell
while : (state -> Bool) -> state -> (state -> state) -> state
```

In plain English: `while` is a function that takes 3 arguments: a condition (a function that takes a `state` and returns a boolean), an initial state, and a body function which takes a state and returns the new state. The return value of `while` is the final state.

The implementation of `while` is only a couple lines of Elm. Once you wrap your head around the functional trick of replacing iteration with recursive calls, it's easy.

```haskell
while condition initialState body =
  if not (condition initialState) then initialState
  else while condition (body initialState) body
```

In English:

- Given a condition, an initial state, and a body function
  - if the condition is false for the initial state, just return the state with no changes.
  - if the condition is true, find the state for the next iteration of the loop by calling the body function. Then return the value that `while` would have returned if the new state had been the initial state.

# Checking the Collatz conjecture

Let's see an example in action. We'll write a program that checks the [Collatz conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture) for any positive integer, and outputs the number of iterations required to reach 1.

The Collatz conjecture is an unproven statement about what happens when you repeatedly apply a certain calculation to a number. The basic idea is this:

- Start with any positive integer. Call it `n`.
- If `n` is even, divide it by 2.
- If `n` is odd, multiply it by 3 and then add 1.
- Repeat until `n` is 1. After that, the sequence becomes boring: a repeating loop of 1, 4, 2, 1, 4, 2....

The sequence of numbers generated by this repeated calculation is sometimes called the "hailstone sequence" because, like a hailstone in a storm, `n` often goes up and down many times before finally falling to 1.

Now, if you pick a positive integer at random and apply the calculation above enough times, you'll eventually reach 1. It works for every number that's ever been tried. But of course that doesn't constitute a proof that this would be true for *any* number, since there could still be a counterexample out there. And that's why the Collatz conjecture is just a conjecture: we don't know how to prove that the sequence always reaches 1. Such a proof is, in the words of mathematician Jeffrey Lagarias, ["completely out of reach of present day mathematics."](https://en.wikipedia.org/wiki/Collatz_conjecture#cite_note-10)

```haskell
collatz : Int -> Int
{-| Returns the number of steps necessary for the hailstone
  | sequence starting at `start` to reach 1.
  -}
collatz start =
  let
    nIsNot1 = \{n} -> n /= 1
    isEven = \n -> n % 2 == 0
    initialState =
      { n = start
      , iterations = 0
      }

    result = while nIsNot1 initialState (\{n, iterations} ->
      if isEven n
      then
        { n = n // 2
        , iterations = iterations + 1
        }
      else
        { n = 3 * n + 1
        , iterations = iterations + 1
        })
  in
    result.iterations
```

In my opinion, the `while` function produces code that is quite readable. The body function reads almost like an imperative procedure, even though we're not actually mutating any variables! The only thing that shows this program's true functional nature is that the while loop must explicitly state which "variables" in the `collatz` function it depends on, by listing them as parameters to the body function. That makes for a bit more typing, but it does make it easy to see at a glance what the while loop is doing.

Note that in trying to keep the code simple, I've created a bug: the while loop runs forever if `collatz` is called with `0` as the starting number. There may also be negative inputs that cause infinite loops. You'd probably want to fix this if you're writing a real program that uses hailstone numbers.
