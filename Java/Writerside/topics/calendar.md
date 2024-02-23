# 日期时间

这篇文档描述了在 Java 中如何对日期和时间进行处理。介绍了其 API 的历史变化，以及给出了新的 API 的示例。希望这篇文档可以对开发中时间的处理有所
帮助。

## 历史的包袱 {id="history"}

在 Java 的早期，提供了 `java.util.Date` 和 `java.util.Calendar` 类来处理日期和时间。其中 `java.util.Date` 类由于方法和设计不一致，
导致了后期废弃了一些 API。下图所示的这些方法都被标记为 `@Deprecated`。

```plantuml
@startmindmap
+ java.util.Date 废弃API
++ getYear() / setYear(int year)
++ getMonth() / setMonth(int month)
++ getDay()
++ getDate() / setDate(int date)
++ getHours() / setHours(int hours)
-- toGMTString()
-- toLocaleString()
-- getMinutes() / setMinutes(int minutes)
-- getSeconds() / setSeconds(int seconds)
@endmindmap
```

> 上图列举的并不完全，只是做一个展示，在实际生产中，并不建议使用`java.util.Date` 类，而是推荐使用 Java8 之后引入的 `java.time` API。

而 `java.util.Calendar` 也被认为是复杂和笨拙的。比如说，月份的计数是从 `0` 而非 `1` 开始的，这就和我们平时的习惯不一致，且会导致难以察觉的
 Bug。

比如说，要配置一个指定的日期，使用`java.util.Calendar` 需要编写更多的代码，下面是是 `java.time.LocalDate` 的示例对比:

<compare first-title="java.util.Calendar" second-title="java.time.LocalDate">
<code-block lang="java">
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.YEAR, 2023);
// Calendar.OCTOBER 表示十月，实际的值却是 9
calendar.set(Calendar.MONTH, Calendar.OCTOBER);
calendar.set(Calendar.DAY_OF_MONTH, 27);
// 输出 Fri Oct 27 21:10:36 CST 2023
System.out.println(calendar.getTime());
</code-block>
<code-block lang="java">
// 输出 2023-10-27
LocalDate date = LocalDate.of(2023, 10, 27);
System.out.println(date);
</code-block>
</compare>

还有, `java.util.Calendar` 是可变的，这个在多线程的程序中可能会引发问题。

## Java8 后新的 API {id="new_api"}

在 Java 8 之前，因为上面提到的历史包袱。所以实际项目中使用 `Joda-Time` 库来代替原生的 API。在Java 8 引入了全新的日期和时间 API，这也是
收到了 `Joda-Time` 的启发，并且是由这个库的主要作者 Stephen Colebourne 主导开发的。

新的 API 位于 `java.time` 包下，提供了 `LocalDate` 、`LocalDateTime`、`ZonedDateTime`、`Duration`、`Period` 等类, 如下表所示:

| 类名               | 作用                                                              |
|------------------|-----------------------------------------------------------------|
| `LocalDate`      | 表示不带时区的日期（例如：2023-10-27）。常用于生日、纪念日等。                            |
| `LocalTime`      | 表示不带时区的时间（例如：14:30:45）。常用于表示一天内的某个时间点。                          |
| `LocalDateTime`  | 表示不带时区的日期和时间（例如：2023-10-27T14:30:45）。                           |
| `ZonedDateTime`  | 表示带有时区的日期和时间。它是 `LocalDateTime` 的时区版本。                          |
| `Instant`        | 表示从1970年1月1日（Unix起始日期）到现在的时间线上的某个具体瞬间。常用于时间戳或与 `Date` 之间的转换。    |
| `Duration`       | 表示两个时间点之间的时间段，如 "2.5 小时"。常用于时间的算术运算。                            |
| `Period`         | 表示两个日期之间的时间段，如 "5 年 6 个月"。常用于日期的算术运算。                           |
| `ZoneId`         | 表示时区ID，如 "Asia/Shanghai"。                                       |
| `ZoneOffset`     | 表示与UTC/Greenwich的时间偏移，如 `+05:30`。                               |
| `Year`           | 表示不带时区的年份。                                                      |
| `Month`          | 枚举，表示月份，如 `JANUARY`、`FEBRUARY` 等。                               |
| `MonthDay`       | 表示年份中的某天，例如2月14日，常用于不涉及年份的重复事件，如情人节。                            |
| `YearMonth`      | 表示某年某月，例如 2023-10，常用于信用卡到期日等。                                   |
| `OffsetDateTime` | 表示日期和时间，与 `ZonedDateTime` 类似，但仅存储与UTC/Greenwich的偏移量，而不是完整的时区信息。 |
| `OffsetTime`     | 表示时间，带有与UTC/Greenwich的偏移量。                                      |
| `DayOfWeek`      | 枚举，表示星期几，如 `MONDAY`、`TUESDAY` 等。                                |

这些类为日期和时间提供了全面、一致且不可变的表示，使得日期和时间操作在 Java 中变得更加简单和直观。其优点如下:

```plantuml
@startmindmap
* java.time 的优点
    * 它是不可变的，有助于避免多线程中的问题
    * 设计和实现是清晰和一致的
    * 提供了对时区和夏令时的全面支持
@endmindmap
```

## 新 API 的示例 {id="examples"}

下面对开发中经常出现的一些时间和日期的操作，给出一些示例，以备查询。

### 获取当前日期和时间 {id="get_datetime_of_now"}

