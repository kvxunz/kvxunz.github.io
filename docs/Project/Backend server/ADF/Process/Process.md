
# 

## **定义**
```c++

// 忽略关闭全部的信号、关闭全部的IO，缺省只忽略信号，不关IO。
void closeioandsignal(bool bcloseio=false);
```
```c++
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
        if (m_inited!=true)               // 循环队列的初始化只能执行一次。
        { 
            m_head=0;                      // 头指针。
            m_tail=MaxLength-1;     // 为了方便写代码，初始化时，尾指针指向队列的最后一个位置。
            m_length=0;                   // 队列的实际长度。
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
```
```c++
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
    // 如果信号量用于生产消费者模型，设置为0。
    short m_sem_flg;

    csemp(const csemp &) = delete;                      // 禁用拷贝构造函数。
    csemp &operator=(const csemp &) = delete;  // 禁用赋值函数。
public:
    csemp():m_semid(-1){}

    // 如果信号量已存在，获取信号量；如果信号量不存在，则创建它并初始化为value。
    // 如果用于互斥锁，value填1，sem_flg填SEM_UNDO。
    // 如果用于生产消费者模型，value填0，sem_flg填0。
    bool init(key_t key,unsigned short value=1,short sem_flg=SEM_UNDO);
    bool wait(short value=-1);    // 信号量的P操作，如果信号量的值是0，将阻塞等待，直到信号量的值大于0。
    bool post(short value=1);     // 信号量的V操作。
    int  getvalue();                       // 获取信号量的值，成功返回信号量的值，失败返回-1。
    bool destroy();                       // 销毁信号量。
    ~csemp();
};
```
```c++
// 进程心跳信息的结构体。
struct st_procinfo
{
    int      pid=0;                      // 进程id。
    char   pname[51]={0};        // 进程名称，可以为空。
    int      timeout=0;              // 超时时间，单位：秒。
    time_t atime=0;                 // 最后一次心跳的时间，用整数表示。
    st_procinfo() = default;     // 有了自定义的构造函数，编译器将不提供默认构造函数，所以启用默认构造函数。
    st_procinfo(const int in_pid,const string & in_pname,const int in_timeout, const time_t in_atime)
                    :pid(in_pid),timeout(in_timeout),atime(in_atime) { strncpy(pname,in_pname.c_str(),50); }
};

// 以下几个宏用于进程的心跳。
#define MAXNUMP     1000     // 最大的进程数量。
#define SHMKEYP    0x5095     // 共享内存的key。
#define SEMKEYP     0x5095     // 信号量的key。

// 查看共享内存：  ipcs -m
// 删除共享内存：  ipcrm -m shmid
// 查看信号量：      ipcs -s
// 删除信号量：      ipcrm sem semid

// 进程心跳操作类。
class cpactive
{
 private:
     int  m_shmid;                   // 共享内存的id。
     int  m_pos;                       // 当前进程在共享内存进程组中的位置。
     st_procinfo *m_shm;        // 指向共享内存的地址空间。

 public:
     cpactive();  // 初始化成员变量。

     // 把当前进程的信息加入共享内存进程组中。
     bool addpinfo(const int timeout,const string &pname="",clogfile *logfile=nullptr);

     // 更新共享内存进程组中当前进程的心跳时间。
     bool uptatime();

     ~cpactive();  // 从共享内存中删除当前进程的心跳记录。
};

}

```


## **实现**

```c++

// 忽略关闭全部的信号、关闭全部的IO，缺省只忽略信号，不关IO。 
// 不希望后台服务程序被信号打扰，需要什么信号可以在程序中设置。
// 实际上关闭的IO是0、1、2。
void closeioandsignal(bool bcloseio)
{
    int ii=0;

    for (ii=0;ii<64;ii++)
    {
        if (bcloseio==true) close(ii);

        signal(ii,SIG_IGN); 
    }
}

```

```c++
 cpactive::cpactive()
 {
     m_shmid=0;
     m_pos=-1;
     m_shm=0;
 }
```

