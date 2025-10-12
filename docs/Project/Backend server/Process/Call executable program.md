## **调用可执行程序**
- `Linux`提供了`system()`函数和`exec`函数族，在`C++`程序中，可以执行其它的程序（二进制文件、操作系统命令或`Shell`脚本）


## **system()函数**

- `system()`函数提供了一种简单的执行程序的方法，把需要执行的程序和参数用一个字符串传给`system()`函数就行了
    - 函数的声明： `int system(const char * string);`
    - `system()`函数的返回值
        - 如果执行的程序不存在，`system()`函数返回非`0`；
        - 如果执行程序成功，并且被执行的程序终止状态是0，`system()`函数返回`0`；
        - 如果执行程序成功，并且被执行的程序终止状态不是`0`，`system()`函数返回非`0`;

    ```c++
    #include <unistd.h>
    #include <iostream>
    using namespace std;

    int main(int argc, char *argv[]) {
        int ret = system("/bin/ls -l /tmp");  
        cout << "ret=" << ret << endl;
        perror("system");
    }
    ```

## **exec函数族**

- `exec`函数族提供了另一种在进程中调用程序（二进制文件或`Shell`脚本）的方法
    - `exec`函数族的声明如下：
        - ***`int execl(const char *path, const char *arg, ...);`***
        - `int execlp(const char *file, const char *arg, ...);`
        - `int execle(const char *path, const char *arg,...,char * const envp[]);`
        - ***`int execv(const char *path, char *const argv[]);`***
        - `int execvp(const char *file, char *const argv[]);`
        - `int execvpe(const char *file, char *const argv[],char *const envp[]);`
    - 注意：
        - 如果执行程序失败则直接返回`-1`，失败原因存于`errno`中
        - 新进程的进程编号与原进程相同，但是，新进程取代了原进程的代码段、数据段和堆栈
        - 如果执行成功则函数不会返回，***当在主程序中成功调用`exec`后，被调用的程序将取代调用者程序***，也就是说，`exec`函数之后的代码都不会被执行
        - 在实际开发中，最常用的是`execl()`和`execv()`，其它的极少使用
    - `execl()` 函数的参数解释如下：

        - 第一个参数：要执行的程序的路径。这个参数指明了可执行程序的完整路径

        - 第二个参数：程序的名称。这个参数是传递给新程序的 `argv[0]`，按照惯例，它应该是程序的名称，但实际上可以是任何字符串

        - 第三个参数及之后的参数：这些参数是传递给新程序的命令行参数，也就是 `argv[1]、argv[2]` 等等。这些参数必须是字符串，并且每个参数都必须单独传递

        - 最后一个参数：必须是一个空指针` 0 `或 `NULL`，表示参数列表的结束
    
    ```c++
    #include <iostream>
    #include <string.h>
    #include <unistd.h>
    using namespace std;
    
    int main(int argc,char *argv[])
    {
      int ret=execl("/bin/ls","/bin/ls","-lt","/tmp",0);  // 最后一个参数0不能省略。
      cout << "ret=" << ret << endl;
      perror("execl");
    
      /*
        char *args[10];
        args[0]=(char *)"/bin/ls";
        args[1]=(char *)"-lt";
        args[2]=(char *)"/tmp";
        args[3]=0;     // 这行代码不能省略。

        int ret=execv("/bin/ls",args);
        cout << "ret=" << ret << endl;
        perror("execv");
      */
    }
    
    ```
    
  
