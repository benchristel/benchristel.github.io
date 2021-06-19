---
layout: post
title: JavaScript Objects Without `this` or `new`
---

# JavaScript Objects Without `this` or `new`

It took me more years than I like to admit to fully grok
JavaScript's `this` and `new` keywords, but I don't think
that's entirely my fault: they are two of the most confusing
parts of the language. Fortunately, they're almost never
necessary. You can do object-oriented programming perfectly
well in JavaScript without using `this` or `new` at all.

## The Problems with JavaScript Classes

Avoiding `this` and `new` gets around several of the
disadvantages that come with JavaScript's built-in `class`
keyword. What follows is a whirlwind tour of some of those
problems.

### No Private Methods or State

You can't have truly private methods or instance variables
if you use `class`. Any properties added to an object or
its prototype will be visible to all and sundry.
A common workaround is to prefix private property names with
underscores:

```
class Counter {
  constructor(count) {
    this._count = count
  }

  increment() {
    this._count++
  }

  value() {
    return this._count
  }
}
```

This doesn't block access to the private properties, but
might make people think twice about reaching in and changing
them. Still, I think we can do better.

### Binding of `this`

Quick! If I have the following line of code:

```
const bar = foo.bar
```

Are `bar` and `foo.bar` equivalent in subsequent
expressions? (assuming we don't do anything that changes
foo.bar)

If `bar` is a function, the answer is no! Calling `bar()`
and `foo.bar()` will, in general, do different things. In
the expression `foo.bar()`, the name `this` refers to `foo`
within

This is hard to understand at first for many newcomers to
JavaScript, who naturally assume that

```
foo.bar()
```

is equivalent to

```
(foo.bar)()
```
