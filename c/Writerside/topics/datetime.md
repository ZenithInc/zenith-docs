# 时间的处理

这一篇文档主要描述了在 C 语言编程过程中，如何对时间进行正确的处理。其中包括如何获取系统时间、如何格式化时间、解析时间以及计算时间差等内容。

### 时间的标准 {id="datetime-standard"}

在日常的生活中，时间的显示利用并没有非常严格的标准。我们可以说 11 点 45 分，也可以说是午时三刻，也可以说是太阳当空的时候。只要人们能看得懂听得懂就好了。在计算机中，常用的时间标准有如下几种:

- UTC 是世界调和时间，是国际时间的标准，我们提及 UTC 时，它一定是一个确定的值，不受时区的影响。

- GMT 是格林尼治时间，与 UTC 的时间是一致的，但我们说起  GMT 的时候其实指的是零时区的时间，它现在已经不是时间标准了。

- Epoch，一般被翻译成纪元，时代，我们通常在计算机程序中使用的时间都是从 UTC 时间  1970 年 1 月 1 日 0 时 0 分 0 秒开始的一个整数值，这是 Unix 的计时方法。Unix 系统对 C 标准的扩展标准 POSIX 也采用了这样的规定，因此这个起始时间就被称之为 Unix Epoch。现在绝大多数的编程语言都采用了  Unix Epoch，Windows 上的 C 语言实现也是如此。

### 与时间相关的类型和结构体 {id="datetime-struct"}

在 C 语言中，有一些与时间相关的类型和结构体，是我们需要了解的。这些类型或者结构体和具体的平台、编译器也有关系，所以如果我们要编写跨平台的代码，也需要准确区不同平台之间的区别。

#### time_t 类型 {id="time-t"}

`time_t` 类型，它表示的就是 Epoch 时间，也就是 Unix 时间戳。有的平台比如 MSVC 上它是一个 `int64` 类型，而在 Linux 下它应该是一个 `long int` 类型。

#### clock_t 类型 {id="clock-t"}

`clock_t` 类型, 它指的是处理器的时间，和我们一般而言的时间并没有太大关系。一般在计算程序运行时间的时候才会使用到它。它是 `long int` 或者 `long` 类型。

#### tm 结构体 {id="tm"}

`struct tm` 结构体，这个结构体当中包含了年月日十分秒之类的信息，结构体拷贝代码如下:
```c
struct tm {
	int	tm_sec;		/* seconds after the minute [0-60] */
	int	tm_min;		/* minutes after the hour [0-59] */
	int	tm_hour;	/* hours since midnight [0-23] */
	int	tm_mday;	/* day of the month [1-31] */
	int	tm_mon;		/* months since January [0-11] */
	int	tm_year;	/* years since 1900 */
	int	tm_wday;	/* days since Sunday [0-6] */
	int	tm_yday;	/* days since January 1 [0-365] */
	int	tm_isdst;	/* Daylight Savings Time flag */
	long	tm_gmtoff;	/* offset from UTC in seconds */
	char	*tm_zone;	/* timezone abbreviation */
}
```
> 注意： 如你所见，小时的取值范围是 0 到 23，而月份的取值范围是 0 到 11。这个在编程中需要注意，不要弄错。其实也是很好理解的，在计算机中计数一般都是从 0 开始的。


#### timespec 结构体 {id="timespec"}

上面的这些类型以及结构体只能够精确到秒级别，如果需要精确到毫秒(10^3s)或者微秒(10^6s), 就可以使用 `timespec` 结构体。结构体代码如下:
```c
_STRUCT_TIMESPEC
{
	__darwin_time_t tv_sec;
	long            tv_nsec;
};
```
> 注意：这个结构体需要对应的平台支持 C11 标准，或者是 MSVC。


#### timeb 结构体 {id="timeb"}

有些平台并不支持 `timespec` 结构体，但是大部分的操作系统都支持 `timeb` 结构体:
```c
struct timeb {
    time_t time;
    unsigned short millitm;		// 毫秒
    short timezone;
    short dstflag;
};
```
在 Unix 系统上(例如 Mac), 并没有这个 `timeb` 的结构体，但是存在一个名为 `timeval` 的结构体，如下:
```c
_STRUCT_TIMEVAL
{
	__darwin_time_t         tv_sec;         /* seconds */
	__darwin_suseconds_t    tv_usec;        /* and microseconds */
};
```

#### 如何选择这些类型以及结构体 {id="how-to-select"}

上面我们分别描述了这些类型以及结构体之间的区别，那么我们在具体的编程过程中又如何选择呢？

