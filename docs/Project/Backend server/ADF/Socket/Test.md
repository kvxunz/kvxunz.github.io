
#

## **接收文本数据**


```c++
/*
 *  ctcpclient类传输文本数据（网络通讯的客户端） 
*/
#include "../_public.h"
using namespace std;
using namespace idc;

int main(int argc,char *argv[])
{
    if (argc!=3)
    {
        printf("Using:./demo44 ip port\n");
        printf("Sample:./demo44 192.168.150.128 5005\n");
        return -1;
    }

    ctcpclient tcpclient;
    if (tcpclient.connect(argv[1],atoi(argv[2]))==false)     // 向服务端发起连接请求。
    {
        printf("tcpclient.connect(%s,%s) failed.\n",argv[1],argv[2]); return -1;
    }

    string sendbuf,recvbuf;

    for (int ii=0;ii<10;ii++)
    {
        sendbuf=sformat("这是第%d个超级女生。",ii);

        if (tcpclient.write(sendbuf)==false)        // 向服务端发送请求报文。
        {
            printf("tcpclient.write() failed.\n"); break;
        }
        cout << "发送：" << sendbuf << endl;

        sleep(1);

        if (tcpclient.read(recvbuf)==false)         // 接收服务端的回应报文。
        {
            printf("tcpclient.read() failed.\n"); break;
        }
        cout << "接收：" << recvbuf << endl;
    }
}

```

```c++
/*
 * ctcpserver类传输文本数据。（网络通讯的服务端） 
*/
#include "../_public.h"
using namespace std;
using namespace idc;

int main(int argc,char *argv[])
{
    if (argc!=2)
    {
        printf("Using:./demo45 port\n");
        printf("Sample:./demo45 5005\n");
        return -1;
    }

    ctcpserver tcpserver;
    if (tcpserver.initserver(atoi(argv[1]))==false)        // 服务端初始化。
    {
        printf("tcpserver.initserver(%s) failed.\n",argv[1]); return -1;
    }

    if (tcpserver.accept()==false)                                // 等待客户端的连接。
    {
        printf("accept() failed.\n"); return -1;
    }
    cout << "客户端已连接(" << tcpserver.getip() << ")。\n";

    string sendbuf,recvbuf;

    while (true)
    {
        if (tcpserver.read(recvbuf)==false)          // 接收客户端的请求报文。
        {
            printf("tcpserver.read() failed.\n"); break;
        }
        cout << "接收：" << recvbuf << endl;

        sendbuf="ok";
        if (tcpserver.write(sendbuf)==false)        // 向客户端发送回应报文。
        {
            printf("tcpserver.write() failed.\n"); break;
        }
        cout << "发送：" << sendbuf << endl;
    }
}


```




## **接收二进制数据**
```c++
/*
 *  ctcpclient类传输二进制数据（网络通讯的客户端） 
*/
#include "../_public.h"
using namespace std;
using namespace idc;

int main(int argc,char *argv[])
{
    if (argc!=3)
    {
        printf("Using:./demo46 ip port\n");
        printf("Sample./demo46 192.168.150.128 5005\n");
        return -1;
    }

    ctcpclient tcpclient;
    if (tcpclient.connect(argv[1],atoi(argv[2]))==false)     // 向服务端发起连接请求。
    {
        printf("tcpclient.connect(%s,%s) failed.\n",argv[1],argv[2]); return -1;
    }

    struct st_girl   // 超女结构体。
    {
        int bh;
        char name[31];
    }stgirl;

    string recvbuf;

    for (int ii=0;ii<10;ii++)
    {
        memset(&stgirl,0,sizeof(stgirl));
        stgirl.bh=ii;
        sprintf(stgirl.name,"西施%d",ii);

        if (tcpclient.write(&stgirl,sizeof(stgirl))==false)        // 向服务端发送请求数据。
        {
            printf("tcpclient.write() failed.\n"); break;
        }
        cout << sformat("发送：bh=%d,name=%s\n",stgirl.bh,stgirl.name);

        sleep(1);

        if (tcpclient.read(recvbuf)==false)         // 接收服务端的回应报文。
        {
            printf("tcpclient.read() failed.\n"); break;
        }
        cout << "接收：" << recvbuf << endl;
    }
}

```

```c++
/*
 *  ctcpserver类传输二进制数据。（网络通讯的服务端）
*/
#include "../_public.h"
using namespace std;
using namespace idc;

int main(int argc,char *argv[])
{
    if (argc!=2)
    {
        printf("Using:./demo47 port\n");
        printf("Sample:./demo47 5005\n");
        return -1;
    }

    ctcpserver tcpserver;
    if (tcpserver.initserver(atoi(argv[1]))==false)        // 服务端初始化。
    {
        printf("tcpserver.initserver(%s) failed.\n",argv[1]); return -1;
    }

    if (tcpserver.accept()==false)                                // 等待客户端的连接。
    {
        printf("accept() failed.\n"); return -1;
    }
    cout << "客户端已连接(" << tcpserver.getip() << ")。\n";

    struct st_girl
    {
        int bh;
        char name[31];
    }stgirl;

    string sendbuf;

    while (true)
    {
        if (tcpserver.read(&stgirl,sizeof(stgirl))==false)          // 接收客户端的请求数据。
        {
            printf("tcpserver.read() failed.\n"); break;
        }
        cout << sformat("接收：bh=%d,name=%s\n",stgirl.bh,stgirl.name);

        sendbuf="ok";
        if (tcpserver.write("ok")==false)        // 向客户端发送回应报文。
        {
            printf("tcpserver.write() failed.\n"); break;
        }
        cout << "发送：" << sendbuf << endl;
    }
}

```
