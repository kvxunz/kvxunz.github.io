
# **ccmdstr类拆分分隔符**

## **ccmdstr类**

```c++ hl_lines="1-4"
// ccmdstr类用于拆分有分隔符的字符串。
// 字符串的格式为：字段内容1+分隔符+字段内容2+分隔符+字段内容3+分隔符+...+字段内容n。
// 例如："messi,10,striker,30,1.72,68.5,Barcelona"，这是足球运动员梅西的资料。
// 包括：姓名、球衣号码、场上位置、年龄、身高、体重和效力的俱乐部，字段之间用半角的逗号分隔。
```

```c++
class ccmdstr
{
private:
    vector<string> m_cmdstr;  // 存放拆分后的字段内容。

    ccmdstr(const ccmdstr &) = delete;                      // 禁用拷贝构造函数。
    ccmdstr &operator=(const ccmdstr &) = delete;  // 禁用赋值函数。
public:
    ccmdstr()  { } // 构造函数。
    ccmdstr(const string &buffer,const string &sepstr,const bool bdelspace=false);

    const string& operator[](int ii) const     // 重载[]运算符，可以像访问数组一样访问m_cmdstr成员。
    {
        return m_cmdstr[ii];
    }

    // 把字符串拆分到m_cmdstr容器中。
    // buffer：待拆分的字符串。
    // sepstr：buffer中采用的分隔符，注意，sepstr参数的数据类型不是字符，是字符串，如","、" "、"|"、"~!~"。
    // bdelspace：拆分后是否删除字段内容前后的空格，true-删除；false-不删除，缺省不删除。
    void splittocmd(const string &buffer,const string &sepstr,const bool bdelspace=false);

    // 获取拆分后字段的个数，即m_cmdstr容器的大小。
    int size() const { return m_cmdstr.size(); }
    int cmdcount() const { return m_cmdstr.size(); }      // 兼容以前的项目。

    // 从m_cmdstr容器获取字段内容。
    // ii：字段的顺序号，类似数组的下标，从0开始。
    // value：传入变量的地址，用于存放字段内容。
    // 返回值：true-成功；如果ii的取值超出了m_cmdstr容器的大小，返回失败。
    bool getvalue(const int ii,string &value,const int ilen=0) const;      // C++风格字符串。视频中没有第三个参数，加上第三个参数更好。
    bool getvalue(const int ii,char *value,const int ilen=0) const;          // C风格字符串，ilen缺省值为0-全部长度。 
    bool getvalue(const int ii,int  &value) const;                                    // int整数。
    bool getvalue(const int ii,unsigned int &value) const;                     // unsigned int整数。
    bool getvalue(const int ii,long &value) const;                                  // long整数。
    bool getvalue(const int ii,unsigned long &value) const;                  // unsigned long整数。
    bool getvalue(const int ii,double &value) const;                              // 双精度double。
    bool getvalue(const int ii,float &value) const;                                  // 单精度float。
    bool getvalue(const int ii,bool &value) const;                                  // bool型。

    ~ccmdstr(); // 析构函数。
};

```


```c++
// 重载<<运算符，输出ccmdstr::m_cmdstr中的内容，方便调试。
ostream& operator<<(ostream& out, const ccmdstr& cc);
```


