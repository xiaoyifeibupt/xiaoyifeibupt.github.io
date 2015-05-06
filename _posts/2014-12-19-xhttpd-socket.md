---
layout: post
title:  从零开始实现自己的httpd——socket
categories: [http]
---

根据连接启动的方式以及本地套接字要连接的目标，套接字之间的连接过程可以分为三个步骤：服务器监听，客户端请求，连接确认。

（1）服务器监听：是服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态。

（2）客户端请求：是指由客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。

（3）连接确认：是指当服务器端套接字监听到或者说接收到客户端套接字的连接请求，它就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，连接就建立好了。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

Socket.h

```cpp
#ifndef __SOCKET_H__
#define __SOCKET_H__

#include <sys/socket.h>
#include <arpa/inet.h>

#include <memory>
#include <string>

#include "Buffer.h"

struct Address {
	Address(std::string host, uint16_t port);
	std::string a_host;
	uint16_t a_port;
};

class Socket;

typedef std::shared_ptr<Socket> SocketPtr; //指向Socket类的智能指针

class Socket
{
public:

	Socket();
	Socket(int fd);
	Socket(Socket&&);
	virtual ~Socket();

	void bind(Address address);
	void listen();
	SocketPtr accept();
	
	void makeNonBlocking();
	
	void read(Buffer<uint8_t>& buffer);
	
	
	size_t write(const uint8_t *buf, size_t size);
	size_t write();
	
	int native();
	bool error();
	bool needToWrite() const;

	friend bool operator==(SocketPtr lhs, int rhs);

private:
	Socket(const Socket&) = delete;
	Socket& operator=(const Socket&) = delete;

	int sockfd;
	struct sockaddr_in addr;

	std::string host;
	uint16_t port;
	Buffer<uint8_t> writeBuffer;
	bool needToWrite;
};

#endif


```

`Socket.c`里面的部分函数

###创建

函数原型：

```cpp

int socket(int domain, int type, int protocol);

```

参数说明：
　　
-	domain：协议域，又称协议族（family）。常用的协议族有AF_INET、AF_INET6、AF_LOCAL（或称AF_UNIX，Unix域Socket）、AF_ROUTE等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址（32位的）与端口号（16位的）的组合、AF_UNIX决定了要用一个绝对路径名作为地址。

-	type：指定Socket类型。常用的socket类型有SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等。流式Socket（SOCK_STREAM）是一种面向连接的Socket，针对于面向连接的TCP服务应用。数据报式Socket（SOCK_DGRAM）是一种无连接的Socket，对应于无连接的UDP服务应用。

-	protocol：指定协议。常用协议有IPPROTO_TCP、IPPROTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC等，分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议。
注意：type和protocol不可以随意组合，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当第三个参数为0时，会自动选择第二个参数类型对应的默认协议。

返回值：

如果调用成功就返回新创建的套接字的描述符，如果失败就返回INVALID_SOCKET（Linux下失败返回-1）。套接字描述符是一个整数类型的值。每个进程的进程空间里都有一个套接字描述符表，该表中存放着套接字描述符和套接字数据结构的对应关系。该表中有一个字段存放新创建的套接字的描述符，另一个字段存放套接字数据结构的地址，因此根据套接字描述符就可以找到其对应的套接字数据结构。每个进程在自己的进程空间里都有一个套接字描述符表但是套接字数据结构都是在操作系统的内核缓冲里。

```cpp

Socket::Socket() : needToWrite(false) {

	sockfd = socket(AF_INET, SOCK_STREAM, 0);
	if (sockfd == -1) {
		THROW_SYSTEM_ERROR(); 
	}
	int one = 1;
	//设置套接字选项
	int ret = setsockopt(sockfd, SOL_SOCKET, SO_REUSEPORT, &one, sizeof(one));
	if (ret != 0) {
		THROW_SYSTEM_ERROR();
	}
}
```
###绑定

函数原型：

```cpp
int bind(SOCKET socket, const struct sockaddr* address, socklen_t address_len);
```
参数说明：

-	socket：是一个套接字描述符。

-	address：是一个sockaddr结构指针，该结构中包含了要结合的地址和端口号。

-	address_len：确定address缓冲区的长度。

返回值：

如果函数执行成功，返回值为0，否则为SOCKET_ERROR。

