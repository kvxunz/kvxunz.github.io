## **文件操作**

- Linux底层文件的操作-创建文件并写入数据
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
using namespace std;

int main()
{
  int fd;    // 定义一个文件描述符/文件句柄。

  // 打开文件，注意，如果创建后的文件没有权限，可以手工授权chmod 777 data.txt。
  fd=open("data.txt",O_CREAT|O_RDWR|O_TRUNC);
  if (fd==-1)
  {
    perror("open(data.txt)"); return -1;
  }

  printf("文件描述符fd=%d\n",fd);

  char buffer[1024];
  strcpy(buffer,"我是一只傻傻鸟。\n");

  if (write(fd,buffer,strlen(buffer))==-1)    // 把数据写入文件。
  {
    perror("write()"); return -1;
  }

  close(fd);  // 关闭文件。
}

```

- Linux底层文件的操作-读取文件数据
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
  int fd;    // 定义一个文件描述符/文件句柄。

  fd=open("data.txt",O_RDONLY); // 打开文件。
  if (fd==-1)
  {
    perror("open(data.txt)"); return -1;
  }

  printf("文件描述符fd=%d\n",fd);

  char buffer[1024];
  memset(buffer,0,sizeof(buffer));
  if (read(fd,buffer,sizeof(buffer))==-1)    // 从文件中读取数据。
  {
    perror("write()"); return -1;
  }

  printf("%s",buffer);

  close(fd);  // 关闭文件。
}

```


## **文件描述符的分配规则**
- `/proc/进程id/fd`目录中，存放了每个进程打开的`fd`
- `Linux`进程默认打开了三个文件描述符：`0`-标准输入（键盘），`1`-标准输出 (显示器)，`2`-标准错误 (显示器) , `cin cout cerr`
- 文件描述符的分配规则是：找到最小的，没有被占用的文件描述符
```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include<iostream>
using namespace std;

int main()
{
  // 关闭文件描述符 0,1,2
  close(0);
  close(1);
  close(2);

  // int i =0;
  // cin>>i;
  // cout<<"i="<<i<<endl;
  // cerr<<"i="<<i<<endl;

  int fd;    // 定义一个文件描述符/文件句柄。

  // 打开文件，注意，如果创建后的文件没有权限，可以手工授权chmod 777 data.txt。
  fd=open("data.txt",O_CREAT|O_RDWR|O_TRUNC);  //由于关闭0，1，2，所以文件描述符从最小的0开始
  if (fd==-1)
  {
    perror("open(data.txt)"); return -1;
  }

  printf("文件描述符fd=%d\n",fd);

  char buffer[1024];
  strcpy(buffer,"我是一只傻傻鸟。\n");

  if (write(fd,buffer,strlen(buffer))==-1)    // 把数据写入文件。
  {
    perror("write()"); return -1;
  }
  sleep(100);
  close(fd);  // 关闭文件。
}
```

- 对``Linux来说，`socket`操作与文件操作没有区别

- 在网络传输数据的过程中，可以使用文件的`I/O`函数

- 文件描述符是`Linux`分配给文件或`socket`的整数