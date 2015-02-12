---
layout: post
title: Tests and color
---

It is a hard choice to pick a good test frameworks for C++.
Generally I'm using [Google Test](https://code.google.com/p/googletest/)
and [Boost Test](http://www.boost.org/doc/libs/1_57_0/libs/test/doc/html/index.html).
Both frameworks are mature, support fixtures, complex test cases, expects and asserts (Google Test suited me a little better
because of non-fatal assertion support)

But there is only one feature that stops me from using Boost Test. It can't produce colored output. Seriously, it's 2015
and you must squeeze into raw output to find which test had failed. Do they remember the concept of green bar for unit tests?

It is especially sad that Boost Test has a wonderful extendable output generator (supports both xml and human-readable output) and
there is no sign of any activity for adding colored output. People on boost mailing list advised to write generator on your own.

So I have chosen my testing framework by the coloring support…
