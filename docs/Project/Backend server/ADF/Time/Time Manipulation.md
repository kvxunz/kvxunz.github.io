
# **时间操作**

## **定义**
```c++

取操作系统的时间（用字符串表示）。
strtime：用于存放获取到的时间。
timetvl：时间的偏移量，单位：秒，0是缺省值，表示当前时间，30表示当前时间30秒之后的时间点，-30表示当前时间30秒之前的时间点。
fmt：输出时间的格式，fmt每部分的含义：yyyy-年份；mm-月份；dd-日期；hh24-小时；mi-分钟；ss-秒，
缺省是"yyyy-mm-dd hh24:mi:ss"，目前支持以下格式：
"yyyy-mm-dd hh24:mi:ss"
"yyyymmddhh24miss"
"yyyy-mm-dd"
"yyyymmdd"
"hh24:mi:ss"
"hh24miss"
"hh24:mi"
"hh24mi"
"hh24"
"mi"
注意：
1）小时的表示方法是hh24，不是hh，这么做的目的是为了保持与数据库的时间表示方法一致；
2）以上列出了常用的时间格式，如果不能满足你应用开发的需求，请修改源代码timetostr()函数增加更多的格式支持；
3）调用函数的时候，如果fmt与上述格式都匹配，strtime的内容将为空。
4）时间的年份是四位，其它的可能是一位和两位，如果不足两位，在前面补0。
```

```c++
string& ltime(string &strtime,const string &fmt="",const int timetvl=0);
char *    ltime(char *strtime   ,const string &fmt="",const int timetvl=0);
// 为了避免重载的岐义，增加ltime1()函数。
string    ltime1(const string &fmt="",const int timetvl=0);

// 把整数表示的时间转换为字符串表示的时间。
// ttime：整数表示的时间。
// strtime：字符串表示的时间。
// fmt：输出字符串时间strtime的格式，与ltime()函数的fmt参数相同，如果fmt的格式不正确，strtime将为空。
string& timetostr(const time_t ttime,string &strtime,const string &fmt="");
char*     timetostr(const time_t ttime,char *strtime   ,const string &fmt="");
// 为了避免重载的岐义，增加timetostr1()函数。
string    timetostr1(const time_t ttime,const string &fmt="");

// 把字符串表示的时间转换为整数表示的时间。
// strtime：字符串表示的时间，格式不限，但一定要包括yyyymmddhh24miss，一个都不能少，顺序也不能变。
// 返回值：整数表示的时间，如果strtime的格式不正确，返回-1。
time_t strtotime(const string &strtime);

// 把字符串表示的时间加上一个偏移的秒数后得到一个新的字符串表示的时间。
// in_stime：输入的字符串格式的时间，格式不限，但一定要包括yyyymmddhh24miss，一个都不能少，顺序也不能变。
// out_stime：输出的字符串格式的时间。
// timetvl：需要偏移的秒数，正数往后偏移，负数往前偏移。
// fmt：输出字符串时间out_stime的格式，与ltime()函数的fmt参数相同。
// 注意：in_stime和out_stime参数可以是同一个变量的地址，如果调用失败，out_stime的内容会清空。
// 返回值：true-成功，false-失败，如果返回失败，可以认为是in_stime的格式不正确。
bool addtime(const string &in_stime,char *out_stime    ,const int timetvl,const string &fmt="");
bool addtime(const string &in_stime,string &out_stime,const int timetvl,const string &fmt="");

```
```c++
// 这是一个精确到微秒的计时器。
class ctimer
{
private:
    struct timeval m_start;    // 计时开始的时间点。
    struct timeval m_end;     // 计时结束的时间点。
public:
    ctimer();          // 构造函数中会调用start方法。

    void start();     // 开始计时。

    // 计算已逝去的时间，单位：秒，小数点后面是微秒。
    // 每调用一次本方法之后，自动调用start方法重新开始计时。
    double elapsed();
};
```

