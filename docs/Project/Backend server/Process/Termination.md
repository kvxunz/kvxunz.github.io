## **进程终止**
- 8种方式中止进程
    - 5种正常终止
        1. 在`main()`函数用`return 0(10)`返回；
        2. 在任意函数中调用`exit(0/1)`函数；  
        3. 在任意函数中调用`_exit()`或`_Exit()`函数；
        4. 最后一个线程从其启动例程（线程主函数）用`return`返回；
        5. 在最后一个线程中调用`pthread_exit()`返回；
    - 3种异常终止
        1. 调用`abort()`函数中止；
        2. 接收到一个信号；kill -9 ...,  内存泄露
        3. 最后一个线程对取消请求做出响应;

    ```c++
    #include <iostream>
    using namespace std;

    void func2() {
    cout << "进入func2函数" << endl;
    //exit(0);         //(终止进程，不会回到man函数中)
    return;           //(一级一级返回，回到mian函数中)
    }

    void func1() {
    cout << "进入func1函数" << endl;
    func2();
    cout << "回到了func1函数" << endl;
    }

    int main() {
    func1();
    cout << "回到了main函数" << endl;
    }
    ```
        

## **进程终止状态**

- 在`main()`函数中，`return`的返回值即终止状态，如果没有`return`语句或调用`exit()`，那么该进程的终止状态是0。
- 在`Shell`中，查看进程终止的状态：echo $?
- 正常终止进程的3个函数`exit()`和`_Exit()`是由`ISO C`说明的，`_exit()`是由`POSIX`说明的：
    - `void exit(int status);`
    - `void _exit(int status);`
    - `void _Exit(int status);`
    - `status`也是进程终止的状态
    - 如果进程被异常终止，终止状态为非`0`
- 作用： 服务程序的调度、日志和监控


## **资源释放问题**

- `retun`表示函数返回，会调用局部对象的析构函数，`main()`函数中的`return`还会调用全局对象的析构函数
- `exit()`表示终止进程，不会调用局部对象的析构函数，只调用全局对象的析构函数
- `exit()`会执行清理工作(例如把缓冲区的数据写入磁盘，关闭文件等等)，然后退出
- `_exit()`和`_Exit()`直接退出，不会执行任何清理工作
```c++
#include <iostream>
using namespace std;

struct AA {
  string name;
  AA(const string &n) : name(n) {}
  ~AA() { cout << name << "调用了析构函数" << endl; }
};

AA a1("a1");
int main() {
  AA a2("a2");
  // return 0;
  exit(0);
}
```


## **进程的终止函数**

- 进程可以用`atexit()`函数登记终止函数（最多32个），这些函数将由`exit()`自动调用。
- `int atexit(void (*function)(void));`
- `exit()`调用终止函数的顺序与登记时相反。 进程退出前的收尾工作
  ```c++
  #include <iostream>

  using namespace std;

  void func2() { cout << "进入func2函数" << endl; }

  void func1() { cout << "进入func1函数" << endl; }

  int main() {
    atexit(func1);     //登记第一个进程终止函数
    atexit(func2);     //登记第二个进程终止函数
    exit(1);
  }
  ```

    

