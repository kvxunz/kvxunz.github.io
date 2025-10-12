[//]: # (---)

[//]: # (hide:)

[//]: # (  - toc)

[//]: # (  - feedback)

[//]: # (---)

## **简介**

- `UNIX`操作系统根据计算机产生的年代把1970年1月1日作为UNIX的纪元时间，1970年1月1日是时间的中间点，将从1970年1月1日起经过的秒数用一个整数存放。




## **操作函数**

1.  **time_t别名**
    - `time_t`用于表示时间类型，它是一个long类型的别名，在`<time.h>`文件中定义，表示从1970年1月1日0时0分0秒到现在的秒数。
    - `typedef long time_t`;

2.  **time()库函数**
    - `time()`库函数用于获取操作系统的当前时间
    - 包含头文件：`<time.h>`
    - 两种调用方法:
        - `time_t now=time(0)`;         
            将空地址传递给`time()`函数，并将`time()`返回值赋给变量`now`
        - `time_t now; time(&now)`;   
            将变量now的地址作为参数传递给`time()`函数


3. **tm结构体**
    - `time_t`是一个长整数，不符合人类的使用习惯，需要转换成`tm`结构体，`tm`结构体在`<time.h>`中声明，如: `2022-10-01 15:30:25   Oct 1,2022 15:30:25`
    - `tm`结构体的定义如下:

        ``` 
        struct tm
        {
            int tm_year;	// 年份：其值等于实际年份减去1900
            int tm_mon;	// 月份：取值区间为[0,11]，其中0代表一月，11代表12月
            int tm_mday;	// 日期：一个月中的日期，取值区间为[1,31]
            int tm_hour; 	// 时：取值区间为[0,23]
            int tm_min;	// 分：取值区间为[0,59]
            int tm_sec;     	// 秒：取值区间为[0,59]
            int tm_wday;	// 星期：取值区间为[0,6]，其中0代表星期天，6代表星期六
            int tm_yday;	// 从每年的1月1日开始算起的天数：取值区间为[0,365] 
            int tm_isdst;   // 夏令时标识符，该字段意义不大
        };
        ```

4. **localtime()库函数**
    - `localtime()`函数用于把`time_t`表示的时间转换为`tm`结构体表示的时间;
    - `localtime()`函数不是线程安全的，`localtime_r()`是线程安全的;
    - 包含头文件：`<time.h>`
    - 函数声明：
        - struct tm *localtime(const time_t *timep);
        - struct tm *localtime_r(const time_t *timep, struct tm *result);
    - 示例:
    ```
      #include <iostream>
      #include <time.h>      // 时间操作的头文件。
      using namespace std;
      
      int main()
      {
        time_t now=time(0);             // 获取当前时间，存放在now中。
      
        cout << "now=" << now << endl;  // 显示当前时间，1970年1月1日到现在的秒数。
      
        tm tmnow;
        localtime_r(&now,&tmnow);       // 把整数的时间转换成tm结构体。
      
        // 根据tm结构体拼接成中国人习惯的字符串格式。
        string stime = to_string(tmnow.tm_year+1900)+"-"
                     + to_string(tmnow.tm_mon+1)+"-"
                     + to_string(tmnow.tm_mday)+" "
                     + to_string(tmnow.tm_hour)+":"
                     + to_string(tmnow.tm_min)+":"
                     + to_string(tmnow.tm_sec);
      
        cout << "stime=" << stime << endl;
      }
    ```  

5. **mktime()库函数**
    - mktime()函数的功能与localtime()函数相反，用于把tm结构体时间转换为time_t时间
    - 包含头文件：<time.h>
    - 函数声明：
        - time_t mktime(struct tm *tm);
        - 该函数主要用于时间的运算，例如：把2022-03-01 00:00:25加30分钟
    - 思路：
        1. 解析字符串格式的时间，转换成`tm`结构体；
        2. 用`mktime()`函数把tm结构体转换成`time_t`时间；
        3. 把`time_t`时间加`30*60`秒；
        4. 用`localtime_r()`函数把`time_t`时间转换成`tm`结构体；
        5. 把`tm`结构体转换成字符串;
        
6. **gettimeofday()库函数**
    - 用于获取1970年1月1日到现在的秒和当前秒中已逝去的微秒数，可以用于程序的计时
    - 包含头文件：`<sys/time.h>`
    - 函数声明：

        ``` 
        int gettimeofday(struct timeval *tv, struct timezone *tz);
        ```
    
        ```
        struct timeval {
        time_t      tv_sec;    	/* 1970-1-1到现在的秒数 */
        suseconds_t tv_usec;   	/* 当前秒中，已逝去的微秒数 */
        }; 
        ```

        ```
        struct timezone {         /* 在实际开发中，派不上用场 */
            int tz_minuteswest;   	/* minutes west of Greenwich */ 
            int tz_dsttime;         	/* type of DST correction */
        }; 
        ```

    - 示例：
        ```
        #include <iostream>
        #include <sys/time.h>  // gettimeofday()需要的头文件。
        using namespace std;
        
        int main()
        {
          timeval start,end;
        
          gettimeofday(&start, 0 ); // 计时开始。
        
          for (int ii=0;ii<1000000000;ii++)
            ;
        
          gettimeofday(&end, 0 );   // 计时结束。
        
          // 计算消耗的时长。
          timeval tv;
          tv.tv_usec=end.tv_usec-start.tv_usec;
          tv.tv_sec=end.tv_sec-start.tv_sec;
          if (tv.tv_usec<0)
          {
            tv.tv_usec=1000000-tv.tv_usec;
            tv.tv_sec--;
          }
        
          cout << "耗时：" << tv.tv_sec << "秒和" << tv.tv_usec << "微秒。\n";
        }
        ```

7. **程序睡眠**
    - 如果需要把程序挂起一段时间，可以使用`sleep()`和`usleep()`两个库函数。
    - 包含头文件：`<unistd.h>`
    - 函数声明：
        - `unsigned int sleep(unsigned int seconds);`
        - `int usleep(useconds_t usec);`
    