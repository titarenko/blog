---
layout: post
comments: true
title: "CUDA Vector Implementation"
categories: Snippet C++ CUDA
---
Dealing with CUDA, you probably noticed how tedious are memory management operations. So, here is my way to get rid of boring things.

Which data structure is widely used while developing with CUDA? Vector (or linear array) - no doubt! OK, so let's create our CUDA-aware vector.

```cpp
/*! vector data structure (in device memory) */
struct cuvec
{
	/*! pointer to data */
	sca* ptr;

	/*! elements count */
	int size;

	/*! bytes taken */
	int bytes;

	/*! skip disposal */
	bool skip;

	/*! constructs vector with n elements */
	cuvec(int n)
	{
		size = n;
		bytes = size*sizeof(sca);

		CUCHECK(cudaMalloc((void**) &ptr, bytes));

		skip = false;
	}

	/*! copy constructor - disables disposal */
	cuvec(const cuvec& other)
	{
		size = other.size;
		bytes = other.bytes;

		ptr = other.ptr;

		// disposal is disabled because resources
		// were allocated by another instance, so
		// management of them is not our responsibility
		skip = true;
	}

	/*! performs disposal if allowed */
	~cuvec()
	{
		if (!skip)
		{
			dispose();
		}
	}

	/*! performs disposal */
	void dispose()
	{
		if (ptr)
		{
			CUCHECK(cudaFree((void*) ptr));

			ptr = 0;
			size = 0;
			bytes = 0;
		}
	}
};
```

Probably, you've noticed some unknown symbols like `sca` and `CUCHEK`. Here they are.

```cpp
/*! scalar type */
typedef float sca;

/*! vector data structure (in host memory) */
typedef std::vector<sca> vec;

/*! throws exception with error message if something went wrong */
#define CUCHECK(x) if ((x) != cudaSuccess) \
	throw std::exception(cudaGetErrorString(cudaGetLastError()))
```

It feels like something is missing! Definitely, it's time for memory transfers from host to device and vice versa!

```cpp
/*! copies data from device to host */
void operator << (vec& to, const cuvec& from)
{
	CUCHECK(cudaMemcpy((void*) to.data(), (void*) from.ptr,
		from.bytes, cudaMemcpyDeviceToHost));
}

/*! copies data from host to device */
void operator >> (const vec& from, cuvec& to)
{
	CUCHECK(cudaMemcpy((void*) to.ptr, (void*) from.data(),
		to.bytes, cudaMemcpyHostToDevice));
}

/*! copies data from host to device */
void operator << (cuvec& to, const vec& from)
{
	from >> to;
}

/*! copies data from device to host */
void operator >> (const cuvec& from, vec& to)
{
	to << from;
}
```

OK, now we are ready to test this infrastructure. We are going further with little example of usage. Let's suppose we want... yes, add two arrays! :)

```cpp
__global__ void add(cuvec a, cuvec b)
{
	int i = threadIdx.x;
	a.ptr[i] += b.ptr[i];
}

int execute()
{
	vec ha(10);
	vec hb(10);

	for (int i = 0; i < 10; ++i)
		ha[i] = hb[i] = i;

	cuvec a(10);
	cuvec b(10);

	a << ha;
	b << hb;

	add<<<1, 10>>>(a, b);
	CUCHECK(cudaGetLastError());

	a >> ha;

	for (int i = 0; i < 10; ++i)
		std::cout << ha[i] << std::endl;

	return 0;
}

int main()
{
	try
	{
		return execute();
	}
	catch (std::exception e)
	{
		std::cerr << e.what() << std::endl;
		return -1;
	}
}
```

I think this code is more readable and easier to write and maintain, than that one which uses a bunch of `cudaMalloc`, `cudaFree` and `cudaMemcpy`.
One more advantage of `cuvec` is kind of smart disposal. Probably, you've spotted that data is passed to `add` kernel by value, not by reference (this is restricted) or by pointer (this is not correct). So, destructor of `cuvec` will be called several times, but we need only one disposal right at the end of the execution, and this happens right as expected.
