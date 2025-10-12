<!-- ## **循环队列** -->
## ***_public.h***
```c++
#ifndef __PUBLIC_HH
#define __PUBLIC_HH 1

#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/sem.h>
using namespace std;

// 循环队列。
template <class TT, int MaxLength>
class squeue
{
private:
  bool m_inited;              // 队列被初始化标志，true-已初始化；false-未初始化。
  TT   m_data[MaxLength];     // 用数组存储循环队列中的元素。
  int  m_head;                // 队列的头指针。
  int  m_tail;                // 队列的尾指针，指向队尾元素。
  int  m_length;              // 队列的实际长度。    
  squeue(const squeue &) = delete;             // 禁用拷贝构造函数。
  squeue &operator=(const squeue &) = delete;  // 禁用赋值函数。
public:

  squeue() { init(); }  // 构造函数。

  // 循环队列的初始化操作。
  // 注意：如果用于共享内存的队列，不会调用构造函数，必须调用此函数初始化。
  void init()  
  { 
    if (m_inited!=true)      // 循环队列的初始化只能执行一次。
    { 
      m_head=0;              // 头指针。
      m_tail=MaxLength-1;    // 为了方便写代码，初始化时，尾指针指向队列的最后一个位置。
      m_length=0;            // 队列的实际长度。
      memset(m_data,0,sizeof(m_data));  // 数组元素清零。
      m_inited=true; 
    }
  }

  // 元素入队，返回值：false-失败；true-成功。
  bool push(const TT &ee)
  {
    if (full() == true)
    {
      cout << "循环队列已满，入队失败。\n"; return false;
    }

    // 先移动队尾指针，然后再拷贝数据。
    m_tail=(m_tail+1)%MaxLength;  // 队尾指针后移。
    m_data[m_tail]=ee;
    m_length++;    

    return true;
  }

  // 求循环队列的长度，返回值：>=0-队列中元素的个数。
  int  size()                   
  {
    return m_length;    
  }

  // 判断循环队列是否为空，返回值：true-空，false-非空。
  bool empty()                    
  {
    if (m_length == 0) return true;    

    return false;
  }

  // 判断循环队列是否已满，返回值：true-已满，false-未满。
  bool full()
  {
    if (m_length == MaxLength) return true;    

    return false;
  }

  // 查看队头元素的值，元素不出队。
  TT& front()
  {
    return m_data[m_head];
  }

  // 元素出队，返回值：false-失败；true-成功。
  bool pop()
  {
    if (empty() == true) return false;

    m_head=(m_head+1)%MaxLength;  // 队列头指针后移。
    m_length--;    

    return true;
  }

  // 显示循环队列中全部的元素。
  // 这是一个临时的用于调试的函数，队列中元素的数据类型支持cout输出才可用。
  void printqueue()                    
  {
    for (int ii = 0; ii < size(); ii++)
    {
      cout << "m_data[" << (m_head+ii)%MaxLength << "],value=" \
           << m_data[(m_head+ii)%MaxLength] << endl;
    }
  }
};

// 信号量。
class csemp
{
private:
  union semun  // 用于信号量操作的共同体。
  {
    int val;
    struct semid_ds *buf;
    unsigned short  *arry;
  };

  int   m_semid;         // 信号量id（描述符）。

  // 如果把sem_flg设置为SEM_UNDO，操作系统将跟踪进程对信号量的修改情况，
  // 在全部修改过信号量的进程（正常或异常）终止后，操作系统将把信号量恢复为初始值。
  // 如果信号量用于互斥锁，设置为SEM_UNDO。
  // 如果信号量用于生产消费者模型，设置为0，操作系统不会把信号量恢复为初始值
  short m_sem_flg;

  csemp(const csemp &) = delete;             // 禁用拷贝构造函数。
  csemp &operator=(const csemp &) = delete;  // 禁用赋值函数。
public:
  csemp():m_semid(-1){}                     //构造函数初始化信号量id=-1,表示信号量没有初始化
  // 如果信号量已存在，获取信号量；如果信号量不存在，则创建它并初始化为value。
  // 如果用于互斥锁，value填1，sem_flg填SEM_UNDO。
  // 如果用于生产消费者模型，value填0，sem_flg填0。
  bool init(key_t key,unsigned short value=1,short sem_flg=SEM_UNDO);
  bool wait(short value=-1);// 信号量的P操作，如果信号量的值是0，将阻塞等待，直到信号量的值大于0。
  bool post(short value=1); // 信号量的V操作。
  int  getvalue();           // 获取信号量的值，成功返回信号量的值，失败返回-1。
  bool destroy();            // 销毁信号量。
 ~csemp();
};

#endif

```

