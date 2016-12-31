---
layout: post
title: "Why I Love ES6"
date: 2016-12-03T11:56:38-08:00
author: Ben Christel
keywords: ""
description: ""
categories:
  - Software Development
---

JavaScript is notorious for being a Bad Language. It was [designed by one dude in ten days in 1995](https://en.wikipedia.org/wiki/JavaScript), so you might expect it to have some rough edges. [And it does!](https://www.destroyallsoftware.com/talks/wat) In case you're not currently disposed to watch that video, here are some of JavaScript's most "beloved" quirks:

```javascript
{} + [] // => 0
[] + {} // "[object Object]"
["1", "2", "3"].map(parseInt) // => [1, NaN, NaN]
0 == '0' // => true
0 == '' // => true
'' == '0' // => false
```

But these are not the worst of the traps JavaScript lays for the unwary programmer—they're fairly easy to avoid. For instance, you can dodge the quirks of `==` by using `===` instead, which doesn't do type coercion. It's pretty easy to ensure that you're always using `===` instead of `==` just by searching your codebase. However, JavaScript programs often have structural problems that are much subtler and even more deadly.

- implicit creation of globals
- binding of `this`
- double-equals type coercion
- semicolon (non-)insertion, especially around function calls

```javascript
function validateFoo(foo) {
  (foo > 0) || alert("foo must be greater than 0")
}
```

function validateFoo(foo) {
  var message = buildMessage("foo", "greater than 0")
  (foo > 0) || alert(message)
}

function buildMessage(name, requirement) {
  return name + " must be " + requirement
}


```

- implicit globals
- non-block-scoped variables in loops that create closures

## ES6 to the rescue

Many of JavaScript's biggest problems are addressed by a recent standard, ECMAScript 6, or ES6 for short.



- block-scoping means you never have to use an IIFE - fewer semicolon problems.
- arrow functions bind `this` to the value it had when the function was defined - fewer this-binding problems
