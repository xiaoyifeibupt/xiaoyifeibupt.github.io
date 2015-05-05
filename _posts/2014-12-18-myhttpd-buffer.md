---
layout: post
title:  从零开始实现自己的httpd——buffer
categories: [http]
---

准备工作

ANSI C标准中有几个标准预定义宏（也是常用的）：

`__LINE__`：在源代码中插入当前源代码行号；

`__FILE__`：在源文件中插入当前源文件名；

`__DATE__`：在源文件中插入当前的编译日期

`__TIME__`：在源文件中插入当前编译时间；

`__STDC__`：当要求程序严格遵循ANSI C标准时该标识被赋值为1；

`__cplusplus`：当编写C++程序时该标识符被定义。

编译器在进行源码编译的时候，会自动将这些宏替换为相应内容。


Log.h

```cpp

#ifndef __LOG_H__
#define __LOG_H__

#include <string.h>

#include <cerrno>
#include <iostream>
#include <system_error>

#define __STR(s) #s
#define WHERE __STR(__FILE__ ":" __LINE__)

#define _E(msg) \
	std::cerr << "[ERROR] " << __FILE__ << ":" << __LINE__ << " " << msg << std::endl;

#define _W(msg) \
	std::cerr << "[WARNING] " << __FILE__ << ":" << __LINE__ << " " << msg << std::endl;

#define _I(msg) \
	std::cerr << "[INFO] " << msg << std::endl;

#define SYSTEM_ERROR() \
do { \
	_E(strerror(errno));\
} while (0)

#define THROW_SYSTEM_ERROR() \
do { \
	SYSTEM_ERROR(); \
	throw std::system_error(std::error_code(errno, std::system_category()));\
} while (0)

#define SYSTEM_WARNING() \
do { \
	_W(strerror(errno)) \
} while (0)

#define THROW(msg) \
do { \
	throw std::runtime_error(msg);\
} while (0)

#endif /* __LOG_H__ */

```

Buffer.h

```cpp

#ifndef __BUFFER_H__
#define __BUFFER_H__

#include <stdlib.h>

#include "Log.h"

template<typename T>
class Buffer
{
public:
	Buffer(size_t capacity = 1024) : capacity(capacity) , size(0) {
		if (capacity == 0) {
			data = nullptr;
			return;
		}

		data = reinterpret_cast<T*>(malloc(capacity * sizeof(T)));
		if (!data) {
			THROW_SYSTEM_ERROR();
		}
	}

	~Buffer() {
		if (data) {
			free(data);
		}
	}

	size_t size() const {
		return size;
	}

	size_t capacity() const {
		return capacity;
	}

	void setCapacity(size_t _capacity) {
		capacity = capacity;
		errno = 0;
		data = reinterpret_cast<T*>(
			realloc(data, capacity * sizeof(T)));
		if (errno) {
			THROW_SYSTEM_ERROR();
		}
	}

	void append(const T *src, size_t _size) {
		if (capacity < size + _size) {
			setCapacity(capacity + size);
		}

		if (src >= data && src < data + capacity) {
			_E("src and dst do ovelap")
		}
		memcpy(data + size, src, _size);

		size += _size;
	}

	void drain(size_t n) {
		if (n == 0)
			return;

		if (n > size) {
			THROW("not enough data");
		}

		size_t count = size - n;
		T *ptr1 = data;
		T *ptr2 = data + n;

		while (count) {
			size_t count2 = std::min(n, count);

			memcpy(ptr1, ptr2, count2);
			ptr1 = ptr2;
			ptr2 += count2;
			count -= count2;
		}

		size -= n;
	}

	const T* data() {
		return data;
	}

private:
	T* data;
	size_t capacity;
	size_t size;
};

#endif

```
