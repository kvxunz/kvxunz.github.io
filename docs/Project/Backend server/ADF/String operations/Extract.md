
# **字符串提取**

## **定义**
```c++
// 从一个字符串中提取出数字、符号和小数点，存放到另一个字符串中。
// src：原字符串。
// dest：目标字符串。
// bsigned：是否提取符号（+和-），true-包括；false-不包括。
// bdot：是否提取小数点（.），true-包括；false-不包括。
// 注意：src和dest可以是同一个变量。
```

```c++
char*     picknumber(const string &src,char *dest,const bool bsigned=false,const bool bdot=false);
string& picknumber(const string &src,string &dest,const bool bsigned=false,const bool bdot=false);
string    picknumber(const string &src,const bool bsigned=false,const bool bdot=false);
```

## **实现**
```c++
char* picknumber(const string &src,char *dest,const bool bsigned,const bool bdot)
{
    if (dest==nullptr) return nullptr;    // 判断空指针。

    string strtemp=picknumber(src,bsigned,bdot);
    strtemp.copy(dest,strtemp.length());
    dest[strtemp.length()]=0;    // string的copy函数不会给C风格字符串的结尾加0。

    return dest;
}

string& picknumber(const string &src,string &dest,const bool bsigned,const bool bdot)
{
    // 为了支持src和dest是同一变量的情况，定义str临时变量。
    string str;

    for (char cc:src)
    {
        // 判断是否提取符号。
        if ( (bsigned==true) && ( (cc == '+') || (cc == '-') ))
        {
            str.append(1,cc); continue;
        }

        // 判断是否提取小数点。
        if ( (bdot==true) && (cc == '.') )
        {
            str.append(1,cc); continue;
        }

        // 提取数字。
        if (isdigit(cc)) str.append(1,cc);
    }

    dest=str;

    return dest;
}

string picknumber(const string &src,const bool bsigned,const bool bdot)
{
    string dest;
    picknumber(src,dest,bsigned,bdot);
    return dest;
}

```

## **测试**
```c++
#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    char str1[30];   
    string str2;

    strcpy(str1,"iab+12.3xy");
    picknumber(str1,str1,false,false);
    printf("str1=%s=\n",str1);    // 出输结果是str1=123=

    str2="iab+12.3xy";
    picknumber(str2,str2,false,false);
    cout << "str2=" << str2 << "=\n";  // 出输结果是str2=123=

    strcpy(str1,"iab+12.3xy");
    picknumber(str1,str1,true,false);
    printf("str1=%s=\n",str1);         // 出输结果是str1=+123=

    str2="iab+12.3xy";
    picknumber(str2,str2,true,false);
    cout << "str2=" << str2 << "=\n";  // 出输结果是str2=+123=

    strcpy(str1,"iab+12.3xy");
    picknumber(str1,str1,true,true);
    printf("str1=%s=\n",str1);         // 出输结果是str1=+12.3=

    str2="iab+12.3xy";
    picknumber(str2,str2,true,true);
    cout << "str2=" << str2 << "=\n";  // 出输结果是str2=+12.3=
}


```