---
layout: post
comments: true
title: "Useful Information In Exceptions"
categories: C++ Snippet
---
Developing image fusion software under my master's work, I do specific assumptions on image channel size, minimal required memory size, etc. And if something goes wrong, the best way is to throw exception :). So, here is some kind of helper for including useful information (besides message itself)  to exception.

```cpp
/*!
	For internal use.
*/
#define __TO_STRING(what) #what
#define __WHAT(message, file, line, date, time) \
	message " (" file ": " __TO_STRING(line) ", built at " date " " time ")"

/*!
	Throws standard exception with given message and
	information about source file name, line and build date.
*/
#define THROW_EXCEPTION(message) throw std::exception( \
	__WHAT(message, __FILE__, __LINE__, __DATE__, __TIME__))
```

So, here is an example of usage of that macro.

```cpp
int main()
{
	try
	{
		THROW_EXCEPTION("Error!");
	}
	catch(const std::exception& e)
	{
		std::cerr << e.what() << std::endl;
	}

	return 0;
}
```

In this case STDERR will contain next line.

```
Error! (d:\projects\test\main.cpp: 22, built at Nov 12 2011 13:51:15)
```

Now we can explicitly see where that exception was thrown, moreover - we know when build was made.
