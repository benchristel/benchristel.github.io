---
layout: post
title: Test-driving the Y Combinator
---

# Test-driving the Y Combinator

I've always found the Y Combinator to be one of computer
science's most mind-boggling mysteries. No, not Y Combinator
the startup accelerator; I'm talking about the [Y
combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator) of functional programming. **`Y` is a function
that lets you express any recursive computation as
an anony&shy;mous function.** For example, the following JavaScript
computes the factorial of 10:

```javascript
Y(function(factorial) {
  return function(n) {
    if (n === 0) return 1
    return n * factorial(n - 1)
  }
})(10)
```

While it's seldom *useful* in practice, the Y combinator has
gained a reputation for being one of those arcane functional
programming concepts that only wizards and geniuses
understand. This reputation probably stems from its
incomprehensibly compact definition, attributed to
[Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry).
Here it is in JavaScript:

```javascript
Y = f => (x => f(x(x)))(x => a => f(x(x))(a))
```

If you try to puzzle out how this `Y` function works, you'll
quickly realize just how bizarre it is. I've made several
attempts in the past to unravel the mystery of `Y`, but
always got stuck. This code, brief as it is, seems simply to
be beyond human comprehension.

It's really not, though! In this post, I hope to thoroughly
demystify the Y Combinator through a combination of
test-driven development and refactoring. I'll write a simple
test for `Y` and some simple code to make it pass. Then I'll
refactor the code in a series of small, behavior-preserving
steps until it's identical to the definition above. I hope
that by following along, you too will see that `Y`'s not so
scary.

That said, this post does assume you have some
experience with functional programming. I'll assume you're
already comfortable with anonymous functions, recursion, and
closures.

## Why is Y?

The Y Combinator lets us do something we otherwise couldn't
do in a purely functional language: define an **anonymous
recursive function**.

At first glance, it's not obvious why this requires a
special technique. If we know how to write an anonymous
function and how to define an algorithm recursively, why
can't we do both at once?

For example, this works:

```javascript
const factorial =
  function(n) {
    if (n === 0) return 1
    return n * factorial(n - 1)
  }
```

But it's cheating by declaring a `const`. This declaration
has a side effect: before it happens, `factorial` is not
defined; afterward, it is. Our goal is to define this
recursive function in a way that has no side-effecting
statements like declarations, because if we want to write
code that's truly [functionally pure](https://en.wikipedia.org/wiki/Pure_function), side effects aren't allowed.

Specifically, we'd like to have a function `Y` that
lets us compute the factorial of `n` using an expression
of the form `(Y(...))(n)`. We should also be able to use `Y`
to express other recursive computations, not just
factorials.

<!--
## When would you ever need this?

The Y combinator isn't exactly the most useful thing. The
only situation in which I can imagine ever reaching for it
is if I'm in the middle of a functional pipeline of `map`s
and `filter`s and `reduce`s and I need a one-off anonymous
function that *just happens* to have a convenient recursive
definition.

For example,

```javascript
const factorials = [1, 2, 3, 4].map(Y(...))
```

We'd like to replace `...` with an expression that makes
this whole statement compute the first four factorials. To
do that elegantly, we'll need a Y combinator.
-->

## A Failing Test