```c++
 // 把当前进程的信息加入共享内存进程组中。
 bool cpactive::addpinfo(const int timeout,const string &pname,clogfile *logfile)
 {
    if (m_pos!=-1) return true;

    // 创建/获取共享内存，键值为SHMKEYP，大小为MAXNUMP个st_procinfo结构体的大小。
    if ( (m_shmid = shmget((key_t)SHMKEYP, MAXNUMP*sizeof(struct st_procinfo), 0666|IPC_CREAT)) == -1)
    { 
        if (logfile!=nullptr) logfile->write("创建/获取共享内存(%x)失败。\n",SHMKEYP); 
        else printf("创建/获取共享内存(%x)失败。\n",SHMKEYP);

        return false; 
    }

    // 将共享内存连接到当前进程的地址空间。
    m_shm=(struct st_procinfo *)shmat(m_shmid, 0, 0);
  
    /*
    struct st_procinfo stprocinfo;    // 当前进程心跳信息的结构体。
    memset(&stprocinfo,0,sizeof(stprocinfo));
    stprocinfo.pid=getpid();            // 当前进程号。
    stprocinfo.timeout=timeout;         // 超时时间。
    stprocinfo.atime=time(0);           // 当前时间。
    strncpy(stprocinfo.pname,pname.c_str(),50); // 进程名。
    */
    st_procinfo stprocinfo(getpid(),pname.c_str(),timeout,time(0));    // 当前进程心跳信息的结构体。

    // 进程id是循环使用的，如果曾经有一个进程异常退出，没有清理自己的心跳信息，
    // 它的进程信息将残留在共享内存中，不巧的是，如果当前进程重用了它的id，
    // 守护进程检查到残留进程的信息时，会向进程id发送退出信号，将误杀当前进程。
    // 所以，如果共享内存中已存在当前进程编号，一定是其它进程残留的信息，当前进程应该重用这个位置。
    for (int ii=0;ii<MAXNUMP;ii++)
    {
        if ( (m_shm+ii)->pid==stprocinfo.pid ) { m_pos=ii; break; }
    }

    csemp semp;                       // 用于给共享内存加锁的信号量id。

    if (semp.init(SEMKEYP) == false)  // 初始化信号量。
    {
        if (logfile!=nullptr) logfile->write("创建/获取信号量(%x)失败。\n",SEMKEYP); 
        else printf("创建/获取信号量(%x)失败。\n",SEMKEYP);

        return false;
    }

    semp.wait();  // 给共享内存上锁。

    // 如果m_pos==-1，表示共享内存的进程组中不存在当前进程编号，那就找一个空位置。
    if (m_pos==-1)
    {
        for (int ii=0;ii<MAXNUMP;ii++)
            if ( (m_shm+ii)->pid==0 ) { m_pos=ii; break; }
    }

    // 如果m_pos==-1，表示没找到空位置，说明共享内存的空间已用完。
    if (m_pos==-1) 
    { 
        if (logfile!=0) logfile->write("共享内存空间已用完。\n");
        else printf("共享内存空间已用完。\n");

        semp.post();  // 解锁。

        return false; 
    }

    // 把当前进程的心跳信息存入共享内存的进程组中。
    memcpy(m_shm+m_pos,&stprocinfo,sizeof(struct st_procinfo)); 

    semp.post();   // 解锁。

    return true;
 }
```
```c++
 // 更新共享内存进程组中当前进程的心跳时间。
 bool cpactive::uptatime()
 {
    if (m_pos==-1) return false;

    (m_shm+m_pos)->atime=time(0);

    return true;
 }
```
```c++
 cpactive::~cpactive()
 {
    // 把当前进程从共享内存的进程组中移去。
    if (m_pos!=-1) memset(m_shm+m_pos,0,sizeof(struct st_procinfo));

    // 把共享内存从当前进程中分离。
    if (m_shm!=0) shmdt(m_shm);
 }
```
```c++
// 如果信号量已存在，获取信号量；如果信号量不存在，则创建它并初始化为value。
// 如果用于互斥锁，value填1，sem_flg填SEM_UNDO。
// 如果用于生产消费者模型，value填0，sem_flg填0。
bool csemp::init(key_t key,unsigned short value,short sem_flg)
{
    if (m_semid!=-1) return false; // 如果已经初始化了，不必再次初始化。

    m_sem_flg=sem_flg;

    // 信号量的初始化不能直接用semget(key,1,0666|IPC_CREAT)
    // 因为信号量创建后，初始值是0，如果用于互斥锁，需要把它的初始值设置为1，
    // 而获取信号量则不需要设置初始值，所以，创建信号量和获取信号量的流程不同。

    // 信号量的初始化分三个步骤：
    // 1）获取信号量，如果成功，函数返回。
    // 2）如果失败，则创建信号量。
    // 3) 设置信号量的初始值。

    // 获取信号量。
    if ( (m_semid=semget(key,1,0666)) == -1)
    {
        // 如果信号量不存在，创建它。
        if (errno==ENOENT)
        {
            // 用IPC_EXCL标志确保只有一个进程创建并初始化信号量，其它进程只能获取。
            if ( (m_semid=semget(key,1,0666|IPC_CREAT|IPC_EXCL)) == -1)
            {
                if (errno==EEXIST) // 如果错误代码是信号量已存在，则再次获取信号量。
                {
                    if ( (m_semid=semget(key,1,0666)) == -1)
                    { 
                        perror("init 1 semget()"); return false; 
                    }
                    return true;
                }
                else  // 如果是其它错误，返回失败。
                {
                    perror("init 2 semget()"); return false;
                }
            }

            // 信号量创建成功后，还需要把它初始化成value。
            union semun sem_union;
            sem_union.val = value;   // 设置信号量的初始值。
            if (semctl(m_semid,0,SETVAL,sem_union) <  0) 
            { 
                perror("init semctl()"); return false; 
            }
        }
        else
        { perror("init 3 semget()"); return false; }
    }

    return true;
}
```
```c++
// 信号量的P操作（把信号量的值减value），如果信号量的值是0，将阻塞等待，直到信号量的值大于0。
bool csemp::wait(short value)
{
    if (m_semid==-1) return false;

    struct sembuf sem_b;
    sem_b.sem_num = 0;      // 信号量编号，0代表第一个信号量。
    sem_b.sem_op = value;   // P操作的value必须小于0。
    sem_b.sem_flg = m_sem_flg;
    if (semop(m_semid,&sem_b,1) == -1) { perror("p semop()"); return false; }

    return true;
}
```
```c++
// 信号量的V操作（把信号量的值减value）。
bool csemp::post(short value)
{
    if (m_semid==-1) return false;

    struct sembuf sem_b;
    sem_b.sem_num = 0;     // 信号量编号，0代表第一个信号量。
    sem_b.sem_op = value;  // V操作的value必须大于0。
    sem_b.sem_flg = m_sem_flg;
    if (semop(m_semid,&sem_b,1) == -1) { perror("V semop()"); return false; }

    return true;
}
```
```c++
// 获取信号量的值，成功返回信号量的值，失败返回-1。
int csemp::getvalue()
{
    return semctl(m_semid,0,GETVAL);
}
```
```c++
// 销毁信号量。
bool csemp::destroy()
{
    if (m_semid==-1) return false;

    if (semctl(m_semid,0,IPC_RMID) == -1) { perror("destroy semctl()"); return false; }

    return true;
}
```
```c++
csemp::~csemp()
{
}


```