**根据是否支持毫秒，我们优先使用 `timespec` ，这也是 C++ 标准支持的。如果没有这个结构体，我们再考虑使用 `timeval` 或者 `timeb` 。如果这些都没有，那么只能够获取 `time_t` 秒了。**

### 获取系统时间 {id="get-system-time"}

首先我们来看如何获取系统的时间戳，精确到秒。利用标准库中提供的 `time` 函数，非常容易做到:
```c
time_t t;
time(&t);		// 也可以写成 t = time(NULL); 使用地址传递更加高效
printf("%ld\n", t);     // Output: 1610016140
```
但是，目前这三两行代码在实际的应用中还是远远不够的。一来它获取的时间只能够精确到秒，二来它并没有考虑到不同平台之间的实现差异。所以，下面我们来写一个函数获取系统的当前时间，不但精确到毫秒而且也能够屏蔽不同的系统之间的差异。

首先，不同的平台引入的头文件是不一样。我们使用条件编译来屏蔽这些差异:
```c
#if defined(_WIN32)		// 如果是 Windows 系统
#include <sys/timeb.h>
#elif defined(__UNIX__) || defined(__APPLE__)		// 如果是 unix 或者 mac 系统
#include <sys/time.h>
#endif
```
然后得到了毫秒时间戳应该是 `long long` 类型的，为这个类型定义一个可读性更好的别名:
```c
typedef long long millisecond_time_t;
```
接着，我们写一个函数来屏蔽不同平台之间的 API 差异:
```c
millisecond_time_t TimeInMillisecond(void) {
#if defined(_WIN32)			// Windows 操作系统
    struct timeb time_buffer;
    ftime(&time_buffer);
    return time_buffer * 1000L + time_buffer.millitm;
#elif defined(__unix__)				// Unix 操作系统
    struct timeval time_value;
    gettimeofday(&time_value, NULL);
    return time_value.tv_sec * 1000LL + time_value.tv_usec / 1000;
#elif defined(__STDC__) && __STDC_VERSION__ == 201112L		// C11 标准
    struct timespec timespec_value;
    timespec_get(&timespec_value, TIME_UTC);
    return timespec_value.tv_sec * 1000LL + timespec_value.tv_nsec / 1000000;
#else			// 如果以上都不支持，那么就降级到精确到秒的时间戳
    time_t current_time = time(NULL);
    return current_time * 1000LL;
#endif
}
```

### 获取日历时间 {id="calender"}

上面我们演示了获取系统时间，也称之为绝对时间，既不随着时区的变化而变化的。接下来，我们来说说如何获取日历时间，也称之为本地时间，既然是本地时间一定会和时区有关系的。示例如下:
```c
time_t t = time(NULL);
struct tm *calendar_time = localtime(&t);
printf("years since 1900 is %d\n", 1900 + calendar_time->tm_year);	// Output:...2021
printf("hours since midnight is %d\n", calendar_time->tm_hour);		// Output:...21
```
那么如何将这个日历时间转换成时间戳呢？如下示例:
```c
time_t t2 = mktime(calendar_time);	// calender_time 变量来源于上面一个示例
printf("timestamp is %ld\n", t2);	// Output: 1610024883
```
> 注意: mkdir 还会格式化时间的作用，比如更改一下时间 calendar->time = 74; 使用 mktime 函数可以进位。


除了 `mktime` 函数之外，还有一个函数为 `gmtime` 函数。这个函数的作用是返回格林尼治时间，它转换出来的时间会按照格林尼治时间(也就是 0 时区)来转换，所以和我们所处的东八区时间会相差 8 个小时。

### 时间的格式化 {id="format"}

在上面的章节中，我们已经介绍了如何获取绝对时间和本地时间，就如同下面的示例一般:
```c
time_t t = time(NULL);
struct tm *calender_time = localtime(&t);
```
那么我们如何将时间格式化成我们熟悉的模样呢？比如说我输出为 `2020-12-31`：

<code-block lang="c">
char current_time_s[20];
size_t size = strftime(current_time_s, 20, "百分号Y-百分号m-%d %H:%M:%S", calender_time);// size:19 
printf("%s\n", current_time_s);     // Output: 2021-01-07 21:30:33
printf("%s", asctime(calender_time)); // Output: Thu Jan  7 21:24:36 2021
printf("%s", ctime(&t)); // Output: Thu Jan  7 21:24:36 2021
</code-block>

`strftime`  函数接受四个传参数，第一个是在格式化后写入的字符串，第二个是字符串的长度，第三个是日期格式化字符串，第四个是日历时间。 `asctime` 和 `ctime` 这两个函数如果是要做国际化那是要用的。

