# Java 8：新的时间和日期API
在Java 8之前，所有关于时间和日期的API都存在各种使用方面的缺陷，因此建议使用新的时间和日期API，分别从旧的时间和日期的API的缺点以及解决方法、Java 8 新的时间和日期API进行讲解。

## 旧的时间和日期的API的缺陷
Java 的 java.util.Date 和 java.util.Calendar 类易用性差，不支持时区，而且都不是线程安全的。

Date如果不格式化，打印出的日期可读性差。
```
Thu Sep 12 13:47:34 CST 2019
```

可以使用 SimpleDateFormat 对时间进行格式化，但 SimpleDateFormat 是线程不安全的，SimpleDateFormat 的 format 方法源码如下：
```
    private StringBuffer format(Date date, StringBuffer toAppendTo,
                                FieldDelegate delegate) {
        // Convert input date to time field list
        calendar.setTime(date);

        boolean useDateFormatSymbols = useDateFormatSymbols();

        for (int i = 0; i < compiledPattern.length; ) {
            int tag = compiledPattern[i] >>> 8;
            int count = compiledPattern[i++] & 0xff;
            if (count == 255) {
                count = compiledPattern[i++] << 16;
                count |= compiledPattern[i++];
            }

            switch (tag) {
            case TAG_QUOTE_ASCII_CHAR:
                toAppendTo.append((char)count);
                break;

            case TAG_QUOTE_CHARS:
                toAppendTo.append(compiledPattern, i, count);
                i += count;
                break;

            default:
                subFormat(tag, count, delegate, toAppendTo, useDateFormatSymbols);
                break;
            }
        }
        return toAppendTo;
    }
```
其中 calendar 是共享变量，并且这个共享变量没有做线程安全控制。当多个线程同时使用相同的 SimpleDateFormat 对象【如用static修饰的 SimpleDateFormat 】调用format方法时，多个线程会同时调用 calendar.setTime 方法，可能一个线程刚设置好 time 值另外的一个线程马上把设置的 time 值给修改了导致返回的格式化时间可能是错误的。

在多并发情况下使用 SimpleDateFormat 需注意。

SimpleDateFormat 除了 format 是线程不安全以外，parse 方法也是线程不安全的。parse 方法实际调用 alb.establish(calendar).getTime() 方法来解析，alb.establish(calendar) 方法里主要完成了
- 重置日期对象cal的属性值
- 使用calb中中属性设置cal
- 返回设置好的cal对象

但是这三步不是原子操作，导致解析出来的时间可以是错误的。

Date对时间处理比较麻烦，比如想获取某年、某月、某星期，以及 n 天以后的时间，如果用Date来处理的话真是太难了，并且 Date 类的 getYear、getMonth 这些方法都被弃用了。

## 多线程并发如何保证线程安全
避免线程之间共享一个 SimpleDateFormat 对象，每个线程使用时都创建一次 SimpleDateFormat 对象 => 创建和销毁对象的开销大

对使用 format 和 parse 方法的地方进行加锁 => 线程阻塞性能差

使用 ThreadLocal 保证每个线程最多只创建一次 SimpleDateFormat 对象 => 较好的方法

## Java 8 新的时间和日期API
Java 8的日期和时间类包含 LocalDate、LocalTime、Instant、Duration 以及 Period，这些类都包含在 java.time 包中，Java 8 新的时间API的使用方式，包括创建、格式化、解析、计算、修改，下面我们看下如何去使用。

LocalDate   只会获取年月日
```
// 创建 LocalDate
// 获取当前年月日
LocalDate localDate = LocalDate.now();
// 构造指定的年月日
LocalDate localDate1 = LocalDate.of(2019, 9, 12);
// 获取年、月、日、星期几
int year = localDate.getYear();
int year1 = localDate.get(ChronoField.YEAR);
Month month = localDate.getMonth();
int month1 = localDate.get(ChronoField.MONTH_OF_YEAR);
int day = localDate.getDayOfMonth();
int day1 = localDate.get(ChronoField.DAY_OF_MONTH);
DayOfWeek dayOfWeek = localDate.getDayOfWeek();
int dayOfWeek1 = localDate.get(ChronoField.DAY_OF_WEEK);
```
LocalTime   只会获取时分秒
```
// 创建 LocalTime
LocalTime localTime = LocalTime.of(14, 14, 14);
LocalTime localTime1 = LocalTime.now();
// 获取小时
int hour = localTime.getHour();
int hour1 = localTime.get(ChronoField.HOUR_OF_DAY);
// 获取分
int minute = localTime.getMinute();
int minute1 = localTime.get(ChronoField.MINUTE_OF_HOUR);
// 获取秒
int second = localTime.getMinute();
int second1 = localTime.get(ChronoField.SECOND_OF_MINUTE);
```
LocalDateTime   获取年月日时分秒，相当于 LocalDate + LocalTime

