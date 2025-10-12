[//]: # (---)

[//]: # (hide:)

[//]: # (  - toc)

[//]: # (  - feedback)

[//]: # (---)

## **目录操作函数**
1. 获取当前工作目录
    - 包含头文件：`<unistd.h>`
    - `char *getcwd(char *buf, size_t size)`; 
    - `char *get_current_dir_name(void)`;
    - 示例：
        ```
        #include <iostream>
        #include <unistd.h>
        using namespace std;
        
        int main()
        {
        char path1[256];   // linux系统目录的最大长度是255。
        getcwd(path1,256);
        cout << "path1=" << path1 << endl;
        
        char *path2=get_current_dir_name();
        cout << "path2=" << path2 << endl;
        free(path2);   // 注意释放内存。malloc() new delete
        }
        
        ```

2. 切换工作目录
    - 包含头文件：`<unistd.h>`
    - `int chdir(const char *path)`;
    - 返回值：`0`-成功；其它-失败（目录不存在或没有权限）

3. 创建目录
    - 包含头文件：`<sys/stat.h>`
    - `int mkdir(const char *pathname, mode_t mode)`;
    - `pathname`-目录名
    - `mode`-访问权限，如`0755`，不要省略前置的`0` 为八进制
    - 返回值：`0`-成功；其它-失败（上级目录不存在或没有权限） `/tmp/aaa  /tmp/aaa/bbb`

4. 删除目录
    - 包含头文件：`<sys/stat.h>`
    - `int rmdir(const char *pathname)`;
    - `path`-目录名
    - 返回值：`0`-成功；其它-失败（目录不存在或没有权限）

## **获取目录中文件的列表**
    文件存放在目录中，在处理文件之前，必须先知道目录中有哪些文件，所以要获取目录中文件的列表

1. 包含头文件
    - `#include <dirent.h>`
2. 相关的库函数
    1.  用`opendir()`函数打开目录
        - `DIR *opendir(const char *pathname)`;
        - 成功-返回目录的地址，失败-返回空地址;
    2.  用`readdir()`函数循环的读取目录
        - `struct dirent *readdir(DIR *dirp)`;
        - 成功-返回`struct dirent`结构体的地址，失败-返回空地址;
    3.  用`closedir()`关闭目录。
        - `int closedir(DIR *dirp)`;
3. 数据结构
    - 目录指针：`DIR *目录指针变量名`;
    - 每次调用`readdir()`，函数返回`struct dirent`的地址，存放了本次读取到的内容
    ```
    struct dirent
    {
       long d_ino;                    			// inode number 索引节点号。
       off_t d_off;                   			// offset to this dirent 在目录文件中的偏移。
       unsigned short d_reclen;     		// length of this d_name 文件名长度。
       unsigned char d_type;         		// the type of d_name 文件类型。
       char d_name [NAME_MAX+1];    // file name文件名，最长255字符。
    };
    
    ```
    - 重点关注结构体的`d_name`和`d_type`成员。
    - `d_name`-文件名或目录名。
    - `d_type`-文件的类型，有多种取值，最重要的是`8`和`4`，`8-常规文件（A regular file）`；`4-子目录（A directory）`，其它的暂时不关心。
      注意，`d_name`的数据类型是字符，不可直接显示。
    - 示例：
        ```
        #include <iostream>
        #include <dirent.h>
        using namespace std;
        
        int main(int argc,char *argv[])
        {
        if (argc != 2) { cout << "Using ./demo 目录名\n"; return -1; }
        
        DIR *dir;   // 定义目录指针。
        
        // 打开目录。
        if ( (dir=opendir(argv[1])) == nullptr ) return -1;
        
        // 用于存放从目录中读取到的内容。
        struct dirent *stdinfo=nullptr;
        
        while (1)
        {
            // 读取一项内容并显示出来。
            if ((stdinfo=readdir(dir)) == nullptr) break;
        
            cout << "文件名=" << stdinfo->d_name << "，文件类型=" << (int)stdinfo->d_type << endl;
        }
        
        closedir(dir);   // 关闭目录指针。
        }
        
        ```

    
    
