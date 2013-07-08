---
layout: post
comments: true
title: "Tracing C++ Code Execution"
categories: C++ Snippet
---
Sometimes it's quite handy to see what is being executed and how much time does it take. OK, so lets define several helpers in order to have that thing (here I assume you do not have advanced tools for profiling).

```cpp
#define TRACE_BEGIN_EX(what, watch) \
	std::clog << what << std::endl; \
	watch.restart()

#define TRACE_END_EX(what, watch) \
	watch.stop(); \
	std::clog << what << \
	" Time: " << watch.getElapsedSeconds() << "s." << std::endl
```

Have you spotted that parameter `watch`? It is an instance of the following class.

```cpp
/*!
	Represents stopwatch for measuring code execution time.
*/
class Stopwatch
{
public:
	/*!
		Initializes and starts stopwatch.
	*/
	Stopwatch();

	/*!
		Clears elapsed time value and starts stopwatch.
	*/
	void restart();

	/*!
		Stops stopwatch.
	*/
	void stop();

	/*!
		Returns number of elapsed seconds since start.
	*/
	numeric_t getElapsedSeconds() const;

private:
	clock_t _begin;
	numeric_t _length;
};
```

If you wonder what is `numeric_t` - this is nothing but simple alias of `float`. So, lets see an example of usage (excerpt from the source code).

```cpp
...
TRACE_BEGIN_EX("Application started (MPI process #"
		<< _processIndex << ").", _watch);
...
TRACE_END_EX("Application finished (MPI process #"
		<< _processIndex << ").", _watch);
```

It will be more useful if we'll introduce few new macros.

```cpp
#define TRACE_PREPARE \
	Stopwatch __trace_stopwatch; \
	std::ostringstream __trace_ostream; \
	std::string __trace_subject

#define TRACE_BEGIN(what) \
	__trace_ostream.str(std::string()); \
	__trace_ostream << what; \
	__trace_subject = __trace_ostream.str(); \
	TRACE_BEGIN_EX("Started: " << __trace_subject << ".", __trace_stopwatch)

#define TRACE_END \
	TRACE_END_EX("Finished: " << __trace_subject << ".", __trace_stopwatch)

#define TRACE(message, action) \
	TRACE_BEGIN(message); \
	{action;} \
	TRACE_END
```

So now we are able to write something like this (excerpt from source code of image fusion application).

```cpp
TRACE("transformation and transfer of fused image",
{
	saveChannel(0);
	saveChannel(1);
	saveChannel(2);
});
```

OK, once we have compiled and run application, we can see something like this.

```
...
Started: transformation and transfer of fused image.
Finished: transformation and transfer of fused image. Time: 0.15s.
...
```

Good, but what if we releasing production version and tracing no more needed? That's simple! Lets change our macros.

```cpp
#if defined(_TRACE)

#include <iostream>
#include <sstream>
#include "Stopwatch.h"

#define TRACE_BEGIN_EX(what, watch) \
	std::clog << what << std::endl; \
	watch.restart()

#define TRACE_END_EX(what, watch) \
	watch.stop(); \
	std::clog << what << \
	" Time: " << watch.getElapsedSeconds() << "s." << std::endl

#define TRACE_PREPARE \
	Stopwatch __trace_stopwatch; \
	std::ostringstream __trace_ostream; \
	std::string __trace_subject

#define TRACE_BEGIN(what) \
	__trace_ostream.str(std::string()); \
	__trace_ostream << what; \
	__trace_subject = __trace_ostream.str(); \
	TRACE_BEGIN_EX("Started: " << __trace_subject << ".", __trace_stopwatch)

#define TRACE_END \
	TRACE_END_EX("Finished: " << __trace_subject << ".", __trace_stopwatch)

#define TRACE(message, action) \
	TRACE_BEGIN(message); \
	{action;} \
	TRACE_END

#else

#define TRACE_BEGIN_EX(what, watch)
#define TRACE_END_EX(what, watch)
#define TRACE_PREPARE
#define TRACE_BEGIN(what)
#define TRACE_END
#define TRACE(message, action) {action;}

#endif
```

So now if we'll compile our code without `_TRACE` being defined, tracing simply disappears and doesn't have any impact on performance.

To summarize, these simple but quite powerful macros let us easily trace our code execution (including time of execution), and at the same time we have an option to disable it by simply not defining `_TRACE` macro.
