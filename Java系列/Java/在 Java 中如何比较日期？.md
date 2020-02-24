在 Java 中有多种方法可以比较日期，日期在计算机内部表示为（long型）时间点——自1970年1月1日以来经过的毫秒数。在Java中，Date是一个对象，包含多个用于比较的方法，任何比较两个日期的方法本质上都会比较日期的时间。

本文主要介绍以下五种方式：
1. 使用 Date.compareTo()
2. 使用 Date.before()、Date.after() 和 Date.equals()
3. 使用 Calender.before()、Calender.after() 和 Calender.equals()
4. 使用 getTime()
4. 使用 Java 8 的 isBefore()、isAfter()、isEqual() 和 compareTo() 

# Date.compareTo()
Date 实现了 Comparable<Date>，因此两个日期可以直接用 compareTo 方法进行比较。
- 如果两个日期相等，则返回值为0。
- 如果 Date1 在 Date2 参数之后，则返回值大于0。
- 如果 Date1 在 Date2 参数之前，则返回值小于0。

```
package com.wupx.date;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateDemo {

    public static void main(String[] args) throws ParseException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date date1 = sdf.parse("2019-10-01");
        Date date2 = sdf.parse("2019-10-17");

        System.out.println("date1 : " + sdf.format(date1));
        System.out.println("date2 : " + sdf.format(date2));

        if (date1.compareTo(date2) > 0) {
            System.out.println("Date1 is after Date2");
        } else if (date1.compareTo(date2) < 0) {
            System.out.println("Date1 is before Date2");
        } else if (date1.compareTo(date2) == 0) {
            System.out.println("Date1 is equal to Date2");
        } else {
            System.out.println("咋到这的？");
        }

    }

}
```

输出结果
```
date1 : 2019-10-01
date2 : 2019-10-17
Date1 is before Date2
```
# Date.before() Date.after()  Date.equals()
可以用 equals、after 和 before 方法比较日期。
- 如果两个日期在同一时间点，equals方法将返回true。
- 如果 date1 在 date2 之前，before 返回 true，否则返回 false。
- 如果 date2 在 date1 之后，after 返回 true，否则返回 false。

```
package com.wupx.date;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateDemo2 {

    public static void main(String[] args) throws ParseException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date date1 = sdf.parse("2019-10-01");
        Date date2 = sdf.parse("2019-10-17");

        System.out.println("date1 : " + sdf.format(date1));
        System.out.println("date2 : " + sdf.format(date2));
        
        if (date1.after(date2)) {
            System.out.println("Date1 is after Date2");
        }

        if (date1.before(date2)) {
            System.out.println("Date1 is before Date2");
        }

        if (date1.equals(date2)) {
            System.out.println("Date1 is equal Date2");
        }
    }
}

```

输出结果
```
date1 : 2019-10-01
date2 : 2019-10-17
Date1 is before Date2
```

# Calender.before() Calender.after() Calender.equals()
Calendar 类也有 compareTo、equals、after 和 before 方法，工作方式与上面描述的 Date 类的方法相同。因此，如果日期信息保存在 Calendar 类中，则不需要提取日期来执行比较。

```
package com.wupx.date;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

public class DateDemo3 {

    public static void main(String[] args) throws ParseException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        Date date1 = sdf.parse("2009-12-31");
        Date date2 = sdf.parse("2010-01-31");

        System.out.println("date1 : " + sdf.format(date1));
        System.out.println("date2 : " + sdf.format(date2));

        Calendar cal1 = Calendar.getInstance();
        Calendar cal2 = Calendar.getInstance();
        cal1.setTime(date1);
        cal2.setTime(date2);

        if (cal1.after(cal2)) {
            System.out.println("Date1 is after Date2");
        }

        if (cal1.before(cal2)) {
            System.out.println("Date1 is before Date2");
        }

        if (cal1.equals(cal2)) {
            System.out.println("Date1 is equal Date2");
        }
    }
}
```

输出结果
```
date1 : 2019-10-01
date2 : 2019-10-17
Date1 is before Date2
```

# getTime()
可以直接比较两个日期的时间点。这是对两种原始数据类型的比较，因此可以使用 < 、 > 和 == 来比较。

在比较日期之前，必须使用前面创建的 Date 对象中的数据来创建长整型。

```
package com.wupx.date;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class DateDemo4 {

    public static void main(String[] args) throws ParseException {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

        Date date1 = sdf.parse("2019-10-01");
        Date date2 = sdf.parse("2019-10-17");
        
        System.out.println("date1 : " + sdf.format(date1));
        System.out.println("date2 : " + sdf.format(date2));

        long time1 = date1.getTime();
        long time2 = date2.getTime();

        if (time1 > time2) {
            System.out.println("Date1 is after Date2");
        } else if (time1 < time2) {
            System.out.println("Date1 is before Date2");
        } else if (time1 == time2) {
            System.out.println("Date1 is equal to Date2");
        } else {
            System.out.println("咋到这的？");
        }
    }
}
```

输出结果
```
date1 : 2019-10-01
date2 : 2019-10-17
Date1 is before Date2
```

# Java 8 中的 isBefore() isAfter() isEqual() compareTo()

在 Java 8 中，可以使用新的 isBefore()、isAfter()、isEqual() 以及 compareTo() 来比较 LocalDate、LocalTime 和 LocalDateTime。

```
package com.wupx.date;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class DateDemo5 {

    public static void main(String[] args) {

        DateTimeFormatter sdf = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate date1 = LocalDate.of(2019, 10, 01);
        LocalDate date2 = LocalDate.of(2019, 10, 17);

        System.out.println("date1 : " + sdf.format(date1));
        System.out.println("date2 : " + sdf.format(date2));

        System.out.println("Is...");
        if (date1.isAfter(date2)) {
            System.out.println("Date1 is after Date2");
        }

        if (date1.isBefore(date2)) {
            System.out.println("Date1 is before Date2");
        }

        if (date1.isEqual(date2)) {
            System.out.println("Date1 is equal Date2");
        }

        System.out.println("CompareTo...");
        if (date1.compareTo(date2) > 0) {
            System.out.println("Date1 is after Date2");
        } else if (date1.compareTo(date2) < 0) {
            System.out.println("Date1 is before Date2");
        } else if (date1.compareTo(date2) == 0) {
            System.out.println("Date1 is equal to Date2");
        } else {
            System.out.println("咋到这的？");
        }
    }
}
```

输出结果
```
date1 : 2019-10-01
date2 : 2019-10-17
Is...
Date1 is before Date2
CompareTo...
Date1 is before Date2
```

# 总结
本文主要讲解了在 Java 中比较日期的几种常用方法，可以自己实际操作一下。

> 参考
> 
> https://blog.csdn.net/qq_27276045/article/details/100792621