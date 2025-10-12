
# **创建目录**

## **定义**
```c++
// 根据绝对路径的文件名或目录名逐级的创建目录。
// pathorfilename：绝对路径的文件名或目录名。
// bisfilename：指定pathorfilename的类型，true-pathorfilename是文件名，否则是目录名，缺省值为true。
// 返回值：true-成功，false-失败，如果返回失败，原因有大概有三种情况：
// 1）权限不足；2）pathorfilename参数不是合法的文件名或目录名；3）磁盘空间不足。
```
```c++
bool newdir(const string &pathorfilename,bool bisfilename=true);

```


## **实现**
```c++
bool newdir(const string &pathorfilename,bool bisfilename)
{
    // /tmp/aaa/bbb/ccc/ddd    /tmp    /tmp/aaa    /tmp/aaa/bbb    /tmp/aaa/bbb/ccc 
     
    // 检查目录是否存在，如果不存在，逐级创建子目录
    int pos=1;          // 不要从0开始，0是根目录/。

    while (true)
    {
        int pos1=pathorfilename.find('/',pos);
        if (pos1==string::npos) break;

        string strpathname=pathorfilename.substr(0,pos1);      // 截取目录。

        pos=pos1+1;       // 位置后移。
        if (access(strpathname.c_str(),F_OK) != 0)  // 如果目录不存在，创建它。
        {
            // 0755是八进制，不要写成755。
            if (mkdir(strpathname.c_str(),0755) != 0) return false;  // 如果目录不存在，创建它。
        }
    }

    // 如果pathorfilename不是文件，是目录，还需要创建最后一级子目录。
    if (bisfilename==false)
    {
        if (access(pathorfilename.c_str(),F_OK) != 0)
        {
            if (mkdir(pathorfilename.c_str(),0755) != 0) return false;
        }
    }

    return true;
}

```

## **测试**
```c++
//newdir函数根据绝对路径的文件名或目录名逐级的创建目录。

#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    // /tmp/aaa/bbb/ccc/ddd    /tmp    /tmp/aaa    /tmp/aaa/bbb    /tmp/aaa/bbb/ccc   /tmp/aaa/bbb/ccc/ddd
    newdir("/tmp/aaa/bbb/ccc/ddd",false);   // 创建"/tmp/aaa/bbb/ccc/ddd"目录。

    newdir("/tmp/111/222/333/444/data.xml",true);   // 创建"/tmp/111/222/333/444"目录。
}

```

