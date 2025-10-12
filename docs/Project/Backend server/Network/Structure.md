
## **Sockaddr结构体**
- 存放协议族、端口和地址信息，客户端和connect()函数和服务端的bind()函数需要这个结构体

```c++
struct sockaddr {
  unsigned short sa_family;	// 协议族，与socket()函数的第一个参数相同，填AF_INET。
  unsigned char sa_data[14];	// 14字节的端口和地址。
};
typedef unsigned short int sa_family_t;
```

## **Sockaddr_in结构体**
- sockaddr结构体是为了统一地址结构的表示方法，统一接口函数，但是，操作不方便，所以定义了等价的sockaddr_in结构体，它的大小与sockaddr相同，可以强制转换成sockaddr

```c++
struct sockaddr_in {  
  unsigned short sin_family;	// 协议族，与socket()函数的第一个参数相同，填AF_INET。
  unsigned short sin_port;		// 16位端口号，大端序。用htons(整数的端口)转换。
  struct in_addr sin_addr;		// IP地址的结构体。192.168.101.138
  unsigned char sin_zero[8];	// 未使用，为了保持与struct sockaddr一样的长度而添加。
};
struct in_addr {				// IP地址的结构体。
  unsigned int s_addr;		// 32位的IP地址，大端序。
};

```


## **字符串IP与大端序IP的转换**
### **客户端**
-  Gethostbyname函数，根据域名/主机名/字符串IP获取大端序IP

    ```c++
    struct hostent *gethostbyname(const char *name);
    struct hostent { 
      char *h_name;     	// 主机名。
      char **h_aliases;    	// 主机所有别名构成的字符串数组，同一IP可绑定多个域名。 
      short h_addrtype; 	// 主机IP地址的类型，例如IPV4（AF_INET）还是IPV6。
      short h_length;     	// 主机IP地址长度，IPV4地址为4，IPV6地址则为16。
      char **h_addr_list; 	// 主机的ip地址，以网络字节序存储。 
    };
    #define h_addr h_addr_list[0] 	// for backward compatibility.

    ```

  - 转换后，用以下代码把大端序的地址复制到`sockaddr_in`结构体的`sin_addr`成员中。
      - `memcpy(&servaddr.sin_addr,h->h_addr,h->h_length);`


### **服务端**
- C语言提供了几个库函数，用于字符串格式的IP和大端序IP的互相转换

    ```c++
    typedef unsigned int in_addr_t;    // 32位大端序的IP地址。

    // 把字符串格式的IP转换成大端序的IP，转换后的IP赋给sockaddr_in.in_addr.s_addr。
    in_addr_t inet_addr(const char *cp); 

    // 把字符串格式的IP转换成大端序的IP，转换后的IP将填充到sockaddr_in.in_addr成员。
    int inet_aton(const char *cp, struct in_addr *inp);	

    // 把大端序IP转换成字符串格式的IP，用于在服务端程序中解析客户端的IP地址。
    char *inet_ntoa(struct in_addr in);

    ```


