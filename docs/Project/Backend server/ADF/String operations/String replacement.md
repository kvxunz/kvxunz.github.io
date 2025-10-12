# **字符串替换函数**

## **定义**
```c++
// 字符串替换函数。
// 在字符串str中，如果存在字符串str1，就替换为字符串str2。
// str：待处理的字符串。
// str1：旧的内容。
// str2：新的内容。
// bloop：是否循环执行替换。
// 注意：
// 1、如果str2比str1要长，替换后str会变长，所以必须保证str有足够的空间，否则内存会溢出（C++风格字符串不存在这个问题）。
// 2、如果str2中包含了str1的内容，且bloop为true，这种做法存在逻辑错误，replacestr将什么也不做。
// 3、如果str2为空，表示删除str中str1的内容。
```

```c++
bool replacestr(char *str   ,const string &str1,const string &str2,const bool bloop=false);
bool replacestr(string &str,const string &str1,const string &str2,const bool bloop=false);
```

## **实现**
```c++
bool replacestr(char *str,const string &str1,const string &str2,bool bloop)
{
    if (str == nullptr) return false;
  
    string strtemp(str);

    replacestr(strtemp,str1,str2,bloop);

    strtemp.copy(str,strtemp.length());
    str[strtemp.length()]=0;    // string的copy函数不会给C风格字符串的结尾加0。

    return true;
}


bool replacestr(string &str,const string &str1,const string &str2,bool bloop)
{
    // 如果原字符串str或旧的内容str1为空，没有意义，不执行替换。
    if ( (str.length() == 0) || (str1.length() == 0) ) return false;
  
    // 如果bloop为true并且str2中包函了str1的内容，直接返回，因为会进入死循环，最终导致内存溢出。
    if ( (bloop==true) && (str2.find(str1)!=string::npos) ) return false;

    int pstart=0;      // 如果bloop==false，下一次执行替换的开始位置。
    int ppos=0;        // 本次需要替换的位置。

    while (true)
    {
        if (bloop == true)
            ppos=str.find(str1);                      // 每次从字符串的最左边开始查找子串str1。
        else
            ppos=str.find(str1,pstart);            // 从上次执行替换的位置后开始查找子串str1。

        if (ppos == string::npos) break;       // 如果没有找到子串str1。

        str.replace(ppos,str1.length(),str2);   // 把str1替换成str2。

        if (bloop == false) pstart=ppos+str2.length();    // 下一次执行替换的开始位置往右移动。
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
    char str1[301];

    strcpy(str1,"name:messi,no:10,job:striker.");
    replacestr(str1,":","=");         // 把冒号替换成等号。
    printf("str1=%s=\n",str1);    // 出输结果是str1=name=messi,no=10,job=striker.=

    strcpy(str1,"name:messi,no:10,job:striker.");
    replacestr(str1,"name:","");    // 把"name:"替换成""，相当于删除内容"name:"。
    printf("str1=%s=\n",str1);      // 出输结果是str1=messi,no:10,job:striker.=

    strcpy(str1,"messi----10----striker");  
    replacestr(str1,"--","-",false);    // 把两个"--"替换成一个"-"，bloop参数为false。
    printf("str1=%s=\n",str1);         // 出输结果是str1=messi--10--striker=

    strcpy(str1,"messi----10----striker");  
    replacestr(str1,"--","-",true);    // 把两个"--"替换成一个"-"，bloop参数为true。
    printf("str1=%s=\n",str1);        // 出输结果是str1=messi-10-striker=

    strcpy(str1,"messi-10-striker");  
    replacestr(str1,"-","--",false);    // 把一个"-"替换成两个"--"，bLoop参数为false。
    printf("str1=%s=\n",str1);         // 出输结果是str1=messi--10--striker=

    // 以下代码把"-"替换成"--"，bloop参数为true，存在逻辑错误，replacestr将不执行替换。
    strcpy(str1,"messi-10-striker");  
    replacestr(str1,"-","--",true);    // 把一个"-"替换成两个"--"，bloop参数为true。
    printf("str1=%s=\n",str1);        // 出输结果是str1=messi-10-striker=

    // ////////////////////////////////////
    string str2;
    str2="name:messi,no:10,job:striker.";
    replacestr(str2,":","=");                        // 把冒号替换成等号。
    cout << "str2=" << str2 << "=\n";    // 出输结果是str2=name=messi,no=10,job=striker.=

    str2="name:messi,no:10,job:striker.";
    replacestr(str2,"name:","");                  // 把"name:"替换成""，相当于删除内容"name:"。
    cout << "str2=" << str2 << "=\n";     // 出输结果是str2=messi,no:10,job:striker.=

    str2="messi----10----striker";  
    replacestr(str2,"--","-",false);               // 把两个"--"替换成一个"-"，bLoop参数为false。
    cout << "str2=" << str2 << "=\n";     // 出输结果是str2=messi--10--striker=

    str2="messi----10----striker";  
    replacestr(str2,"--","-",true);                // 把两个"--"替换成一个"-"，bLoop参数为true。
    cout << "str2=" << str2 << "=\n";     // 出输结果是str2=messi-10-striker=

    str2="messi-10-striker";  
    replacestr(str2,"-","--",false);               // 把一个"-"替换成两个"--"，bLoop参数为false。
    cout << "str2=" << str2 << "=\n";     // 出输结果是str2=messi--10--striker=

    // 以下代码把"-"替换成"--"，bloop参数为true，存在逻辑错误，updatestr将不执行替换。
    str2="messi-10-striker";  
    replacestr(str2,"-","--",true);                // 把一个"-"替换成两个"--"，bloop参数为true。
    cout << "str2=" << str2 << "=\n";     // 出输结果是str2=messi-10-striker=
}


```