## **封装socket通讯的客户端**
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

class ctcpclient         // TCP通讯的客户端类。
{
private:
  int m_clientfd;        // 客户端的socket，-1表示未连接或连接已断开；>=0表示有效的socket。
  string m_ip;           // 服务端的IP/域名。
  unsigned short m_port; // 通讯端口。
public:
  ctcpclient():m_clientfd(-1) {}

  // 向服务端发起连接请求，成功返回true，失败返回false。
  bool connect(const string &in_ip,const unsigned short in_port)
  {
    if (m_clientfd!=-1) return false; // 如果socket已连接，直接返回失败。

    m_ip=in_ip; m_port=in_port;       // 把服务端的IP和端口保存到成员变量中。

    // 第1步：创建客户端的socket。
    if ( (m_clientfd = socket(AF_INET,SOCK_STREAM,0))==-1) return false;

    // 第2步：向服务器发起连接请求。
    struct sockaddr_in servaddr;               // 用于存放协议、端口和IP地址的结构体。
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family = AF_INET;             // ①协议族，固定填AF_INET。
    servaddr.sin_port = htons(m_port);         // ②指定服务端的通信端口。

    struct hostent* h;                         // 用于存放服务端IP地址(大端序)的结构体的指针。
    if ((h=gethostbyname(m_ip.c_str()))==nullptr ) // 把域名/主机名/字符串格式的IP转换成结构体。
    {
      ::close(m_clientfd); m_clientfd=-1; return false;
    }
    memcpy(&servaddr.sin_addr,h->h_addr,h->h_length); // ③指定服务端的IP(大端序)。

    // 向服务端发起连接清求。
    if (::connect(m_clientfd,(struct sockaddr *)&servaddr,sizeof(servaddr))==-1)  
    {
      ::close(m_clientfd); m_clientfd=-1; return false;
    }

    return true;
  }

  // 向服务端发送报文，成功返回true，失败返回false。
  bool send(const string &buffer)   // buffer不要用const char *//因为前者支持C风格字符串也支持string，后者只支持C风格字符串
  {
    if (m_clientfd==-1) return false; // 如果socket的状态是未连接，直接返回失败。
        //::send(m_clientfd,buffer.c_str(),buffer.length(),0) //语义不准确，buffer可能是字符串，也可能是一块内存
    if ((::send(m_clientfd,buffer.data(),buffer.size(),0))<=0) return false;

    return true;
  }

  // 接收服务端的报文，成功返回true，失败返回false。
  // buffer-存放接收到的报文的内容，maxlen-本次接收报文的最大长度。
  bool recv(string &buffer,const size_t maxlen)
  { // 如果直接操作string对象的内存，那么string对象的自动管理内存的功能失效，
    // 因为string对象中有一个成员变量存放了数据的大小，如果通过string类的成员函数来操作string对象，那么会自动设置数据的大小
    // 必须保证：1)不能越界；2）操作后手动设置数据的大小。
    buffer.clear();         // 清空容器。
    buffer.resize(maxlen);  // 设置容器的大小为maxlen。
    int readn=::recv(m_clientfd,&buffer[0],buffer.size(),0);  // 直接操作buffer的内存。
                            //获取string对象内存空间的首地址，buffer[0]，buffer.data(),buffer.c_str(),后两个获取到的是const
                            // -1 失败; 0 socket断开; >0 接收到的字节数
    if (readn<=0) { buffer.clear(); return false; }
    buffer.resize(readn);   // 重置buffer的实际大小。

    return true;
  }

  // 断开与服务端的连接。
  bool close()
  {
    if (m_clientfd==-1) return false; // 如果socket的状态是未连接，直接返回失败。

    ::close(m_clientfd);
    m_clientfd=-1;
    return true;
  }

 ~ctcpclient(){ close(); }
};

int main(int argc,char *argv[])
{
  if (argc!=3)
  {
    cout << "Using:./demo7 服务端的IP 服务端的端口\nExample:./demo7 192.168.101.138 5005\n\n"; 
    return -1;
  }

  ctcpclient tcpclient;
  if (tcpclient.connect(argv[1],atoi(argv[2]))==false)  // 向服务端发起连接请求。
  {
    perror("connect()"); return -1;
  }

  // 第3步：与服务端通讯，客户发送一个请求报文后等待服务端的回复，收到回复后，再发下一个请求报文。
  string buffer;
  for (int ii=0;ii<10;ii++)  // 循环3次，将与服务端进行三次通讯。
  {
    buffer="这是第"+to_string(ii+1)+"个超级女生，编号"+to_string(ii+1)+"。";
    // 向服务端发送请求报文。
    if (tcpclient.send(buffer)==false)
    { 
      perror("send"); break; 
    }
    cout << "发送：" << buffer << endl;

    // 接收服务端的回应报文，如果服务端没有发送回应报文，recv()函数将阻塞等待。
    if (tcpclient.recv(buffer,1024)==false)
    {
      perror("recv()"); break;
    }
    cout << "接收：" << buffer << endl;

    sleep(1);
  }
}
```





## **封装socket通讯的服务端**


```c++
#include <iostream> 
#include <cstdio>
#include <cstring>
#include <unistd.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
using namespace std;

