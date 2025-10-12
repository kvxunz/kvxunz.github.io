## **僵尸进程**
- 如果父进程比子进程先退出，子进程将被1号进程托管（这也是一种让程序在后台运行的方法）。
- 如果子进程比父进程先退出，而父进程没有处理子进程退出的信息，那么，子进程将成为僵尸进程。
    ```c++
    #include <unistd.h>
    #include <iostream>
    using namespace std;

    int main() {
    if (fork() == 0) {
        return 0;
    }

    while (1) {
        cout << "hello" << endl;
        sleep(1);
     }
    }
    ```
- 僵尸进程有什么危害？
    - 内核为每个子进程保留了一个数据结构，包括进程编号、终止状态、使用CPU时间等
    - 父进程如果处理了子进程退出的信息，内核就会释放这个数据结构，父进程如果没有处理子进程退出的信息，内核就不会释放这个数据结构，子进程的进程编号将一直被占用
    - 系统可用的进程编号是有限的，如果产生了大量的僵尸进程，将因为没有可用的进程编号而导致系统不能产生新的进程

## **僵尸进程的避免**
- 子进程退出的时候，内核会向父进程发头`SIGCHLD`信号，如果父进程用`signal(SIGCHLD,SIG_IGN)`通知内核，表示自己对子进程的退出不感兴趣，那么子进程退出后会立即释放数据结构，但是父进程得不到子进程退出的信息
    ```c++
    #include <signal.h>
    #include <unistd.h>
    #include <iostream>
    using namespace std;

    int main() {
    signal(SIGCHLD, SIG_IGN);  // 忽略子进程退出的信号
    if (fork() == 0) {
        return 0;
    }

    while (1) {
        cout << "hello" << endl;
        sleep(1);
     }
    }
    ```
- 父进程通过`wait()/waitpid()`等函数等待子进程结束，在子进程退出之前，父进程将被阻塞待
    - ***`pid_t wait(int *stat_loc);`***
    - `pid_t waitpid(pid_t pid, int *stat_loc, int options);`
    - `pid_t wait3(int *status, int options, struct rusage *rusage);`
    - `pid_t wait4(pid_t pid, int *status, int options, struct rusage *rusage);`
    - 返回值是子进程的编号
    - `stat_loc`是子进程终止的信息：
        - 如果是正常终止，宏`WIFEXITED(stat_loc)`返回真，宏`WEXITSTATUS(stat_loc)`可获取终止状态；
        - 如果是异常终止，宏`WTERMSIG(stat_loc)`可获取终止进程的信号

        ```c++
        #include <iostream>
        #include <unistd.h>
        #include <sys/types.h>
        #include <sys/wait.h>
        using  namespace std;
        
        int main()
        {
        if (fork()>0)
        { // 父进程的流程。
            int sts;
            pid_t pid=wait(&sts);
        
            cout << "已终止的子进程编号是：" << pid << endl;
        
            if (WIFEXITED(sts)) { cout << "子进程是正常退出的，退出状态是：" << WEXITSTATUS(sts) << endl; }
            else { cout << "子进程是异常退出的，终止它的信号是：" << WTERMSIG(sts) << endl; }
        }
        else
        { // 子进程的流程。
            //sleep(100);
            int *p=0; *p=10;
            exit(1);
         }
        }
        ```

    - 如果父进程很忙，可以捕获`SIGCHLD`信号，在信号处理函数中调用`wait()/waitpid()`

        ```c++
        #include <iostream>
        #include <unistd.h>
        #include <sys/types.h>
        #include <sys/wait.h>
        using  namespace std;
        
        void func(int sig)   // 子进程退出的信号处理函数。
        {
          int sts;
          pid_t pid=wait(&sts);
        
          cout << "已终止的子进程编号是：" << pid << endl;
        
          if (WIFEXITED(sts)) { cout << "子进程是正常退出的，退出状态是：" << WEXITSTATUS(sts) << endl; }
          else { cout << "子进程是异常退出的，终止它的信号是：" << WTERMSIG(sts) << endl; }
        }
        
        int main()
        {
          signal(SIGCHLD,func);  // 捕获子进程退出的信号。
        
          if (fork()>0)
          { // 父进程的流程。
            while (true)
            {
              cout << "父进程忙着执行任务。\n";
              sleep(1);
            }
          }
          else
          { // 子进程的流程。
            sleep(5);
            // int *p=0; *p=10;
            exit(1);
          }
        }
        
        ```