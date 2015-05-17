---
layout: post
title:  从零开始实现自己的httpd——epoll读写数据的注意事项
categories: [http]
---
Linux中 ”一切皆文件的概念“使得我们可以很方便地理解与文件相关的操作。网络编程中遇到的基本函数是 select、epoll、accept、read、write这5个，在阻塞和非阻塞模式下，对他们的调用，尤其是对返回值的判断处理很不同，下面做深入描述

阻塞简单理解就是当程序调用该函数时，在一段时间内函数不返回，进程会挂起，直到某些条件发生，函数返回，进程才能继续之下函数下面的语句。

1、read  write
函数原型如下：

```cpp
#include <unistd.h>  
ssize_t read(int filedes, void* buf, size_t nbytes);

ssize_t write(int filedes, const void* buf, size_t nbytes);
```

其中，read返回实际读取到的字节数。但实际读取的字节很有可能少于指定要读取的字节数nbytes。因此会分为：
-   返回值大于0。 读取正常，返回实际读取到的字节数
-   返回值等于0。 读取异常，读取到文件filedes结尾处了。这里逻辑上要理解为read已经读取完数据
-   返回值小于0（-1）。 读取出错，在处理网络请求时可能是网络异常。着重注意当返回-1，此时errno的值EAGAIN、EWOULLDBLOCK，表示内核对应的读缓冲区为空

而write返回的实际写入字节数正常情况是与制定写入的字节数nbytes相同的，不相等说明写入异常了，着重注意，此时errno的值EAGAIN、EWOULLDBLOCK，表示内核对应的写缓冲区为空。注，EAGAIN等同于EWOULLDBLOCK。

总之，这个错误表示资源暂时不够，可能read时读缓冲区没有数据， 或者write时写缓冲区满了。遇到这种情况，如果是阻塞socket、 read/write就要阻塞掉。而如果是非阻塞socket、 read/write立即返回-1, 同时errno设置为EAGAIN。

所以对于阻塞socket、 read/write返回-1代表网络出错了。但对于非阻塞socket、read/write返回-1不一定网络真的出错了。可能支持缓冲区空或者满，这时应该再试，直到Resource available。

综上，对于非阻塞的socket，正确的读写操作为：读： 忽略掉errno = EAGAIN的错误，下次继续读；写：忽略掉errno = EAGAIN的错误，下次继续写。对于select和epoll的LT模式，这种读写方式是没有问题的。但对于epoll的ET模式，这种方式还有漏洞。

下面来介绍下epoll事件的两种模式LT（水平触发）和ET（边沿触发），根据可以理解为，文件描述符的读写状态发生变化才会触发epoll事件，具体说来如下：二者的差异在于 level-trigger 模式下只要某个 socket 处于 readable/writable 状态，无论什么时候进行 epoll_wait 都会返回该 socket；而 edge-trigger 模式下只有某个 socket 从 unreadable 变为 readable，或从unwritable 变为writable时，epoll_wait 才会返回该 socket。

所以在epoll的ET模式下，正确的读写方式为：
读： 只要可读， 就一直读，直到返回0，或者 errno = EAGAIN
写：只要可写， 就一直写，直到数据发送完，或者 errno = EAGAIN
这里的意思是，对于ET模式，相当于我们要自己重写read和write，使其像”原子操作“一样，保证一次read 或 write能够完整的读完缓冲区的数据或者写完要写入缓冲区的数据。因此，实现为用while包住read和write即可。但是对于select或者LT模式，我们可以只使用一次read和write，因为在主程序中会一直while，而事件再下一次select时还会被获取到。但也可以实现为用while包住read和write。从逻辑上讲，一次性把数据读取完整可以保证数据的完整性。

```cpp
int n = 0;    
while(1)
{
	nread = read(fd, buf + n, BUFSIZ-1);//读时，用户进程指定的接收数据缓冲区大小固定，一般要比数据大
	if(nread < 0)
	{
		if(errno == EAGAIN || errno == EWOULDBLOCK)
		{
			continue;
		}
		else
		{
			break;//or return;
		}
	}
	else if(nread == 0)
	{
		break;//or return. because read the EOF
	}
	else
	{
		n += nread;
	}
}
```


```cpp

int data_size = strlen(buf);
int n = 0;
while(1)
{
	nwrite = write(fd, buf + n, data_size);//写时，数据大小一直在变化
	if(nwrite < data_size)
	{
		if(errno == EAGAIN || errno == EWOULDBLOCK)
		{
			continue;
		}
		else
		{
			break;//or return;		
		}

	}
	else
	{
		n += nwrite;
		data_size -= nwrite;
	}
}
```
