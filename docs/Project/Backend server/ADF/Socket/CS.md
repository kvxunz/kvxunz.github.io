 
# **socket通讯的函数和类**

## **定义**
```c++
// socket通讯的客户端类
class ctcpclient
{
private:
    int  m_connfd;    // 客户端的socket.
    string m_ip;        // 服务端的ip地址。
    int  m_port;        // 服务端通讯的端口。
public:
    ctcpclient(): m_connfd(-1),m_port(0) { }  // 构造函数。

    // 向服务端发起连接请求。
    // ip：服务端的ip地址。
    // port：服务端通讯的端口。
    // 返回值：true-成功；false-失败。
    bool connect(const string &ip,const int port);

    // 接收对端发送过来的数据。
    // buffer：存放接收数据缓冲区。
    // ibuflen: 打算接收数据的大小。
    // itimeout：等待数据的超时时间（秒）：-1-不等待；0-无限等待；>0-等待的秒数。
    // 返回值：true-成功；false-失败，失败有两种情况：1）等待超时；2）socket连接已不可用。
    bool read(string &buffer,const int itimeout=0);                           // 接收文本数据。
    bool read(void *buffer,const int ibuflen,const int itimeout=0);   // 接收二进制数据。

    // 向对端发送数据。
    // buffer：待发送数据缓冲区。
    // ibuflen：待发送数据的大小。
    // 返回值：true-成功；false-失败，如果失败，表示socket连接已不可用。
    bool write(const string &buffer);                          // 发送文本数据。
    bool write(const void *buffer,const int ibuflen);   // 发送二进制数据。

    // 断开与服务端的连接
    void close();

    ~ctcpclient();  // 析构函数自动关闭socket，释放资源。
};

```
```c++
// socket通讯的服务端类
class ctcpserver
{
private:
    int m_socklen;                                // 结构体struct sockaddr_in的大小。
    struct sockaddr_in m_clientaddr;   // 客户端的地址信息。
    struct sockaddr_in m_servaddr;     // 服务端的地址信息。
    int  m_listenfd;                               // 服务端用于监听的socket。
    int  m_connfd;                                // 客户端连接上来的socket。
public:
    ctcpserver():m_listenfd(-1),m_connfd(-1) {}  // 构造函数。

    // 服务端初始化。
    // port：指定服务端用于监听的端口。
    // 返回值：true-成功；false-失败，一般情况下，只要port设置正确，没有被占用，初始化都会成功。
    bool initserver(const unsigned int port,const int backlog=5); 

    // 从已连接队列中获取一个客户端连接，如果已连接队列为空，将阻塞等待。
    // 返回值：true-成功的获取了一个客户端连接，false-失败，如果accept失败，可以重新accept。
    bool accept();

    // 获取客户端的ip地址。
    // 返回值：客户端的ip地址，如"192.168.1.100"。
    char *getip();

    // 接收对端发送过来的数据。
    // buffer：存放接收数据的缓冲区。
    // ibuflen: 打算接收数据的大小。
    // itimeout：等待数据的超时时间（秒）：-1-不等待；0-无限等待；>0-等待的秒数。
    // 返回值：true-成功；false-失败，失败有两种情况：1）等待超时；2）socket连接已不可用。
    bool read(string &buffer,const int itimeout=0);                           // 接收文本数据。
    bool read(void *buffer,const int ibuflen,const int itimeout=0);   // 接收二进制数据。

    // 向对端发送数据。
    // buffer：待发送数据缓冲区。
    // ibuflen：待发送数据的大小。
    // 返回值：true-成功；false-失败，如果失败，表示socket连接已不可用。
    bool write(const string &buffer);                          // 发送文本数据。
    bool write(const void *buffer,const int ibuflen);   // 发送二进制数据。

    // 关闭监听的socket，即m_listenfd，常用于多进程服务程序的子进程代码中。
    void closelisten();

    // 关闭客户端的socket，即m_connfd，常用于多进程服务程序的父进程代码中。
    void closeclient();

    ~ctcpserver();  // 析构函数自动关闭socket，释放资源。
};

```
```c++
// 接收socket的对端发送过来的数据。
// sockfd：可用的socket连接。
// buffer：接收数据缓冲区的地址。
// ibuflen：本次成功接收数据的字节数。
// itimeout：读取数据超时的时间，单位：秒，-1-不等待；0-无限等待；>0-等待的秒数。
// 返回值：true-成功；false-失败，失败有两种情况：1）等待超时；2）socket连接已不可用
```
```c++
bool tcpread(const int sockfd,string &buffer,const int itimeout=0);                            // 读取文本数据。
bool tcpread(const int sockfd,void *buffer,const int ibuflen,const int itimeout=0);     // 读取二进制数据
```
```c++
// 向socket的对端发送数据。
// sockfd：可用的socket连接。
// buffer：待发送数据缓冲区的地址。
// ibuflen：待发送数据的字节数。
// 返回值：true-成功；false-失败，如果失败，表示socket连接已不可用
```
```c++
bool tcpwrite(const int sockfd,const string &buffer);                             // 写入文本数据。
bool tcpwrite(const int sockfd,const void *buffer,const int ibuflen);      // 写入二进制数据
```
```c++

// 从已经准备好的socket中读取数据。
// sockfd：已经准备好的socket连接。
// buffer：存放数据的地址。
// n：本次打算读取数据的字节数。
// 返回值：成功接收到n字节的数据后返回true，socket连接不可用返回false
```
```c++
bool readn(const int sockfd,char *buffer,const size_t n);
```
```c++
// 向已经准备好的socket中写入数据。
// sockfd：已经准备好的socket连接。
// buffer：待写入数据的地址。
// n：待写入数据的字节数。
// 返回值：成功写入完n字节的数据后返回true，socket连接不可用返回false
```