## **实现**
```c++
ccmdstr::ccmdstr(const string &buffer,const string &sepstr,const bool bdelspace)
{
    splittocmd(buffer,sepstr,bdelspace);
}

// 把字符串拆分到m_cmdstr容器中。
// buffer：待拆分的字符串。
// sepstr：buffer字符串中字段内容的分隔符，注意，分隔符是字符串，如","、" "、"|"、"~!~"。
// bdelspace：是否删除拆分后的字段内容前后的空格，true-删除；false-不删除，缺省不删除。
void ccmdstr::splittocmd(const string &buffer,const string &sepstr,const bool bdelspace)
{
    // 清除所有的旧数据
    m_cmdstr.clear();

    int pos=0;        // 每次从buffer中查找分隔符的起始位置。
    int pos1=0;      // 从pos的位置开始，查找下一个分隔符的位置。
    string substr;   // 存放每次拆分出来的子串。

    while ( (pos1=buffer.find(sepstr,pos)) != string::npos)   // 从pos的位置开始，查找下一个分隔符的位置。
    {
        substr=buffer.substr(pos,pos1-pos);            // 从buffer中截取子串。

        if (bdelspace == true) deletelrchr(substr);   // 删除子串前后的空格。

        m_cmdstr.push_back(std::move(substr));     // 把子串放入m_cmdstr容器中，调用string类的移动构造函数。

        pos=pos1+sepstr.length();                           // 下次从buffer中查找分隔符的起始位置后移。
    }

    // 处理最后一个字段（最后一个分隔符之后的内容）。
    substr=buffer.substr(pos);

    if (bdelspace == true) deletelrchr(substr);

    m_cmdstr.push_back(std::move(substr));

    return;
}

bool ccmdstr::getvalue(const int ii,string &value,const int ilen) const
{
    if (ii>=m_cmdstr.size()) return false;

    // 从xml中截取数据项的内容。
    // 视频中是以下代码：
    // value=m_cmdstr[ii];
    // 改为：
    int itmplen=m_cmdstr[ii].length();
    if ( (ilen>0) && (ilen<itmplen) ) itmplen=ilen;
    value=m_cmdstr[ii].substr(0,itmplen);
  
    return true;
}

bool ccmdstr::getvalue(const int ii,char *value,const int len) const
{
    if ( (ii>=m_cmdstr.size()) || (value==nullptr) ) return false;

    if (len>0) memset(value,0,len+1);   // 调用者必须保证value的空间足够，否则这里会内存溢出。

    if ( (m_cmdstr[ii].length()<=(unsigned int)len) || (len==0) )
    {
        m_cmdstr[ii].copy(value,m_cmdstr[ii].length());
        value[m_cmdstr[ii].length()]=0;    // string的copy函数不会给C风格字符串的结尾加0。
    }
    else
    {
        m_cmdstr[ii].copy(value,len);
        value[len]=0;
    }

    return true;
}

bool ccmdstr::getvalue(const int ii,int &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    try
    {
        value = stoi(picknumber(m_cmdstr[ii],true));  // stoi有异常，需要处理异常。
    }
    catch(const std::exception& e)
    {
        return false;
    }
    
    return true;
}

bool ccmdstr::getvalue(const int ii,unsigned int &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    try
    {
       value = stoi(picknumber(m_cmdstr[ii]));  // stoi有异常，需要处理异常。不提取符号 + -
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool ccmdstr::getvalue(const int ii,long &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    try
    {
        value = stol(picknumber(m_cmdstr[ii],true));  // stol有异常，需要处理异常。
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool ccmdstr::getvalue(const int ii,unsigned long &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    try
    {
        value = stoul(picknumber(m_cmdstr[ii]));  // stoul有异常，需要处理异常。不提取符号 + -
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool ccmdstr::getvalue(const int ii,double &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    try
    {
        value = stod(picknumber(m_cmdstr[ii],true,true));  // stod有异常，需要处理异常。提取符号和小数点。
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool ccmdstr::getvalue(const int ii,float &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    try
    {
        value = stof(picknumber(m_cmdstr[ii],true,true));  // stof有异常，需要处理异常。提取符号和小数点。
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool ccmdstr::getvalue(const int ii,bool &value) const
{
    if (ii>=m_cmdstr.size()) return false;

    string str=m_cmdstr[ii];
    toupper(str);     // 转换为大写来判断。

    if (str=="TRUE") value=true; 
    else value=false;

    return true;
}

ccmdstr::~ccmdstr()
{
    m_cmdstr.clear();
}

ostream& operator<<(ostream& out, const ccmdstr& cmdstr)
{
    for (int ii=0;ii<cmdstr.size();ii++)
        out << "[" << ii << "]=" << cmdstr[ii] << endl;

    return out;
}


```

## **测试**
```c++
#include "../_public.h"
using namespace std;
using namespace idc;

// 用于存放足球运动员资料的结构体。
struct st_player
{
    char name[51];    // 姓名
    char no[6];           // 球衣号码
    bool striker;         // 场上位置是否是前锋，true-是；false-不是。
    int  age;               // 年龄
    double weight;    // 体重，kg。
    long sal;              // 年薪，欧元。
    char club[51];      // 效力的俱乐部
}stplayer;

int main()
{
    memset(&stplayer,0,sizeof(struct st_player));

    string buffer="messi~!~10~!~true~!~a30~!~68.5~!~2100000~!~Barc,elona";    // 梅西的资料。

    //ccmdstr cmdstr;                               // 定义拆分字符串的对象。
    //cmdstr.splittocmd(buffer,"~!~");           // 拆分buffer。
    ccmdstr cmdstr(buffer,"~!~");                 // 定义拆分字符串的对象并拆分字符串。

    // 像访问数组一样访问拆分后的元素。
    for (int ii=0;ii<cmdstr.size();ii++)
    {
        cout << "cmdstr["<<ii<<"]=" << cmdstr[ii] << endl;
    }

    // 输出拆分后的元素，一般用于调试。
    cout << cmdstr;

    // 获取拆分后元素的内容。
    cmdstr.getvalue(0, stplayer.name,50);     // 获取姓名
    cmdstr.getvalue(1, stplayer.no,5);            // 获取球衣号码
    cmdstr.getvalue(2, stplayer.striker);         // 场上位置
    cmdstr.getvalue(3, stplayer.age);             // 获取年龄
    cmdstr.getvalue(4, stplayer.weight);        // 获取体重
    cmdstr.getvalue(5, stplayer.sal);               // 获取年薪，欧元。
    cmdstr.getvalue(6, stplayer.club,50);        // 获取效力的俱乐部
  
    printf("name=%s,no=%s,striker=%d,age=%d,weight=%.1f,sal=%ld,club=%s\n",\
               stplayer.name,stplayer.no,stplayer.striker,stplayer.age,\
               stplayer.weight,stplayer.sal,stplayer.club);
    // 输出结果:name=messi,no=10,striker=1,age=30,weight=68.5,sal=21000000,club=Barcelona
}


```