---
layout: post
title: Test Doubles in C
---

# Test Doubles in C

## The Problem

## Creating a test seam

The first thing we have to do to make this code testable is to pull out the part we want to test into a function that's not `main()`. Our test code is going to have its own `main()` function that runs the tests, and we don't want them to conflict.

A side effect of this is we can't test our production `main()` function. Therefore, `main()` should do as little as possible; ideally it will just call another function and return its result.

Here's how the code looks after extracting the loop and return code into a `hello.c` file.

**`hello.h`:**

```c
#ifndef HELLO_H
#define HELLO_H

int hello();

#endif
```

**`hello.c`:**

```c
#include "hello.h"
#include <stdio.h>

int hello() {
  for (int i = 0; i < 10; i++) {
    printf("Hello, world!\n");
  }
  return 0;
}
```

**`main.c`:**

```c
#include "hello.h"

int main() {
  return hello();
}
```

We can build this code with `gcc main.c hello.c -o main.o`, and then run `./main.o`.

## Writing the first test

Now we can write a test that checks the return value of `hello()`.

**`test.c`:**

```c
#include "hello.h"

int main() {
  int returned = hello();

  if (returned != 0) {
    printf("Expected return code to be 0, but was %d", returned);
    return 1;
  }

  return 0;
}
```