```c++
bool writen(const int sockfd,const char *buffer,const size_t n);

```

```c++
- send函数:
    - 功能是把待发送的数据拷贝到发送缓冲区。

    - 返回值是已拷贝的字节数，正常情况下，与待发送数据的字节数相同。

    - 如果发送缓冲区的空间不足，则返回本次已拷贝的字节数。
```


## **实现**
```c++
bool ctcpserver::initserver(const unsigned int port,const int backlog)
{
    // 如果服务端的socket>0，关掉它，这种处理方法没有特别的原因，不要纠结。
    if (m_listenfd > 0) { ::close(m_listenfd); m_listenfd=-1; }

    if ( (m_listenfd = socket(AF_INET,SOCK_STREAM,0))<=0) return false;

    // 忽略SIGPIPE信号，防止程序异常退出。
    // 如果往已关闭的socket继续写数据，会产生SIGPIPE信号，它的缺省行为是终止程序，所以要忽略它。
    signal(SIGPIPE,SIG_IGN);   

    // 打开SO_REUSEADDR选项，当服务端连接处于TIME_WAIT状态时可以再次启动服务器，
    // 否则bind()可能会不成功，报：Address already in use。
    int opt = 1; 
    setsockopt(m_listenfd,SOL_SOCKET,SO_REUSEADDR,&opt,sizeof(opt));    

    memset(&m_servaddr,0,sizeof(m_servaddr));
    m_servaddr.sin_family = AF_INET;
    m_servaddr.sin_addr.s_addr = htonl(INADDR_ANY);   // 任意ip地址。
    m_servaddr.sin_port = htons(port);
    if (bind(m_listenfd,(struct sockaddr *)&m_servaddr,sizeof(m_servaddr)) != 0 )
    {
        closelisten(); return false;
    }

    if (listen(m_listenfd,backlog) != 0 )
    {
        closelisten(); return false;
    }

    return true;
}

```

```c++
bool ctcpclient::connect(const string &ip,const int port)
{
    // 如果已连接到服务端，则断开，这种处理方法没有特别的原因，不要纠结。
    if (m_connfd!=-1) { ::close(m_connfd); m_connfd=-1; }
 
    // 忽略SIGPIPE信号，防止程序异常退出。
    // 如果send到一个disconnected socket上，内核就会发出SIGPIPE信号。这个信号
    // 的缺省处理方法是终止进程，大多数时候这都不是我们期望的。我们重新定义这
    // 个信号的处理方法，大多数情况是直接屏蔽它。
    signal(SIGPIPE,SIG_IGN);   

    m_ip=ip;
    m_port=port;

    struct hostent* h;
    struct sockaddr_in servaddr;

    if ( (m_connfd = socket(AF_INET,SOCK_STREAM,0) ) < 0) return false;

    if ( !(h = gethostbyname(m_ip.c_str())) )
    {
        ::close(m_connfd);  m_connfd=-1; return false;
    }

    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(m_port);  // 指定服务端的通讯端口
    memcpy(&servaddr.sin_addr,h->h_addr,h->h_length);

    if (::connect(m_connfd, (struct sockaddr *)&servaddr,sizeof(servaddr)) != 0)
    {
        ::close(m_connfd);  m_connfd=-1; return false;
    }

    return true;
}
```
```c++
bool ctcpserver::accept()
{
    if (m_listenfd==-1) return false;

    int m_socklen = sizeof(struct sockaddr_in);
    if ((m_connfd=::accept(m_listenfd,(struct sockaddr *)&m_clientaddr,(socklen_t*)&m_socklen)) < 0)
        return false;

    return true;
}

```
```c++
char *ctcpserver::getip()
{
    return(inet_ntoa(m_clientaddr.sin_addr));
}

```
```c++

写
```
```c++
bool ctcpserver::write(const string &buffer)
{
    if (m_connfd==-1) return false;

    return(tcpwrite(m_connfd,buffer));
}
```
```c++

bool ctcpserver::write(const void *buffer,const int ibuflen)  // 发送二进制数据。
{
    if (m_connfd==-1) return false;

    return(tcpwrite(m_connfd,(char*)buffer,ibuflen));
}

```
```c++
bool ctcpclient::write(const string &buffer)
{
    if (m_connfd==-1) return false;

    return(tcpwrite(m_connfd,buffer));
}


```
```c++

bool ctcpclient::write(const void *buffer,const int ibuflen)
{
    if (m_connfd==-1) return false;

    return(tcpwrite(m_connfd,(char*)buffer,ibuflen));
}
```


