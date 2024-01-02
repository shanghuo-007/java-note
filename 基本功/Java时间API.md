# Java时间API

## 几种常见的时间

1. GMT
   GMT(格林威治标准时间)：也就是0时区的时间，理论上来说，格林尼治标准时间的正午是指当太阳横穿格林尼治子午线时（也就是在格林尼治上空最高点时）的时间。由于地球在它的椭圆轨道里的运动速度不均匀，这个时刻可能和实际的太阳时相差16分钟。地球每天的自转是有些不规则的，而且正在缓慢减速。所以，格林尼治时间已经不再被作为标准时间使用。现在的标准时间——协调世界时（UTC）——由原子钟提供。
2. UTC
   UTC（世界协调时间），协调世界时是以原子时秒长为基础，国际原子时的准确度为每日数纳秒，而世界时的准确度为每日数毫秒，**现在我们使用的互联网就采用该计时标准**。UTC是GMT微调过的时间，我们可以认为两者是一致的。
3. CET
   欧洲中部时间(CET)比世界标准时间(UTC)早一个小时，部分欧洲国家和北非国家使用，这个标准是地理加政治的产物；
4. CST
   北京时间

```ini
    CET=UTC(GMT) + 1小时
    CST=UTC(GMT) + 8小时
```

## 时间戳

> 时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数(Java中获得的秒数是以毫秒为单位的)。
>
> 例如现在北京时间2015-12-31 17:00:00的时间戳是1451552400，就是指从北京时间1970-01-01 08:00:00到2015-12-31 17:00:00已经过去了1451552400秒。

使用时间戳有如下好处：

- 时间戳没有时区概念，比如如果用'2015-12-31 17:00:00'这么一个字符串表示时间的话，北京时间和美国时间是不一样的，但是用时间戳1451552400来表示的话，那就是一定是唯一的时间，不会有歧义；
- 时间戳在编程语言中一般是长整形数据类型，无论何种编程语言都能认识时间戳，如果用字符串表示时间，还需要转换。

## Java8之前

在Java8以前操作时间的常见API有：

- java.util.Date:表示Java中的日期，但是能够操作到时间级别，如今这个类中的很多方法都已经被废弃，不建议使用；
- java.sql.Date:表示数据库时间，只能操作到日期，不能读取和修改时间；
- java.sql.Time:表示数据库时间；
- java.sql.Timestamp:时间戳；
- Calendar:工具类，提供时间的加减等复杂操作，支持时区；
- TimeZone:表示时区；
- SimpleDateFormat：日期格式化类，非常常用。

Date主要负责存储一个绝对时间，并对两边提供操作接口。Calendar负责对Date中特定信息，比如这个时间是该年的第几个星期，此外，还可以通过set,add,roll接口来进行日期时间的增减。SimpleDateFormat主要作为一些格式化的输入输出。

### SimpleDateFormat使用列子

SimpleDateFormat接收一个String pattern和Local参数来构造对象。其中pattern是预定义的：

```powershell
G    年代标志符
y    年
M   月
d    日
h    时 在上午或下午 (1~12)
H    时 在一天中 (0~23)
m   分
s    秒
S    毫秒
E    星期
D    一年中的第几天
F    一月中第几个星期几
w   一年中第几个星期
W   一月中第几个星期
a    上午 / 下午 标记符 
k    时 在一天中 (1~24)
K   时 在上午或下午 (0~11)
z    时区
//默认的方言
SimpleDateFormat sf = new SimpleDateFormat("G yyyy-MM-dd a z");
SimpleDateFormat sf1 = new SimpleDateFormat("G yyyy-MM-dd a z", Locale.US);
SimpleDateFormat sf2 = new SimpleDateFormat("G yyyy-MM-dd a z", Locale.KOREA);
String format1 = sf.format(date1);
String format2 = sf1.format(date1);
String format3 = sf2.format(date1);
System.out.println("format1:"+format1);
System.out.println("format2:"+format2);
System.out.println("format3:"+format3);
//解析Date
Date date = sf.parse(format1);
```

### Calendar的使用

Calendar还有许多其他API，比如获得当前月天数，当前年天数等等，需要时可以查询使用。

```java
Calendar calendar = Calendar.getInstance();
int year = calendar.get(Calendar.YEAR);
// 获取月，这里需要需要月份的范围为0~11，因此获取月份的时候需要+1才是当前月份值
int month = calendar.get(Calendar.MONTH) + 1;
// 获取日
int day = calendar.get(Calendar.DAY_OF_MONTH);
// 获取时
int hour = calendar.get(Calendar.HOUR);
// 获取分
int minute = calendar.get(Calendar.MINUTE);
// 获取秒
int second = calendar.get(Calendar.SECOND);
System.out.println("现在是" + year + "年" + month + "月" + day + "日" + hour
		+ "时" + minute + "分" + second + "秒");
//获取一个月后的今天
calendar.add(Calendar.MONTH,1);
```

### 存在的问题

- java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。
- java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
- 对于时间、时间戳、格式化以及解析，并没有一些明确定义的类。对于格式化和解析的需求，我们有java.text.DateFormat抽象类，但通常情况下，SimpleDateFormat类被用于此类需求。
- 所有的日期类都是可变的，因此他们都**不是线程安全的**，这是Java日期类最大的问题之一。
- 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

### SimpleDateFormat解决线程安全问题

看看阿里巴巴 java 开发手册中推荐的 ThreadLocal 的用法:

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
 
public class DateUtils {
    public static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd");
        }
    };
}
```

然后我们再要用到 DateFormat 对象的地方，这样调用：

```java
DateUtils.df.get().format(new Date());
```



## Java8中新添加的时间API

- java.time包：这是新的Java日期/时间API的基础包，所有的主要基础类都是这个包的一部分，如：LocalDate, LocalTime, LocalDateTime, Instant, Period, Duration等等。所有这些类都是不可变的和线程安全的，在绝大多数情况下，这些类能够有效地处理一些公共的需求。
- java.time.chrono包：这个包为非ISO的日历系统定义了一些泛化的API，我们可以扩展AbstractChronology类来创建自己的日历系统。
- java.time.format包：这个包包含能够格式化和解析日期时间对象的类，在绝大多数情况下，我们不应该直接使用它们，因为java.time包中相应的类已经提供了格式化和解析的方法。
- java.time.temporal包：这个包包含一些时态对象，我们可以用其找出关于日期/时间对象的某个特定日期或时间，比如说，可以找到某月的第一天或最后一天。你可以非常容易地认出这些方法，因为它们都具有“withXXX”的格式。
- java.time.zone包：这个包包含支持不同时区以及相关规则的类。

### LocalDateTime

需要格式化日期时，我们直接调用相关类的format方法即可，如下：

```java
//获取当前系统时间
LocalDateTime today = LocalDateTime.now();
System.out.println(today);

//格式化为yyyyMMdd
String yyyyMMdd = today.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
System.out.println("yyyyMMdd = " + yyyyMMdd);

//格式化为yyyyMMddHHmmss
String yyyyMMddHHmmss = today.format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
System.out.println("yyyyMMddHHmmss = " + yyyyMMddHHmmss);

//获取下一天日期
LocalDateTime tomorrow = today.plusDays(1);
System.out.println("tomorrow = " + tomorrow);

//获取下一周日期
LocalDateTime nextWeek = today.plusWeeks(1);
System.out.println("nextWeek = " + nextWeek);

//字符串转LocalDateTime
LocalDateTime today1 = LocalDateTime.parse(yyyyMMddHHmmss, DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));
System.out.println("today1:"+today1);
```