## **实现**
```c++
// 把整数表示的时间转换为字符串表示的时间。
// ttime：整数表示的时间。
// strtime：字符串表示的时间。
// fmt：输出字符串时间strtime的格式，与ttime函数的fmt参数相同，如果fmt的格式不正确，strtime将为空。
```
```c++
string& timetostr(const time_t ttime,string &strtime,const string &fmt)
{
    //struct tm sttm = *localtime ( &ttime );        // 非线程安全。
    struct tm sttm; localtime_r (&ttime,&sttm);   // 线程安全。
    sttm.tm_year=sttm.tm_year+1900;                // tm.tm_year成员要加上1900。
    sttm.tm_mon++;                                            // sttm.tm_mon成员是从0开始的，要加1。

    // 缺省的时间格式。
    if ( (fmt=="") || (fmt=="yyyy-mm-dd hh24:mi:ss") )
    {
        strtime=sformat("%04u-%02u-%02u %02u:%02u:%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday,\
                   sttm.tm_hour,sttm.tm_min,sttm.tm_sec);
        return strtime;
    }

    if (fmt=="yyyy-mm-dd hh24:mi")
    {
        strtime=sformat("%04u-%02u-%02u %02u:%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday,\
                   sttm.tm_hour,sttm.tm_min);
        return strtime;
    }

    if (fmt=="yyyy-mm-dd hh24")
    {
        strtime=sformat("%04u-%02u-%02u %02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday,sttm.tm_hour);
        return strtime;
    }

    if (fmt=="yyyy-mm-dd")
    {
        strtime=sformat("%04u-%02u-%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday); 
        return strtime;
    }

    if (fmt=="yyyy-mm")
    {
        strtime=sformat("%04u-%02u",sttm.tm_year,sttm.tm_mon); 
        return strtime;
    }

    if (fmt=="yyyymmddhh24miss") 
    {
        strtime=sformat("%04u%02u%02u%02u%02u%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday,\
                   sttm.tm_hour,sttm.tm_min,sttm.tm_sec);
        return strtime;
    }

    if (fmt=="yyyymmddhh24mi")
    {
        strtime=sformat("%04u%02u%02u%02u%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday,\
                   sttm.tm_hour,sttm.tm_min);
        return strtime;
    }

    if (fmt=="yyyymmddhh24")
    {
        strtime=sformat("%04u%02u%02u%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday,sttm.tm_hour);
        return strtime;
    }

    if (fmt=="yyyymmdd")
    {
        strtime=sformat("%04u%02u%02u",sttm.tm_year,sttm.tm_mon,sttm.tm_mday); 
        return strtime;
    }

    if (fmt=="hh24miss")
    {
        strtime=sformat("%02u%02u%02u",sttm.tm_hour,sttm.tm_min,sttm.tm_sec); 
        return strtime;
    }

    if (fmt=="hh24mi") 
    {
        strtime=sformat("%02u%02u",sttm.tm_hour,sttm.tm_min); 
        return strtime;
    }

    if (fmt=="hh24")
    {
        strtime=sformat("%02u",sttm.tm_hour); 
        return strtime;
    }

    if (fmt=="mi")
    {
        strtime=sformat("%02u",sttm.tm_min); 
        return strtime;
    }

    return strtime;
}

```
```c++
char* timetostr(const time_t ttime,char *strtime,const string &fmt)
{
    if (strtime==nullptr) return nullptr;    // 判断空指针。

    string str;
    timetostr(ttime,str,fmt);           // 直接调用string& timetostr(const time_t ttime,string &strtime,const string &fmt="");
    str.copy(strtime,str.length());
    strtime[str.length()]=0;           // string的copy函数不会给C风格字符串的结尾加0。

    return strtime;
}

```


```c++
string timetostr1(const time_t ttime,const string &fmt)
{
    string str;
    timetostr(ttime,str,fmt);           // 直接调用string& timetostr(const time_t ttime,string &strtime,const string &fmt="");
    return str;
}

```

```c++
string& ltime(string &strtime,const string &fmt,const int timetvl)
{
    time_t  timer;
    time(&timer );                          // 获取系统当前时间。

    timer=timer+timetvl;              // 加上时间的偏移量。

    timetostr(timer,strtime,fmt);   // 把整数表示的时间转换为字符串表示的时间。

    return strtime;
}

```

```c++
char* ltime(char *strtime,const string &fmt,const int timetvl)
{
    if (strtime==nullptr) return nullptr;    // 判断空指针。

    time_t  timer;
    time(&timer );                          // 获取系统当前时间。

    timer=timer+timetvl;              // 加上时间的偏移量。

    timetostr(timer,strtime,fmt);   // 把整数表示的时间转换为字符串表示的时间。

    return strtime;
}

```

```c++
string ltime1(const string &fmt,const int timetvl)
{
    string strtime;

    ltime(strtime,fmt,timetvl);   // 直接调用string& ltime(string &strtime,const string &fmt="",const int timetvl=0);

    return strtime;
}

```
```c++
bool addtime(const string &in_stime,string &out_stime,const int timetvl,const string &fmt)
{
    time_t  timer;

    // 把字符串表示的时间转换为整数表示的时间，方便运算。
    if ( (timer=strtotime(in_stime))==-1) { out_stime=""; return false; }

    timer=timer+timetvl;  // 时间运算。

    // 把整数表示的时间转换为字符串表示的时间。
    timetostr(timer,out_stime,fmt);

    return true;
}
```

