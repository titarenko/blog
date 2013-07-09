---
layout: post
title: "What Have I Done During Last Week?"
date: 2013-07-09 23:24
comments: true
categories: git
---

While being in a state of active development, you really can be confused by a question: "What have you done during last week?". You are focused on current task, so obviously you are out of context of previous tasks - this is OK.

Anyway, since you are using Git, it's extremely easy to answer initial question:

```bash
git log --since="2013-06-23" --until="2013-06-29" --author="Constantin Titarenko" --pretty="%ad %s"
``` 

This command will yield something like this:

```
Sat Jun 29 12:51:32 2013 +0300 Fixed spelling.
Sat Jun 29 12:49:31 2013 +0300 Translated comments template to Russian.
Sat Jun 29 12:48:59 2013 +0300 Russian date format.
...
```
