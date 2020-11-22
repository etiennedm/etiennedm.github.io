---
layout: post
title: "C Programming Tips and Tricks (and associated rationale)"
date: 2019-05-21 12:00:00 +0200
categories: software c
---
## `do { } while(0)` Macro Construct
The idea is to make a macro which will expand into a regular statement, in order to make the use of function-style macros uniform with the use of ordinary functions. The code wrapped in a `do { } while(0)` is technically a single statement and can for instance be used after an `if` statement without braces. Simply wrapping the macro with braces would not work, since this is a compound statement and adding a semicolon after it would finish the whole `if` statement (and create an orphaned `else` block).

https://stackoverflow.com/questions/1067226/c-multi-line-macro-do-while0-vs-scope-block

## Be Aware of Compiler Optimizations
This article should make you scared! Or at least aware of some optimizations that can mess with code running in different threads (e.g. when using tasks in FreeRTOS): [Who's afraid of a big bad optimizing compiler](https://lwn.net/Articles/793253/).