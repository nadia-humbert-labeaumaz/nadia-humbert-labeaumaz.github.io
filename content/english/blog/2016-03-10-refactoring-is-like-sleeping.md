---
type: "post"
title: "Refactoring Is Like Sleeping"
slug: "refactoring-is-like-sleeping"
date: 2016-03-10 15:12:42 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/refactoring-sleeping/header.jpg"
bg_image: "images/blog/refactoring-sleeping/header.jpg"
categories: ["Software"]
tags: ["Refactoring"]
draft: false
comments: true
---

Bob is a developer. He has been asked to add a brand new feature to the application. So, he would like to take this opportunity to refactor the code a little bit since it doesn't respect the good practices that he learnt from the last Software Craftsmanship meetup. However, Bob was told that he couldn't do it because there was not enough time to do anything other than producing features. So, Bob thought, "there is never enough time anyways"...

Sounds familiar? Indeed, refactoring is often seen as an activity without any value. Other things in life can also seem worthless. There is one that takes almost a third of our lives during which we do nothing literally: *sleeping*!

<!--more-->

The number of things that we could do if we didn't sleep! However, after a few days, even hours, unpleasant events would eventually occur. Without sleep, the most trivial tasks can become very difficult to accomplish, and weird mistakes can take place (like pouring orange juice into your bowl of cereal). After a while, sleep would suddenly take over, leading to accidents (because of drowsy driving, for example).

In the same way, postponing refactoring harms a project. Developments may take much longer than necessary, and bugs may occur in features very apart from those modified. Moreover, there is a moment where refactoring imposes itself unexpectedly: code can no longer be modified without risking important regressions. This can happen at a critical time of the project and thus jeopardize it. At this point, the feared verdict may be rendered: "we must rewrite everything."

In general, regular refactoring is necessary to a project's good health, just like sleeping every night is for ours. Thereafter, we will define refactoring, when it is good to do it, how to prepare it and what to do if time is missing.

## What does refactoring mean?

According to Michael Feathers, refactoring is "the act of improving design without changing its behaviour."

It is not necessarily about changing a whole class hierarchy or implementing a complex design pattern, but it could be as simple as renaming a variable, method or class, extracting a private method in an external class, gathering the attributes of a method into an object, etc.

Furthermore, we also consider adding tests to an existing codebase as refactoring. Indeed, a testable code is the first step towards a better design.

## When to refactor?

The idea is not to try to fix the entire system every time. This would be unproductive and impossible to carry out. Also, it would be difficult to justify causing regressions in a portion of code unrelated to the one that was supposed to be changed.

Refactoring is very efficient when it is targeted at the code related to the development of a feature. Moreover, it is preferable to refactor before adding new code to start on a good basis.

## How to prepare it?

The first thing to do before refactoring is to ensure that tests cover the code to be modified. Tests permit to verify that refactoring doesn't generate regression. If tests are missing, you need to add them before starting.

Sometimes it might be impossible to test a portion of the code. In this case, you can perform the minimal refactoring required for implementing the tests.

## What to do if time is missing?

If you lack the time, like Bob, you must be pragmatic and do small refactoring for each development. This allows to improve the design without consuming too much time and prepare the ground for a larger refactoring.

## Conclusion

Refactoring is essential in a project and should be done regularly. It ensures new features are developed within a reasonable timeframe and limits regressions by improving the design. Furthermore, it allows developers to (re)take pleasure to make the product evolve.

Let's conclude with the boy scout rule that inspires us to always leave the code in a better condition than when we found it. Applying this rule leads to improving the overall quality of the code and to the reversal of technical debt.
