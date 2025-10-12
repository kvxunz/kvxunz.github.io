
# **文件操作相关的函数**

## **定义**

```c++
// 重命名文件，类似Linux系统的mv命令。
// srcfilename：原文件名，建议采用绝对路径的文件名。
// dstfilename：目标文件名，建议采用绝对路径的文件名。
// 返回值：true-成功；false-失败，失败的主要原因是权限不足或磁盘空间不够，如果原文件和目标文件不在同一个磁盘分区，重命名也可能失败。
// 注意，在重命名文件之前，会自动创建dstfilename参数中包含的目录。
// 在应用开发中，可以用renamefile()函数代替rename()库函数。


```

```c++

bool renamefile(const string &srcfilename,const string &dstfilename);
```


```c++
// 复制文件，类似Linux系统的cp命令。
// srcfilename：原文件名，建议采用绝对路径的文件名。
// dstfilename：目标文件名，建议采用绝对路径的文件名。
// 返回值：true-成功；false-失败，失败的主要原因是权限不足或磁盘空间不够。
// 注意：
// 1）在复制文件之前，会自动创建dstfilename参数中的目录名。
// 2）复制文件的过程中，采用临时文件命名的方法，复制完成后再改名为dstfilename，避免中间状态的文件被读取。
// 3）复制后的文件的时间与原文件相同，这一点与Linux系统cp命令不同。



```
```c++
bool copyfile(const string &srcfilename,const string &dstfilename);

```
```c++

// 获取文件的大小。
// filename：待获取的文件名，建议采用绝对路径的文件名。
// 返回值：如果文件不存在或没有访问权限，返回-1，成功返回文件的大小，单位是字节。

```
```c++

int filesize(const string &filename);


```
```c++
// 获取文件的时间。
// filename：待获取的文件名，建议采用绝对路径的文件名。
// mtime：用于存放文件的时间，即stat结构体的st_mtime。
// fmt：设置时间的输出格式，与ltime()函数相同，但缺省是"yyyymmddhh24miss"。
// 返回值：如果文件不存在或没有访问权限，返回false，成功返回true。


```

```c++

bool filemtime(const string &filename,char *mtime    ,const string &fmt="yyyymmddhh24miss");
bool filemtime(const string &filename,string &mtime,const string &fmt="yyyymmddhh24miss");


```

```c++
// 重置文件的修改时间属性。
// filename：待重置的文件名，建议采用绝对路径的文件名。
// mtime：字符串表示的时间，格式不限，但一定要包括yyyymmddhh24miss，一个都不能少，顺序也不能变。
// 返回值：true-成功；false-失败，失败的原因保存在errno中。


```

```c++

bool setmtime(const string &filename,const string &mtime);
```




## **实现**
```c++
bool renamefile(const string &srcfilename,const string &dstfilename)
{
    // 如果原文件不存在，直接返回失败。
    if (access(srcfilename.c_str(),R_OK) != 0) return false;

    // 创建目标文件的目录。
    if (newdir(dstfilename,true) == false) return false;

    // 调用操作系统的库函数rename重命名文件。 mv
    if (rename(srcfilename.c_str(),dstfilename.c_str()) == 0) return true;

    return false;
}

```


```c++
bool copyfile(const string &srcfilename,const string &dstfilename)
{
    // 创建目标文件的目录。
    if (newdir(dstfilename,true) == false) return false;

    cifile ifile;
    cofile ofile;
    int ifilesize=filesize(srcfilename);

    int  total_bytes=0;
    int  onread=0;
    char buffer[5000];

    if (ifile.open(srcfilename,ios::in|ios::binary)==false) return false;

    if (ofile.open(dstfilename,ios::out|ios::binary)==false) return false;

    while (true)
    {
        if ((ifilesize-total_bytes) > 5000) onread=5000;
        else onread=ifilesize-total_bytes;

        memset(buffer,0,sizeof(buffer));
        ifile.read(buffer,onread);
        ofile.write(buffer,onread);

        total_bytes = total_bytes + onread;

        if (total_bytes == ifilesize) break;
    }

    ifile.close();
    ofile.closeandrename();

    // 更改文件的修改时间属性
    string strmtime;
    filemtime(srcfilename,strmtime);
    setmtime(dstfilename,strmtime);

    return true;
}

```

```c++
int filesize(const string &filename)
{
    struct stat st_filestat;      // 存放文件信息的结构体。

    // 获取文件信息，存放在结构体中。
    if (stat(filename.c_str(),&st_filestat) < 0) return -1;

    return st_filestat.st_size;   // 返回结构体的文件大小成员。
}

```


```c++
bool filemtime(const string &filename,string &mtime,const string &fmt)
{
    struct stat st_filestat;      // 存放文件信息的结构体。

    // 获取文件信息，存放在结构体中。
    if (stat(filename.c_str(),&st_filestat) < 0) return false;

    // 把整数表示的时间转换成字符串表示的时间。
    timetostr(st_filestat.st_mtime,mtime,fmt);

    return true;
}



```

```c++

bool filemtime(const string &filename,char *mtime,const string &fmt)
{
    struct stat st_filestat;      // 存放文件信息的结构体。

    // 获取文件信息，存放在结构体中。
    if (stat(filename.c_str(),&st_filestat) < 0) return false;

    // 把整数表示的时间转换成字符串表示的时间。
    timetostr(st_filestat.st_mtime,mtime,fmt);

    return true;
}
```

```c++
bool setmtime(const string &filename,const string &mtime)
{
    struct utimbuf stutimbuf;

    stutimbuf.actime=stutimbuf.modtime=strtotime(mtime);

    if (utime(filename.c_str(),&stutimbuf)!=0) return false;

    return true;
}
```






## **测试**
```c++

#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    // 重命名文件。
    if (renamefile("/project/public/lib_public.so","/tmp/aaa/bbb/ccc/lib_public.so")==false)
    {
        printf("renamefile(/project/public/lib_public.so) %d:%s\n",errno,strerror(errno));
    }

    // 复制文件。
    if (copyfile("/project/public/libftp.a","/tmp/aaa/ddd/ccc/libftp.a")==false)
    {
        printf("copyfile(/project/public/libftp.a) %d:%s\n",errno,strerror(errno));
    }

    // 获取文件的大小。
    printf("size=%d\n",filesize("/project/public/_public.h"));

    // 重置文件的时间。
    setmtime("/project/public/_public.h","2020-01-05 13:37:29");

    // 获取文件的时间。
    string mtime;
    filemtime("/project/public/_public.h",mtime,"yyyy-mm-dd hh24:mi:ss");
    cout << "mtime=" << mtime << endl;   // 输出mtime=2020-01-05 13:37:29
    filemtime("/project/public/_public.h",mtime);
    cout << "mtime=" << mtime << endl;   // 输出mtime=20200105133729
}

```

