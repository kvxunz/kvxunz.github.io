

## **简介**
- 在`C++`程序中，如果调用了库函数，可以通过函数的返回值判断调用是否成功。其实，还有一个整型的全局变量`errno`，存放了函数调用过程中产生的错误代码
- 如果调用库函数失败，可以通过`errno`的值来查找原因，这也是调试程序的一个重要方法
- errno在`<errno.h>`中声明。
- 配合 `strerror()`和`perror()`两个库函数，可以查看出错的详细信息



## **strerror()库函数**
- `strerror()` 在`<string.h>`中声明，用于获取错误代码对应的详细信息。
- `char *strerror(int errnum)`;                       	// 非线程安全。
- `int strerror_r(int errnum, char *buf, size_t buflen)`;		// 线程安全。
- 示例：
    ```
    #include <iostream>
    #include <cstring>
    using namespace std;
     
    int main()
    {
      int ii;
     
      for(ii=0;ii<150;ii++)		// gcc8.3.1一共有133个错误代码。
      {
        cout << ii << ":" << strerror(ii) << endl;
      }
    }
    
    ```

    ```
    #include <iostream>
    #include <cstring>
    #include <cerrno>
    #include <sys/stat.h>
    using namespace std;
    
    int main()
    {
      int iret=mkdir("/tmp/aaa",0755);
    
      cout << "iret=" << iret << endl;
      cout << errno << ":" << strerror(errno) << endl;
    }
    
    ```

## **perror()库函数**
- `perror()` 在`<stdio.h>`中声明，用于在控制台显示最近一次系统错误的详细信息，在实际开发中，服务程序在后台运行，通过控制台显示错误信息意义不大。（对调试程序略有帮助）
- `void perror(const char *s)`;



## **注意事项**
1. 调用库函数失败不一定会设置`errno`
    - 并不是全部的库函数在调用失败时都会设置e`rrno`的值，以`man`手册为准（一般来说，不属于系统调用的函数不会设置`errno`，属于系统调用的函数才会设置`errno`）
2. `errno`不能作为调用库函数失败的标志
    - `errno`的值只有在库函数调用发生错误时才会被设置，当库函数调用成功时，`errno`的值不会被修改，不会主动的置为 `0`
    - 在实际开发中，判断函数执行是否成功还得靠函数的返回值，只有在返回值是失败的情况下，才需要关注`errno`的值
    - 示例：
        ```
        #include <iostream>
        #include <cstring>    // strerror()函数需要的头文件。
        #include <cerrno>     // errno全局变量的头文件。
        #include <sys/stat.h> // mkdir()函数需要的头文件。
        using namespace std;
        
        int main()
        {
        int iret=mkdir("/tmp/aaa/bb/cc/dd",0755);
        if (iret!=0)
        {
            cout << "iret=" << iret << endl;
            cout << errno << ":" << strerror(errno) << endl;
            perror("调用mkdir(/tmp/aaa/bb/cc/dd)失败");
        }
        
        iret=mkdir("/tmp/dd",0755);
        if (ireet!=0)
          {
            cout << "iret=" << iret << endl;
            cout << errno << ":" << strerror(errno) << endl;
            perror("调用mkdir(/tmp/dd)失败");
          }
        }
        
        ```

    