## ***_public.cpp***
```c++
#include "_public.h"

// 如果信号量已存在，获取信号量；如果信号量不存在，则创建它并初始化为value。
// 如果用于互斥锁，value填1，sem_flg填SEM_UNDO。
// 如果用于生产消费者模型，value填0，sem_flg填0。
bool csemp::init(key_t key, unsigned short value, short sem_flg) {
  if (m_semid != -1) return false;  // 如果已经初始化了，不必再次初始化。

  m_sem_flg = sem_flg;

  // 信号量的初始化不能直接用semget(key,1,0666|IPC_CREAT)
  // 因为信号量创建后，初始值是0，如果用于互斥锁，需要把它的初始值设置为1，
  // 而获取信号量则不需要设置初始值，所以，创建信号量和获取信号量的流程不同。

  // 信号量的初始化分三个步骤：
  // 1）获取信号量，如果成功，函数返回。
  // 2）如果失败，则创建信号量。
  // 3) 设置信号量的初始值。

  // 获取信号量。
  if ((m_semid = semget(key, 1, 0666)) == -1) {
    // 如果信号量不存在，创建它。
    if (errno == ENOENT) {
      // 用IPC_EXCL标志确保只有一个进程创建并初始化信号量，其它进程只能获取。
      if ((m_semid = semget(key, 1, 0666 | IPC_CREAT | IPC_EXCL)) == -1) {
        if (errno == EEXIST)  // 如果错误代码是信号量已存在，则再次获取信号量。
        {
          if ((m_semid = semget(key, 1, 0666)) == -1) {
            perror("init 1 semget()");
            return false;
          }
          return true;
        } else  // 如果是其它错误，返回失败。
        {
          perror("init 2 semget()");
          return false;
        }
      }

      // 信号量创建成功后，还需要把它初始化成value。
      union semun sem_union;
      sem_union.val = value;  // 设置信号量的初始值。
      if (semctl(m_semid, 0, SETVAL, sem_union) < 0) {
        perror("init semctl()");
        return false;
      }
    } else {
      perror("init 3 semget()");
      return false;
    }
  }

  return true;
}

// 信号量的P操作（把信号量的值减value），如果信号量的值是0，将阻塞等待，直到信号量的值大于0。
bool csemp::wait(short value) {
  if (m_semid == -1) return false;

  struct sembuf sem_b;
  sem_b.sem_num = 0;     // 信号量编号，0代表第一个信号量。
  sem_b.sem_op = value;  // P操作的value必须小于0。
  sem_b.sem_flg = m_sem_flg;
  if (semop(m_semid, &sem_b, 1) == -1) {
    perror("p semop()");
    return false;
  }

  return true;
}

// 信号量的V操作（把信号量的值减value）。
bool csemp::post(short value) {
  if (m_semid == -1) return false;

  struct sembuf sem_b;
  sem_b.sem_num = 0;     // 信号量编号，0代表第一个信号量。
  sem_b.sem_op = value;  // V操作的value必须大于0。
  sem_b.sem_flg = m_sem_flg;
  if (semop(m_semid, &sem_b, 1) == -1) {
    perror("V semop()");
    return false;
  }

  return true;
}

// 获取信号量的值，成功返回信号量的值，失败返回-1。
int csemp::getvalue() { return semctl(m_semid, 0, GETVAL); }

// 销毁信号量。
bool csemp::destroy() {
  if (m_semid == -1) return false;

  if (semctl(m_semid, 0, IPC_RMID) == -1) {
    perror("destroy semctl()");
    return false;
  }

  return true;
}

csemp::~csemp() {}

```

## ***循环队列-demo1***
```c++ 
#include "_public.h"

int main()
{
  using ElemType=int;

  squeue<ElemType,5> QQ;

  ElemType ee;      // 创建一个数据元素。

  cout << "元素（1、2、3）入队。\n";
  ee=1;  QQ.push(ee);
  ee=2;  QQ.push(ee);
  ee=3;  QQ.push(ee);

  cout << "队列的长度是" << QQ.size() << endl;
  QQ.printqueue();

  ee=QQ.front(); QQ.pop(); cout << "出队的元素值为" << ee << endl;
  ee=QQ.front(); QQ.pop(); cout << "出队的元素值为" << ee << endl;

  cout << "队列的长度是" << QQ.size() << endl;
  QQ.printqueue();

  cout << "元素（11、12、13、14、15）入队。\n";
  ee=11;  QQ.push(ee);
  ee=12;  QQ.push(ee);
  ee=13;  QQ.push(ee);
  ee=14;  QQ.push(ee);
  ee=15;  QQ.push(ee);

  cout << "队列的长度是" << QQ.size() << endl;
  QQ.printqueue();
}
```


