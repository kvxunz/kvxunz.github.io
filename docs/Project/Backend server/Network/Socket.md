# **Socket套接字**

## **简介**
- 对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。 一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口， 是应用程序与网络协议根进行交互的接口。

- socket 可以看成是两个网络应用程序进行通信时，各自通信连接中的端点，这是一个逻辑上的概念。
- 它是网络环境中进程间通信的API，也是可以被命名和寻址的通信端点，使用中的每一个套接字都有其类型和一个与之相连进程。
- **通信时其中一个网络应用程序将要传输的一段信息写入它所在主机的socket中，该socket通过与网络接口卡(NIC)相连的传输介质将这段信息送到另外一台主机的socket中，使对方能够接收到这段信息。socket 是由 IP 地址和端口结合的，提供向应用层进程传送数据包的机制**
- socket本身有“插座”的意思，在Linux环境下，用于表示进程间网络通信的特殊文件类型。本质为内核借助缓冲区形成的伪文件。既然是文件，那么理所当然的，我们可以使用文件描述符引用套接字
    - 与管道类似的，Linux 系统将其封装成文件的目的是为了统一接口，使得读写套接字和读写文件的操作一致。区别是管道主要应用于本地进程间通信，而套接字多应用于网络进程间数据的传递。



## **创建socket**
- `int socket(int domain, int type, int protocol);`
    - 成功返回一个有效的`socket`，失败返回`-1`，`errno`被设置
    - 全部网络编程的函数，失败时基本上都是返回`-1`，`errno`被设置
    - 只要参数没填错，基本上不会失败
    - 不过，单个进程中创建的`socket`数量与受系统参数`open files`的限制`（ulimit -a ）`
- `domain` 通讯的协议家族
    - `PF_INET`		`IPv4`互联网协议族
    - `PF_INET6`		`IPv6`互联网协议族
    - `PF_LOCAL`		本地通信的协议族
    - `PF_PACKET`		内核底层的协议族
    - `PF_IPX`			`IPX Novell`协议族
    - `IPv6`尚未普及，其它的不常用
- `type` 数据传输的类型
    - `SOCK_STREAM`		面向连接的socket：
        - 数据不会丢失
        - 数据的顺序不会错乱
        - 双向通道
    - `SOCK_DGRAM`		无连接的socket：
        - 数据可能会丢失
        - 数据的顺序可能会错乱
        - 传输的效率更高
- `protocol` 最终使用的协议
    - 在`IPv4`网络协议家族中，数据传输方式为`SOCK_STREAM`的协议只有`IPPROTO_TCP`，数据传输方式为`SOCK_DGRAM`的协议只有`IPPROTO_UDP`
    - 本参数也可以填`0`
    - ` socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);`    // 创建`tcp`的`sock`
    - `socket(PF_INET, SOCK_DGRAM, IPPROTO_UDP);`    // 创建`udp`的`sock`


## **Socket函数**

```c++
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h> // 包含了这个头文件，上面两个就可以省略
```


```c++
int socket(int domain, int type, int protocol); 
    - 功能:创建一个套接字
    - 参数:
        - domain: 协议族 
                AF_INET : ipv4
                AF_INET6 : ipv6
                AF_UNIX, AF_LOCAL : 本地套接字通信(进程间通信) 
        - type: 通信过程中使用的协议类型
                SOCK_STREAM : 流式协议
                SOCK_DGRAM : 报式协议
        - protocol : 具体的一个协议。一般写0
            - SOCK_STREAM : 流式协议默认使用 TCP
            - SOCK_DGRAM : 报式协议默认使用 UDP 
        - 返回值:
            - 成功:返回文件描述符，操作的就是内核缓冲区。 
            - 失败:-1
```

```c++
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);//socket命名
    - 功能:绑定，将fd 和本地的IP + 端口进行绑定 
    - 参数:
        - sockfd : 通过socket函数得到的文件描述符
        - addr : 需要绑定的socket地址，这个地址封装了ip和端口号的信息 
        - addrlen : 第二个参数结构体占的内存大小
```

```c++
int listen(int sockfd, int backlog); // /proc/sys/net/core/somaxconn 
    - 功能:监听这个socket上的连接
    - 参数:
        - sockfd : 通过socket()函数得到的文件描述符
        - backlog : 未连接的和已经连接的和的最大值， 5
```

```c++
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); 
    - 功能:接收客户端连接，默认是一个阻塞的函数，阻塞等待客户端连接
    - 参数:
        - sockfd : 用于监听的文件描述符
        - addr : 传出参数，记录了连接成功后客户端的地址信息(ip，port) 
        - addrlen : 指定第二个参数的对应的内存大小
    - 返回值:
        - 成功 :用于通信的文件描述符
        - -1 : 失败
```

