---
layout: post
comments: true
title: "Why Programmer Must Have Math Background"
categories: Thoughts
---

Now I definitely sure that each programmer (or software developer, or software engineer, call he/she as you want) **must** have math background.

Lack of analytical skills will lead to unclear, tangled, repetitious, inconsistent code. 

Let's consider simple example - from web development area (which is believed to be not so complicated, although this is, of course, arguable) - we have several custom controls which are based on same HTML elements which are, in their turn, transformed into aforementioned controls by applying to them some jQuery plugin. Person without, so to say, "pattern recognition" skill will code something like this on different pages: `$("#cars").slider()`, `$("#bikes").slider()` and so on, instead of just writing `$(".slider").slider()`. 

More depressing case is when it comes to design - you'll see lots of similar code snippets (with only slight differences which can be eliminated via parameterizing). Of course, even experienced developers can end up with unclear architecture, but this is definitely easier when they do not have analytical skills which are developed, I strongly believe, while doing math excercises.

Maybe you've already considered me to be Captain Obvious, but, since I still can hear from here and there: "Why do I need to do these stupid exercises in algebra/trigonometry/differential calculus/your option instead of just learning jQuery?" I think this post to some extent can be useful.
