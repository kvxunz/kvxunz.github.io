
# **解析xml格式字符串的函数族**

## **定义**
```c++
// xml格式的字符串的内容如下：
// <filename>/tmp/_public.h</filename><mtime>2020-01-01 12:20:35</mtime><size>18348</size>
// <filename>/tmp/_public.cpp</filename><mtime>2020-01-01 10:10:15</mtime><size>50945</size>
// xmlbuffer：待解析的xml格式字符串。
// fieldname：字段的标签名。
// value：传入变量的地址，用于存放字段内容，支持bool、int、insigned int、long、
//       unsigned long、double和char[]。
// 注意：当value参数的数据类型为char []时，必须保证value数组的内存足够，否则可能发生内存溢出的问题，
//           也可以用ilen参数限定获取字段内容的长度，ilen的缺省值为0，表示不限长度。
// 返回值：true-成功；如果fieldname参数指定的标签名不存在，返回失败。
```
```c++
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,string &value,const int ilen=0);    // 视频中没有第三个参数，加上第三个参数更好。
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,char *value,const int ilen=0);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,bool &value);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,int  &value);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,unsigned int &value);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,long &value);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,unsigned long &value);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,double &value);
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,float &value);
```

## **实现**
```c++
bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,char *value,const int len)
{
    if (value==nullptr) return false;

    if (len>0) memset(value,0,len+1);   // 调用者必须保证value的空间足够，否则这里会内存溢出。

    string str;
    getxmlbuffer(xmlbuffer,fieldname,str);

    if ( (str.length()<=(unsigned int)len) || (len==0) )
    {
        str.copy(value,str.length());
        value[str.length()]=0;    // string的copy函数不会给C风格字符串的结尾加0。
    }
    else
    {
        str.copy(value,len);
        value[len]=0;
    }

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,bool &value)
{
    string str;
    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    toupper(str);    // 转换为大写来判断（也可以转换为小写，效果相同）。

    if (str=="TRUE") value=true; 
    else value=false;

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,int &value)
{
    string str;

    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    try
    {
       value = stoi(picknumber(str,true));  // stoi有异常，需要处理异常。
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,unsigned int &value)
{
    string str;

    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    try
    {
       value = stoi(picknumber(str));  // stoi有异常，需要处理异常。不提取符号 + -
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,long &value)
{
    string str;

    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    try
    {
        value = stol(picknumber(str,true));  // stol有异常，需要处理异常。
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,unsigned long &value)
{
    string str;

    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    try
    {
        value = stoul(picknumber(str));  // stoul有异常，需要处理异常。不提取符号 + -
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,double &value)
{
    string str;

    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    try
    {
        value = stod(picknumber(str,true,true));  // stod有异常，需要处理异常。提取符号和小数点。
    }
    catch(const std::exception& e)
    {
        return false;
    }

    return true;
}

bool getxmlbuffer(const string &xmlbuffer,const string &fieldname,float &value)
{
    string str;

    if (getxmlbuffer(xmlbuffer,fieldname,str)==false) return false;

    try
    {
        value = stof(picknumber(str,true,true));  // stof有异常，需要处理异常。提取符号和小数点。
    }
    catch(const std::exception& e)
    {
        return false;
    }

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
    // 球员梅西的资料存放在xml中。
    string buffer="<name>梅西</name><no>10</no><striker>true</striker><age>30</age><weight>68.5</weight><sal>21000000</sal><club>Barcelona</club>";

    // 用于存放足球运动员资料的结构体。
    struct st_player
    {
        string name;      // 姓名
        char no[6];       // 球衣号码
        bool striker;     // 场上位置是否是前锋，true-是；false-不是。
        int  age;         // 年龄
        double weight;    // 体重，kg。
        long sal;         // 年薪，欧元。
        char club[51];    // 效力的俱乐部
    }stplayer;

    getxmlbuffer(buffer,"name",stplayer.name);
    cout << "name=" << stplayer.name << endl;

    getxmlbuffer(buffer,"no",stplayer.no,5);
    cout << "no=" << stplayer.no << endl;

    getxmlbuffer(buffer,"striker",stplayer.striker);
    cout << "striker=" << stplayer.striker << endl;

    getxmlbuffer(buffer,"age",stplayer.age);
    cout << "age=" << stplayer.age << endl;

    getxmlbuffer(buffer,"weight",stplayer.weight);
    cout << "weight=" << stplayer.weight << endl;

    getxmlbuffer(buffer,"sal",stplayer.sal);
    cout << "sal=" << stplayer.sal << endl;

    getxmlbuffer(buffer,"club",stplayer.club,50);
    cout << "club=" << stplayer.club << endl;
}

```