```cpp

void Socket::bind(Address address) {
	struct hostent *hosten = gethostbyname(address.a_host.c_str());

	if (!hosten) {
		const char *errMsg = hstrerror(h_errno);
		throw std::runtime_error(errMsg);
	}

	memset(&addr, 0, sizeof(addr));
	addr.sin_family = AF_INET;
	addr.sin_port = htons(address.port);
	addr.sin_addr.s_addr = htonl(INADDR_ANY);
	
	int ret = ::bind(sockfd, reinterpret_cast<struct sockaddr*>(&addr),sizeof(addr));
	if (ret != 0) {
		THROW_SYSTEM_ERROR();
	}
}
```

###监听

```cpp
void Socket::listen() {
	int ret = ::listen(sockfd, 10000);

	if (ret != 0) {
		THROW_SYSTEM_ERROR();
	}
}

SocketPtr Socket::accept() {
	socklen_t size = sizeof(addr);
	int ret = ::accept(sockfd, reinterpret_cast<struct sockaddr*>(&addr), &size);

	if (ret == -1) {
		THROW_SYSTEM_ERROR();
	}

	return std::make_shared<Socket>(ret);
}
```
控制I/O设备 ，提供了一种获得设备信息和向设备发送控制参数的手段。用于向设备发控制和配置命令 ，有些命令需要控制参数，这些数据是不能用read / write 读写的,称为Out-of-band数据。也就是说,read / write 读写的数据是in-band数据,是I/O操作的主体，而ioctl 命令传送的是控制信息,其中的数据是辅助的数据。
函数原型

```cpp
int ioctl(int handle, int cmd,[int *argdx, int argcx]);
```

返回值：成功为0，出错为-1

```cpp
void Socket::makeNonBlocking() {
	long one = 1;

	int ret = ioctl(sockfd, FIONBIO, &one);

	if (ret == -1) {
		THROW_SYSTEM_ERROR();
	}
}
```
###接收

函数原型：

```cpp
int recv(SOCKET socket, char FAR* buf, int len, int flags);
```

参数说明：
　　
-	socket：一个标识已连接套接口的描述字。

-	buf：用于接收数据的缓冲区。

-	len：缓冲区长度。

-	flags：指定调用方式。取值：`MSG_PEEK` 查看当前数据，数据将被复制到缓冲区中，但并不从输入队列中删除；`MSG_OOB` 处理带外数据。

返回值：

若无错误发生，`recv()`返回读入的字节数。如果连接已中止，返回0。否则的话，返回`SOCKET_ERROR`错误，应用程序可通过`WSAGetLastError()`获取相应错误代码。

```cpp
void Socket::read(Buffer<uint8_t>& buffer) {
	uint8_t buf[1024];
	int ret = 0;

	ret = recv(sockfd, buf, sizeof(buf), 0);
	while (ret > 0) {
		buffer.append(buf, ret);
		ret = recv(sockfd, buf, sizeof(buf), 0);
	}
}
```

###发送

函数原型

```cpp
Ssize_t write(int fd,const void *buf,size_t nbytes);
```
`Write`函数将`buf`中的`nbytes`字节内容写入到文件描述符中，成功返回写的字节数，失败返回-1.并设置`errno`变量。在网络程序中，当我们向套接字文件描述舒服写数据时有两种可能：
    
1、`write`的返回值大于0，表示写了部分数据或者是全部的数据，这样用一个`while`循环不断的写入数据，但是循环过程中的`buf`参数和nbytes参数是我们自己来更新的，也就是说，网络编程中写函数是不负责将全部数据写完之后再返回的，说不定中途就返回了！


2、返回值小于0，此时出错了，需要根据错误类型进行相应的处理。如果错误是`EINTR`表示在写的时候出现了中断错误，如果是`EPIPE`表示网络连接出现了问题。

```cpp
size_t Socket::write(const uint8_t *buf, size_t size) {
	writeBuffer.append(buf, size);

	return write();
}

size_t Socket::write() {
	int ret = 0;

	ret = ::write(sockfd, writeBuffer.data(), writeBuffer.size());
	if (ret == -1) {
		if (errno != EAGAIN && errno != EWOULDBLOCK) {
			_E("Can't write to socket. Socket will be closed");
		}
	} else {
		writeBuffer_.drain(ret);
	}

	needToWrite = !!writeBuffer.size();

	return writeBuffer.size();
}
```
