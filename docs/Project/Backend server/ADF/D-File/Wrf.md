
# **读写文件**

## **定义**

```c++
// 写文件的类
//步骤：
// 1) 打开临时文件。
// 2) 向临时文件中写入数据。
// 3）关闭临时文件。
// 4）把临时文件名改名为正式的文件名。
```

```c++
class cofile  // class out file
{
private:
    ofstream fout;                  // 写入文件的对象。
    string   m_filename;         // 文件名，建议采用绝对路径。
    string   m_filenametmp;  // 临时文件名，在m_filename后面加".tmp"。
public:
    cofile() {}
    bool isopen() const { return fout.is_open(); }     // 文件是否已打开。

    // 打开文件。
    // filename，待打开的文件名。
    // btmp，是否采用临时文件的方案。
    // mode，打开文件的模式。
    // benbuffer，是否启用文件缓冲区。
     bool open(const string &filename,const bool btmp=true,const ios::openmode mode=ios::out,const bool benbuffer=true);

    // 把数据以文本的方式格式化输出到文件。
    template< typename... Args >
    bool writeline(const char* fmt, Args... args) 
    {
        if (fout.is_open()==false) return false;

        fout << sformat(fmt,args...);

        return fout.good();
    }

    // 重载<<运算符，把数据以文本的方式输出到文件。
    // 注意：换行只能用\n，不能用endl。
    template<typename T>
    cofile& operator<<(const T &value)
    {
        fout << value; return *this;
    }

    // 把二进制数据写入文件。
    bool write(void *buf,int bufsize);

    // 关闭文件，并且把临时文件名改为正式文件名。
    bool closeandrename();

    // 关闭文件，如果有临时文件，则删除它。
    void close();

    ~cofile() { close(); };
};


```

```c++

// 读取文件的类
```

```c++

class cifile    // class in file
{
private:
    ifstream fin;                     // 读取文件的对象。
    string   m_filename;         // 文件名，建议采用绝对路径。
public:
    cifile() {}

    // 判断文件是否已打开。
    bool isopen() const { return fin.is_open(); }

    // 打开文件。
    // filename，待打开的文件名。
    // mode，打开文件的模式。
    bool open(const string &filename,const ios::openmode mode=ios::in);

    // 以行的方式读取文本文件，endbz指定行的结尾标志，缺省为空，没有结尾标志。
    bool readline(string &buf,const string& endbz="");

    // 读取二进制文件，返回实际读取到的字节数。
    int read(void *buf,const int bufsize);

    // 关闭并删除文件。
    bool closeandremove();

    // 只关闭文件。
    void close();

    ~cifile() { close(); }
};


```



## **实现**
```c++
bool cofile::open(const string &filename,const bool btmp,const ios::openmode mode,const bool benbuffer)
{
    // 如果文件是打开的状态，先关闭它。
    if (fout.is_open()) fout.close();

    m_filename=filename;

    newdir(m_filename,true);     // 如果文件的目录不存在，创建目录。

    if (btmp==true) 
    {   // 采用临时文件的方案。
        m_filenametmp=m_filename+".tmp";
        fout.open(m_filenametmp,mode);
    }
    else
    {   // 不采用临时文件的方案。
        m_filenametmp.clear();
        fout.open(m_filename,mode);
    }

    // 不启用文件缓冲区。
    if (benbuffer==false) fout << unitbuf;

    return fout.is_open();
}

bool cofile::write(void *buf,int bufsize)
{
    if (fout.is_open()==false) return false;

    // fout.write((char *)buf,bufsize);
    fout.write(static_cast<char *>(buf),bufsize);

    return fout.good();
}

// 关闭文件，并且把临时文件名改为正式文件名。
bool cofile::closeandrename()
{
    if (fout.is_open()==false) return false;

    fout.close();

    //  如果采用了临时文件的方案。
    if (m_filenametmp.empty()==false) 
        if (rename(m_filenametmp.c_str(),m_filename.c_str())!=0) return false;

    return true;
}

// 关闭文件，删除临时文件。
void cofile::close() 
{ 
    if (fout.is_open()==false) return;

    fout.close(); 

    //  如果采用了临时文件的方案。
    if (m_filenametmp.empty()==false) 
        remove(m_filenametmp.c_str());
}


```

