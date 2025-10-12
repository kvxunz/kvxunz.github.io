# **网络通信**
``` mermaid
%%{init: {"flowchart": {"htmlLabels": false}} }%%
flowchart TD



subgraph "`**服务器端**`"
  6["`socket()
  创建流式socket-B`"]
  7["`bind()
  绑定服务器ip地址和端口号`"]
  8["`listen()
  监听连接请求`"]
  9["`accept()
  受理客户端的连接`"]
  10["`recv()
  接收数据服务器端`"]
  11["`send()
  发送数据服务器端`"]
  12["`recv()
  接收数据服务器端`"]
  13["`close()
  关闭socket连接
  释放资源服务器端`"]

  6 --> 7
  7 --> 8
  8 --socket-B--> 9
  9 --> 10  
  10 --> 11
  11 --> 12
  12 --> 13
end

subgraph "`**建立连接**`"
  14["`建立连接`"]
  
  14 <--> 8
  
end

subgraph "`**建立通信**`"
  15["`socket-A套接字`"]
  15<-->9
end

subgraph "`**客户端**`"
  1["`socket()
  创建流式socket-A`"]
  2["`connect()
  连接服务器`"]
  3["`send()
  发送数据服务器端`"]
  4["`recv()
  接收数据服务器端`"]
  5["`close()
  关闭socket连接
  释放资源客户端`"]

  1 --> 2
  2 --> 3
  3 --> 4
  4 --> 5
end
  2<-->14
  3-->10
  11-->4
  5-->12


```


## **Socket-客户端**
1. 创建一个用于通信的套接字（fd）
2. 连接服务器，需要指定连接的服务器的 IP 和 端口
3. 连接成功了，客户端可以直接和服务器通信
    - 接收数据
    - 发送数据
4. 通信结束，断开连接

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
using namespace std;
 
int main(int argc,char *argv[])
{
  if (argc!=3)
  {
    cout << "Using:./demo5 服务端的IP 服务端的端口\nExample:./demo5 192.168.101.138 5005\n\n"; 
    return -1;
  }

  // 第1步：创建客户端的socket。  
  int sockfd = socket(AF_INET,SOCK_STREAM,0);
  if (sockfd==-1)
  {
    perror("socket"); return -1;
  }
 
  // 第2步：向服务器发起连接请求。 
  struct sockaddr_in servaddr;               // 用于存放协议、端口和IP地址的结构体。
  memset(&servaddr,0,sizeof(servaddr));
  servaddr.sin_family = AF_INET;             // ①协议族，固定填AF_INET。
  servaddr.sin_port = htons(atoi(argv[2]));  // ②指定服务端的通信端口。

  struct hostent* h;                         // 用于存放服务端IP地址(大端序)的结构体的指针。
  if ( (h = gethostbyname(argv[1])) == nullptr )  // 把域名/主机名/字符串格式的IP转换成结构体。
  { 
    cout << "gethostbyname failed.\n" << endl; close(sockfd); return -1;
  }
  memcpy(&servaddr.sin_addr,h->h_addr,h->h_length); // ③指定服务端的IP(大端序)。
  
  //servaddr.sin_addr.s_addr=inet_addr(argv[1]); // ③指定服务端的IP，只能用IP，不能用域名和主机名。
  if (connect(sockfd,(struct sockaddr *)&servaddr,sizeof(servaddr))==-1)  // 向服务端发起连接清求。
  { 
    perror("connect"); close(sockfd); return -1; 
  }
  
  // 第3步：与服务端通讯，客户发送一个请求报文后等待服务端的回复，收到回复后，再发下一个请求报文。
  char buffer[1024];
  for (int ii=0;ii<10;ii++)  // 循环3次，将与服务端进行三次通讯。
  {
    int iret;
    memset(buffer,0,sizeof(buffer));
    sprintf(buffer,"这是第%d个超级女生，编号%03d。",ii+1,ii+1);  // 生成请求报文内容。
    // 向服务端发送请求报文。
    if ( (iret=send(sockfd,buffer,strlen(buffer),0))<=0)
    { 
      perror("send"); break; 
    }
    cout << "发送：" << buffer << endl;

    memset(buffer,0,sizeof(buffer));
    // 接收服务端的回应报文，如果服务端没有发送回应报文，recv()函数将阻塞等待。
    if ( (iret=recv(sockfd,buffer,sizeof(buffer),0))<=0)
    {
       cout << "iret=" << iret << endl; break;
    }
    cout << "接收：" << buffer << endl;

    sleep(1);
  }
 
  // 第4步：关闭socket，释放资源。
  close(sockfd);
}

