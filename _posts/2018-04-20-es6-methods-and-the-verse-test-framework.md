---
layout: post
title: ES6 Methods and the Verse Test Framework
---

# ES6 Methods and the Verse Test Framework

You're probably familiar with ECMAScript 6's method syntax:

```javascript
let obj = {
  foo() {
    return 'hello from foo'
  },

  bar() {
    return 'hello from bar'
  }
}
```

This is just shorthand for

```javascript
let obj = {
  foo: function () {
    return 'hello from foo'
  },

  bar: function () {
    return 'hello from bar'
  }
}
```

You're also probably aware that object keys can contain
arbitrary characters if you put them in quotes. When written
this way, JavaScript object literals look a lot like JSON:

```javascript
let obj = {
  "a method name with spaces": function () {
    return 'whoa'
  }
}
```

But did you know... that you can combine these things?

```javascript
let obj = {
  'a method name with spaces'() {
    return 'whoa'
  }
}
```

Craziness!

This turns out to work really nicely with the test framework
I'd envisioned for
[Verse](/blog/2018/04/16/verse-not-just-for-poets/). I
wanted to avoid the verbosity of frameworks like Jest and
Jasmine and make tests as syntactically minimal as possible.
I hoped that by minimizing the activation energy required
for testing, I might encourage people to write more tests.

To that end, tests are simply globally `define()`d
functions whose names start with `"test "`.

```javascript
define({
  'test math works'() {
    check(1 + 1, equals, 2)
  },

  'test math still works'() {
    check(2 + 2, equals, 4)
  }
})
```

Isn't that nice?

We can take this *even further* with yet another ES6 feature:
dynamic keys in object literals. In modern browsers, you can
do this:

```javascript
let obj = {
  ['foo' + (1 + 2 + 3)]: function() {
    return 'you called foo6()'
  }
}
```

And, you guessed it, this works with method syntax too:

```javascript
let obj = {
  ['foo' + (1 + 2 + 3)]() {
    return 'you called foo6()'
  }
}
```

This means that if you have a test with an awkwardly long
name, you can just break it into multiple lines:

```javascript
define({
  ['a test with a really really long description that ' +
   'needs multiple lines to properly express itself']() {
    check(1 + 1, equals, 2)
  }
})
```

The Verse test framework isn't ready yet, but soon you'll be
able to write tests like this, right in your browser. Until
then, you can check out [this
demo](https://benchristel.github.io/pico-fermi-bagel/), or
try your hand at writing Verse code at
[druidic.github.io](https://druidic.github.io/)