```
// 创建 LocalDateTime
LocalDateTime localDateTime = LocalDateTime.now();
LocalDateTime localDateTime1 = LocalDateTime.of(2019, Month.SEPTEMBER, 10, 14, 46, 56);
LocalDateTime localDateTime2 = LocalDateTime.of(localDate, localTime);
LocalDateTime localDateTime3 = localDate.atTime(localTime);
LocalDateTime localDateTime4 = localTime.atDate(localDate);
// 获取LocalDate
LocalDate localDate2 = localDateTime.toLocalDate();
// 获取LocalTime
LocalTime localTime2 = localDateTime.toLocalTime();
```
Instant 获取秒数，用于表示一个时间戳（精确到纳秒）

如果只是为了获取秒数或者毫秒数，可以使用 System.currentTimeMillis()。
```
// 创建Instant对象
Instant instant = Instant.now();
// 获取秒数
long currentSecond = instant.getEpochSecond();
// 获取毫秒数
long currentMilli = instant.toEpochMilli();
```
Duration    表示一个时间段
```
// Duration.between()方法创建 Duration 对象
LocalDateTime from = LocalDateTime.of(2017, Month.JANUARY, 1, 00, 0, 0);    // 2017-01-01 00:00:00
LocalDateTime to = LocalDateTime.of(2019, Month.SEPTEMBER, 12, 14, 28, 0);     // 2019-09-15 14:28:00
Duration duration = Duration.between(from, to);     // 表示从 from 到 to 这段时间
long days = duration.toDays();              // 这段时间的总天数
long hours = duration.toHours();            // 这段时间的小时数
long minutes = duration.toMinutes();        // 这段时间的分钟数
long seconds = duration.getSeconds();       // 这段时间的秒数
long milliSeconds = duration.toMillis();    // 这段时间的毫秒数
long nanoSeconds = duration.toNanos();      // 这段时间的纳秒数
```
修改 LocalDate、LocalTime、LocalDateTime、Instant。

LocalDate、LocalTime、LocalDateTime、Instant 为不可变对象，修改这些对象对象会返回一个副本。

增加、减少年数、月数、天数等，以LocalDateTime为例:
```
LocalDateTime localDateTime = LocalDateTime.of(2019, Month.SEPTEMBER, 12, 14, 32, 0);
// 增加一年
localDateTime = localDateTime.plusYears(1);
localDateTime = localDateTime.plus(1, ChronoUnit.YEARS);
// 减少一个月
localDateTime = localDateTime.minusMonths(1);
localDateTime = localDateTime.minus(1, ChronoUnit.MONTHS);  
// 通过with修改某些值
// 修改年为2020
localDateTime = localDateTime.withYear(2020);
localDateTime = localDateTime.with(ChronoField.YEAR, 2020);
// 时间计算
// 获取该年的第一天
LocalDate localDate = LocalDate.now();
LocalDate localDate1 = localDate.with(firstDayOfYear());
```
TemporalAdjusters   包含许多静态方法，可以直接调用，以下列举一些：

方法名| 描述
---|---
dayOfWeekInMonth |	返回同一个月中每周的第几天
firstDayOfMonth |	返回当月的第一天
firstDayOfNextMonth |	返回下月的第一天
firstDayOfNextYear |	返回下一年的第一天
firstDayOfYear |	返回本年的第一天
firstInMonth |	返回同一个月中第一个星期几
lastDayOfMonth |	返回当月的最后一天
lastDayOfNextMonth |	返回下月的最后一天
lastDayOfNextYear |	返回下一年的最后一天
lastDayOfYear |	返回本年的最后一天
lastInMonth	 |返回同一个月中最后一个星期几
next / previous	 |返回后一个/前一个给定的星期几
nextOrSame / previousOrSame |	返回后一个/前一个给定的星期几，如果这个值满足条件，直接返回

格式化时间
```
LocalDate localDate = LocalDate.of(2019, 9, 12);
String s1 = localDate.format(DateTimeFormatter.BASIC_ISO_DATE);
String s2 = localDate.format(DateTimeFormatter.ISO_LOCAL_DATE);
// 自定义格式化
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String s3 = localDate.format(dateTimeFormatter);
```
解析时间
```
LocalDate localDate1 = LocalDate.parse("20190912", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate localDate2 = LocalDate.parse("2019-09-12", DateTimeFormatter.ISO_LOCAL_DATE);
```
## 总结
和 SimpleDateFormat 相比，DateTimeFormatter 是线程安全的。

Instant 的精确度更高，可以精确到纳秒级。

Duration 可以便捷得到时间段内的天数、小时数等。

LocalDateTime 能够快速地获取年、月、日、下一月等。

TemporalAdjusters 类中包含许多常用的静态方法，避免自己编写工具类。