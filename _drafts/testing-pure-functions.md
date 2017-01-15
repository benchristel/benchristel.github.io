---
layout: post
title: Testing Pure Functions
---

Pure functions always have a range of outputs that is smaller than (or the same size as) their domain of inputs. (Mathematical notions of countable infinities don't apply here—nothing in computing is really infinite). Many inputs may produce the same output, and you cannot (in general) derive a unique input given the output of a function.

Take, for example, the following `square` function:

```javascript
let square = x => x * x
```

There are a few reasons why one pure function might call another. Let's say there's a function `a` that calls `b`.

- `a` might do some computation that is not solely a function of its own arguments to determine the arguments to pass to `b`.
- `a` might pass `b`'s return value to another function.

Any nontrivial function `a` must do one of the above. If it does neither, that means that one of the following is true:

- `a` is a middle man: it is just passing its parameters to `b` and returning `b`'s return value
- `a` composes its parameters in some way to compute the arguments to `b`. This violates the single-responsibility principle