I'm going to write all the tests and code in
[Verse](https://verse.js.org/2.0.0-alpha.0), my
browser-based development environment. Feel free to follow
along—it's probably the best way to get a feel for what's
actually happening in the code.

We'll start with this test, which demonstrates how we want
to use `Y`:

```javascript
define({
  'test anonymous factorial function'() {
    let firstFourFactorials =
      [1, 2, 3, 4].map(Y(function(recurse) {
        return function(n) {
          if (n === 0) return 1
          return n * recurse(n - 1)
        }
      }))

    assert(firstFourFactorials, equals, [1, 2, 6, 24])
  },
})
```

What's going on here?

The function `Y` is helping us define a recursive factorial
function.

`Y` gives us a function `recurse` that we can call when we
want our factorial function to call itself. We receive
the reference to `recurse` via a callback:

```javascript
  Y(function(recurse) {
    // ...
  })
```

Within that
callback, we can define our anonymous factorial function
in terms of `recurse`:

```javascript
    function(n) {
      if (n === 0) return 1
      return n * recurse(n - 1)
    }
```

Finally, we `return` our factorial
definition from the callback so that `Y` knows what to do
when we call `recurse`.

```javascript
  Y(function(recurse) {
    return function(n) {
      // ...
    }
  })
```

## Making the test pass

Here's a first attempt at defining `Y`. This passes
the test:

```javascript
define({
  'test anonymous factorial function'() {
    // ...
  },

  Y(definer) {
    function recurse(arg) {
      return recursiveFunc(arg)
    }
    const recursiveFunc = definer(recurse);
    return recursiveFunc
  },
})
```

`Y` takes in a function named `definer`. In our
test case, the `definer` we're passing in is:

```javascript
function(recurse) {
  return function(n) {
    if (n === 0) return 1
    return n * recurse(n - 1)
  }
}
```

`Y` needs to pass a `recurse` function to the `definer` so
our factorial function can call itself. So the first thing
`Y` does is to define the `recurse` function. `recurse` just
calls `recursiveFunc`, which in our test case is the
factorial function returned by the `definer`.

I've intentionally chosen variable names that are quite
abstract to highlight the point that `Y` isn't specific
to factorials, but to clarify what's going on we can also
give these concepts factorial-specific names:

```javascript
Y(createFactorialFunction) {
  function recurse(n) {
    return factorial(n)
  }
  const factorial = createFactorialFunction(recurse);
  return factorial
}
```

## Comparison with Curry's Y combinator

The code we've written works, and it's probably the easiest
way to write a Y combinator in JavaScript. However, our
goal isn't just to get a working Y combinator, but to
understand the magic of [Haskell Curry's definition](https://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator):

```javascript
Y = f => (x => f(x(x)))(x => a => f(x(x))(a))
```

One reason Curry's Y combinator is so obtuse is that it's
defined in the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus),
a formal mathematical system for expressing computation. You
can think of the lambda calculus as being like a functional
programming language with some additional, rather severe
restrictions:

- A function must have exactly one parameter.
- No variables or constants (other than the parameters of
  functions) may be declared.

Looking at the above JavaScript translation of Curry's Y
combinator, we can verify that it adheres to these
restrictions and thus is easy to express in lambda calculus.
We can also see that the code we've written for `Y` is *not*
lambda-calculus compatible. It defines two names
that aren't parameters: the `recurse` function and the
`recursiveFunc` constant.

The rest of this post will be dedicated to refactoring our
Y combinator until it looks like Curry's definition.

## Refactoring toward Curry's Y combinator

Currently, our code looks like this:

```javascript
Y(definer) {
  function recurse(arg) {
    return recursiveFunc(arg)
  }
  const recursiveFunc = definer(recurse);
  return recursiveFunc
}
```

Our goal is to eliminate the declarations of `recurse` and
`recursiveFunc`.

The `const recursiveFunc` statement is easy to get rid of.
We can use the principle of [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency) to replace the references
to `recursiveFunc` with its definition:

```javascript
Y(definer) {
  function recurse(arg) {
    return definer(recurse)(arg)
  }
  return definer(recurse)
}
```

Now we just need to get rid of the declaration of `recurse`.
That's not so easy, though, because `recurse` references
itself.

Fortunately, there's a non-obvious trick we can use to turn
`recurse` into an anonymous function. First we have to
introduce some seemingly pointless indirection, defining a
function `createRecurse` that returns an anonymous version
of our `recurse` function.

```javascript
Y(definer) {
  function createRecurse() {
    // returns the function formerly known as `recurse`
    return function(arg) {
      return definer(recurse)(arg)
    }
  }
  const recurse = createRecurse()
  return definer(recurse)
}
```

At first glance, it seems we've made things worse. Now we
have not only a named function, `createRecurse`, but a
constant defined for `recurse` itself.

But look what happens next: we can use
[referential transparency](https://en.wikipedia.org/wiki/Referential_transparency) again to replace the references to
`recurse` with calls to `createRecurse()`. This lets us
delete the `const` declaration.

```javascript
Y(definer) {
  function createRecurse() {
    // returns the function formerly known as `recurse`
    return function(arg) {
      return definer(createRecurse())(arg)
    }
  }
  return definer(createRecurse())
}
```

Then, we can avoid referencing the `createRecurse` name inside
`createRecurse`, by passing `createRecurse` to itself. I've
added an underscore to the parameter name to make it clear
that `createRecurse` references only its parameter, not its
own name.

```javascript
Y(definer) {
  function createRecurse(createRecurse_) {
    // returns the function formerly known as `recurse`
    return function(arg) {
      return definer(createRecurse_(createRecurse_))(arg)
    }
  }
  return definer(createRecurse(createRecurse))
}
```

Having eliminated the self-reference, we can make
`createRecurse` anonymous. As an intermediate step, we'll
define it using anonymous function syntax and assign it to a
constant `x`:

```javascript
Y(definer) {
  const x = function(createRecurse) {
    // returns the function formerly known as `recurse`
    return function(arg) {
      return definer(createRecurse(createRecurse))(arg)
    }
  }
  return definer(x(x))
}
```

Then, relying on referential transparency again, we can
inline the constant `x`, replacing references to it with
its definition:

```javascript
Y(definer) {
  return definer(
    function(createRecurse) {
      // returns the function formerly known as `recurse`
      return function(arg) {
        return definer(createRecurse(createRecurse))(arg)
      }
    }(function(createRecurse) {
      // returns the function formerly known as `recurse`
      return function(arg) {
        return definer(createRecurse(createRecurse))(arg)
      }
    })
  )
}
```

Finally, we should do something about these statements:

```javascript
return function(arg) {
  return definer(createRecurse(createRecurse))(arg)
}
```

The anonymous `function(arg)` here is needless indirection,
since its argument is just passed along to another function.
We can remove it:

```javascript
Y(definer) {
  return definer(function(createRecurse) {
    // returns the function formerly known as `recurse`
    return definer(createRecurse(createRecurse))
  }(function(createRecurse) {
    // returns the function formerly known as `recurse`
    return definer(createRecurse(createRecurse))
  }))
}
```

Oops! This code results in an `InternalError: too much
recursion` on Firefox. As it turns out, we *do* need the
indirection of one of those anonymous functions because
otherwise the `createRecurse` function calls itself
recursively forever. By adding the indirection, we're making
`createRecurse` lazy, so it only generates a `recurse`
function when one is required.

Here's the code we end up with:

```javascript
Y(definer) {
  return definer(
    function(createRecurse) {
      // returns the function formerly known as `recurse`
      return definer(createRecurse(createRecurse))
    }(function(createRecurse) {
      // returns the function formerly known as `recurse`
      return function(arg) {
        return definer(createRecurse(createRecurse))(arg)
      }
    })
  )
}
```


We now have a Y combinator that is fully lambda-calculus
compatible.

In fact, if we rename the parameters and
use ES6 arrow function syntax, our code can be written as

```javascript
// renames:
//   definer       --> f
//   createRecurse --> x
//   arg           --> a

Y: f => (
     x => f(x(x))
   )(
     x => a => f(x(x))(a)
   )
```

which translates almost exactly to [Haskell Curry's
formulation of the Y combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator) in the lambda calculus:

```
Y = λf.
  (λx.
    (f (x x)))
  (λx.
    (f (x x)))
```

The only difference is that in our version, we've had to
preserve a layer of indirection to prevent infinite
recursion, as described above. We have to do this because
JavaScript, like most languages, uses
[_applicative evaluation
order_](https://people.csail.mit.edu/jastr/6001/spring06/r23.pdf).

## Conclusion

I honestly still can't follow the code we ended up with, despite
having written it. However, we took small enough steps when
refactoring that I'm confident the behavior hasn't changed
from the original implementation. I hope you have similar
confidence, even if you don't grok the final code.

I'd also like to draw attention to the techniques we used in
this post. I honestly didn't think when I started this
exercise that it would be possible to test-drive something
as arcane as the Y combinator, much less to refactor it from
a naïve implementation to one resembling an expression in
the lambda calculus. And yet, we did it! I hope this
exercise serves to showcase the power and versatility of
test-driven development and refactoring, and gives you some
hope that you can apply these techniques to your own work,
even when the going gets tough.