## ***共享内存循环队列-demo2***
```c++
#include "_public.h"

int main()
{
  using ElemType=int;

  // 初始化共享内存。
  int shmid=shmget(0x5005, sizeof(squeue<ElemType,5>), 0640|IPC_CREAT);
  if ( shmid ==-1 )
  {
    cout << "shmget(0x5005) failed.\n"; return -1;
  }

  // 把共享内存连接到当前进程的地址空间。
  squeue<ElemType,5> *QQ=(squeue<ElemType,5> *)shmat(shmid,0,0);
  if ( QQ==(void *)-1 )
  {
    cout << "shmat() failed\n"; return -1;
  }

  QQ->init();       // 初始化循环队列。

  ElemType ee;      // 创建一个数据元素。

  cout << "元素（1、2、3）入队。\n";
  ee=1;  QQ->push(ee);
  ee=2;  QQ->push(ee);
  ee=3;  QQ->push(ee);

  cout << "队列的长度是" << QQ->size() << endl;
  QQ->printqueue();

  ee=QQ->front(); QQ->pop(); cout << "出队的元素值为" << ee << endl;
  ee=QQ->front(); QQ->pop(); cout << "出队的元素值为" << ee << endl;

  cout << "队列的长度是" << QQ->size() << endl;
  QQ->printqueue();

  cout << "元素（11、12、13、14、15）入队。\n";
  ee=11;  QQ->push(ee);
  ee=12;  QQ->push(ee);
  ee=13;  QQ->push(ee);
  ee=14;  QQ->push(ee);
  ee=15;  QQ->push(ee);

  cout << "队列的长度是" << QQ->size() << endl;
  QQ->printqueue();

  shmdt(QQ);  // 把共享内存从当前进程中分离。
}

```
## ***信号量***

- 操作：
    - `P`操作 ***wait*** 将信号量的值减 `1` ，如果信号量的值为 `0` ，将阻塞等待 ，直到信号量的值大于 `0 `
    - `V`操作 ***post*** 将信号量的值加`1`，任何时候都不会阻塞
    - 用`ipcs -s`可以查看系统的信号量
        - 键值`（key）`，共享内存`id（shmid）`，拥有者`（owner）`，权限`（perms）`，信号量数组中信号量的数量`（nsems）`
    - 用`ipcrm -sem id`可以手工删除信号量
- 应用：
    - 如果约定信号量的取值只是 `0` 和 `1` （`0`-资源不可用；`1`-资源可用），可以实现互斥锁
    - 如果约定信号量的取值表示可用资源的数量，可以实现生产`/`消费者模型





## ***共享内存加锁-信号量-demo3***
```c++
#include "_public.h"

struct stgirl     // 超女结构体。
{
  int  no;        // 编号。
  char name[51];  // 姓名，注意，不能用string。
};

int main(int argc,char *argv[])
{
  if (argc!=3) { cout << "Using:./demo no name\n"; return -1; }

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

  // 创建、初始化二元信号量。
  csemp mutex;
  if (mutex.init(0x5005)==false)
  {
    cout << "mutex.init(0x5005) failed.\n"; return -1;
  }

  cout << "申请加锁...\n";
  mutex.wait(); // 申请加锁。
  cout << "申请加锁成功。\n";

  // 第3步：使用共享内存，对共享内存进行读/写。
  cout << "原值：no=" << ptr->no << ",name=" << ptr->name << endl;  // 显示共享内存中的原值。
  ptr->no=atoi(argv[1]);        // 对超女结构体的no成员赋值。
  strcpy(ptr->name,argv[2]);    // 对超女结构体的name成员赋值。
  cout << "新值：no=" << ptr->no << ",name=" << ptr->name << endl;  // 显示共享内存中的当前值。
  sleep(10);

  mutex.post(); // 解锁。
  cout << "解锁。\n";

  // 查看信号量  ：ipcs -s    // 删除信号量  ：ipcrm sem 信号量id
  // 查看共享内存：ipcs -m    // 删除共享内存：ipcrm -m  共享内存id

  // 第4步：把共享内存从当前进程中分离。
  shmdt(ptr);

  // 第5步：删除共享内存。
  // if (shmctl(shmid,IPC_RMID,0)==-1)
  // { 
  //  cout << "shmctl failed\n"; return -1; 
  // }
}
```