```c++
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    - 功能: 客户端连接服务器 
    - 参数:
        - sockfd : 用于通信的文件描述符
        - addr : 客户端要连接的服务器的地址信息 
        - addrlen : 第二个参数的内存大小
    - 返回值:成功 0， 失败 -1
```

```c++
ssize_t write(int fd, const void *buf, size_t count);// 写数据
ssize_t read(int fd, void *buf, size_t count); // 读数据
```


## **Socket 地址**
- socket地址其实是一个结构体，封装端口号和IP等信息。后面的socket相关的api中需要使用到这个socket地址
- 客户端 -> 服务器(IP, Port)

### **通用 socket 地址**
- socket 网络编程接口中表示 socket 地址的是结构体 sockaddr，其定义如下:
```c++
#include <bits/socket.h>
struct sockaddr {
    sa_family_t sa_family;
    char        sa_data[14];
};
typedef unsigned short int sa_family_t;
```

- sa_family 成员是地址族类型(sa_family_t)的变量。地址族类型通常与协议族类型对应。常见的协议 族(protocol family，也称 domain)和对应的地址族入下所示
    - 宏 PF_* 和 AF_* 都定义在 bits/socket.h 头文件中，且后者与前者有完全相同的值，所以二者通常混 用。

        | 协议族 | 地址族 | 描述
        |------|-------|-------|
        | PF_UNIX | AF_UNIX |UNIX本地域协议族|
        | PF_INET | AF_INET |TCP/IPv4协议族|
        | PF_INET6 | AF_INET6 |TCP/IPv6协议族|

- sa_data 成员用于存放 socket 地址值。但是，不同的协议族的地址值具有不同的含义和长度，如下所示:

    | 协议族 | 地址值含义和长度|
    |------|-------|
    | PF_UNIX| 文件的路径名，长度可达到108字节 |
    |PF_INET |16 bit 端口号和 32 bit IPv4 地址，共 6 字节 |
    | PF_INET6 | 16 bit 端口号，32 bit 流标识，128 bit IPv6 地址，32 bit 范围 ID，共 26 字节 |

- 14 字节的 sa_data 根本无法容纳多数协议族的地址值。因此，Linux 定义了下面这个新的通用的 socket 地址结构体，这个结构体不仅提供了足够大的空间用于存放地址值，而且是内存对齐的

    ```c++
    #include <bits/socket.h>
    struct sockaddr_storage
    {
        sa_family_t sa_family;
        unsigned long int __ss_align;
        char __ss_padding[ 128 - sizeof(__ss_align) ];
    };
    typedef unsigned short int sa_family_t;
    ```

### **专用 socket 地址**
- 很多网络编程函数诞生早于 IPv4 协议，那时候都使用的是 struct sockaddr 结构体，为了向前兼容，现 在sockaddr 退化成了(void *)的作用，传递一个地址给函数，至于这个函数是 sockaddr_in 还是 sockaddr_in6，由地址族确定，然后函数内部再强制类型转化为所需的地址类型

- UNIX 本地域协议族使用如下专用的 socket 地址结构体:

    ```c++
    #include <sys/un.h>
    struct sockaddr_un
    {
        sa_family_t sin_family;
        char sun_path[108];
    };
    ```

- TCP/IP 协议族有 sockaddr_in 和 sockaddr_in6 两个专用的 socket 地址结构体，它们分别用于 IPv4 和 IPv6:

    ```c++
    #include <netinet/in.h>
    struct sockaddr_in
    {
        sa_family_t sin_family; /* __SOCKADDR_COMMON(sin_) */
        in_port_t sin_port; /* Port number.  */
        struct in_addr sin_addr; /* Internet address.  */
        /* Pad to size of `struct sockaddr'. */
        unsigned char sin_zero[sizeof (struct sockaddr) - __SOCKADDR_COMMON_SIZE - sizeof (in_port_t) - sizeof (struct in_addr)];
    };
    struct in_addr
    {
        in_addr_t s_addr;
    };
    struct sockaddr_in6
    {
        sa_family_t sin6_family;
        in_port_t sin6_port;    /* Transport layer port # */
        uint32_t sin6_flowinfo; /* IPv6 flow information */
        struct in6_addr sin6_addr;  /* IPv6 address */
        uint32_t sin6_scope_id; /* IPv6 scope-id */
    };
    typedef unsigned short  uint16_t;
    typedef unsigned int    uint32_t;
    typedef uint16_t in_port_t;
    typedef uint32_t in_addr_t;
    #define __SOCKADDR_COMMON_SIZE (sizeof (unsigned short int))
    ```

- 所有专用 socket 地址(以及 sockaddr_storage)类型的变量在实际使用时都需要转化为通用 socket 地 址类型 sockaddr(强制转化即可)，因为所有 socket 编程接口使用的地址参数类型都是 sockaddr