```c++
bool cifile::open(const string &filename,const ios::openmode mode)
{
    // 如果文件是打开的状态，先关闭它。
    if (fin.is_open()) fin.close();

    m_filename=filename;

    fin.open(m_filename,mode);

    return fin.is_open();
}

int cifile::read(void *buf,const int bufsize)
{
    // fin.read((char *)buf,bufsize);
    fin.read(static_cast<char *>(buf),bufsize);

    return fin.gcount();          // 返回读取的字节数。
}

bool cifile::closeandremove()
{
    if (fin.is_open()==false) return false;

    fin.close(); 

    if (remove(m_filename.c_str())!=0) return false;

    return true;
}

void cifile::close() 
{ 
    if (fin.is_open()==false) return;

    fin.close(); 
}

bool cifile::readline(string &buf,const string& endbz)
{
    buf.clear();            // 清空buf。

    string strline;        // 存放从文件中读取的一行。

    while (true)
    {
        getline(fin,strline);    // 从文件中读取一行。
      
        if (fin.eof()) break;    // 如果文件已读完。

        buf=buf+strline;      // 把读取的内容拼接到buf中。

        if (endbz=="")
            return true;          // 如果行没有结尾标志。
        else 
        {
            // 如果行有结尾标志，判断本次是否读到了结尾标志，如果没有，继续读，如果有，返回。
            if (buf.find(endbz,buf.length()-endbz.length()) != string::npos) return true;
        }

        buf=buf+"\n";        // getline从文件中读取一行的时候，会删除\n，所以，这里要补上\n，因为这个\n不应该被删除。
    }

    return false;
}


```


## **测试**
```c++
//cofile类向文件中写入文本数据。

#include "../_public.h"
using namespace std;
using namespace idc;  

int main()
{
    cofile ofile;      

    // 创建文件，实际创建的是临时文件，例如/tmp/data/girl.xml.tmp。
    if (ofile.open("/tmp/data/girl.xml")==false)
    {
        printf("ofile.open(/tmp/data/girl.xml) failed.\n"); return -1;
    }

    // 用<<输出到文件，与cout的用法相同。
    ofile << "<data>" << "\n";         // 换行只能用\n，不能用endl,endl在输出流中插入一个换行符(\n)并刷新输出缓冲区

    // 格式化输出到文件。
    ofile.writeline("<name>%s</name><age>%d</age><sc>%s</sc><yz>%s</yz><memo>%s</memo><endl/>\n",\
                           "妲已",28,"火辣","漂亮","商要亡，关我什么事。");
    ofile.writeline("<name>%s</name><age>25</age><sc>火辣</sc><yz>漂亮</yz><memo>1、中国排名第一的美女；\n"\
         "2、男朋友是范蠡；\n"\
         "3、老公是夫差，被勾践弄死了。</memo><endl/>\n","西施");

    ofile << "</data>\n";                 // 换行只能用\n，不能用endl。

    // sleep(10);   // 用ls /tmp/data/*.tmp可以看到生成的临时文件。

    // 关闭文件，并把临时文件名改为正式的文件名。
    ofile.closeandrename();
}



```





```c++
//cifile类从文本文件中读取数据。

#include "../_public.h"
using namespace std;
using namespace idc;
 
int main()
{
    cifile ifile;

    // 打开文件。
    if (ifile.open("/tmp/data/girl.xml")==false)
    {
        printf("ofile.open(/tmp/data/girl.xml) failed.\n"); return -1;
    }

    string strline;   // 存放从文本文件中读取的一行。

    while (true)
    {
        // 从文件中读取一行。
        if (ifile.readline(strline,"<endl/>")==false) break;

        cout << "=" << strline << "=\n";
    }

    // ifile.closeandremove();     // 关闭并删除文件。
    ifile.close();                       // 关闭文件。
}

```

```c++
//cofile类向文件中写入二进制数据

#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    cofile ofile;

    // 创建文件，实际创建的是临时文件，例如/tmp/data/girl.dat.tmp。
    if (ofile.open("/tmp/data/girl.dat",true,ios::binary)==false)
    {
        printf("ofile.open(/tmp/data/girl.dat) failed.\n"); return -1;
    }

    struct st_girl
    {
        int bh;
        char name[21];
    }girl;

    memset(&girl,0,sizeof(girl));
    girl.bh=8;
    strcpy(girl.name,"西施");
    ofile.write(&girl,sizeof(girl));

    // sleep(30);   // 用ls /tmp/data/*.tmp可以看到生成的临时文件。

    // 关闭文件，并把临时文件名改为正式的文件名。
    ofile.closeandrename();
}

```