下面的示例演示了获取当前的日期和时间:

```Java
// 2023-10-27
System.out.println("LocalDate.now() = " + LocalDate.now());
// 21:38:33.507939
System.out.println("LocalTime.now() = " + LocalTime.now());
// 2023-10-27T21:38:33.507999
System.out.println("LocalDateTime.now() = " + LocalDateTime.now());
// 2023-10-27T21:38:33.508717+08:00[Asia/Shanghai]
System.out.println("ZonedDateTime.now() = " + ZonedDateTime.now());
```
`LocalDateTime` 和 `ZonedDateTime` 的区别是，前者不带时区，后者是带时区的。

### 格式化和解析日期和时间 {id="format"}

下面的示例演示了如何格式化日期，这个在输出用户界面的时候特别有用:

```Java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String datetime = LocalDateTime.now().format(formatter);
System.out.println("datetime = " + datetime);  // 2023-10-27 21:46:00
```

然后将格式化的日期转换为 `LocalDateTime` 对象，如下:

```Java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime datetime = LocalDateTime.parse("2023-10-27 21:46:10", formatter);
System.out.println("datetime = " + datetime);
```

### 日期和时间的加减 {id="calc"}

下面的示例演示了日期和时间的加减:

```Java
LocalDateTime plusDays = LocalDateTime.now().plusDays(1);
LocalDateTime minusDays = LocalDateTime.now().minusDays(1);
LocalDateTime minusWeeks = LocalDateTime.now().minusWeeks(1);
```

### 日期和时间的对比 {id="diff_datetime"}

下面的示例演示了日期和时间的对比:

```Java
boolean isBefore = LocalDateTime.now().isBefore(LocalDateTime.MAX);
boolean isAfter = LocalDateTime.now().isAfter(LocalDateTime.MIN);
```

### 获取日期和时间的特定部分 {id="get_date_time_part"}

下面的示例演示了获取时间和日期的特定部分:

```Java
int year = LocalDateTime.now().getYear();
int month = LocalDateTime.now().getMonthValue();
int day = LocalDateTime.now().getDayOfMonth();
```

### 时区的偏移 {id="timezone"}

下面的示例演示了时区的偏移:

```Java
ZonedDateTime zonedDateTime = ZonedDateTime.now().withZoneSameInstant(ZoneId.of("Europe/Paris"));
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String datetime = zonedDateTime.format(formatter);
System.out.println("datetime = " + datetime); // 2023-10-27 16:16:43
```

### 时期和时间的间隔 {id="interval"}

下面的示例演示了时间的间隔:

```Java
Duration between = Duration.between(LocalDateTime.of(2023, 10, 27, 22, 19), LocalDateTime.now());
long seconds = between.getSeconds();   // 65
```

### 日期和时间戳的相互转换 {id="ts2str"}

从时间戳转换到日期:

```Java
long ts = System.currentTimeMillis();
Instant instant = Instant.ofEpochMilli(ts);
LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
```

从日期转换到时间戳:

```Java
long epochSecond = localDateTime.toInstant(ZoneOffset.ofHours(8)).getEpochSecond();
long epochMilli = localDateTime.toInstant(ZoneOffset.ofHours(8)).toEpochMilli();
```

## 关于时间和时区的答疑 {id="fqa"}

时间和时区有时候挺让人感到迷惑的，我整理了一些问题，并做出解答。

### 时间戳和时区有关系吗 {id="timestamp_with_timezone"}

很多人的回答是时间戳和时区没有关系，其实这样的回答不完全正确。时间戳的设计确实是为了独立于时区的，但并不表示其没有时区或者和时区无关。因为时间
戳是以零时区为基准的，或者说时间戳是 1970 年 1 月 1 日 0 时 0 分 0 秒到现在的秒数，这个时间是零时区。

### GMT 和 UTC 有什么区别 {id="diff_gmt_and_utc"}

GMT (格林尼治平均时间) 和 UTC (协调世界时) 都是时间标准，用于参考和协调全球的时间的。他们的区别仅仅是定义和历史背景不同，在时间上几乎是一样的。

GMT 基于地球的旋转并与太阳时间相关联。具体来说，当太阳位于格林尼治子午线上的最高点时，格林尼治的太阳时间正好是正午。

UTC 是基于原子时间的，与由大约 300 多个高度精确的原子钟组成的国际原子时 (TAI) 相关联。为了使UTC与地球的旋转保持一致，有时会插入或删除闰秒。

GMT 历史更加久远，因为地球旋转的速度并不恒定，所以精度并不高。现在更倾向于使用 UTC 标准，它基于原子时间，提供了非常高的精准性。

### 什么是夏令时 {id="what_is_summer_time"}

中国已经不再使用夏令时，早在 1986 年的时候，会在每年的 4 月中旬将时钟向前调整一个小时，然后在 9 月中旬再将时钟调回。因为夏季的阳光更长，
将时间向前调整一个小时，会让白天的时间更长，人们的活动更久，可以减少煤炭、电力等资源的使用。

但是，实际上并没有起到很好的节能效果，也会让生活和工作造成不方便。所以，从 1992 年其，就不再实行夏令时了。

夏令时的英文有多种表达方式，比如 Summer Time 或者 British Summer Time。但是在国际上更标准的说法是 "Daylight Saving Time(DST)", 这也
说明了其是为了利用日光来节省能源的一种做法。