```c++
bool tcpwrite(const int sockfd,const string &buffer)      // 发送文本数据。
{
    if (sockfd==-1) return false;

    int buflen=buffer.size();

    // 先发送报头（报文长度）可解决粘包分包问题
    if (writen(sockfd,(char*)&buflen,4) == false) return false;

    // 再发送报文体。
    if (writen(sockfd,buffer.c_str(),buflen) == false) return false;

    return true;
}

```

```c++
bool tcpwrite(const int sockfd,const void *buffer,const int ibuflen)        // 发送二进制数据。
{
    if (sockfd==-1) return false;

    if (writen(sockfd,(char*)buffer,ibuflen) == false) return false;

    return true;
}

```


```c++
bool writen(const int sockfd,const char *buffer,const size_t n)
{
    int nleft=n;       // 剩余需要写入的字节数。
    int idx=0;          // 已成功写入的字节数。
    int nwritten;      // 每次调用send()函数写入的字节数。
  
    while(nleft > 0 )
    {    
      if ( (nwritten=send(sockfd,buffer+idx,nleft,0)) <= 0) return false;      
    
      nleft=nleft-nwritten;
      idx=idx+nwritten;
    }

    return true;
}

```

```c++
读

```
```c++
bool ctcpserver::read(void *buffer,const int ibuflen,const int itimeout)   // 接收二进制数据。
{
    if (m_connfd==-1) return false;

    return(tcpread(m_connfd,buffer,ibuflen,itimeout));
}
```
```c++
bool ctcpserver::read(string &buffer,const int itimeout)  // 接收文本数据。
{
    if (m_connfd==-1) return false;

    return(tcpread(m_connfd,buffer,itimeout));
}
```
```c++
bool ctcpclient::read(void *buffer,const int ibuflen,const int itimeout)   // 接收二进制数据。
{
    if (m_connfd==-1) return false;

    return(tcpread(m_connfd,buffer,ibuflen,itimeout));
}
```
```c++
bool ctcpclient::read(string &buffer,const int itimeout)  // 接收文本数据。
{
    if (m_connfd==-1) return false;

    return(tcpread(m_connfd,buffer,itimeout));
}
```
```c++
bool tcpread(const int sockfd,void *buffer,const int ibuflen,const int itimeout)    // 接收二进制数据。
{
    if (sockfd==-1) return false;

    // 如果itimeout>0，表示需要等待itimeout秒，如果itimeout秒后还没有数据到达，返回false。
    if (itimeout>0)
    {
        struct pollfd fds;
        fds.fd=sockfd;
        fds.events=POLLIN;
        if ( poll(&fds,1,itimeout*1000) <= 0 ) return false;
    }

    // 如果itimeout==-1，表示不等待，立即判断socket的缓冲区中是否有数据，如果没有，返回false。
    if (itimeout==-1)
    {
        struct pollfd fds;
        fds.fd=sockfd;
        fds.events=POLLIN;
        if ( poll(&fds,1,0) <= 0 ) return false;
    }

    // 读取报文内容。
    if (readn(sockfd,(char*)buffer,ibuflen) == false) return false;

    return true;
}
```
```c++
bool tcpread(const int sockfd,string &buffer,const int itimeout)    // 接收文本数据。
{
    if (sockfd==-1) return false;

    // 如果itimeout>0，表示等待itimeout秒，如果itimeout秒后接收缓冲区中还没有数据，返回false。
    if (itimeout>0)
    {
        struct pollfd fds;
        fds.fd=sockfd;
        fds.events=POLLIN;
        if ( poll(&fds,1,itimeout*1000) <= 0 ) return false;
    }

    // 如果itimeout==-1，表示不等待，立即判断socket的接收缓冲区中是否有数据，如果没有，返回false。
    if (itimeout==-1)
    {
        struct pollfd fds;
        fds.fd=sockfd;
        fds.events=POLLIN;
        if ( poll(&fds,1,0) <= 0 ) return false;
    }

    int buflen=0;

    // 先读取报文长度，4个字节。
    if (readn(sockfd,(char*)&buflen,4) == false) return false;

    buffer.resize(buflen);   // 设置buffer的大小。

    // 再读取报文内容。
    if (readn(sockfd,&buffer[0],buflen) == false) return false;

    return true;
}


```
```c++

bool readn(const int sockfd,char *buffer,const size_t n)
{
    int nleft=n;    // 剩余需要读取的字节数。
    int idx=0;       // 已成功读取的字节数。
    int nread;       // 每次调用recv()函数读到的字节数。

    while(nleft > 0)
    {
        if ( (nread=recv(sockfd,buffer+idx,nleft,0)) <= 0) return false;

        idx=idx+nread;
        nleft=nleft-nread;
    }

    return true;
}

```
## **测试**
```c++


```

