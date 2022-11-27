# 步步深入MUDUO库：日志系统与时间

在开发过程中，程序可能会出现很多错误，为了记录这些错误方便我们进行调试，于是我们要用一个日志类记载运行日志，日志记载的时候需要时间作为依据。故我们要实现一个日志类，和一个时间类

## 日志类

##### 需求：

###### Logger首先是不含拷贝语义的：

日志只有一个唯一的实例对象

###### Logger是具有等级的：

有四个等级：普通信息>错误信息>core信息>调试信息。等级的作用就是如果定义了等级大的，那么将等级小的将不会显示，只显示等级大的。用户可以改变日志的等级。

###### Logger是具有写功能：

用户可以写日志，日志的信息是可变的，能够自动识别有各种数据



##### 具体实现思路：

1.我们可以用nocopyable限制Logger，其次日志有一个唯一的实例对象，我们用单例模式创建日志（采用静态类手法，保证了全局唯一，线程安全，且资源自动释放）。

2.等级我们可以用枚举类枚举四个等级，Logger类中唯一一个描述等级的变量

3.我们类中定义一个写日志的接口，至于写日志的核心，采用可变参数+宏定义运算符`__VA_ARGS__`配合写日志。

##### 具体实现

```cpp
#include "Logger.h"
#include <iostream>
//#include "Timestamp.h"






Logger& Logger::instance() {
	static Logger logger;
	return logger;
}


void Logger::setloglevel(int level) {
	this->level_ = level;
}

void Logger::log(string msg){
	switch (this->level_)
	{
	case INFO:
		cout << "[INFO]";
		break;
	case ERROR:
		cout << "[ERROR]";
		break;
	case FATAL:
		cout << "[FATAL]";
		break;
	case DEBUG:
		cout << "[DEBUG]";
		break;
	default:
		break;
	}
	cout << "[TIME]" << ":" << msg << endl;

}

```



头文件

```cpp
#pragma once

#include <string>
#include "noncopyable.h"
#include <iostream>

using namespace std;


#define LOG_INFO(logmsgFormat, ...) \
    do \
    { \
        Logger &logger = Logger::instance(); \
        logger.setloglevel(INFO); \
        char buf[1024] = {0}; \
        snprintf(buf, 1024, logmsgFormat, ##__VA_ARGS__); \
        logger.log(buf); \
    } while(0) 

#define LOG_ERROR(logmsgFormat, ...) \
    do \
    { \
        Logger &logger = Logger::instance(); \
        logger.setLogLevel(ERROR); \
        char buf[1024] = {0}; \
        snprintf(buf, 1024, logmsgFormat, ##__VA_ARGS__); \
        logger.log(buf); \
    } while(0) 

#define LOG_FATAL(logmsgFormat, ...) \
    do \
    { \
        Logger &logger = Logger::instance(); \
        logger.setLogLevel(FATAL); \
        char buf[1024] = {0}; \
        snprintf(buf, 1024, logmsgFormat, ##__VA_ARGS__); \
        logger.log(buf); \
        exit(-1); \
    } while(0) 

#ifdef MUDEBUG
#define LOG_DEBUG(logmsgFormat, ...) \
    do \
    { \
        Logger &logger = Logger::instance(); \
        logger.setLogLevel(DEBUG); \
        char buf[1024] = {0}; \
        snprintf(buf, 1024, logmsgFormat, ##__VA_ARGS__); \
        logger.log(buf); \
    } while(0) 
#else
#define LOG_DEBUG(logmsgFormat, ...)
#endif

//日志等级类别
enum {
    INFO = 0, //普通信息
    ERROR, //错误信息
    FATAL, //core信息
    DEBUG, //调试信息
};



class Logger :noncopyable
{
public:
    static Logger& instance(); //创建日志实例
    void setloglevel(int level); //设置日志等级
    void log(string msg); //写日志
private:
    int level_;

};
```







## 时间类



##### 需求：

###### 能够在终端显示当前时间

##### 具体实现思路：

用64位存储时间，有一个转化函数将该位转化成字符串string。对外有个接口now()



##### 使用函数细节：

**结构体**

###### 1.time_t

64位长整型

```cpp
typedef __time64_t time_t;   /* 时间值time_t 为长整型的别名*/

//从一个时间点（一般是1970年1月1日0时0分0秒）到那时的秒数.
```

###### 2.tm结构体

```cpp
struct tm
{
    int tm_sec;  /*秒，正常范围0-59， 但允许至61*/
    int tm_min;  /*分钟，0-59*/
    int tm_hour; /*小时， 0-23*/
    int tm_mday; /*日，即一个月中的第几天，1-31*/
    int tm_mon;  /*月， 从一月算起，0-11*/  1+p->tm_mon;
    int tm_year;  /*年， 从1900至今已经多少年*/  1900＋ p->tm_year;
    int tm_wday; /*星期，一周中的第几天， 从星期日算起，0-6*/
    int tm_yday; /*从今年1月1日到目前的天数，范围0-365*/
    int tm_isdst; /*日光节约时间的旗标*/
};
```

###### 3.timeval结构体

```cpp
struct timeval
{
    time_t      tv_sec;     /* seconds */
    suseconds_t tv_usec;    /* microseconds */
};
//该结构体以1970-01-01 00:00:00 +0000 (UTC)，也就是Unix中的Epoch作为0，之后的时间都是相对于Epoch流逝的秒和毫秒数。直接用函数 gettimeofday 就可以获得时间。
```

****

**获取时间函数**

###### 4.time()

```cpp
函数定义:time_t time(time_t *seconds)

函数说明：C 库函数 time_t time(time_t *seconds) 返回自纪元 Epoch（1970-01-01 00:00:00 UTC）起经过的时间，以秒为单位。如果 seconds 不为空，则返回值也存储在变量 seconds 中。

返回值:以 time_t 对象返回当前日历时间。

```

###### 5.gettimeofday():  (Linux)

```cpp
头文件：#include<sys/time.h>

函数定义：int gettimeofday(struct timeval *tv,struct timezone *tz)

函数说明：用于获取调用该代码时，距离Epoch的时间
```



###### 6.localtime();

```cpp
struct tm *localtime(const time_t *clock);
localtime是 把从1970-1-1零点零分到当前所偏移的秒数(clock的值)时间转换为本地时间，而gmtime函数转换后的时间没有经过时区变换，是UTC时间 
```

**时间转化函数**

###### 7.ctime();

```cpp
函数原型：char *ctime(const time_t *timer)

函数说明：将timer转换成字符串。字符串格式如下：time now:Sun Nov 14 21:55:19 2021

```

