# 三次握手和socket的关系

## 1. socket

```c++
int socket(int domain, int type, int protocol);
```

socket函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而`socket()`用于创建一个socket描述符（socket descriptor），它唯一标识一个socket。这个socket描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。

正如可以给fopen的传入不同参数值，以打开不同的文件。创建socket的时候，也可以指定不同的参数创建不同的socket描述符，socket函数的三个参数分别为：

- domain：即协议域，又称为协议族（family）。常用的协议族有，AF_INET、AF_INET6、AF_LOCAL（或称AF_UNIX，Unix域socket）、AF_ROUTE等等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址（32位的）与端口号（16位的）的组合、AF_UNIX决定了要用一个绝对路径名作为地址。
- type：指定socket类型。常用的socket类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等等（socket的类型有哪些？）。
- protocol：故名思意，就是指定协议。常用的协议有，IPPROTO_TCP、IPPTOTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC等，它们分别对应TCP传输协议、 UDP传输协议、STCP传输协议、TIPC传输协议（这个协议我将会单独开篇讨论！）。

注意：并不是上面的type和protocol可以随意组合的，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当protocol为0时，会自动选择type类型对应的默认协议。

## 2. listen

如果作为一个服务器，在调用socket()、bind()之后就会调用listen()来监听这个socket，如果客户端这时调用connect()发出连接请求，服务器端就会接收到这个请求。

```c++
int listen(int sockfd, int backlog);
```

listen函数的第一个参数即为要监听的socket描述字，第二个参数backlog为相应socket可以排队的最大连接个数，其包括两个队列，一个是未完成队列，包含的套接字状态为SYN_RCVD，另一个为已完成队列，包含的套接字为established,两队列之和小于backlog。<u>**因此三次握手成功与否与accept()无关！socket()函数创建的socket默认是一个主动类型的，listen函数将socket变为被动类型的，等待客户的连接请求。**</u>

## 3.connect

```c++
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

connect函数的第一个参数即为客户端的socket描述字，第二参数为服务器的socket地址，第三个参数为socket地址的长度。客户端通过调用connect函数来建立与TCP服务器的连接。

## 4.accept

```c++
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

TCP服务器端依次调用socket()、bind()、listen()之后，就会监听指定的socket地址了。调用accept()函数会从已完成队列取出一个以建立连接的套接字。若已完成队列为空，则会阻塞(若为非阻塞，则errno置为EAGAIN)。之后就可以开始网络I/O操作了，即类同于普通文件的读写I/O操作。

accept函数的第一个参数为服务器的socket描述字，第二个参数为指向struct sockaddr *的指针，用于返回客户端的协议地址，第三个参数为协议地址的长度。如果accpet成功，那么其返回值是由内核自动生成的一个全新的描述字，代表与返回客户的TCP连接。