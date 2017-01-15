---
layout: post
title: Isolated Unit Tests
---

In order to properly isolate your tests, you need to destroy the system under test and recreate it for each test that runs. It won't do to simply reset the state. That's too error prone. Correct test teardown doesn't look different from incorrect teardown, so it's hard to verify at a glance that it's correct. It's better to create a new system under test because that will obviously be isolated (as long as it doesn't depend on external mutable state).

Dependency injection gives the reader assurance that the test is truly isolated from global state.
