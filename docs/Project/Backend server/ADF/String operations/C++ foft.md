
# **C++格式化输出函数模板**

## **定义**
```c++
template< typename... Args >
bool sformat(string &str,const char* fmt, Args... args ) 
{
    int len = snprintf( nullptr, 0, fmt, args... );      // 得到格式化输出后字符串的总长度。
    if (len < 0) return false;                                  // 如果调用snprintf失败，返回-1。
    if (len == 0) { str.clear(); return true; }            // 如果调用snprintf返回0，表示格式化输出的内容为空。

    str.resize(len);                                                 // 为string分配内存。
    snprintf(&str[0], len + 1, fmt, args... );           // linux平台第二个参数是len+1，windows平台是len。
    return true;
}

template< typename... Args >
string sformat(const char* fmt, Args... args ) 
{
    string str;

    int len = snprintf( nullptr, 0, fmt, args... );      // 得到格式化后字符串的长度。
    if (len < 0) return str;              // 如果调用snprintf失败，返回-1。
    if (len == 0) return str;           // 如果调用snprintf返回0，表示格式化输出的内容为空。;

    str.resize(len);                                                // 为string分配内存。
    snprintf(&str[0], len + 1, fmt, args... );          // linux平台第二个参数是len+1，windows平台是len。
    return str;
}

```



## **测试**
```c++
#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    int bh=1;
    char name[31]; strcpy(name,"西施");
    double weight=48.2;
    string yz="漂亮";

    char s1[100];
    int len=snprintf(s1,100,"编号=%02d,姓名=%s,体重=%.2f,颜值=%s",bh,name,weight,yz.c_str());
    cout << "s1=" << s1 << endl;
    
    printf("len=%d\n",len);                 // 为50字节 一个中文占3个字节
    printf("strlen(s1)=%zu\n",strlen(s1)); // 使用 %zu 输出 size_t 类型

    string s2;
    s2="编号="+to_string(bh)+",姓名="+name+",体重="+to_string(weight)+",颜值="+yz;
    cout << "s2=" << s2 << endl;

    sformat(s2,"编号=%02d,姓名=%s,体重=%.2f,颜值=%s",bh,name,weight,yz.c_str());
    cout << "s2=" << s2 << endl;

    s2=sformat("编号=%02d,姓名=%s,体重=%.2f,颜值=%s",bh,name,weight,yz.c_str());
    cout << "s2=" << s2 << endl;



    return 0;
}
```

