---
layout: post
title: Don't listen to your tests
---

# Don't listen to your tests

["Listen to your tests"](http://chimera.labs.oreilly.com/books/1234000000754/ch19.html#_listen_to_your_tests_ugly_tests_signal_a_need_to_refactor). Such good advice! If you haven't encountered the phrase before, it means:

> If it's hard to write a test you should reconsider the design of your code.

I wish people took the time to say that instead of just a curt "Listen to your tests" (henceforth _LTYT_). The problem with LTYT is that it's often misconstrued; although it's advice about [TDD](http://wiki.c2.com/?TestDrivenDevelopment), it's incorrect to assume that knowledge of TDD is sufficient to apply it. In order to effectively LTYT, you need a lot of knowledge and experience in the area of software design.

Consequently, when TDD experts say "listen to your tests" to junior developers, what the latter *hear* is often one of these things:

- If I think about my tests hard enough, I'll know how to design my code.
- I know my tests are messy, but I have no idea how to fix them. This is supposed to be obvious. I guess I'm just not smart enough.
- [Oh, I know, I'll use an event bus!](https://grysz.com/2015/11/05/listen-to-your-tests-a-real-example/)

But these interpretations are incorrect! No matter how good you are at red-green-refactor, TDD *by itself* will never tell you what tests to write. It will not tell you what boundary to test at. It will not teach you about the single-responsibility principle or the law of Demeter. TDD can tell you when your design is bad, but it will never tell you which other designs might be good.

Don't listen to your tests! Tests can't talk! They can, at best, emit a high-pitched whine that may induce you to go pick up [_Head-First Design Patterns_](http://shop.oreilly.com/product/9780596007126.do) or some other such resource. When doing TDD, it is perfectly sufficient to *hear* your tests complain, and reserve *listening* for things that can actually teach you how to improve the design of your code.
