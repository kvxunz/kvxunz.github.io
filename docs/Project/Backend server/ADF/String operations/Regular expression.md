
# **正则表达式**

## **定义**
```c++
// 正则表达式，判断一个字符串是否匹配另一个字符串。
// str：需要判断的字符串，是精确表示的，如文件名"_public.cpp"。
// rules：匹配规则的表达式，用星号"*"代表任意字符，多个表达式之间用半角的逗号分隔，如"*.h,*.cpp"。
// 注意：1）str参数不需要支持"*"，rules参数支持"*"；2）函数在判断str是否匹配rules的时候，会忽略字母的大小写。
```

```c++
bool matchstr(const string &str,const string &rules);
```

## **实现**
```c++
bool matchstr(const string &str,const string &rules)
{
    // 如果匹配规则表达式的内容是空的，返回false。
    if (rules.length() == 0) return false;

    // 如果如果匹配规则表达式的内容是"*"，直接返回true。
    if (rules == "*") return true;

    int  ii,jj;
    int  pos1,pos2;
    ccmdstr cmdstr,cmdsubstr;

    string filename=str;
    string matchstr=rules;

    // 把字符串都转换成大写后再来比较
    toupper(filename);
    toupper(matchstr);

    cmdstr.splittocmd(matchstr,",");

    for (ii=0;ii<cmdstr.size();ii++)
    {
        // 如果为空，就一定要跳过，否则就会被匹配上。
        if (cmdstr[ii].empty() == true) continue;

        pos1=pos2=0;
        cmdsubstr.splittocmd(cmdstr[ii],"*");

        for (jj=0;jj<cmdsubstr.size();jj++)
        {
            // 如果是文件名的首部
            if (jj == 0)
                if (filename.substr(0,cmdsubstr[jj].length())!=cmdsubstr[jj]) break;

            // 如果是文件名的尾部
            if (jj == cmdsubstr.size()-1)
                if (filename.find(cmdsubstr[jj],filename.length()-cmdsubstr[jj].length()) == string::npos) break;

            pos2=filename.find(cmdsubstr[jj],pos1);

            if (pos2 == string::npos) break;

            pos1=pos2+cmdsubstr[jj].length();
        }

        if (jj==cmdsubstr.size()) return true;
    }

    return false;
}


```

## **测试**
```c++
#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    // 以下代码将输出yes。
    if (matchstr("_public.h","*.h,*.cpp")==true) printf("yes\n");
    else printf("no\n");

    // 以下代码将输出yes。
    if (matchstr("_public.h","*.H")==true) printf("yes\n");
    else printf("no\n");

    // 以下代码将输出no。
    if (matchstr("_public.h","*p*k*.h")==true) printf("yes\n");
    else printf("no\n");
}
```