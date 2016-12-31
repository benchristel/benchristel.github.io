---
layout: post
title: "Simple JavaScript Builds"
date: 2016-12-03T11:55:34-08:00
author:
keywords: ""
description: ""
---

One of the reasons I love JavaScript is that it's so easy to get a new project up and running—easy, that is, if you're not doing it professionally. For tiny projects, you can slap an inline script into an HTML page and call it a day. If you're building a large web application, you probably want something like `webpack` or `browserify` to build your myriad JavaScript source files into a single über-file that can be easily referenced by your HTML.

```javascript
const SomethingSomething = require('../../src/lib/my-namespace/SomethingSomething.js')

describe('SomethingSomething', function() {
  // ...
})
```


# Problems Solved by Build Systems

Build systems in general exist because developers prefer code to be structured one way, and computers prefer code to be structured a different way. The build system's purpose is to convert the _source code_ that developers work with into a format that's convenient for the computer.



- JS code for the web must ultimately be loaded into an HTML page with `<script>` tags. This means that if you want to break up your JavaScript code into multiple files, you have to reference each of them from your HTML.
- To avoid having to change the HTML every time a script changes, it's convenient to "build" all the JS source files into a single file which can be referenced in the HTML. The simplest way to do this is to just concatenate all the files.
- Scripts may have dependencies on other scripts. They can reference those dependencies in a few different ways:
  - references to global variables (requires the order of the files to be determined so variables are defined before they're used)
  - references to the files themselves, which are then resolved by something like `browserify` or `webpack`
  - dependency injection