## ***多进程-生消者模型-生产-incache***
```c++
#include "_public.h"

int main()
{
  struct stgirl  // 循环队列的数据元素是超女结构体。
  {
    int no;
    char name[51];
  };

  using ElemType=stgirl;

  // 初始化共享内存。
  int shmid=shmget(0x5005, sizeof(squeue<ElemType,5>), 0640|IPC_CREAT);
  if ( shmid ==-1 )
  {
    cout << "shmget(0x5005) failed.\n"; return -1;
  }

  // 把共享内存连接到当前进程的地址空间。
  squeue<ElemType,5> *QQ=(squeue<ElemType,5> *)shmat(shmid,0,0);
  if ( QQ==(void *)-1 )
  {
    cout << "shmat() failed\n"; return -1;
  }

  QQ->init();       // 初始化循环队列。

  ElemType ee;      // 创建一个数据元素。

  csemp mutex; mutex.init(0x5001);     // 用于给共享内存加锁。
  csemp cond;  cond.init(0x5002,0,0);  // 信号量的值用于表示队列中数据元素的个数。

  mutex.wait();  // 加锁。
  // 生产3个数据。
  ee.no=3; strcpy(ee.name,"西施"); QQ->push(ee);
  ee.no=7; strcpy(ee.name,"冰冰"); QQ->push(ee);
  ee.no=8; strcpy(ee.name,"幂幂"); QQ->push(ee);
  mutex.post();  // 解锁。
  cond.post(3);  // 实参是3，表示生产了3个数据。

  shmdt(QQ);  // 把共享内存从当前进程中分离。
}
```

## ***多进程-生消者模型-消费-outcache***
```c++
#include "_public.h"

int main()
{
  struct stgirl  // 循环队列的数据元素是超女结构体。
  {
    int no;
    char name[51];
  };

  using ElemType=stgirl;

  // 初始化共享内存。
  int shmid=shmget(0x5005, sizeof(squeue<ElemType,5>), 0640|IPC_CREAT);
  if ( shmid ==-1 )
  {
    cout << "shmget(0x5005) failed.\n"; return -1;
  }

  // 把共享内存连接到当前进程的地址空间。
  squeue<ElemType,5> *QQ=(squeue<ElemType,5> *)shmat(shmid,0,0);
  if ( QQ==(void *)-1 )
  {
    cout << "shmat() failed\n"; return -1;
  }

  QQ->init();       // 初始化循环队列。

  ElemType ee;      // 创建一个数据元素。

  csemp mutex; mutex.init(0x5001);     // 用于给共享内存加锁。
  csemp cond;  cond.init(0x5002,0,0);  // 信号量的值用于表示队列中数据元素的个数。

  while (true)
  {
    mutex.wait();  // 加锁。

    while (QQ->empty())    // 如果队列空，进入循环，否则直接处理数据。必须用循环，不能用if
    {
      mutex.post();   // 解锁。
      cond.wait();    // 等待生产者的唤醒信号。
      mutex.wait();   // 加锁。
    }

    // 数据元素出队。
    ee = QQ->front();  QQ->pop();
    mutex.post(); // 解锁。

    // 处理出队的数据（把数据消费掉）。
    cout << "no=" << ee.no << ",name=" << ee.name << endl;
    usleep(100);    // 假设处理数据需要时间，方便演示。
  }

  shmdt(QQ);
}
```

## ***makefile***

```makefile
all:demo1 demo2 demo3 incache outcache

demo1:demo1.cpp _public.h _public.cpp
    g++ -g -o demo1 demo1.cpp _public.cpp

demo2:demo2.cpp _public.h _public.cpp
    g++ -g -o demo2 demo2.cpp _public.cpp

demo3:demo3.cpp _public.h _public.cpp
    g++ -g -o demo3 demo3.cpp _public.cpp

incache:incache.cpp _public.h _public.cpp
    g++ -g -o incache incache.cpp _public.cpp

outcache:outcache.cpp _public.h _public.cpp
    g++ -g -o outcache outcache.cpp _public.cpp

clean:
    rm -f demo1 demo2 demo3 incache outcache

```