```c++
bool addtime(const string &in_stime,char *out_stime,const int timetvl,const string &fmt)
{
    if (out_stime==nullptr) return false;    // 判断空指针。

    time_t  timer;

    // 把字符串表示的时间转换为整数表示的时间，方便运算。
    if ( (timer=strtotime(in_stime))==-1) { strcpy(out_stime,""); return false; }

    timer=timer+timetvl;  // 时间运算。

    // 把整数表示的时间转换为字符串表示的时间。
    timetostr(timer,out_stime,fmt);

    return true;
}
```

```c++
// 计时开始。
void ctimer::start()
{
    memset(&m_start,0,sizeof(struct timeval));
    memset(&m_end,0,sizeof(struct timeval));

    gettimeofday(&m_start, 0);    // 获取当前时间，精确到微秒。
}


```

```c++

// 计算已逝去的时间，单位：秒，小数点后面是微秒
// 每调用一次本方法之后，自动调用Start方法重新开始计时。
double ctimer::elapsed()
{
    gettimeofday(&m_end,0);     // 获取当前时间作为计时结束的时间，精确到微秒。

    string str;
    str=sformat("%ld.%06ld",m_start.tv_sec,m_start.tv_usec);
    double dstart=stod(str);      // 把计时开始的时间点转换为double。

    str=sformat("%ld.%06ld",m_end.tv_sec,m_end.tv_usec);
    double dend=stod(str);       // 把计时结束的时间点转换为double。

    start();                                  // 重新开始计时。

    return dend-dstart;
}

```


## **测试**
```c++
//ltime时间函数的使用（获取操作系统时间）
#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    // C风格的字符串。
    char strtime1[20];     // 存放系统时间。
    memset(strtime1,0,sizeof(strtime1));

    ltime(strtime1,"yyyy-mm-dd hh24:mi:ss");        // 获取当前时间。
    printf("strtime1=%s\n",strtime1);

    ltime(strtime1,"yyyy-mm-dd hh24:mi:ss",-30);  // 获取30秒前的时间。
    printf("strtime1=%s\n",strtime1);

    ltime(strtime1,"yyyy-mm-dd hh24:mi:ss",30);    // 获取30秒后的时间。
    printf("strtime1=%s\n",strtime1);

    // C++风格的字符串。
    string strtime2;

    ltime(strtime2,"yyyy-mm-dd hh24:mi:ss");        // 获取当前时间。
    cout << "strtime2=" << strtime2 << "\n";

    ltime(strtime2,"yyyy-mm-dd hh24:mi:ss",-30);  // 获取30秒前的时间。
    cout << "strtime2=" << strtime2 << "\n";

    ltime(strtime2,"yyyy-mm-dd hh24:mi:ss",30);    // 获取30秒后的时间。
    cout << "strtime2=" << strtime2 << "\n";
}


```

```c++
//整数表示的时间和字符串表示的时间之间的转换。
#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    string strtime;
    strtime="2020-01-01 12:35:22";

    time_t ttime;
    ttime=strtotime(strtime);        // 转换为整数的时间
    printf("ttime=%ld\n",ttime);    // 输出ttime=1577853322
  
    char s1[20];                             // C风格的字符串。
    timetostr(ttime,s1,"yyyy-mm-dd hh24:mi:ss");  // 转换为字符串的时间
    cout << "s1=" << s1 << endl;

    string s2;                               // C++风格的字符串。
    timetostr(ttime,s2,"yyyy-mm-dd hh24:mi:ss");  // 转换为字符串的时间
    cout << "s2=" << s2 << endl;
}


```


```c++

//采用addtime函数进行时间的运算。

#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    char strtime[20];

    memset(strtime,0,sizeof(strtime));
    strcpy(strtime,"2020-01-20 12:35:22");

    char s1[20];         // C风格的字符串。
    addtime(strtime,s1,0-1*24*60*60);        // 减一天。
    printf("s1=%s\n",s1);           // 输出s1=2020-01-19 12:35:22
  
    string s2;            // C++风格的字符串。
    addtime(strtime,s2,2*24*60*60);      // 加两天。  172800
    cout << "s2=" << s2 << endl;         // 输出s2=2020-01-22 12:35:22
}



```


```c++

//演示ctimer类（计时器）的用法。

#include "../_public.h"
using namespace std;
using namespace idc;

int main()
{
    ctimer timer;

    printf("elapsed=%lf\n",timer.elapsed());
    sleep(1);
    printf("elapsed=%lf\n",timer.elapsed());
    sleep(1);
    printf("elapsed=%lf\n",timer.elapsed());
    usleep(1000);
    printf("elapsed=%lf\n",timer.elapsed());
    usleep(100);
    printf("elapsed=%lf\n",timer.elapsed());
    sleep(10);
    printf("elapsed=%lf\n",timer.elapsed());
}
```
