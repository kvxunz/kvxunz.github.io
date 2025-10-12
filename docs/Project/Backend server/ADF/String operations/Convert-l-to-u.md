
# **小写字母转换成大写**


## **定义**

```c++
// 把字符串中的小写字母转换成大写，忽略不是字母的字符。
// str：待转换的字符串。
char*     toupper(char *str);
string& toupper(string &str);

// 把字符串中的大写字母转换成小写，忽略不是字母的字符。
// str：待转换的字符串。
char*     tolower(char *str);
string& tolower(string &str);
```

## **实现**
```c++
char* toupper(char *str)
{
    if (str == nullptr) return nullptr;

    char* p = str;				// 指向字符串的首地址。
    while (*p != 0)			  // 遍历字符串。
    {
        if ( (*p >= 'a') && (*p <= 'z') ) *p=*p - 32;
        p++;
    }

    return str;
}

string& toupper(string &str)
{
    for (auto &cc:str)
    {
        if ( (cc >= 'a') && (cc <= 'z') ) cc=cc - 32;
    }

    return str;
}

char* tolower(char *str)
{
    if (str == nullptr) return nullptr;

    char* p = str;				// 指向字符串的首地址。
    while (*p != 0)			  // 遍历字符串。
    {
        if ( (*p >= 'A') && (*p <= 'Z') ) *p=*p + 32;
        p++;
    }

    return str;
}

string& tolower(string &str)
{
    for (auto &cc:str)
    {
        if ( (cc >= 'A') && (cc <= 'Z') ) cc=cc + 32;
    }

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
    char str1[31];   // C风格的字符串。

    strcpy(str1,"12abz45ABz8西施。");
    toupper(str1);                       // 把str1中的小写字母转换为大写。
    printf("str1=%s=\n",str1);    // 出输结果是str1=12ABZ45ABZ8西施。=

    strcpy(str1,"12abz45ABz8西施。");
    tolower(str1);                       // 把str1中的大写字母转换为小写。
    printf("str1=%s=\n",str1);    // 出输结果是str1=12abz45abz8西施。=

    string str2;    // C++风格的字符串。
  
    str2="12abz45ABz8西施。";
    toupper(str2);                                    // 把str2中的小写字母转换为大写。
    cout << "str2=" << str2 << "=\n";   // 出输结果是str2=12ABZ45ABZ8西施。=

    str2="12abz45ABz8西施。";
    tolower(str2);                                     // 把str2中的大写字母转换为小写。
    cout << "str2=" << str2 << "=\n";   // 出输结果是str1=12abz45abz8西施。=
}
```