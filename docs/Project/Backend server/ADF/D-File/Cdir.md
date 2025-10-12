
# **获取某目录及其子目录中文件列表**

## **定义**

```c++
class cdir
{
private:
    vector<string> m_filelist;  // 存放文件列表的容器（绝对路径的文件名）。
    int m_pos;                          // 从文件列表m_filelist中已读取文件的位置。
    string m_fmt;                     // 文件时间格式，缺省"yyyymmddhh24miss"。

    cdir(const cdir &) = delete;                      // 禁用拷贝构造函数。
    cdir &operator=(const cdir &) = delete;  // 禁用赋值函数。
public:
    // /project/public/_public.h
    string m_dirname;        // 目录名，例如：/project/public
    string m_filename;       // 文件名，不包括目录名，例如：_public.h
    string m_ffilename;      // 绝对路径的文件，例如：/project/public/_public.h
    int      m_filesize;          // 文件的大小，单位：字节。
    string m_mtime;           // 文件最后一次被修改的时间，即stat结构体的st_mtime成员。
    string m_ctime;            // 文件生成的时间，即stat结构体的st_ctime成员。
    string m_atime;            // 文件最后一次被访问的时间，即stat结构体的st_atime成员。

    cdir():m_pos(0),m_fmt("yyyymmddhh24miss") {}  // 构造函数。

    // 设置文件时间的格式，支持"yyyy-mm-dd hh24:mi:ss"和"yyyymmddhh24miss"两种，缺省是后者。
    void setfmt(const string &fmt);

    // 打开目录，获取目录中文件的列表，存放在m_filelist容器中。
    // dirname，目录名，采用绝对路径，如/tmp/root。
    // rules，文件名的匹配规则，不匹配的文件将被忽略。
    // maxfiles，本次获取文件的最大数量，缺省值为10000个，如果文件太多，可能消耗太多的内存。
    // bandchild，是否打开各级子目录，缺省值为false-不打开子目录。
    // bsort，是否按文件名排序，缺省值为false-不排序。
    // 返回值：true-成功，false-失败。
    bool opendir(const string &dirname,const string &rules,const int maxfiles=10000,const bool bandchild=false,bool bsort=false);

private:
    // 这是一个递归函数，被opendir()的调用，在cdir类的外部不需要调用它。
    bool _opendir(const string &dirname,const string &rules,const int maxfiles,const bool bandchild);

public:
    // 从m_filelist容器中获取一条记录（文件名），同时获取该文件的大小、修改时间等信息。
    // 调用opendir方法时，m_filelist容器被清空，m_pos归零，每调用一次readdir方法m_pos加1。
    // 当m_pos小于m_filelist.size()，返回true，否则返回false。
    bool readdir();

    unsigned int size() { return m_filelist.size(); }

    ~cdir();  // 析构函数。
};

```

## **实现**
```c++
void cdir::setfmt(const string &fmt)
{
    m_fmt=fmt;
}
```
```c++

bool cdir::opendir(const string &dirname,const string &rules,const int maxfiles,const bool bandchild,bool bsort)
{
    m_filelist.clear();    // 清空文件列表容器。
    m_pos=0;              // 从文件列表中已读取文件的位置归0。

    // 如果目录不存在，创建它。
    if (newdir(dirname,false) == false) return false;

    // 打开目录，获取目录中的文件列表，存放在m_filelist容器中。
    bool ret=_opendir(dirname,rules,maxfiles,bandchild);

    if (bsort==true)    // 对文件列表排序。
    {
      sort(m_filelist.begin(), m_filelist.end());
    }

    return ret;
}

```

```c++
// 这是一个递归函数，在opendir()中调用，cdir类的外部不需要调用它。
bool cdir::_opendir(const string &dirname,const string &rules,const int maxfiles,const bool bandchild)
{
    DIR *dir;   // 目录指针。

    // 打开目录。
    if ( (dir=::opendir(dirname.c_str())) == nullptr ) return false; // opendir与库函数重名，需要加::

    string strffilename;            // 全路径的文件名。
    struct dirent *stdir;            // 存放从目录中读取的内容。

    // 用循环读取目录的内容，将得到目录中的文件名和子目录。
    while ((stdir=::readdir(dir)) != 0) // readdir与库函数重名，需要加::
    {
        // 判断容器中的文件数量是否超出maxfiles参数。
        if ( m_filelist.size()>=maxfiles ) break;

        // 文件名以"."打头的文件不处理。.是当前目录，..是上一级目录，其它以.打头的都是特殊目录和文件。
        if (stdir->d_name[0]=='.') continue;
        
        // 拼接全路径的文件名。
        strffilename=dirname+'/'+stdir->d_name;  

        // 如果是目录，处理各级子目录。
        if (stdir->d_type==4)
        {
            if (bandchild == true)      // 打开各级子目录。
            {
                if (_opendir(strffilename,rules,maxfiles,bandchild) == false)   // 递归调用_opendir函数。
                {
                    closedir(dir); return false;
                }
            }
        }
        
        // 如果是普通文件，放入容器中。
        if (stdir->d_type==8)
        {
            // 把能匹配上的文件放入m_filelist容器中。
            if (matchstr(stdir->d_name,rules) == false) continue;

            m_filelist.push_back(std::move(strffilename));
        }
    }

    closedir(dir);   // 关闭目录。

    return true;
}

```

```c++
bool cdir::readdir()
{
    // 如果已读完，清空容器
    if (m_pos >= m_filelist.size()) 
    {
      m_pos=0; m_filelist.clear(); return false;
    }

    // 文件全名，包括路径
    m_ffilename=m_filelist[m_pos];

    // 从绝对路径的文件名中解析出目录名和文件名。
    int pp=m_ffilename.find_last_of("/");
    m_dirname=m_ffilename.substr(0,pp);
    m_filename=m_ffilename.substr(pp+1);

    // 获取文件的信息。
    struct stat st_filestat;
    stat(m_ffilename.c_str(),&st_filestat);
    m_filesize=st_filestat.st_size;                                     // 文件大小。
    m_mtime=timetostr1(st_filestat.st_mtime,m_fmt);   // 文件最后一次被修改的时间。
    m_ctime=timetostr1(st_filestat.st_ctime,m_fmt);      // 文件生成的时间。
    m_atime=timetostr1(st_filestat.st_atime,m_fmt);      // 文件最后一次被访问的时间。

    m_pos++;       // 已读取文件的位置后移。

    return true;
}

```



```c++

cdir::~cdir()
{
    m_filelist.clear();
}

```


## **测试**
```c++
// cdir类获取某目录及其子目录中的文件列表信息。

#include "../_public.h"
using namespace std;
using namespace idc;

int main(int argc,char *argv[])
{
    if (argc != 3) 
    { 
        printf("Using:./demo34 pathname matchstr\n");
        printf("Sample:./demo34 /project \"*.h,*.cpp\"\n");
        return -1;
    }

    cdir dir;       // 创建读取目录的对象。

    if (dir.opendir(argv[1],argv[2],100,false,true)==false)             // 打开目录，获取目录中文件的列表。
    {   
        printf("dir.opendir(%s) failed.\n",argv[1]); return -1; 
    }

    while(dir.readdir()==true)        // 遍历文件列表。
    {
        cout << "filename=" << dir.m_ffilename << ",mtime=" << dir.m_mtime << ",size=" << dir.m_filesize << endl;
    }
}


```

