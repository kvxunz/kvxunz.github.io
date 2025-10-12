## **access()库函数**
-   `access()`函数用于判断当前用户对目录或文件的存取权限。
-   包含头文件：`#include <unistd.h>`
-   函数声明：`int access(const char *pathname, int mode);`
-   参数说明：
    -   `pathname`	目录或文件名。
    -   `mode` 		需要判断的存取权限。在头文件`<unistd.h>`中的预定义如下：
        - `#define R_OK 	4` 	   判断是否有读权限。
        - `#define W_OK	    2`	   判断是否有写权限。
        - `#define X_OK 	1` 	   判断是否有执行权限。
        - `#define F_OK 	0` 	   判断是否存在。
    - 返回值：
        - 当`pathname`满足`mode`权限返回`0`，不满足返回`-1，errno`被设置。
        
- access()函数主要用于判断目录或文件是否存在

## **stat()库函数**
1. `stat`结构体
    - `struct stat`结构体用于存放目录或文件的详细信息，如下：
        ```
        struct stat
        {
            dev_t st_dev;   	// 文件的设备编号。
            ino_t st_ino;   	// 文件的i-node。
            mode_t st_mode; 	// 文件的类型和存取的权限。
            nlink_t st_nlink;   // 连到该文件的硬连接数目，刚建立的文件值为1。
            uid_t st_uid;   	// 文件所有者的用户识别码。
            gid_t st_gid;   	// 文件所有者的组识别码。
            dev_t st_rdev;  	// 若此文件为设备文件，则为其设备编号。
            off_t st_size;  	// 文件的大小，以字节计算。
            size_t st_blksize;	// I/O 文件系统的I/O 缓冲区大小。
            size_t st_blocks;  	// 占用文件区块的个数。
            time_t st_atime;  	// 文件最近一次被存取或被执行的时间，
                                // 在用mknod、 utime、read、write 与tructate 时改变。
            time_t st_mtime;  	// 文件最后一次被修改的时间，
                                // 在用mknod、 utime 和write 时才会改变。
            time_t st_ctime;  	// 最近一次被更改的时间，在文件所有者、组、 权限被更改时更新。
        };
        
        ```
    
    - `struct stat`结构体的成员变量比较多，重点关注`st_mode、st_size`和`st_mtime`成员
        - 注意：`st_mtime`是一个整数表示的时间，需要程序员自己写代码转换格式
    - `st_mode`成员的取值很多，用以下两个宏来判断：
        - `S_ISREG(st_mode)`   是否为普通文件，如果是，返回真。 
        - `S_ISDIR(st_mode)`   是否为目录，如果是，返回真。
        
2. `stat()`库函数
    - 包含头文件：`#include <sys/stat.h>`
    - 函数声明：`int stat(const char *path, struct stat *buf);`
        - `stat()`函数获取`path`参数指定目录或文件的详细信息，保存到`buf`结构体中。
        - 返回值：`0-`成功，`-1-`失败，`errno`被设置。
    - 示例：
        ```
        #include <stdio.h>
        #include <iostream>
        #include <cstdio>
        #include <sys/stat.h>
        #include <unistd.h>
        using namespace std;
        
        int main(int argc,char *argv[])
        {
          if (argc != 2)  { cout << "Using:./demo 文件或目录名\n"; return -1; }
        
          struct stat st;  // 存放目录或文件详细信息的结构体。
        
          // 获取目录或文件的详细信息
          if (stat(argv[1],&st) != 0)
          {
            cout << "stat(" << argv[1] << "):" << strerror(errno) << endl; return -1;
          }
        
          if (S_ISREG(st.st_mode))
            cout << argv[1] << "是一个文件(" << "mtime=" << st.st_mtime << ",size=" << st.st_size << ")\n";
          if (S_ISDIR(st.st_mode))
            cout << argv[1] << "是一个目录(" << "mtime=" << st.st_mtime << ",size=" << st.st_size << ")\n";
        }
        ```
    
3. `utime()`库函数
    - `utime()`函数用于修改目录或文件的时间。
    - 包含头文件：`#include <sys/types.h> #include <utime.h>`
    - 函数声明：`int utime(const char *filename, const struct utimbuf *times);`
        - 函数用来修改参数`filename`的`st_atime`和`st_mtime`。如果参数`times`为空地址，则设置为当前时间
        - 结构`utimbuf `声明如下：
            ```
            struct utimbuf
            {
                time_t actime;
                time_t modtime;
            };
            
            ```
        - 返回值：`0-`成功，`-1-`失败，`errno`被设置

4. `rename()`库函数
    - `rename()`函数用于重命名目录或文件，相当于操作系统的`mv`命令。
    - 包含头文件：`#include <stdio.h>`
    - 函数声明：`int rename(const char *oldpath, const char *newpath);`
    - 参数说明：
        - `oldpath` 	原目录或文件名。
        - `newpath` 	目标目录或文件名。
        - 返回值：`0-`成功，`-1-`失败，`errno`被设置。

5. remove()库函数
    - remove()函数用于删除目录或文件，相当于操作系统的rm命令。
    - 包含头文件：`#include <stdio.h>`
    - 函数声明：`int remove(const char *pathname);`
    - 参数说明：
        - `pathname` 待删除的目录或文件名。
        - 返回值：`0-`成功，`-1-`失败，`e`rrno`被设置
    
    
    
    