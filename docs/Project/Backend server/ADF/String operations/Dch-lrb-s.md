
# **删除字符串左、右、两边指定字符**

## **定义**
```c++
// C++风格字符串操作的若干函数。
// 删除字符串左边指定的字符。
// str：待处理的字符串。
// cc：需要删除的字符，缺省删除空格。
char*     deletelchr(char* str, const int cc=' ');
string& deletelchr(string &str, const int cc=' ');

// 删除字符串右边指定的字符。
// str：待处理的字符串。
// cc：需要删除的字符，缺省删除空格。
char*     deleterchr(char *str,const int cc=' ');
string& deleterchr(string &str,const int cc=' ');

// 删除字符串左右两边指定的字符。
// str：待处理的字符串。
// chr：需要删除的字符，缺省删除空格。
char*     deletelrchr(char *str,const int cc=' ');
string& deletelrchr(string &str,const int cc=' ');

// 把字符串中的小写字母转换成大写，忽略不是字母的字符。
// str：待转换的字符串。
char*     toupper(char *str);
string& toupper(string &str);
```

## **实现**
```c++
char *deletelchr(char* str, const int cc)
{
    if (str == nullptr) return nullptr;     // 如果传进来的是空地址，直接返回，防止程序崩溃。

    char* p = str;                 // 指向字符串的首地址。
    while (*p == cc)            // 遍历字符串，p将指向左边第一个不是cc的字符。
        p++;        

    memmove(str, p, strlen(str) - (p - str)+1);  // 把结尾标志0也拷过来。

    return str;
}

string& deletelchr(string &str, const int cc)
{
    auto pos=str.find_first_not_of(cc);    // 从字符串的左边查找第一个不是cc的字符的位置。

    if (pos!= 0) str.replace(0,pos,"");       // 把0-pos之间的字符串替换成空。

    return str;
}

char* deleterchr(char *str,const int cc)
{
    if (str == nullptr) return nullptr; // 如果传进来的是空地址，直接返回，防止程序崩溃。

    char* p = str;              // 指向字符串的首地址。
    char* piscc = 0;          // 右边全是字符cc的第一个位置。

    while (*p != 0)            // 遍历字符串。
    {
        if (*p == cc && piscc == 0) piscc = p;        // 记下字符cc的第一个位置。
        if (*p != cc) piscc = 0;                                  // 只要当前字符不是cc，清空piscc。
        p++;        
      }

      if (piscc != 0) *piscc = 0;   // 把piscc位置的字符置为0，表示字符串已结束。

    return str;
}

string& deleterchr(string &str,const int cc)
{
    auto pos=str.find_last_not_of(cc);     // 从字符串的右边查找第一个不是cc的字符的位置。

    if (pos!= 0) str.erase(pos+1);            // 把pos之后的字符删掉。

    return str;
}

char* deletelrchr(char *str,const int cc)
{
    deletelchr(str,cc);
    deleterchr(str,cc);

    return str;
}

string& deletelrchr(string &str,const int cc)
{
    deletelchr(str,cc);
    deleterchr(str,cc);

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
    char str1[31];        // C风格的字符串。
    string str2;            // C++风格的字符串。

    strcpy(str1,"  西施  ");
    deletelchr(str1,' ');                // 删除str1左边的空格
    printf("str1=%s=\n",str1);    // 出输结果是str1=西施  =

    str2="  西施  ";
    deletelchr(str2,' ');
    cout << "str2=" << str2 << "=\n";

    strcpy(str1,"  西施  ");
    deleterchr(str1,' ');                // 删除str1左边的空格
    printf("str1=%s=\n",str1);    // 出输结果是str1=西施  =

    str2="  西施  ";
    deleterchr(str2,' ');
    cout << "str2=" << str2 << "=\n";

    strcpy(str1,"  西施  ");
    deletelrchr(str1,' ');               // 删除str1两边的空格
    printf("str1=%s=\n",str1);    // 出输结果是str1=西施=

    str2="  西施  ";
    deletelrchr(str2,' ');
    cout << "str2=" << str2 << "=\n";
}

```