```


## **Socket-服务端**
1. 创建一个用于监听的套接字
    - 监听：监听有客户端的连接
    - 套接字：这个套接字其实就是一个文件描述符
2. 将这个监听文件描述符和本地的IP和端口绑定（IP和端口就是服务器的地址信息）
    - 客户端连接服务器的时候使用的就是这个IP和端口
3. 设置监听，监听的fd开始工作
4. 阻塞等待，当有客户端发起连接，解除阻塞，接受客户端的连接，会得到一个和客户端通信的套接字
（fd）
5. 通信
        - 接收数据
        - 发送数据
6. 通信结束，断开连接

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
using namespace std;
 
int main(int argc,char *argv[])
{
  if (argc!=2)
  {
    cout << "Using:./demo6 通讯端口\nExample:./demo6 5005\n\n";   // 端口大于1024，不与其它的重复。
    cout << "注意：运行服务端程序的Linux系统的防火墙必须要开通5005端口。\n";
    cout << "      如果是云服务器，还要开通云平台的访问策略。\n\n";
    return -1;
  }

  // 第1步：创建服务端的socket。 
  int listenfd = socket(AF_INET,SOCK_STREAM,0);
  if (listenfd==-1) 
  { 
    perror("socket"); return -1; 
  }
  
  // 第2步：把服务端用于通信的IP和端口绑定到socket上。 
  struct sockaddr_in servaddr;                // 用于存放协议、端口和IP地址的结构体。
  memset(&servaddr,0,sizeof(servaddr));
  servaddr.sin_family=AF_INET;                // ①协议族，固定填AF_INET。
  servaddr.sin_port=htons(atoi(argv[1]));     // ②指定服务端的通信端口。
  servaddr.sin_addr.s_addr=htonl(INADDR_ANY); // ③如果操作系统有多个IP，全部的IP都可以用于通讯。
  //servaddr.sin_addr.s_addr=inet_addr("192.168.101.138"); // ③指定服务端用于通讯的IP(大端序)。
  // 绑定服务端的IP和端口。
  if (bind(listenfd,(struct sockaddr *)&servaddr,sizeof(servaddr))==-1)
  { 
    perror("bind"); close(listenfd); return -1; 
  }
 
  // 第3步：把socket设置为可连接（监听）的状态。
  if (listen(listenfd,5) == -1 ) 
  { 
    perror("listen"); close(listenfd); return -1; 
  }
 
  // 第4步：受理客户端的连接请求，如果没有客户端连上来，accept()函数将阻塞等待。
  int clientfd=accept(listenfd,0,0);
  if (clientfd==-1)
  {
    perror("accept"); close(listenfd); return -1; 
  }

  cout << "客户端已连接。\n";
 
  // 第5步：与客户端通信，接收客户端发过来的报文后，回复ok。
  char buffer[1024];
  while (true)
  {
    int iret;
    memset(buffer,0,sizeof(buffer));
    // 接收客户端的请求报文，如果客户端没有发送请求报文，recv()函数将阻塞等待。
    // 如果客户端已断开连接，recv()函数将返回0。
    if ( (iret=recv(clientfd,buffer,sizeof(buffer),0))<=0) 
    {
       cout << "iret=" << iret << endl;  break;   
    }
    cout << "接收：" << buffer << endl;
 
    strcpy(buffer,"ok");  // 生成回应报文内容。
    // 向客户端发送回应报文。
    if ( (iret=send(clientfd,buffer,strlen(buffer),0))<=0) 
    { 
      perror("send"); break; 
    }
    cout << "发送：" << buffer << endl;
  }
 
  // 第6步：关闭socket，释放资源。
  close(listenfd);   // 关闭服务端用于监听的socket。
  close(clientfd);   // 关闭客户端连上来的socket。
}

```