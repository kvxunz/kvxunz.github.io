## **共享内存**

- 线程共享进程的地址空间，如果多个线程需要访问同一块内存，用全局变量就可以了
- 在多进程中，每个进程的地址空间是独立的，不共享的，如果多个进程需要访问同一块内存，不能用全局变量，只能用共享内存
- 共享内存（Shared Memory）允许多个进程（不要求进程之间有血缘关系）访问同一块内存空间，是多个进程之间共享和传递数据最高效的方式。进程可以将共享内存连接到它们自己的地址空间中，如果某个进程修改了共享内存中的数据，其它的进程读到的数据也会改变
- 共享内存没有提供锁机制，也就是说，在某一个进程对共享内存进行读/写的时候，不会阻止其它进程对它的读/写。如果要对共享内存的读/写加锁，可以使用信号量
- Linux中提供了一组函数用于操作共享内存

## **shmget函数**
- 该函数用于创建/获取共享内存
- `int shmget(key_t key, size_t size, int shmflg);`
    - `key`	 	共享内存的键值，是一个整数`（typedef unsigned int key_t）`，一般采用十六进制，例如`0x5005`，不同共享内存的`key`不能相同
    - `size` 	共享内存的大小，以字节为单位
    - `shmflg`	共享内存的访问权限，与文件的权限一样，例如`0666|IPC_CREAT`，`0666`表示全部用户对它可读写，`IPC_CREAT`表示如果共享内存不存在，就创建它
    - 返回值：成功返回共享内存的`id`（一个非负的整数），失败返回`-1`（系统内存不足、没有权限）
    - 用`ipcs -m`可以查看系统的共享内存，包括：键值`（key）`，共享内存`id（shmid）`，拥有者`（owner）`，权限`（perms）`，大小`（bytes）`
    - 用`ipcrm -m 共享内存 id`可以手工删除共享内存

## **shmat函数**

- 该函数用于把共享内存连接到当前进程的地址空间。
- `void *shmat(int shmid, const void *shmaddr, int shmflg);`
    - `shmid`		由`shmget()`函数返回的共享内存标识
    - `shmaddr` 	指定共享内存连接到当前进程中的地址位置，通常填0，表示让系统来选择共享内存的地址
    - `shmflg`		标志位，通常填0
    - 调用成功时返回共享内存起始地址，失败返回`(void*)-1`


## **shmdt函数**
- 该函数用于将共享内存从当前进程中分离，相当于`shmat()`函数的反操作。
- `int shmdt(const void *shmaddr);`
    - `shmaddr`	`shmat()`函数返回的地址
    - 调用成功时返回`0`，失败时返回`-1`

## **shmctl函数**

- 该函数用于操作共享内存，最常用的操作是删除共享内存。
- `int shmctl(int shmid, int command, struct shmid_ds *buf);`
    - `shmid`		`shmget()`函数返回的共享内存`id`
    - `command`	操作共享内存的指令，如果要删除共享内存，填`IPC_RMID`
    - `buf`			操作共享内存的数据结构的地址，如果要删除共享内存，填`0`
    - 调用成功时返回`0`，失败时返回`-1`
    - 注意，用`root`创建的共享内存，不管创建的权限是什么，普通用户无法删除


```c++
#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
using  namespace std;

struct stgirl     // 超女结构体。
{
  int  no;        // 编号。
  char name[51];  // 姓名，注意，不能用string，因为其会动态在堆上分配内存，不属于共享内存。
                  //string对象在动态分配内存时，它会在堆上为自己的数据分配一块内存空间。
                  //这块内存空间是由string对象所属的进程或线程独占的，并不是共享的。其他进程或线程无法直接访问这个string对象的内存空间，
};

int main(int argc,char *argv[])
{
  // if (argc!=3) { cout << "Using:./demo no name\n"; return -1; }

  // 第1步：创建/获取共享内存，键值key为0x5005，也可以用其它的值。
  int shmid=shmget(0x5005, sizeof(stgirl), 0640|IPC_CREAT);
  if ( shmid ==-1 )
  {
    cout << "shmget(0x5005) failed.\n"; return -1;
  }

  cout << "shmid=" << shmid << endl;

  // 第2步：把共享内存连接到当前进程的地址空间。
  stgirl *ptr=(stgirl *)shmat(shmid,0,0);
  if ( ptr==(void *)-1 )
  {
    cout << "shmat() failed\n"; return -1;
  }

  // // 第3步：使用共享内存，对共享内存进行读/写。
  cout << "原值：no=" << ptr->no << ",name=" << ptr->name << endl;  // 显示共享内存中的原值。
  ptr->no=atoi(argv[1]);        // 对超女结构体的no成员赋值。
  strcpy(ptr->name,argv[2]);    // 对超女结构体的name成员赋值。
  // ptr->name=argv[2];
  cout << "新值：no=" << ptr->no << ",name=" << ptr->name << endl;  // 显示共享内存中的当前值。

  // // 第4步：把共享内存从当前进程中分离。
  shmdt(ptr);

  // // 第5步：删除共享内存。
  if (shmctl(shmid,IPC_RMID,0)==-1)
  {
   cout << "shmctl failed\n"; return -1;
  }
}


```