class ctcpserver         // TCP通讯的服务端类。
{
private:
  int    m_listenfd;        // 监听的socket，-1表示未初始化。
  int    m_clientfd;        // 客户端连上来的socket，-1表示客户端未连接。
  string m_clientip;        // 客户端字符串格式的IP。
  unsigned short m_port;    // 服务端用于通讯的端口。
public:
  ctcpserver():m_listenfd(-1),m_clientfd(-1) {}

  // 初始化服务端用于监听的socket。
  bool initserver(const unsigned short in_port)
  {
    // 第1步：创建服务端的socket。 
    if ( (m_listenfd=socket(AF_INET,SOCK_STREAM,0))==-1) return false;

    m_port=in_port;
  
    // 第2步：把服务端用于通信的IP和端口绑定到socket上。 
    struct sockaddr_in servaddr;                // 用于存放协议、端口和IP地址的结构体。
    memset(&servaddr,0,sizeof(servaddr));
    servaddr.sin_family=AF_INET;                // ①协议族，固定填AF_INET。
    servaddr.sin_port=htons(m_port);            // ②指定服务端的通信端口。
    servaddr.sin_addr.s_addr=htonl(INADDR_ANY); // ③如果操作系统有多个IP，全部的IP都可以用于通讯。

    // 绑定服务端的IP和端口（为socket分配IP和端口）。
    if (bind(m_listenfd,(struct sockaddr *)&servaddr,sizeof(servaddr))==-1)
    { 
      close(m_listenfd); m_listenfd=-1; return false; 
    }
 
    // 第3步：把socket设置为可连接（监听）的状态。
    if (listen(m_listenfd,5) == -1 ) 
    { 
      close(m_listenfd); m_listenfd=-1; return false;
    }

    return true;
  }

  // 受理客户端的连接（从已连接的客户端中取出一个客户端），
  // 如果没有已连接的客户端，accept()函数将阻塞等待。
  bool accept()
  {
    struct sockaddr_in caddr;        // 客户端的地址信息。  
    socklen_t addrlen=sizeof(caddr); // struct sockaddr_in的大小。
    if ((m_clientfd=::accept(m_listenfd,(struct sockaddr *)&caddr,&addrlen))==-1) 
        return false;

    m_clientip=inet_ntoa(caddr.sin_addr);  // 把客户端的地址从大端序转换成字符串。

    return true;
  }

  // 获取客户端的IP(字符串格式)。
  const string & clientip() const
  {
    return m_clientip;
  }

  // 向对端发送报文，成功返回true，失败返回false。
  bool send(const string &buffer)   
  {
    if (m_clientfd==-1) return false;

    if ( (::send(m_clientfd,buffer.data(),buffer.size(),0))<=0) return false;
   
    return true;
  }

  // 接收对端的报文，成功返回true，失败返回false。
  // buffer-存放接收到的报文的内容，maxlen-本次接收报文的最大长度。
  bool recv(string &buffer,const size_t maxlen)
  { 
    buffer.clear();         // 清空容器。
    buffer.resize(maxlen);  // 设置容器的大小为maxlen。
    int readn=::recv(m_clientfd,&buffer[0],buffer.size(),0);  // 直接操作buffer的内存。
    if (readn<=0) { buffer.clear(); return false; }
    buffer.resize(readn);   // 重置buffer的实际大小。

    return true;
  }
  
  // 关闭监听的socket。
  bool closelisten()
  {
    if (m_listenfd==-1) return false; 

    ::close(m_listenfd);
    m_listenfd=-1;
    return true;
  }

  // 关闭客户端连上来的socket。
  bool closeclient()
  {
    if (m_clientfd==-1) return false;

    ::close(m_clientfd);
    m_clientfd=-1;
    return true;
  }

 ~ctcpserver() { closelisten(); closeclient(); }
};
 
int main(int argc,char *argv[])
{
  if (argc!=2)
  {
    cout << "Using:./demo8 通讯端口\nExample:./demo8 5005\n\n";   // 端口大于1024，不与其它的重复。
    cout << "注意：运行服务端程序的Linux系统的防火墙必须要开通5005端口。\n";
    cout << "      如果是云服务器，还要开通云平台的访问策略。\n\n";
    return -1;
  }

  ctcpserver tcpserver;
  if (tcpserver.initserver(atoi(argv[1]))==false) // 初始化服务端用于监听的socket。
  {
    perror("initserver()"); return -1;
  }

  // 受理客户端的连接（从已连接的客户端中取出一个客户端），  
  // 如果没有已连接的客户端，accept()函数将阻塞等待。
  if (tcpserver.accept()==false)
  {
    perror("accept()"); return -1;
  }
  cout << "客户端已连接(" << tcpserver.clientip() << ")。\n";

  string buffer;
  while (true)
  {
    // 接收对端的报文，如果对端没有发送报文，recv()函数将阻塞等待。
    if (tcpserver.recv(buffer,1024)==false)
    {
      perror("recv()"); break;
    }
    cout << "接收：" << buffer << endl;
 
    buffer="ok";  
    if (tcpserver.send(buffer)==false)  // 向对端发送报文。
    {
      perror("send"); break;
    }
    cout << "发送：" << buffer << endl;
  }
}
```