这里还有一个问题，使用类似于 `百分号Y-百分号m-百分号d %H:%M:%S` 这样的格式化字符串没有办法格式化毫秒，因为 C 语言在早期对毫秒支持并不好。但是也没事，有其他方法, 可以利用我们之前写的获取绝对时间的函数来实现:
```c
// Output:2021-01-07 21:47:14.521
sprintf(current_time_s + size, ".%3lld", TimeInMillisecond() % 1000);
```
### 解析时间 {id="datetime-parse"}

接下来，我们来说说如何解析时间，既根据给定的时间字符串解析出时间的每一个部分：年月日时分秒以及毫秒。听上去是不是有些耳熟？之前我们不也根据时间戳，使用 `localtime` 函数解析得到了 `tm` 结构体嘛？区别在于现在不是根据时间戳，而是根据时间字符串来解析时间组成。

在 Unix 环境下，提供了一个名为 `strptime` 的函数，可以来做这件事情:
```c
struct tm parsed_time;
char *unparsed_string = strptime("2021-01-08 13:21:20.131", "%F %T", &parsed_time);
printf("%d\n", parsed_time.tm_year);        // Output: 121
printf("%d\n", parsed_time.tm_hour);        // Output: 13
printf("%s\n", unparsed_string);            // Output: .131
```
调用 `strptime` 函数之后，返回了字符串中未被解析的子串。在这个示例中，指的就是 `.131` ，它是微秒。因为上文中我们已经提到，类似于“%F %T”这样的格式化字符串是不支持毫秒甚至是微妙的，所以无法被解析。那么就只能自己想办法解析这部分内容了:
```c
int millisecond;
char *end;
// 之所以 + 1 是为了跳过 131 前面的.
millisecond = strtol(unparsed_string + 1, &end, 10);
printf("%ld\n", millisecond);		// Output: 131
```
剩下最后一个问题，上面的 `strptime` 函数只有在 `Unix` 环境下才能使用，如果要程序要在 Windows 环境下运行呢？我们可以使用 `sscanf` 函数来解析:
```c
struct tm parsed_time;
int millisecond;
sscanf("2021-01-08 13:21:20.131", "%4d-%2d-%2d %2d:%2d:%2d.%3d",
	&parsed_time.tm_year, &parsed_time.tm_mon, &parsed_time.tm_mday,
	&parsed_time.tm_hour, &parsed_time.tm_min, &parsed_time.tm_sec, &millisecond);
parsed_time.tm_year -= 1990;	// 年是从 1990 年开始计数的
parsed_time.tm_mon -= 1;		// 月是从 0 开始计数的
mktime(&parsed_time);			// 传入的日期不一定合法，所以使用 mktime 格式化
```

### 计算时间差 {id="difference-between-time-and-time"}

特别是在一些性能检测的程序中，我们需要计算程序运行所花费的时间，这就需要计算时间的差(截止时间减去开始时间)。下面我们就来写一个程序演示一下:
```c
void DoHardWork() {
    double sum = 0;
    for (int i = 0; i < 10000000; ++i) {
        sum += i * i / 3.1415926;
    }
}
```
这个函数来做一些耗时的操作，具体什么内容也不用过多理会。下面这段程序利用了前文中我们写的 `TimeInMillisecond` 函数来获取当前的毫秒数，前后两个毫秒数的差使用减法即可:
```c
long long cost_time;
long long start_time = TimeInMillisecond();
DoHardWork();
long long end_time = TimeInMillisecond();
cost_time = end_time - start_time;
printf("%lld", cost_time);      // Output: 70, 你的和我的可能不一样
```
但是这样计算出的时间差只是程序运行过程中，经过了多少系统时间，并不准确。更准确的是获取 CPU 运行这个程序所消耗的时间，这里就用到了 `clock_t` 这个结构体:
```c
clock_t start_time = clock();   // clock 函数返回的一定是一个 int 类型的数值
DoHardWork();
clock_t end_time = clock();
// Output: 0.063807, 你的可能和我的不一样
// CLOCKS_PER_SEC 是一个宏，一般是整数 1000，表示每秒 1000 个时钟周期
printf("%f\n", (float)(end_time - start_time) * 1.0 / CLOCKS_PER_SEC);
```
计算出来的结果是 `0.063807` ，大概是 6 毫秒。因为在一台计算机中，同一时刻有好多的程序运行，所以这个 6 毫秒是 CPU 来运行这个程序所需要的时间，相对会更加的真实一些。

### 参考资料 {id="also-articles"}

1. 慕课网-《C语言系统化精讲》
