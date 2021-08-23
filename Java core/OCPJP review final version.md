# Stream

---

## 1. API 相关

### 1.1 Stream接口中包括了 min max 但是没有其他数学类的比如sum，并且intStream或者doublestream这样的是重写了min 和max 的 但是Stream需要传入 comparator

### 1.2 Optional会在可能出现空值的操作的时候返使用比如 min max  reduce 和那些判断类比如（findAny） 但是sum不会。 average永远是Optional double

### 1.3 findAny 会提前终止，注意打印

```java
OptionalInt first = stream.peek(System.out::print).findAny();
//  22
```



### 1.4 primitive version 调用 match 类数据需要的是 DoublePredicte 不是 predicte<double>

### 1.5 flatMap 的参数的返回值**必须**是stream

返回的是新的stream 默认是单线程的 不是并行流

```java
     	 Stream<String> stream = Stream.of("SE","EE","ME");
       Stream<Integer> list = stream.flatMap(s -> Stream.of(s.length()));// 这种就不行
       System.out.println(list.collect(Collectors.toSet()));



				Stream<Double> stream = Stream.of(12.1,12.5,12.9);
        Stream list = stream.flatMap(d -> Stream.of(d.intValue())); //这种就可以
        System.out.println(list.collect(Collectors.toSet()));
```



### 1.6 如果调用了 sort 再调用foreach 那么会在最后统一打印(没啥用)

```java
stream.peek(System.out::print).sorted().distinct().forEach(System.out::print); 
//581351358
```

### 1.7 针对不同的primitive version stream 对于的 4大内置函数式接口也必须要对于的primitive version

### 1.8 boxed()

该方法会转变stream类型 比如说 

```java
IntStream.boxed() // 会变成 Stream<Integer>
```

### 1.9 **Integer::toString**

一般来说拉姆达表达式会自动匹配 但这个method 的两个重载 输入都是一样的 但是输出不同， 所以不能匹配 

### 1.10 reduce

* 第一个参数必须对上流类型 如果有第一个参数的话 比如是Double  就必须是double
* 如果没有第一个参数 **那么返回值是 optional** 否则 直接**返回identity的类型**
* 为空 返回空的 optional

```java
 class Test1 {
    public static void main(String[] args) {
        Stream<Integer> stream = Arrays.asList(1, 2, 3).stream();
        System.out.println(stream.reduce(null, (s1, s2) -> s1 + s2)); // 这里会空指针 因为 null 无法 和 数字进行加
    }
}

 class Test {
    public static void main(String[] args) {
        Stream<String> stream = Arrays.asList("One", "Two", "Three").stream();
        System.out.println(stream.reduce(null, (s1, s2) -> s1 + s2)); // 这里会是 nullOneTwoThree 因为NULL + string 是可以的
    }
}
```

#### 1.10.1 三个参数的版本

思路是 

* 第一个参数--- 数据类型
* 第二个参数 -- 如何处理
* 第三个参数 -- 如果并行 如何汇聚其他的小分支

#### 1.10.2 如果想得到并行和串行相同的结果

* combiner.apply(u,accumulator.apply(identity,t)) is equal to accumulator.apply(u,t) .

* 对于identity 的要求 combiner.apply(identity, u) is equal to u . 说白了最好是空的

* Accumulator 必须是 associative 和 stateless                associative是表示执行顺序是否影响结果

  比如说(s1.concat(s2)).concat(s3) equals to s1.concat(s2.concat(s3))

  ```java
    public static void main(String[] args) {
          String s1 = Arrays.asList("A", "E", "I", "O", "U").stream()
                  .reduce("", String::concat);
          String s2 = Arrays.asList("A", "E", "I", "O", "U").parallelStream()
                  .reduce("", String::concat);
          System.out.println(s1.equals(s2));
      } // 总是相同
  
   public static void main(String[] args) {
          Stream<String> stream = Stream.of("J", "A", "V", "A");
          String text = stream.parallel().reduce(String::concat).get();
          System.out.println(text);
      }// 这个也一样
  ```

  违反的例子

  ```java
  public class Test {
      public static void main(String[] args) {
          String s1 = Arrays.asList("A", "E", "I", "O", "U").stream()  // 违反了 combiner.apply(identity, u) is equal to u .
                  .reduce("_", String::concat);
          String s2 = Arrays.asList("A", "E", "I", "O", "U").parallelStream()
                  .reduce("_", String::concat);
          System.out.println(s1.equals(s2));
      }
  }
  ```

  

### 1.11 mapToObj

和boxed类似 用于转换

```java
public static void main(String[] args) throws IOException {
        Stream<String> stringStream = IntStream.of(1, 2, 3, 4, 5, 6, 7).mapToObj(elem -> "a" + elem); // primitive to stander
        LongStream longStream = IntStream.of(1, 2, 3, 4, 5, 6, 7).mapToLong(elem -> elem * 100L); // primitive to primitive
    }
```



### 1.12  stream 的 map 只接受 function 不接受奇奇怪怪的

### 1.13  stream 的mapToXXX 才接受 ToXXXFunction

### 1.14 **findFirst** 比较特殊 因为他永远都是第一个 无论是什么流

### 1.15 SummaryStatistics

在不同的primetive version中就是不同的 并且修改了 tostring 会返回 几乎所有的统计类的结果

### 1.16 peek

里面的操作仍然会影响stream

### 1.17 forEachOrdered

premitive 独有

### 1.18 range

```java
IntStream.range(-10, -10);// 空的
IntStream.rangeClosed(-10, -10);// -10
IntStream.range(2, -10)//空的
```

### 1.19 **orElseThrow**

可以抛出异常 **但是如果是Checked 则必须处理**

```java
 public static void main(String[] args) {
        OptionalInt optional = OptionalInt.empty();
        System.out.println(optional.orElseThrow(MyException::new)); // 如果是checked 会编译错误 因为没有处理
    }
```

### 1.20 sort

stream 可以调用 编译不会报错 但是如果没有实现compatible 那么会 castclass Exception

### 1.21 count

返回值值long

### 1.22 generate

一般需要搭配 limit来使用 或者其他终止操作 否则会一直创建 程序可能会卡死

```java
 public static void main(String[] args) {
        Stream<Double> stream = Stream.generate(() -> new Double("1.0"));
        System.out.println(stream.sorted().findFirst());
    } // 程序不会停止 会一只在添加 一直卡在 sorted 因为 sorted必须等待前面结束
```

###  1.23 Collectors.joining

```java
String str = ss.collect(Collectors.joining(",", "-", "+"));
// 第一个是分隔符 然后是 前缀和后缀
```

### 1.24 Collectors.toMap

如果出现key重复 那么会 运行时错误





## 2. 注意点

### 2.1 声明的时候 形参不能和接口变量相同！

### 2.2 BooleanSupplier 存在

### 2.3 lambda表达式如果使用了{ } 就一定要写;

### 2.4 如果流为空 那么其他的都不会执行 也就不会有空指针

### 2.5 Map 不可以直接调用stream 需要取得K or V

```java
map.entrySet();
map.keySet();
```

### 2.6 如果lambda表达式中需要使用 变量 则必须是 final 或者 effectively final , 但是如果变量不是当前作用域内定义的就无所谓

```java
class Counter {
     int count = 1;// 其他类中定义
}
public class Test {
    private static int a = 3; // 方法外定义
    public static void main(String[] args) {
        Counter counter = new Counter();
        Consumer<Integer> add = i -> counter.count += a ++; //都可以不需要是final的
        Consumer<Integer> print = System.out::println;
        add.andThen(print).accept(10); //Line 10
    }
} //编译通过
```

### 2.7 拉姆达表达式的参数相当于局部变量

* 如果是基本类型 那么就不会改变原来的值
* 如果是引用类型 那么就会改变

## 例题

```java
 public static void main(String[] args) {
        BiFunction<Integer, Integer, Character> compFunc = (i, j) -> i + j; // 编译错误
        System.out.println(compFunc.apply(0, 65));
  } // 会报错 因为 int 转 char 可以自动发生 但是 Integer转Character不会
```

```java
System.out.println(stream.reduce(res ++, (i, j) -> i * j)); public class Test {
    public static void main(String[] args) {
        int res = 1;
        IntStream stream = IntStream.rangeClosed(1, 4);
 
        System.out.println(stream.reduce(res++, (i, j) -> i * j)); //看清楚！这他妈res不是lambda
    } 
}
```

### 2.7  stream在关闭后再调用 是 RuntimeException  不是 编译错误

## 3. 各类操作的关系

* 中间操作在终止操作前不会执行， **但是可以保存** 会直到调用终止的时候执行 否则不执行

  ```java
  public static void main(String[] args) {
          Stream<String> stream = Stream.of("1.0","1.2","1.6","1.4","1.3");
          DoubleStream ds = stream.mapToDouble(s->Double.valueOf(s)).filter(d -> d>1.3);
          System.out.println(ds.sum());// 直到这里才执行
  
      }
  ```

  经典例子！！

  ```java
  class Whiz {
      public static void main(String[] args) {
          List<String> list = new ArrayList<>();
          list.add("1");
          list.add("2");
          list.add("3");
          list.add("4");
          list.add("5");
  
        // 下面这里只是保存 并没有执行
          Stream<Integer>  ints  = list.stream().map(         s -> {
                      System.out.print(s);
                      return Integer.parseInt(s);
                  }
          );
  
          System.out.print("Count : ");
  
        // 这里才开始执行 所以 Count 在前面
          System.out.print(ints.count());
  
      } // Count:123455 
    
  }
  ```

  

# Optional

---

* Optional version 之间无法转换 定义的时候就已经定死了 

## 1. API 相关

### 1.1 primitive version 的get 对应是 getAsXXX

### 1.2 如果value为空 调用get会抛出异常

### 1.3 如果添加了  .orElse则返回正常类型  而不是 optional

### 1.4 **ifPresent**

一定要有参数！！！ Comsumer

### 1.5 Map

只存在于 Optional 不存在 primitive里面

## 2. 创建

### 2.1 创建空

```java
Optional ops = Optional.empty();
```

```java
Optional<Integer> optional = Optional.of(null);  // 空指针
```



## 3. 转换

```java
op.flatMap(s -> Optional.of(Integer.decode(s)));

op.map(s -> Integer.decode(s));

// map return的 必须是值
// flatMap return 的必须是 Optional
```

## 4. 注意

即使用empty创建 仍然会报空值错误

```java
OptionalLong optional = OptionalLong.empty();
optional.getAsLong(); // 空值错误
```



# 函数式接口



## 1 Function

### 1.1 使用compose的时候 传入的参数的返回值，必须是这个function声明的参数

```java
    public static void main(String[] args) throws IOException {
        Function<Double, Double> mul = d -> d * 2; // 这里的double必须和 func的参数对应

        Function<Double, Integer> f = d -> d.intValue();
        Function<Double, Integer> func = f.compose(mul);

        System.out.println(func.apply(12.6));
    }
```

### 1.2 ObjInt 开头的

两个都是参数 一般后面一个是指定的 obj需要通过泛型来指定

```java
 ObjIntConsumer<Integer>
```

### 1.3 没有泛型的函数式接口默认为Object不可以指定其类型

```java
Predicate test2 = (String w) -> w.length() > 3; // 报错
```

### 1.4 Comsumer 注意点

```java
IntConsumer consumer = i -> i = i * i * i; // 这样是可以的
IntConsumer consumer = i ->  i * i * i; // 编译错误
```

## 2. Notation

如果用了notation那么编译的时候要有一个 非默认或者继承来的方法 **可以是重载**

## 散碎

### 1. primitive 类型的 各种 （DoubleSupplier）这样的 返回值是primitive  不是封装类 所以如果返回是null 会空指针

```java
 public static void main(String[] args) {
        DoubleConsumer c = s -> System.out.println(s);
        ArrayList<Double> list = new ArrayList<>(Arrays.asList(null));
        list.stream().mapToDouble(s -> s).forEach(c);
    } // 空指针 因为 null 无法被当做 double（可以被当做Double）
```



# Locale 

---

创建 并且都可以为空字符串

```java
Locale(String language)

Locale(String language, String country)

Locale(String language, String country, String variant)
   
```

可以创建错误的值 并且可以被打印出来 只会在后面的操作中出问题

```java
 public static void main(String[] args) {
        Locale locale = new Locale("temp", "UNKNOWN"); //Line 7
        System.out.println(locale.getLanguage() + ":" + locale.getCountry()); //Line 8
        System.out.println(locale); //Line 9
    }  // temp:UNKNOWN
// temp:UNKNOWN
```





## **1 ResourceBundle** 

**先查找同等级的 Java文件后查找同等级的 Properties**

查找顺序

1. bundleName + "_" + localeLanguage + "_" + localeCountry + "_" + localeVariant
2. bundleName + "_" + localeLanguage + "_" + localeCountry
3. bundleName + "_" + localeLanguage
4. bundleName + "_" + defaultLanguage + "_" + defaultCountry + "_" + defaultVariant
5. bundleName + "_" + defaultLanguage + "_" + defaultCountry
6. bundleName + "_" + defaultLanguage
7. bundleName

* 注意不会有 base + Country 必须有语言才考虑国家！

* **最后会检查 default**

* **大小写都支持** 比如说 声明的是 en  但是 basefile.EN 也支持

* 如果一个确定的被找到了 那么只能在他的继承树内继续查找

  ```java
  /**
  Dolphins.properties 
  name=The 
  Dolphin age=0  
  
  Dolphins_en.properties 
  name=Dolly 
  age=4  
  
  Dolphins_fr.properties 
  name=Dolly
  **/
  Locale fr = new Locale("fr");
  Locale.setDefault(new Locale("en", "US"));
  ResourceBundle b = ResourceBundle.getBundle("Dolphins", fr);
  b.getString("name");
  b.getString("age");
  // 最后调用 Dolphins_fr.properties and Dolphins.properties
  /**
  	解释：一旦找到了确切的 路径 那么就只能基于这个路径来向下查找 不能在使用上面的顺序了，所以默认包被跳过了！
  **/
  ```

  

### 1.1  **ResourceBundle** 会从上向下遍历到base

```java
Locale locale = new Locale("de");
ResourceBundle rb = ResourceBundle.getBundle("SRBundel", locale);
Enumeration bundleKeys = rb.getKeys();
// 这里会先遍历 SRBundel_de 然后遍历 SRBundel的 key 如果有子包有重复key取high level
// 如果 key不重复 则都输出
```

### 1.2  如果没有匹配的 则找base 如果实在没找到 就runtime exception

### 1.3 **getDisplayLanguage** 

如果没有设置 不会报错 打印空

### 1.4  如果使用class模式（继承List...) 那么需要初始化所有的Key至少， 因为Key不可以为空

![image-20201029115922628](/Users/bohanxiao/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/0adcf94f7ebd29a4cc07b0b6b94f2208/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/JavaSE 考点注意.assets/image-20201029115922628.png)

上面的这种不可以



### 1.5 一个例子

```java
    import java.util.*;
    import java.util.ResourceBundle.Control;
   
    public class RB{
           public static void main(String[] a){
                   Locale locale = new Locale("de");
                   ResourceBundle rb = ResourceBundle.getBundle("SRBundel", locale);
                   Enumeration bundleKeys = rb.getKeys();
               // 这里会一直遍历到 base的key
                            while (bundleKeys.hasMoreElements()) {
                                    String key = (String)bundleKeys.nextElement();
                                    String value = rb.getString(key);
                                    System.out.println("Key = " + key + ", " + "value = " + value);
                           }
 
               }
 

    }  
```



### 1.6 get & getProperties

Get 不可以带默认值 getProperties可以

## 2. 各种支持

### 2.1 instant 

- `NANO_OF_SECOND`
- `MICRO_OF_SECOND`
- `MILLI_OF_SECOND`
- `INSTANT_SECONDS`

## 3. ListResourceBundle

抽象类 继承重写 getContents

## 4. Control

用于配合 getbundle使用 来设置一些策略

```java
ResourceBundle.Control rbc = ResourceBundle.Control.getControl(Control.FORMAT_CLASS);
Locale locale = new Locale("de");
ResourceBundle rb = ResourceBundle.getBundle("SRBundel",locale, rbc);
// 这里会runtime exception 因为当前没有class类型的
```

## 注意点

### 1.1 equals

```java
 @Override
    public boolean equals(Object obj) {
        if (this == obj)                      // quick check
            return true;
        if (!(obj instanceof Locale))
            return false;
        BaseLocale otherBase = ((Locale)obj).baseLocale;
        if (!baseLocale.equals(otherBase)) {
            return false;
        }
        if (localeExtensions == null) {
            return ((Locale)obj).localeExtensions == null;
        }
        return localeExtensions.equals(((Locale)obj).localeExtensions);
    } // 也就是说实际上是判断内容是否相同
```



```java
 public static void main(String[] args) {
        Locale l1 = new Locale.Builder().setLanguage("en").setRegion("US").build();
        Locale l2 = Locale.US;
        Locale l3 = new Locale("en");
 
        System.out.println(l1.equals(l2));
        System.out.println(l2.equals(l3));
    } // true false
 
```

### 1.2 创建Locale 也可以直接创建单个的国家或者地区 

```java
        Locale loc = Locale.ENGLISH;// 这个时候国家是空
```



# 时间类型

---

**不可变类型！！！！**

* LocalDate 等等类 创建的时候需要注意必须符合规则 不能创建类似 月份大于12这样的 会有runtime exception
* 全部都是 **线程安全的**

## 1. Duration

基于时间

### 1.1 support

* Seconds 
* Nanos

### 1.2 ofdays 

自动转换为hours

### 1.3 between

静态方法

* 两个参数如果类型不同， 那么会将第二个参数转换到第一个参数 如果不能转换 则会 runningTime exception
* duration的 between可以用如下 只要第二个参数可以转换到第一个来
* 都是后面减前面 **并且有正负**

```java
public static Duration between(Temporal startInclusive, Temporal endExclusive) 
public static Period between(LocalDate startDateInclusive, LocalDate endDateExclusive)
// 也已看到 duration的只要是后面的可以转换到前即可（第一个必须需要是时间单位 不能是日期的） 但是Period 就必须都是localDate
public static void main(String [] args) {
        LocalTime t1 = LocalTime.now(); // 
        LocalDateTime t2 = LocalDateTime.now();
        System.out.println(Duration.between(t2, t1)); // runnting time exception 这里反过来就可以
 }	


 public static void main(String [] args) {
        LocalDate d1 = LocalDate.now();
        LocalDateTime d2 = LocalDateTime.now();
        System.out.println(Duration.between(d1, d2)); // 也会报错 因为第一个必须是时间 而这里是日期
    }
}
```



## 2.Period

基于日历

### 2.1 api

#### 2.1.1 withDays 等不是static 只有of 和一些计算是static 别的类也类似

#### 2.1.2 normalized

* **会标准化年和月** **不会标准化Day**
* 将“ 1年15个月”的期限归一化为“ 2年3个月” &  1年-25个月”的期限将被归一化为“ -1年-1个月

### 2.3 ZERO

```java
 System.out.println(Period.ZERO.getUnits()); // {years,Months,Days}
```





## 3. ChronoUnit 和  ChronoField

ChronoUnit(of 类型只有一个HALF_DAYS)  时间单位  ChronoField 日历单位（of）

## 4. LocalDate

at 和 with 都是 instance methods

### 4.1 support

- `DAY_OF_WEEK`
- `ALIGNED_DAY_OF_WEEK_IN_MONTH`
- `ALIGNED_DAY_OF_WEEK_IN_YEAR`
- `DAY_OF_MONTH`
- `DAY_OF_YEAR`
- `EPOCH_DAY`
- `ALIGNED_WEEK_OF_MONTH`
- `ALIGNED_WEEK_OF_YEAR`
- `MONTH_OF_YEAR`
- `PROLEPTIC_MONTH`
- `YEAR_OF_ERA`
- `YEAR`
- `ERA`

### 4.2 plus

参数：1. long + unit 2.  **Period**  

duration会--RunTime Exception

对应localTime 只可以用 duration

### 4.3 adjustInto 

参数才是**被修改的数据** 

## 5. LocalTime

at 和 with 都是 instance methods

### 5.1 support

Fields

- `NANO_OF_SECOND`
- `NANO_OF_DAY`
- `MICRO_OF_SECOND`
- `MICRO_OF_DAY`
- `MILLI_OF_SECOND`
- `MILLI_OF_DAY`
- `SECOND_OF_MINUTE`
- `SECOND_OF_DAY`
- `MINUTE_OF_HOUR`
- `MINUTE_OF_DAY`
- `HOUR_OF_AMPM`
- `CLOCK_HOUR_OF_AMPM`
- `HOUR_OF_DAY`
- `CLOCK_HOUR_OF_DAY`
- `AMPM_OF_DAY`

Units

- `NANOS`
- `MICROS`
- `MILLIS`
- `SECONDS`
- `MINUTES`
- `HOURS`
- `HALF_DAYS`

## 6 Dateformat

### 6.1 创建

抽象类 不可以 new

* getInstance 错误方法！
* **getDateInstance 可以带参数**

```java
DateFormat df = DateFormat.getDatelnstance();
```

### 6.2 DateTimeFormatter

```java
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL);
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL);
```



## 7. APIs

### 7.1  format() 返回值是String 不是 LocalDate or LocalDateTime

## 8. instant

永远和UTC为基准 和其他地方地区没有任何关系

### 8.1 getLong 支持

NANO_OF_SECOND

MICRO_OF_SECOND

MILLI_OF_SECOND

INSTANT_SECONDS

### 8.2 转换

不可以和 Date 或者 time 直接转换 因为 Instance **有区域信息**

#### 8.2.1 LocalDateTime

```java
instant.atZone(ZoneId.systemDefault()).toLocalDateTime()
// 先转换到 ZonedDateTime 再转换到LocalDateTime
 Instant instant = zonedDateTime.toInstant();// 2015–05–25T15:55:00Z 两个0
```

### 8.3 自动补全机制 3 6 9

ISO_LOCAL_TIME represents time in following format:

HH:mm (if second-of-minute is not available), 

HH:mm:ss (if second-of-minute is available), 

HH:mm:ss.SSS (if nano-of-second is 3 digit or less), 

HH:mm:ss.SSSSSS (if nano-of-second is 4 to 6 digits), 

HH:mm:ss.SSSSSSSSS (if nano-of-second is 7 to 9 digits). 

### 8.4  EPOCH

```java
 public static void main(String [] args) {
        System.out.println(Instant.EPOCH);
    } // 1970-01-01T00:00:00Z 两个0结尾
```



## 9. Zone类型

 DateTime 在他们后面加上区域信息

```java
public static void main(String[] args) {

    ZoneId zoneId = ZoneId.of("US/Eastern");

    ZonedDateTime zonedDateTime = ZonedDateTime.of(2015, 1, 20, 6, 15, 30, 200, zoneId);
    
    // 通过变量创建
    LocalDate localDate = LocalDate.of(2034, 2, 3);
    LocalTime localTime = LocalTime.now();
    ZonedDateTime zonedDateTime1 = ZonedDateTime.of(localDate, localTime, zoneId);
}
```

用来计算转换规则 instant 和其他

## 10. Format

* day & month 都分一位数和两位数！

* MMM 首字母大写其他小写的

* MMMM 全称

* ISO_LOCAL_TIME represents time in following format:

  HH:mm (if second-of-minute is not available), 

  HH:mm:ss (if second-of-minute is available), 

  HH:mm:ss.SSS (if nano-of-second is 3 digit or less), 

  HH:mm:ss.SSSSSS (if nano-of-second is 4 to 6 digits), 

  HH:mm:ss.SSSSSSSSS (if nano-of-second is 7 to 9 digits). 

  

  Valid values for hour-of-day (HH) is: 0 to 23. 

  Valid values for minute-of-hour (mm) is: 0 to 59. 

  Valid values for second-of-minute (ss) is: 0 to 59. 

  Valid values for nano-of-second is: 0 to 999999999.

  ```java
   public static void main(String [] args) {
          LocalTime time = LocalTime.parse("14:14:59.1111");
          System.out.println(time); //14:14:59.111100 
      }
  ```

  ```java
   public static void main(String [] args) {
          LocalDate date = LocalDate.of(2018, 2, 1);
          DateTimeFormatter formatter = DateTimeFormatter
                  .ofPattern("DD'nd day of' uuuu");
          System.out.println(formatter.format(date));
      }// 32nd day of 2018
  ```

  ```java
   public static void main(String [] args) {
           LocalDate date = LocalDate.of(2018, 11, 4);
           DateTimeFormatter formatter = DateTimeFormatter.ofPattern("D-MM-uuuu");
           System.out.println(formatter.format(date));
       } // 需要考虑 会不会大于两位数 这里是300多天 如果用D 和 DDD是可以的 用DD就不可以
  ```
  
  

### 10.1 使用方法

```java
 LocalDate date = LocalDate.of(2018, 11, 4);
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-uuuu"); 
System.out.println(formatter.format(date).equals(date.format(formatter))); // 两种使用时一样的结果 都是String
```

### 10.2 NumberFormat 

```java
NumberFormat currFormatter = NumberFormat.getCurrencyInstance(Locale.US); 
System.out.println(currFormatter.parse("$7.00")); // parse需要使用 catch
```

### 10.3 例子！

```java
public static void main(String[] args) {
        LocalDate ldt = LocalDate.of(2015, 12, 1);

        DateTimeFormatter format = DateTimeFormatter.ofPattern("D/M/yyyy"); // 可以
        DateTimeFormatter format1 = DateTimeFormatter.ofPattern("DD/M/yyyy");// 不可以
        DateTimeFormatter format2 = DateTimeFormatter.ofPattern("DDD/M/yyyy");// 可以
        // 如果用d/m 那么就是能用一个的时候就用一个 能用两个的时候就用两个
        // 如果用dd/mm 那么就是必须用0填充

        System.out.println(ldt.format(format));
    }
```



## 11 year & month

### 11.1 创建

```java
Year y = Year.of(2015);
```

### 11.2 year to LocalDate

其他的类似 照着推

```java
 LocalDate ym = y.atMonthDay(MonthDay.of(4,10));
```





## 散碎

### 1. 如果创建的格式不合理--Runtime Excption

```java
        LocalTime t1 = LocalTime.parse("141:03:15.987"); // Runtime Exception
```

### 2. 如果添加后超过时间了 则会自动计算

```java
LocalTime t1 = LocalTime.parse("11:03:15.987");
System.out.println(t1.plus(22, ChronoUnit.HOURS));
// 09:03:15.987
```

### 3. 例题

```java
public class Test {
    public static void main(String [] args) {
        LocalDateTime dt = LocalDateTime.parse("2018-03-16t10:15:30.22");
        System.out.println(dt.toLocalDate() + dt.toLocalTime()); // 这里的两个类型不可以直接进行加减 所以会编译错误
        // 正确的应该是 + "" + 
    }
}
```

### 4. plus的使用

LocalDate 的支持 Preiod

LocalTime 的支持 Duration

不会编译错误 但是会运行错误 如果乱用

### 5. XX.now 就返回什么类型

```java
Year y = Year.now();
```

### 6. 例题

```java
public class Test {
    public static void main(String [] args) {
        System.out.println(Instant.EPOCH);
    }
} // 1970-01-01T00:00:00Z 不是三个0
```

```java
  public static void main(String [] args) {
        LocalDateTime dt = LocalDateTime.parse("2018-03-16t10:15:30.22");
        System.out.println(dt.toLocalDate() + dt.toLocalTime()); // 编译错误 这两个返回的不是字符串 而是对于Object
    }
```

### 7. 时区转换

都是当前时间减去 后面的时间 这里是负数 所以其实是加 

```java
2016–08–28T05:00 GMT-04:00 // 9 点
2016–08–28T09:00 GMT-06:00 // 15点
```



# Lambda

---

条件 一个**接口**仅有**一个**待实现的方法 

* 不可以初始化抽象类

* 不可以直接用 比如说 System.out.println((String s1, String s2) -> s1 + s2); 不可以这样使用

  ```java
  interface Operator<T> {
      public abstract T operation(T t1, T t2);
  }
   
  public class Test {
      public static void main(String[] args) {
          System.out.println(new Operator<String>() {
              public String operation(String s1, String s2) {
                  return s1 + s2;
              }
          });
          
     	//System.out.println((String s1,String s2) -> s1 + s2);
      // 不可以直接这样替代 应该先创建出来 这样无法识别
          /*
          	这道题的问题是 println需要传入的参数是Object lambda表达式不能直接表示一个object 只能用来声明
          */
      }
  }
  ```

  

  

# 枚举类

---

## 1. 注意点

* 常数必须写在最前面 ---否则**编译报错**

* 构造器必须是 **private** 如果没写 默认是private

* **不可以**继承任何其他枚举和类 但是**可以**继承接口

* 所有枚举隐晦的继承了  **java.lang.Enum class**

* 不可以被clone 编译错误

* 枚举类成员默认是 **public static final** --所以可以被switch使用

* 枚举类可以被定义为 static 

* 可以被在内部类中声明

* 可以有具体方法 和 抽象方法 **但是所有的抽象方法 实例必须实现**

  ```java
  protected final Object clone() throws CloneNotSupportedException {
      throw new CloneNotSupportedException();
  } // 会直接报错 因为是protected 包外都无法访问 并且返回值是Object 一定要注意
  ```

* 枚举类不可以通过New来创建 因为**构造器默认私有**，单例模式

* 枚举类会为**每个成员初始化一次**！

* 枚举类的构造器不可以访问static变量

  ```java
  enum Course{
      JAVA(100);
      private static int cost;
      private Course(int c){
          this.cost = c; // 编译错误
      }
  }
  ```

* 重写过 toString 打印成员名字

* **不可以被定义在非静态方法中** 因为他的成员都是static的

## 2. 顺序

枚举类实现了 Comparable  排序方法是声明顺序 **所以说任何排序相关 都是按着常数定义顺序**

```java
 public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }
```

## 3. API

```java
values(); // 返回所有值
ordinal(); // 每个枚举索引
valueOf(); // 指定字符串枚举常量

Color.valueOf("RED")  // 返回的仍然是枚举类还是
Color.values();
col.ordinal() // instance method
```



# **Inner** class

---

* **外部类 是不可以定义为静态的！** **静态类只存在于内部类中**
* 

## 1 注意点

* 初始化顺序 static initialization block, instance initialization block and then constructor.
* 如果在内部的方法里声明 那么可以用 Outer.Inner 或者 直接Inner
* this 不可以在static methods中使用 所以不可以这样 this.new Inner();
* **可以继承外部类** 

## 2. 成员内部类

可以静态或者非静态 和**正常的类一样** 该有啥都可以用 也可以再添加内部类

* 可以被 final修饰
* 可以被 abstract 修饰

### 2.2  调用outer的东西

如果需要 通过 outerclass.this.(method or fields)

### 2.3 非静态成员内部类

**不可以有静态方法！！！！经常考！！！**

可以有静态常量 也就是 static final

```java
class A  //This is a standard Top Level class.
{
    class X {
        public static  void test(){}; // 报错
        static final int j = 10;  //compiles fine!
    }

    public static class B //This is a static nested class
    {

    }
}
```



### 2.4 静态成员内部类

* 只有在被访问的时候静态内部类才会加载 不会和外部类一样在main方法执行前加载 

* 可以有非静态的内部类，也可以有静态或者非静态的属性

  ```java
  class A {
      public static class B {
          class C{
              
          }
      }
  }
  ```

  

#### 2.4.1 静态方法

可以被Outer.Inner.MethodName 直接调用 **要注意** **该内部类也必须是静态的才可以这么调用 否则还是要创建先**

## 3. 局部内部类（定义在方法中）

* 不可以添加任何 access modifier 但是可以添加 abstract 或者 static（不属于access modifier）
* JDK7 inner class 可以访问 static 和 final修饰的变量（在外部类中）
* JDK8 之后inner class 有权限访问的是 final修饰 或者 **实际上是final的**（effectively final）即初始化后从来没有修改过的
* **不可以定义任何static的除非是final**
* **只可以在定义的方法内被创建**
* This 指向的是自己这个内部类类 不是方法 需要考虑访问变量问题

```java
class Outer {
    public static void sayHello() {}
    static {
        class Inner {
             final static int a = 1;  // 不可以定义static  除非是 final static
//              static int a = 1;  这里不可以
        }
        new Inner();
    }
}
```



```java
 public class Whiz{
        
           int x = 10;
          
           public static void method(int c,int i){
                      
                       class Test{
                                       public void in(){
                                       // here
                                         // 这里只可以访问i 因为i从来没有被修改过 但是可以通过创建Whiz来访问x
                                   }
                   }
                   c +=2;
                   new Test().in();
       }
    
       public static void main(String p[]){
                   Whiz.method(3,4);
      }
   }
```

```java
 
class Outer {
    private String msg = "A";
    public void print() {
        final String msg = "B";
        class Inner {
            public void print() {
                System.out.println(this.msg); // this 是在inner内的 所以编译错误
            }
        }
        Inner obj = new Inner();
        obj.print();
    }
}
 
public class Test {
    public static void main(String[] args) {
        new Outer().print(); 
    }
}
```



## 4.  匿名内部类

**可以继承 或者 实现 一个类或者接口 不可以同时 也不可以多个** （这里说的继承或者实现指的就是自己本身）

**无法调用private 属性**

不可以写 构造器， 因为没有名字

```java
class Point {
    private int x;
    private int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public String toString() {
        return "Point(" + x + ", " + y + ")";
    }
}
 
public class TestPoint {
    public static void main(String [] args) {
        List<Point> points = new ArrayList<>();
        points.add(new Point(4, 5));
        points.add(new Point(6, 7));
        points.add(new Point(2, 2));
        
        Collections.sort(points, new Comparator<Point>() {
            @Override
            public int compare(Point o1, Point o2) {
                return o1.x - o2.x; // 报错
            }
        });
    }
}
```



## 内部类总结 

```java
package com.bohan.test;

import com.sun.xml.internal.ws.api.model.wsdl.WSDLOutput;
import org.w3c.dom.ls.LSOutput;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.time.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.function.*;
import java.util.stream.IntStream;
import java.util.stream.Stream;

/**
 * @Project : netty
 * @Author : Bohan Xiao
 * @Create : 2020-11-13
 **/
class A {
    private String name = "bohan";
    private static int age = 25;

    // test for local inner class
    private int time = 12;

    // 静态内部类 可以被任何access modifier 修饰
    static class B {
        private int a ;
        private static int b = 3;

        public void test(){
            System.out.println(name); // 编译错误 因为是非静态的
            System.out.println(age);  // 编译通过 因为是静态的
        }
        class C{ // 可以有非静态的内部类

        }
    }
    // 非静态内部类 可以被任何 access modifier 修饰
     class D{
        private int a ;
        private static int b = 3; // 编译错误 非静态内部类不可以有静态属性 除非是final
        private static final int c = 3; // 编译通过

        public static void method1(){}; // 编译错误 方法不可以被定义为final  所以费静态类也不可以有非静态方法
        public void method2(){} // 编译通过
        static  class E{}
    }

    // 局部内部类 不可以被任何access modifier 修饰

    // 1. 定义在非静态方法内(静态方法一样 这里不做举例)
    public void outerMethod1(){
        int a = 3;
        a++;
        int b = 3;
        static class localClassA{ // 编译错误 局部内部类就不可以定义为静态的

        }

        abstract class LocalClassB{ // abstract 可以
            public static final int age1 = 3; //可以
            public static  int age2 = 3; // 编译报错
            public void methodB(){
                System.out.println(a); // 错误 只能访问 final 或者 effective final 的局部变量
                System.out.println(b);
                A.age ++;
                System.out.println(A.age); // 可以访问静态 尽管age发生了改变
                A a1 = new A();
                a1.time ++;
                System.out.println(new A().time); // 也可以访问非静态 不管是否发生变化
            }
            
            class Test{}
        }

        final class LocalClassC{  // final 也可以

        }

    }

}
/**
 * 总结
 *      1. 所以内部类都可以再用内部类 都可以再有内部类 非静态内部类的不可以有非静态的子内部类
 * 非静态成员内部类
 *      1. 可以被所有access modifier修饰 包括 final 或者 abstract
 *      2. 不可以定义static属性 除非是static final
 *      3. 不可以定义static方法 （因为方法没有 final）
 *
 * 静态成员内部类
 *      1. 可以被所有的access modifier修饰 包括final 或者 abstract
 *      2. 不可以有非静态的成员和静态成员
 *      3. 所拥有的非静态方法不可以访问非静态变量（所有类都是）
 * 局部内部类
 *      1. 在静态方法和非静态方法中一样
 *      2. 不可以定义static属性 除非是static final
 *      3. 不可以定义static方法 （因为方法没有 final）
 *      4. 方法只可以访问 final 或者 effective final 的局部变量（定义在这个局部方法内部的）
 *      5. 可以访问外部的静态属性和非静态属性（通过 new 创建的object的属性） 无论是否发生改变
 */


```



# JDBC

---

## 1. JdbcRowSet

高级版 ResultSet **数据连接必须在线** 和普通版一样

### 1.1 创建

#### 1.1.1使用 实现类（**JdbcRowSetImpl**）

* 直接创建
* 接收ResultSet 必须是可以滚动的
* 接收Connection

#### 1.1.2使用 **RowSetProvider**

* 用工厂模式创建

```java
jdbcRs = new JdbcRowSetImpl();
jdbcRs = new JdbcRowSetImpl(rs);
jdbcRs = new JdbcRowSetImpl(con);

myRowSetFactory = RowSetProvider.newFactory();
jdbcRs = myRowSetFactory.createJdbcRowSet();

// 除了用 ResultSet 创建 其他直到 调用
//	jdbcRs.setCommand("select * from COFFEES");
//	jdbcRs.execute(); 之前都不保存数据
```

### 1.2 使用例子

```java
 public void getPlayer(int id)throws Exception{
                   RowSetFactory myRowSetFactory = RowSetProvider.newFactory();
                   jrs = myRowSetFactory.createJdbcRowSet();
   
                   jrs.setUrl("jdbc:mysql://:3306/teams");
                   jrs.setUsername("root");
                   jrs.setPassword("");
                   jrs.setCommand("SELECT FName, Country FROM allstar WHERE ID="+ id);
                   jrs.execute();
                  
                   while(jrs.next()){
                                   System.out.println(jrs.getString("FName") +"  " + jrs.getString("Country"));
                   }
           }
```



## 2  CachedRowSet

**离线模式** 短暂接触数据源 内容保存在内存中 获取方式和 RS类似 

```java
crs.populate(rs); // 读取一般的RS数据 需要catch SQLException
```

### 2.1 创建

和JdbcRowSet 类似

### 2.2 更新

必须调用acceptChanges来更新数据源 而上面两种不需要

```java
  crs.updateShort(3, 58);
  crs.updateInt(4, 150000);
  crs.updateRow();    // 更新set
  crs.acceptChanges();// 更新数据源
```

## 3. 事务

### 3.1 savepoint 在事务提交了之后则不被保存 所以在commit前必须调用 否则会报运行是错误

## 4. ResultSet

* 数据可以更新，但是更新到数据库需要调用 

  ```java
  rs.updateRow();
  ```

* 起点在第一行之前 需要先调用next才可以获得真正数据 否则其他操作会 -- Runtime Exception

* **如果当前在insert行** 调用 refreshRow updateRow deleteRow 会 SQLException

* 如果statment 关闭了 对应的 ResultSet也会自动关闭

### 4.1 API

全都抛出**SQLException**

一共分为三大类型 

* 指针类型 absolute previous **需要结果集可以滚动**
* 更新和获取结果集类型（不更新数据库）get updateInt
* 更新结果集 update 每次更新后必须要调用！ 否则 RuntimeException

#### 4.1.1 absolute

正负和0都可以 绝对位置 负数就从后往前数

#### 4.1.2 relative

相对位置 也是前后都可以

#### 4.1.3 moveToCurrentRow

如果光标处于插入行，则将光标返回到当前行，**其他情况下，这个方法不执行任何操作**

#### 4.1.2 update类型

## 5. Statement

#### 5.1.1 execute & executeQuery & executeUpdate

* execute 返回值为 boolean 如果是查询并且有ResultSet 返回 true， 如果没有数据或者是Update acount 返回false
* executeQuery 返回 ResultSet
* executeUpdate 返回 int 更新数量
* **如果再次进行查询会自动关闭前一个打开的结果集**

#### 5.1.2 createStatement

createStatement(), createStatement(int, int) and createStatement(int, int, int).

## 6. Class.forName

* 默认jar包在classpath 下

* 参数是 class name 全称 不是URL

  ```java
  Class.forName("jdbc:mysql://localhost:3306/ocp"); // 编译错误
  ```

  

## 散碎

### 1. Starting with JDBC 4, there is no need to manually load the driver class.

### 2. getStrng 可以用来获取int

# 5.Files

**Files 主要和 Path操作 而不是file** Files 是NIO.2 而file是IO包的 属于两个版本的产物

* File的初始化！ 

```java
File f1 = new File("F:\\f1.txt"); // 不会创建File文件 只是对象
FileWriter fw = new FileWriter("F:\\dir\\f2.txt"); // 会创建 但是因为 dir不存在 所以报错
PrintWriter pw = new PrintWriter("F:\\f3.txt"); // 会创建
```

* 大部分API处理的时候有可能会出现无法访问这样的 RuntimeException

## 1. APIs

通过new 来创建file  **没有Path作为参数的构造器**！！！

**属于IO产品 和Path无关  所以没有用Path**

```java
File(File parent, String child);
File(String pathname);
File(String parent, String child);
```



### 1.1 读取内容

参数都是Path 并且都需要处理异常

* lines 返回值 Stream<String> 参数是path 也可以加字符集
* readAllLines 返回值是List<String> 参数是 path 也可以加字符集  如果不加字符集 PDF 和 doc会报错
* list Stream<Path> 

### 1.2 读取属性 & 修改数据

#### 1.2.1 读取数据

* Files.readAttributes(path, PosixFileAttributes.class) ; 这种return 一个 Object 通过属性来访问
* readAttributes(Path path, String attributes, LinkOption... options) 返回Map String部分可以是表达式 比如 *
* Files.getAttribute(Path path, String attribute, LinkOption... options) 获取指定属性 或者 *获取所有 返回List<String>

#### 1.2.2 修改数据

* 值能通过 **BasicFileAttributeView** 来修改数据 例子 通过**getFileAttributeView**来获取前者

  ```java
   public static void main(String[] args) throws IOException {
          Path path = Paths.get("/turtles/sea.txt"); 
          BasicFileAttributeView view = Files.getFileAttributeView(path, BasicFileAttributeView.class);// 获取 View
          BasicFileAttributes data = view.readAttributes(); 
          FileTime lastModifiedTime = FileTime.fromMillis(data.lastModifiedTime().toMillis() + 10_000); 
          view.setTimes(lastModifiedTime, null, null); // 修改
      }
  // view.setTimes(lastModifiedTime,null,null);
  ```

  

### 1.3 find  

* 返回stream 查找 所有的文件和目录 满足条件的 
* Bipredicate 第一个参数是 Path 第二个参数是 BasicAttitude

```java
public static Stream find(Path start,int maxDepth, BiPredicate matcher,FileVisitOption... options)throws IOException
  
   Path path = Paths.get("turkey");
        Stream<Path> walk = Files.find(path,1,(p,att) -> {
            return p.isAbsolute();
        });
```

### 1.4 walk

遍历所有的文件和目录

```java
public static Stream walk(Path start, int maxDepth, FileVisitOption... options) throws IOException
```

### 1.5 **list**

static method

参数 path return Stream<Path>

### 1.6 delete

* **处理异常**

如果删除文件或文件夹 路径上有别的  --不报错 但是不会删除

如果删除的是链接 --直接删除掉

### 1.7 copy

* **需要处理异常**

需要考虑的情况 

* 如果是文件夹 那么会报重名错 -- 已经存在  因为如果是文件夹 那么会考虑再上一目录创建一个和文件夹名字一样的文件 那么会重名

```java
public class Test {
    public static void main(String[] args) throws IOException {
        Path src = Paths.get("F:\\A\\B\\C\\Book.java");
        Path tgt = Paths.get("F:\\A\\B"); // 重名错误 因为 B是目录 那么会企图创建 A下的b.java 但是不可以 因为不可以和文件夹重名
        Files.copy(src, tgt);
    }
}
/**
F:.
└───A
    └───B
        └───C
                Book.java
                **/
```

* 如果是 链接 那么会拷贝实际的文件 而不是链接

另一种写法

```java
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE); // 会删除source
```





### 1.8 resolveSibling

和 resolve 一样 如果参数是 绝对路径 直接返回参数

```java
path1.resolveSibling(path2) // 是取自己的sibling 然后再合并
```

### 1.9 getParentFile & getParent

一个返回 file 类型 一个返回String Path 接口有一模一样的使用和方法

并且都可以返回根目录 

```java
public static void main(String [] args) {
        File file = new File("C:\\Users");
        System.out.println(file.getParentFile());
        System.out.println(file.getParent());
  } // C: 都是
// 如果是相对路径 那么getRoot 返回空 
// 如果路径没有父级 那么返回Null
```



### 1.10 relativize

* 必须是都是绝对或者都是相对

* 如果root不同 --RuntimeException

```java
// 如果出现看似没有关联的相对路径 那么退到头
 public static void main(String[] args) {

        Path p1 = Paths.get("x\\y");
        Path p2 = Paths.get("z");
        Path p3 = p1.relativize(p2);
        System.out.println(p3);//../z
    }
```

* 不会进行 简化操作 

  ```java
  Path p1 = Paths.get("c:\\personal\\.\\photos\\..\\readme.txt");
  Path p2 = Paths.get("c:\\personal\\index.html");
  Path p3 = p1.relativize(p2);
   System.out.println(p3);
  // ..\..\..\..\index.html 
  ```

  

### 1.11 createDirectory & createDirectorys

* **需要处理异常**

前者可以中间**不可以有不存在的**---NoSuchFileException 一定要注意 和 mkdir (必须存在) & mkdirs(不一定)的区别

后者可以可以有不存在的 一起创建

### 1.12 toRealPath

* **需要捕获IO异常**
* 返回真实路径 **必须是真是存在的** 否则会有**IO 异常**

### 1.13 mkdir & mkdirs

这些是IO包下 File class的 不是 Files的

如果路径都存在 那么可以用前者 如果有不存在的 那么需要用后者

### 1.14 **newBufferedReader**

返回值是 BufferedReader

```java
public static BufferedWriter newBufferedWriter(Path path, Charset cs,  OpenOption... options)  throws IOException
  
  public static BufferedWriter newBufferedWriter(Path path,  OpenOption... options)  throws IOException
```



### 1.15 write

写的file必须存在 否则会 报错 文件不存在

```java
public static Path write(Path path,
                         Iterable<? extends CharSequence> lines,
                         Charset cs,
                         OpenOption... options)
                  throws IOException
```

### 1.16 move

* 处理异常

* 如果已经存在 那么需要 +参数REPLACE_EXISTING FileAlreadyExistsException
* 如果是文件夹 且不为空 那么 DirectoryNotEmptyException
* metadata 总会被传递 其他属性需要flag标识
* target 文件不需要指定后缀名 会根据source来变化 比如source是文件 那么target也一定是文件 source是文件夹 那么target也一定是文件夹

### 1.17 setAttribute

* 注意没有s结尾 
* **需要处理异常**

```java
Files.setAttributes(path,"dos:readonly",true);
```

### 1.18 **toAbsolutePath**

* 绝对路径 那么返回绝对路径 无论是否真实存在
* 相对路径 那么添加到当前目录的末尾

### 1.19 isSameFile

* 需要处理IOException
* 先用equls对比 如果不同 那么 会比对真实文件系统的文件（也就是说会处理路径 equlas 就不会）

* 可以包含链接 并且会自动导航链接

  ```java
  try{
    System.out.println(Files.isSameFile(Paths.get("/user/home/cobra"),          Paths.get("/user/home/snake")));
    // cobra -> snake
    // 所以结果是true
  }catch(IOException e){
    
  }
  ```

### 1.20 isRegularFile

只有file 是 regular的 其他的都不是 比如链接 文件夹

但是！如果链接指向的是file  那么判断下也是regular的

### 1.21 isHidden

**需要处理异常** 

### 1.22  size

* 需要处理异常

和file 的length 一样 都是返回文件长度 in byte.

### 1.23 getLastModifiedTime & setLastModifiedTime & getOwner & setOwner

* 需要处理异常

### 1.24 subpath

不包括返回不包括root



## 2. FileVisitor

接口 4个方法

![image-20201028192709077](/Users/bohanxiao/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/0adcf94f7ebd29a4cc07b0b6b94f2208/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/JavaSE 考点注意.assets/image-20201028192709077.png)

### 2.1  **SimpleFileVisitor**

实现 FileVisotor的 类 可以想重写谁就重写谁 方便（默认都实现了简单的）

#### 2.1.1 需要注意的是 

4个方法中 有两个只针对file 有两个全都可以

## 3. attribute

都是接口

### 3.1 BasicFileAttributes

公共部分

![image-20201029113724365](/Users/bohanxiao/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/0adcf94f7ebd29a4cc07b0b6b94f2208/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/JavaSE 考点注意.assets/image-20201029113724365.png)

### 3.2 DosFileAttributes

windows的

![image-20201029113757668](/Users/bohanxiao/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/0adcf94f7ebd29a4cc07b0b6b94f2208/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/JavaSE 考点注意.assets/image-20201029113757668.png)

### 3.3PosixFileAttributes

unix系统的

![image-20201029113826213](/Users/bohanxiao/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/0adcf94f7ebd29a4cc07b0b6b94f2208/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/JavaSE 考点注意.assets/image-20201029113826213.png)

# Collection

---

* Map类除了ConcurrentHashMap 都 **没有实现Collection!!!!**
* 查重复使用 HashCode + equels  排序是用 compareTo

## 1. List

### 1.1 removeAll 

接收Collection 移除List里面参数List的所有elements

### 1.2 注意泛型问题

```java
public static void main(String[] args) {
    List list = new ArrayList<>();
    list.add(new Integer(10));
    list.add(new Integer(11));
    list.add(new Integer(13));
    list.add(new Integer(14));
                               
    list.removeIf(in -> in%2 != 0); // 编译错误 因为没有指定泛型Object不可以当做Integer来做处理
                                 
System.out.println(list);
}
```

### 1.3 asList

**返回的list size固定**

## 2. Map

* hashmap 默认无序  LinkedHashMap  默认为插入顺序
* 普通Map **Key可以为Null**

### 2.1 replaceAll

接收 Bifunction 两个参数一个返回值

### 2.3 **putIfAbsent**

通过 get 查询 如果 返回值是Null 则重新添加

* 可能没有K
* 可能有K但是V为Null

### 2.4 **computeIfPresent**

### 2.5 **subMap**

这个方法定义在 treemap中 不在 map中

### 2.6 Treemap 的key 只可以用可以排序的来定义

### 2.7 **replace**

```java
default boolean replace(K key,V oldValue,V newValue);
```

如果当前k对应的值等于 oldvalue 那么久替换成newvalue

### 2.8 merge

```java
// 三个参数 第一个是 key  第二是 新的值 第三个是merge方法
// 如果第一个key 存在 那么就把他的值 和 新值 通过方法3 合并
// 如果不存在 那么把这个key的值赋值为 新的值
Map<String, String> map1 = new HashMap<>();
map1.put("a", "x");
map1.put("b", "x");
BiFunction<String, String, String> f = String::concat;
map1.merge("b", "y", f); // 
map1.merge("c", "y", f);
System.out.println(map1);//{a=x, b=xy, c=y}

```



## 3. Queue

### 3.1 removeLastOccurrence

移除最后一个出现的啥啥啥



## 4. TreeSet

* 自然排序 + 单一值
* 必须实现了**Comparable** 的调用 compareTo **不需要重写 HashCode 和 equals**
* 不可以存放Null

### 4.1 API

* floor(value)刚刚好 比value小的 包含value

* ceiling 刚刚好比value大的 包含value

* higher 不包括value

* lower 不包含value

* subSet backed by original set 如果插入和删除 必须在指定范围内

  ```java
   public static void main(String[] args) {
  
          TreeSet<Integer> s = new TreeSet<Integer>();
          TreeSet<Integer> subs = new TreeSet<Integer>();
  
          for(int i = 324; i<=328; i++)
          {
              s.add(i);
          }
          subs = (TreeSet) s.subSet(326, true, 328, true );
          subs.add(329);// 过界 runtimeException
          System.out.println(s+" "+subs);
          
    }
  ```

  

## 5. Deque 

不可以存放Null， 因为Poll 可能返回null 表示为空 如果存放NULL 那么出现歧义

* 两套操作 可以是stack 也可以是queue
* **一定注意只要不是按着 stack的特有方法 就是按着queue的 先进先出**

### 5.1 API

* 有一系列精确操作 比如 addFirst offerFirst
* add remove get 类的 都是有Exception的  offer poll peek 这样的都是没有Exception的
* add remove offer poll 都是 **queue-based**
* push pop peek 都是 **stack-based**

#### 5.1.1 remove

* 不带参数 移除头部 
* 带参数移除object  没有**index**
* 如果没有会 RuntimeException

## 6. TreeMap

* **key 不可以为Null** -- 空指针
* 如果传入不可排序 （no-compatible） Cast Exception

### 6.1 API

#### 6.1.1 headMap & tailMap

返回比当前数值小的其他值在Map中 默认 head是不包含  tail是包含本值的

并且对于原来的map会进行修改

```java
     System.out.println(map.headMap(25)); // 不包括25
        System.out.println(map.tailMap(25));// 包括25
```



## 7. HashSet

* 可以存在一个null
* 确保唯一性 HashCode + equals 

## 8. ConcurrentHashMap 

* Key 和 Value 都不可以为 Null -- 空指针

## 9. LinkedHashSet

* 可以用一个null 
* 严格按着插入顺序排放
* 确保唯一性 HashCode + equals 

## 10. BlockingDeque

* 和Deque不同 这里的 需要添加等待时间的方法 都需要处理异常 

## 11. ConcurrentSkipListSet

* 允许在迭代的时候修改

## 5. 注意

### 5.1  泛型问题

```java
List<? extends Number> list = new ArrayList<>(); 
```

如果这样声明， 那么这个List是不可以 insert数据的 因为集合的泛型是确定的

### 5.2 sort可以传入空值 那么就会按着自然排序来排序

### 5.3 sort 如果出现了完全一样无法排序 就按着插入顺序排

### 5.4 HashSet 的查重会先查找HashCode 然后再比对equals 如果没有重写HashCode 那理论上来说大部分应该都不会重复

### 5.5 listIterator 带参数表示七点 

```java
ListIterator<String> iter = list.listIterator(2); // 参数代表起点 start 
```



# Path

---

* 大多数API 处理toString 都返回一个新的Path

* 可以直接遍历  实现了 iteratable

  ```java
  path.forEach(System.out::println); 
  
  for(int i = 0; i < path.getNameCount(); i++) {
      System.out.println(path.getName(i));
  }
  
  ```

for(Path p : path) {
      System.out.println(p);
  }


  Iterator<Path> iterator = path.iterator();
  while(iterator.hasNext()) {
      System.out.println(iterator.next());
  }
  ```
  
  

## 1 APIs

### 1.1  拼接例子

root 代表磁盘 subpath 代表后面的以\\\分割的

​```java
Path path1 = Paths.get("F:\\Whizlabs\\java\\NIO");
Path path2 = Paths.get("c:\\output");
Path path  = Paths.get(path2.getRoot().toString(), path1.subpath(0,3).toString());
System.out.print(path.toString());
  ```

### 1.2 **resolve**

instance method

如果传入的是绝对路径 那么直接返回传入值 

如果传入的是空 返回调用者

如果传入的是别的 那么进行合并

```java
Path path1 = Paths.get("F:\\Whizlabs\\java\\NIO\\myfiles");
Path path2 = Paths.get("myfiles\\myfile.txt");
Path path  = path2.resolve(path1);
System.out.print(path.toString());
```

```java
public class Test {
    public static void main(String[] args) {
        Path path1 = Paths.get("F:", "Other", "Logs");
        Path path2 = Paths.get("..", "..", "Shortcut", "Child.lnk", "Message.txt");
        Path path3 = path1.resolve(path2).normalize();
        Path path4 = path1.resolveSibling(path2).normalize();
        System.out.println(path3.equals(path4));
        // 如果在root  那么 .. 会保持在root 不会报错
        // 而且这道题用的是path 根本就不考虑file是否真实存在 一切都是基于字符串的操作
    }
}
/**
F:.
├───Parent
│   └───Child
│           Message.txt
│
├───Shortcut
│       Child.lnk
│
└───Other
    └───Logs
**/
```



### 1.3 getRoot

如果用非绝对路径创建 返回 null

### 1.4 toAbsolutePath 

无论如何都会返回绝对路径

### 1.5 getNameCount

用什么创建 就返回多长, driver 和 root不算在内 从0开始index

### 1.6 endsWith

参数可以是 Path 也可以是String **参数应该是全称** 比如 c.txt 这样的 **切记！** 因为两个都是Object 且只有一个参数 所以不能传Null 不然无法判断选用哪个  **编译错误**

判断此路径是否**以参数路径结尾** 



### 1.8 getNameCount & getFileName & getName

只有getName可以有参数 从0开始**都不算root**

```java
  public static void main(String[] args) throws IOException {
        Path path = Paths.get("F:\\A\\B\\C\\");
        System.out.printf("%d, %s, %s", path.getNameCount(), path.getFileName(),path.getName(0)); // 3, C, A
    }
```

### 1.9 toFile

格式转换

### 1.10 toRealPath

需要处理异常

### 1.11 normalize

* 不会把相对路径处理为绝对路径

```java
final Path path = Paths.get(".").normalize();  // h1 
int count = 0; for(int i=0; i<path.getNameCount(); ++i) {   
  count++; 
} 
System.out.println(count);
// 当前路径 /seals/harp/food
// 处理完还是 . 所以答案为1
```



# 断言 && Exception

---

## 1. 断言

1. **assert 条件;**

2. **assert 条件 : 表达式;**

3. 第二中写法 第二个表达式会变成Message

   ```java
   public class Test {
       private static void checkStatus() {
           assert 1 == 2 : 2 == 2;
       }
    
       public static void main(String[] args) {
           try {
               checkStatus();
           } catch (AssertionError ae) {
               System.out.println(ae.getMessage()); // true
           }
       }
   }
   ```

   

4. 不可以写lambda or 方法引用

```java
assert 1 == 2 : new RuntimeException("aaa"); // 也可以这要抛异常
```

 5. 只有抛出异常才有下面的操作

    ```java
    public class Test {
        private static void checkStatus() {
            assert 1 == 2 : 2 == 2;
        }
     
        public static void main(String[] args) {
            try {
                checkStatus();
            } catch (AssertionError ae) { // 需要抛出异常才可以捕获到 否则是Null
                System.out.println(ae.getCause());
            }
        }
    }
    ```

    

### 1.1 enable 和 disable

```bash
java -ea -da:Lab Whiz # -da: 表示disable的 

# 细颗粒
java -ea:bad... -da:good... Main # 表示 bad下的所有class激活 good下的不激活
java -ea:bad... Main # 或者直接不写 不激活的 因为默认不激活
```

### 1.2 断言不应该使用在 public的方法中用来测试输入数据 因为public属于公开的 而非只是程序员调用的  那么应该保证任何情况下都是合理的，应该抛出运行时异常。

## 2. Exception

### 2.1 在声明多个Excepion在catch的里面的时候 不可以有继承关系的 Exception 

### 2.2 throw null 会报空指针

### 2.3 Exception 可以创建方法  但是在多重声明的时候  变量获取的是公共基类

```java
class MyException extends Exception {
    public void log() {
        System.out.println("Logging MyException");
    }
}

class YourException extends Exception {
    public void log() {
        System.out.println("Logging YourException");
    }
}

public class Test {
    public static void main(String[] args) {
        try {
            throw new MyException();
        } catch(MyException | YourException ex){
            ex.log(); // EX 是多重声明的公共的基类 也就是 runtime 但是创建的时候不会报错
        }
    }
}
```

### 2.4 java不允许捕获绝对不可能出现的checked异常--编译错误

```java
public class Test {
    public static void main(String[] args) {
        try {
            System.out.println(1);
        } catch (NullPointerException ex) {  // unchecked 所以无所谓
            System.out.println("ONE");
        } catch (FileNotFoundException ex) { // 编译错误 因为绝对不可能抛出
            System.out.println("TWO");
        }
        System.out.println("THREE");
    }
}
```

### 2.5 catch 过后的Exception变量 可以赋值成声明更小的Exception

```java
private static void m() throws SQLException {
         try {
             throw new SQLException();
         } catch (RuntimeException e) {
             e = new ArithmeticException(); //反过来报错
             throw e; //这里要注意 如果抛出需要考虑捕获或者throws
         }
     }
```

### 2.6 方法异常声明

对于override的带有异常的方法 重写的时候可以选择

* 不抛出异常

* 抛出一样的异常

* 抛出异常的子类

* 无法抛出更大Checked异常

* **Unchecked **异常不受限制

  ```java
  class Base {
      public void m1() throws NullPointerException {
          System.out.println("Base: m1()");
      }
  }
  
  class Derived extends Base {
      public void m1() throws RuntimeException {
          System.out.println("Derived: m1()");
      }
  }// 编译通过 因为都是 RuntimeException  所以不受限制
  ```

  

### 2.7 修改被捕获的异常

```java
private static void m() throws SQLException {
        try {
            throw new SQLException();
        } catch (Exception e) {
            e = new SQLException(); 
            throw e; // 这里会报错 因为e可能是任何Exception  所以必须处理 Exception 而不是SQLException
        }
    }
```

报错的原因是一旦e被赋值过后 **那么再抛出异常必须抛出Exception**, 但是如果没有修改直接抛出 **编译器会识别真实的Exception**是什么 

```java
public class Test {
    private static void m() throws SQLException {
        try {
            throw new SQLException();
        } catch (Exception e) {
            throw e; // 看似抛出Exception 上面会编译错误 实际上 编译器知道抛出的真实的是SQL 所以可以 但是如果修改后就不行 
        }
    }
 
    public static void main(String[] args) {
        try {
            m();
        } catch(SQLException e) {
            System.out.println("Caught Successfully.");
        }
    }
}
```



### 2.8 在catch中多重声明的变量e是 final的 不可以修改

```java
class MyException extends RuntimeException {}
 
class YourException extends RuntimeException {}
 
public class Test {
    public static void main(String[] args) {
        try {
            throw new YourException();
        } catch(MyException | YourException e){
            e = null; //编译错误  e是final
        }
    }
}
```

### 2.9 普通的 try-catch-finally 如果finally报错 那么只优先抛出finally, catch里面的会丢失 而不是被 suppressed

```java
public class Test {
    private static void m() {
        try {
            throw new RuntimeException();
        } catch(RuntimeException ex) {
            throw new MyException1();
        } finally {
            throw new MyException2();//抛出这里 
        }
    }
 
    public static void main(String[] args) {
        try {
            m();
        } catch(MyException1 e) {
            System.out.println("MyException1");
        } catch(MyException2 e) {
            System.out.println("MyException2"); //捕获 
        } catch (RuntimeException e) {
            System.out.println("RuntimeException");
        }
    }
}
```

### 2.10 普通的try-catch-finally 如果catch 报错 那么先执行finallly 然后向上抛出

```java
public class Test {
    public static void main(String[] args) {
        try {
            try {
                System.out.println(1/0); 
            } catch(ArithmeticException e) {
                System.out.println("Inner"); 
                throw e;
            } finally {
                System.out.println("Finally 1");
            }
        } catch(ArithmeticException e) {
            System.out.println("Outer");
        }
    }
} // Inner Finally1 Outer
```

### 2.11 出现多个接口 抛出不同的Exception 重写需要写同时满足的



```java
interface I1
{
   void m1() throws java.io.IOException;
}
interface I2
{
   void m1() throws java.sql.SQLException;
} // 这里只可以重写为不抛出
```



# 多线程

## Api

### 1. Shutdown & shutdownNow

前者使得线程池进入停止状态 拒接接受新任务 但是会处理后续任务

后者让线程池直接终止，并且返回未完成任务

### 2. setAutoCommit

是作用在 connection的 不是statement的

设置true的时候会自动commit一次

## 1. forkjoin

### 1.1 注意

#### 1.1.1  It is more efficient to call fork twice for two subprograms and then call join twice. 是错的，对于少数据创建线程开销太大

#### 1.1.2 In some cases fork-join computation might run slower. 是错的，对于少数据创建线程开销太大

#### 1.1.3 join 会计算结果并且等待，所以一定要保证 compute或者 fork在join之前（所有的fork或者compute）

### 1.2 ForkJoinPool



## 2. 线程池

标准使用

```java
public static void main(String []args)throws Exception{
                   final ExecutorService pool = Executors.newFixedThreadPool(2);
                   List<Callable<Integer>> cal = new ArrayList<>();
                  
                   cal.add(new Task2());
                   cal.add(new Task1());
                                  
                   System.out.print(pool.invokeAny(cal));
   
                   pool.shutdownNow();
       }
   }
```

也可以这么用

```java
public class Test {
    private static void print() {
        System.out.println("PRINT");
    }
 
    private static Integer get() {
        return 10;
    }
 
    public static void main(String [] args) throws InterruptedException, 
                                                         ExecutionException {
        ExecutorService es = Executors.newFixedThreadPool(10);
        Future<?> future1 = es.submit(Test::print); // 直接lambda表达式 这个匹配到 runnable
        Future<?> future2 = es.submit(Test::get); // callable
        System.out.println(future1.get());
        System.out.println(future2.get());
        es.shutdown();
    }
}
```

### 2.4 

```java
ExecutorService service = Executors.newSingleThreadScheduledExecutor(); service.scheduleWithFixedDelay(() -> {   // 这里报错 因为声明为ExecutorService 并没有这个方法 
  System.out.println("Open Zoo");     
  return null;   // 没有返回值
}, 0, 1, TimeUnit.MINUTES); 
Future<?> result = service.submit(() -> System.out.println("Wake Staff"));
// submit因为不知道接受的是Runable还是Callable 所以默认返回为Future<?>
System.out.println(result.get()); 
```



### 2.1 **invokeAny**

返回其中任意一个结果 不是**future**

### 2.2 execute & submit

* execute 定义在父类的 Executor 里面 只可以接受 Runnable
* submit 定义在 ExecutorService里面 可以接受 Runable 和 Callable ps: Thread 继承了 runable

### 2.3 shutdown

**使用线程池一定要注意关闭**！ 否则会资源浪费

强制关闭， 并不会等待未完成的线程

## 3 CyclicBarrier

相当于让所有的线程到同一起跑线再开始行动 用法有点类似锁

```java
public CyclicBarrier(int parties)
public CyclicBarrier(int parties, Runnable barrierAction) // 放行一起执行的一个线程 同时第一个参数表示必须达到这么多个线程同时处于 barrier的状态 才可以放行
```

### 3.1  await 

让当前线程处于barrier状态

```java
public int await()
public int await(long timeout, TimeUnit unit) // 过了时间则放行
```

## 4 Lock

#### 4.1 lock 没有返回值 tryLock 返回boolean 

## 注意点

### 1. 用同一个对象创建的线程 属性共有

```java
package com.udayan.ocp;
 
import java.util.concurrent.*;
 
class MyCallable implements Callable<Integer> {
    private Integer i;
 
    public MyCallable(Integer i) {
        this.i = i;
    }
 
    public Integer call() throws Exception {
        return --i;
    }
}
 
public class Test {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService es = Executors.newSingleThreadExecutor();
        MyCallable callable = new MyCallable(100);
        System.out.println(es.submit(callable).get());
        System.out.println(es.submit(callable).get());
        es.shutdown();
    }
}

// 99 98
```

### 2. **AtomicInteger**

* **一定**线程安全

原子化 打印顺序可能不同 但是一定不会出现 重复数

```java
class Counter implements Runnable {
    private static AtomicInteger ai = new AtomicInteger(3);
 
    public void run() {
        System.out.print(ai.getAndDecrement());
    }
}
 
public class Test {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Counter());
        Thread t2 = new Thread(new Counter());
        Thread t3 = new Thread(new Counter());
        Thread[] threads = {t1, t2, t3};
        for(Thread thread : threads) {
            thread.start();
        }
    }
}
```

### compareAndSet(old,new) 比对 如果当前值和old相同 那么就set

### 3. Future

#### 3.1 get 方法需要抛出异常

```java
class MyCallable implements Callable<Integer> {
    private Integer i;
 
    public MyCallable(Integer i) {
        this.i = i;
    }
 
    public Integer call() throws Exception { // 这里抛出异常
        return --i;
    }
}
 
public class Test {
    public static void main(String[] args) { 
  //public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService es = Executors.newSingleThreadExecutor();
        MyCallable callable = new MyCallable(1);
        System.out.println(es.submit(callable).get()); // 编译错误 需要处理异常 
        es.shutdown();
    }
}
```



### 4. Thead 只能以Runnable 为参数 不能以 Callable

### 5. Callable 和 Runnable 都没有参数

### 6. 同步方法的锁

* 非静态方法的锁 是实例-当前对象
* 静态方法的锁 是类

### 7. Sleep 不会释放锁

# System

---



## 1. API

### 1.1 System.out.printf 和 System.out.format 

两个用法一样

```java
System.out.printf("A%nB%nC"); //换行符 注意 不是println
```

### 1.2 符号类

```java
 System.out.printf("%2$d + %1$d", 10, 20); // 2$ 表示第二个参数
```

### 1.3 path.separator & file.separator

前者是 :(windows) ;(linux)

后者是用来区分路径之间的子路径的

## 2. Console

属于IO 包的 final class

只可以操作Character data

创建

```java
System.console();
```

有可能无法返回实例，因为可能会没有可用的console 根据系统而定 **所有有可能会** **空指针 如果直接调用方法**

* d 10进制
* x 16 进制
* < 前一个参数

```java
public class Test {
    public static void main(String[] args) {
        Console console = System.console();
        if(console != null) {
            console.format("%d %<x", 10);
        }
    }
}// 10 a
```

```java
 public static void main(String[] args) {
        Console console = System.console();
        if(console != null) {
            console.format("%d %x", 10); // 报错 注意参数数目 runTime exception
        }
    }
```



### 2.1 读取

```java
readline(); // String
readPassword(); // char[]
reader(); // 返回一个 reader() 
```

### 2.3 写

```java
write() // return Print Wirter 和reader一样属于 内部自己创建的 并且和当前的console相关联的 也就是说读也是这里读 写也是这里写
```

### 2.4 打印

```java
printf();
format(); // 两者一样
writer().println; //或者也可以用writer方法获得一个PrintWriter
```



# 基础

---

## 1. 接口

可以有 publi static final variables， default or static methods

访问权限

```java
interface study{
     String date = "1.1"; // 默认添加 static final
     void test1();
     static void test2(){}
     default void test3(){}

}

class Student implements study{

    @Override
    public void test1() {
        
    }
}

class Test4{
    public static void main(String[] args) {
        Student student = new Student();
        System.out.println(student.date);// 可以通过实例访问无法修改
        
        student.test3(); // default method可以直接访问
        study.test2(); // static method只能通过接口名访问
    }
}


```



* **可以继承多个接口 不可以继承类** 会获得所有的 **abstract method**

* 可以声明重写一个父类的default方法来作为抽象方法

* 不可以被final 修饰 不可以有构造器或者被初始化

* 

  ```java
  interface Printer {
      default void print() {
          System.out.println("Printer");
      }
  }
   
  @FunctionalInterface
  interface ThreeDPrinter extends Printer {
      @Override
      void print(); // 正确写法
  }
  ```

* 不可以用default method 去覆盖object里面的方法

* 实现类**不继承**static方法

  ```java
  interface Walk {
      public static int getSpeed() {
          return 5;
      }
  }
  
  interface Run {
      public static int getSpeed() {
          return 10;
      }
  }
  
   class Animal implements Walk,Run{ // 这里不会报错 因为不继承 所以不冲突
  
      public static void main(String args[]){
          Animal an = new Animal();
          System.out.println(an.getSpeed()); // 这里报错 因为不继承 所以找不到
      }
  }
  ```

  

### 1.1 不可以实现多个接口有相同名字以及signature的 default method且自己没有实现（如果自己有一个相同的实现则可以通过编译）

### 1.3 可以在内部声明 类 和接口 会被默认添加 public  static

```java
interface I1 {
    void m1();
 
    interface I2 { 
        void m2();
    }
 
    abstract class A1 { 
        public abstract void m3();
    }
 
    class A2 { 
        public void m4() {
            System.out.println(4);
        }
    }
} // 可以编译通过
```

### 1.4 如果出现了同名的方法被继承 那么class优先级最高 其次是interface 如果 是两个interface 有继承关系 那么久子类更高，如果还不能分别 必须显示的调用

```java
public class C implements B, A {
    void hello() {
        B.super.hello();
    }
```

### 1.5  多个接口有重名静态方法

### 1.6 如果一个接口继承了另一个接口

编译会报错，但是可以通过重写该方法来解决

```java
public void print() {
    System.out.println("Printer");
}// 解决编译错误

// 调用实现接口里的静态方法
public void print() {
    System.out.println("Printer");
    Printer1.super.print();
    Printer2.super.print();
}
```



### 1.7 如果继承的接口里面出现了abstract method 同时又有继承的接口或者自己有匹配的默认方法那么会自动配对 不再需要在子接口里面实现  但是静态方法不可以！！

```java
interface StringConsumer extends Consumer<String> { // 已经不再是接口式函数了
    @Override
    public default void accept(String s) {
        System.out.println(s.toUpperCase());
    }
    
   //@Override
   // public static void accept(String s) {
   //     System.out.println(s.toUpperCase());
   // }
    // static不可以
}

class Test implements StringConsumer{ // 这里不会报错 因为 consumer里面的 accept已经被默认方法实现了

    public static void main(String[] args) {

    }
}
```

```java
interface StringConsumer extends Consumer<String> {
    @Override
    public default void accept(String s) { // 第一次重写
        System.out.println(s.toUpperCase());
    }
}
 
public class Test {
    public static void main(String[] args) {
        StringConsumer consumer = new StringConsumer() {
            @Override
            public void accept(String s) { // 这里相当于再一次重写
                System.out.println(s.toLowerCase());
            }
        };
        List<String> list = Arrays.asList("Dr", "Mr", "Miss", "Mrs");
        list.forEach(consumer);
    }
}
```

### 1.8 可以声明 Object中的方法 并且不需要重写

```java
@FunctionalInterface
interface I4 {
    void print();
    boolean equals(Object obj); //这个不需要重写 因为默认继承了Object 所以自动匹配了
    //  boolean equals(Number obj); 这个就会报错 因为无法匹配到
}// 编译通
```

### 1.9 default方法可以调用 abstract 方法

```java
interface Boiler{
    public void boil();
    public static void shutdown(){
        System.out.println("shutting down");
    }
}
interface Vaporizer extends Boiler{  
    public default void vaporize(){
        boil(); // 这里可以调用abstract方法 甚至来自于父类
        System.out.println("Vaporized!");
    }
}
  
```





## 2. IO

* 关闭应该写在finally 里面 否则还是会有资源泄露的可能
* 大多数在关闭后都不可以再调用 -- Runtime Exception  **目前已知 除了 打印流**
* **除了 4个抽象类 InputStream OutPutStream Reader Writer 和 FileInputStream FileOutStream FileReader FileWriter其他的所有类的都是Hight Level的**

### 2.1  Writer

基类 基于char

#### 2.1.1 API

* flush
* close
* append
* write

### 2.2 Reader

基类 基于char

#### 2.2.1 API

* read 不带参数 读取一个byte 带参数（char【】） 返回读取数组的长度
* skip 从1开始
* ready
* mark 标记当前，（**后面有待参考**）可以带参数 参数表示如果在超过参数数字后仍然没有使用reset 那么mark就会失效 比如 mark（4） 表示4个以内如果不调用reset那么这个mark就失效, **一直到没有过期前可以被重复调用**
* reset 回到最近的一个mark
* close
* markSupported 并不是所有情况下都支持mark 在调用mark前一般先使用

### 2.3 InputStream

#### 2.3.1 API

* read
* skip
* available
* close
* mark
* reset



### 2.4 OutputStream

* write  
* flush
* close

### 2.5 处理File的4个

##### 2.5.1 FileWriter & File & PrintWrite

这三个除了File在创建的时候不会真的创建文件 其他都会 

FileWriter 只能创建file  如果有dir需要创建 那么 -IOException

```java
 public static void main(String[] args) throws IOException {
        File f1 = new File("F:\\f1.txt");
        FileWriter fw = new FileWriter("F:\\dir\\f2.txt"); // 报错 因为无法创建dir
        PrintWriter pw = new PrintWriter("F:\\f3.txt");
    }
```



### 2.5 **BufferedWriter**

* 关闭后不可以再调用 -- Runtime Exception

### 2.6 **BufferedReader** 

#### 2.6.1 read 源自Reader

无参数返回一个单个char 但是返回值是int类型（其实也可以使char）

有参数的话 参数是一个buffer（char [ ]）返回读取多少个

#### 2.6.2 readLine

返回String 读取一行

#### 2.6.3 lines

读取所有行 返回值是 Stream<String>

#### 2.6.4 close

调用close会自动调用flush先

### 2.7 Scanner

* 可以被关闭多次 并且不需要处理异常

* 接受为String

  ```java
  Scanner scan = new Scanner(System.in)
  String s = scan.nextLine(); // 这里是String
  System.out.println(s);
  scan.close();
  ```

### 2.8 ObjectOutputStream

#### 2.8.1 writeObject

* 参数只能是实现了 Serializable 的Object ---Runtime Exception
* Serializa 的时候 t**ransient 和 static 不会被保存** 当de-Serialize的时候会将transient的值初始化为默认值，static的初始化为当前存在内存中的值
* 1.4 反序列化的时候 如果父类没有实现序列化接口 那么需要**调用父类的无参构造器** 如果没有 --Runtime Exception

### 2.9 PrintWriter & PrintWriter

* High-level stream 可以直接操作file

* System.out  和System.err 实际上是他们的实例

  ```java
  public static void main(String[] args) throws IOException {
          System.out.println(System.out.getClass());
          System.out.println(System.in.getClass());
          System.out.println(System.err.getClass());
          /**
           * class java.io.PrintStream
           * class java.io.BufferedInputStream
           * class java.io.PrintStream
           */
      }
  ```

  

创建的时候会有IOException  **但是其他public 方法不抛出异常**

```java
public class Test {
    public static void main(String[] args) {
        try(PrintWriter bw = new PrintWriter("F:\\test.txt"))
        {
            bw.close();
            bw.write(1); // 不会报错 正常结束
        } catch(IOException e) {
            System.out.println("IOException");
        }
    }
}
```

#### 2.9.1 PrintStream

* 继承了OutPutStream



#### 2.9.2 PrintWriter

* 继承了 Writer
* 没有与之对应的流
* write. Print. append的结果都一样
* **printf** 返回PrintWeiter 所以可以进行链式操作

#### 2.9.3 共同

* 两者都是输出流 并且都可以带参数来指定打印的地方（可以是file 也可以是其他同类流）
* 大多数情况可以混用 但是writer可以只能处理字符



## 3. 抽象类

可以用构造方法

## 4. super

如果不写 则会默认调用父类的*无参构造* （对于无参构造 如果没有配置构造器则会自动添加，但是！！如果配置了 则不会添加） 需要向上检查是否满足条件 如果找不到会编译时错误

## 5. resource try catch

* 资源既可以 是实现了 closeble 也可以是 Autoclosble

* 资源默认会添加final **不可以修改** 也就是说也不可以使用之前创建的变量 必须重新定义变量

* 不要求一定写 finally 和catch

* 如果close里面声明了异常就需要处理（有可能不抛出）

* 会在try**的括号结束前关闭连接**

* 如果资源为空 那么close将不会执行 也不会有编译错误

  ```java
  try(PrintWriter writer = null) {
              System.out.println("HELLO");
  }
  ```

* **不可以不初始化** --编译错误

  ```java
   public static void main(String[] args) {
          try(PrintWriter writer;) { // 编译错误
              writer = new PrintWriter(System.out);
              writer.println("HELLO");
          }
      }
  ```

* 如果finally 报错 那么try catch中的错误会先被忽略 抛出finally的错误

```java
public class Test {
    private static void m() {
        try {
            throw new RuntimeException();
        } catch(RuntimeException ex) {
            throw new MyException1(); // 这里因为下面的finally抛出而被忽略了
        } finally {
            throw new MyException2();
        }
    }
 
    public static void main(String[] args) {
        try {
            m();
        } catch(MyException1 e) {
            System.out.println("MyException1");
        } catch(MyException2 e) {
            System.out.println("MyException2"); // 这里捕获
        } catch (RuntimeException e) {
            System.out.println("RuntimeException");
        }
    }
}
```

### 5.1 Suppressed问题

try-catch-resource 如果要抛出多个异常 那么通常在 close里面写的异常会被压制 

### 5.2 如果声明为AutoClosable

那么一定要处理异常 因为重写的时候可以不抛出 但是原版是有异常的

## 6. properties 类

### 6.1 创建

```java
// 创建 Properties 类
Properties prop = new Properties ();
// 创建 inputStream
FileInputStream fis = new FileInputStream ("F:\\Message.properties");
// load
prop.load(fis);
```

### 6.2 获取键值对

```java
prop.getProperty("key3"); // 正常使用
prop.getProperty("key2", "Good Day!")); // 带一个默认值 没找到就返回这个
```

## 7. Object

### 7.1 不可重写

getClass(), notify(), notifyAll() and overloaded wait methods are final in Object class and hence cannot be overridden.

### 7.2 toString

注意书写格式  默认的是 Object参数！有的可能不是重写 是重载

```java
 public boolean equals(Player player) { //重载
        if(player != null && this.name.equals(player.name)
                && this.age == player.age) {
            return true;
        }
        return false;
}
```

### 7.3 例题

```java
package com.bohan.test;

import java.time.Duration;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.Period;
import java.util.*;
import java.util.concurrent.*;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.UnaryOperator;

/**
 * @Project : netty
 * @Author : Bohan Xiao
 * @Create : 2020-11-13
 **/

class Player {
    String name;
    int age;

    Player(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String toString() {
        return name + ", " + age;
    }

    public boolean equals(Player player) {
        if(player != null && this.name.equals(player.name)
                && this.age == player.age) {
            return true;
        }
        return false;
    }
}

public class Test {
    public static void main(String[] args) {
        Player p1 = new Player("Sachin", 44); 
        Player p2 = new Player("Sachin", 44);
        Object p3 = new Player("Sachin", 44);
        Object p4 = new Player("Sachin", 44);
        System.out.println(p1.equals(p2));
        System.out.println(p3.equals(p4));
        System.out.println(p3.equals((Object) p4));
    }
}
// True false false
```

```java
public class Resource {
    private String data = "DATA";
    String getData(){
        return data;
    }

    void setData(String data){
        this.data = data == null ? "" : data;
    }
    
  // 这里是重载不是重写 并且没有用public声明 所以如果在包外调用equals 会自动匹配到object中的 equals 但是如果在范围内 那么就会匹配更具体的
    boolean equals(Resource r){
        return (r != null && r.getData().equals(this.getData()));
    }
}
```



## 8. 泛型

* 静态方法不可以由泛型

* 异常不可以使用泛型

* 只有 extends 和supper 没有 implement

  ```java
  class Printer<T implements Cloneable> {} // 编译错误
  ```

* 初始化不可以用泛型

  ```java
   public Test() {
          obj = new T[100];
      }
   // 编译错误
  ```

  

### 8.1  多重声明 如果有class一定要写在最前面

```java
class Generic<T extends M & N & A> {} // 编译错误 因为A 是class 没有写在最前面
```

```java
package com.udayan.ocp;
 
class Animal {}
 
class Dog extends Animal {}
 
class Cat extends Animal {}
 
class A<T> {
    T t;
    void set(T t) {
        this.t = t;
    }
 
    T get() {
        return t;
    }
}
 
public class Test {
    public static <T> void print1(A<? extends Animal> obj) {
        obj.set(new Dog()); //Line 22  这里因为如果传入的A声明的是Cat，那么无法复制为cat 所以编译器不会通过 
        System.out.println(obj.get().getClass());
    }
 
    public static <T> void print2(A<? super Dog> obj) {
        obj.set(new Dog()); //Line 27  这里因为无论你声明的是什么 都必须是以Dog为起点 所以都可以转换 所以编译器可以通过
        System.out.println(obj.get().getClass());
    }
 
    public static void main(String[] args) {
        A<Dog> obj = new A<>();
        print1(obj); //Line 33
        print2(obj); //Line 34
    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        List<String> list1 = new ArrayList<>();
        list1.add("A");
        list1.add("B");
 
        List<? extends Object> list2 = list1;
        list2.remove("A"); //Line 13 // remove 可以是因为remove是非泛型方法
        list2.add("C"); //Line 14 编译报错 add不可以是因为add是使用泛型的method 可以添加null 
 
        System.out.println(list2);
        /**
        	对于这类问题 如果使用了 super(>=)那么就会默认使用最低的 也就是说比如 ? super Object 那么就会默认使用Object当做泛型 那么所有类型都可以被 add 使用 因为尽管我们不知道类型传进来的时候 但是他总是可以被视作Object.
        	如果使用 extend <= 那么编译器无法指定泛型 因为相当于只知道了最高层，但是子类之间是会出现无法转换的情况 所以编译器无法通过
        **/
        // 总结 extend 不能加  super 不能取
      
    }
}
```

### 8.2 要注意加减乘除只能针对数字类型 一般的object不可以，也就是说如果泛型没有标明 那么是不可以做计算的

### 8.3 super is used with wildcard (?) only.

### 8.4 泛型方法

不是使用了类的泛型就叫泛型方法 而是 编译的时候不知道参数类型 相当于另一个泛型

* 必须要在List<E>之前写 <E> 相当于告诉编译器 E不是一个类 **而是一个泛型变量 **
* 可以用static 修饰 
* 调用的时候可以在方法前写明泛型 也可以不写

```java
public <E> List<E> copyFormarrayToList(E[] arr){
         return null;
}

Foo<String, String> pair = Foo.<String>twice (“Hello World!”);
```

```java
public class Test {
    private <T extends Number> static void print(T t) { // 编译错误 必须正好写在返回值的前面
  //private static <T extends Number> void print(T t) { 正确写法
        System.out.println(t.intValue());
    }

    public static void main(String[] args) {
        print(new Double(5.5));
    }
}
```



### 8.5 集合泛型

主要看左边 

```java
        List list = new ArrayList<String>();
// 等于  List list = new ArrayList<>();
```

### 8.6 泛型变量可以随便写

```java
class Printer<Integer> {
    private Integer t;
    private Integer a;
    Printer(Integer t){
        this.t = t;
    }
} // 缺点是无法再使用 Integer 在内部
```

但是不可以和Object冲突

```java
class Printer<String> {
    private String t;

    Printer(String t){
        this.t = t;
    }

    public String toString() { // 这里编译器无法判断到底是重写 还是使用了泛型的
        return null;
    }
    // 可以这样写
    public java.lang.String toString() {
        return null;
    }
}

public class Test {
    public static void main(String[] args) {
        Printer<Integer> obj = new Printer<>(100);
        System.out.println(obj);
    }
}

```



### 8.7 泛型前后必须相同 不可以子类父类

```java
        Foo<Object, Object> percentage = new Foo<String, Integer>("Steve", 100);
// 报错
```

### 8.9 例题

```java
 public static void main(String[] args) {
        Animal [] animals = new Dog[2];
        animals[0] = new Animal(); // 这里Runtime Exption  因为尽管声明的是Animal （编译不报错） 但是实际上 Animal 不可以转换为Dog
        animals[1] = new Dog();
 
        animals[0].eat();
        animals[1].eat();
    }
```

### 8.10 协变 逆变不变

* 不变 不可以进行转换 泛型就是不变的 比如说 

  ```java
  List<Object> list = new ArrayList<String>(); // 编译错误 因为不可以转换
  ```

* 协变 extends  

* 逆变 super

* PECE  支持生产者Productor（返回值默认为？后声明的）-Extends Comsumer（参数默认为 ？ 后声明的 返回值默认为Object）-Super

```java
class Person{}
class Student extends Person{}


class Test{
    public static void main(String[] args) {
        /**
         *  协变
         */
        // 可以取不可以添加 限制消费者方法（参数泛型） 允许生产者方法（return 泛型--用底层Person做返回）
       List<? extends Person> list = new ArrayList<>();
       Person person = list.get(0);

        /**
         *  逆变
         */
        // 可以添加不可以取 限制生产者方法（return 泛型 因为什么都可能添加 不知道返回啥） 允许消费者类型（参数泛型 都当做Person）
        List<? super Person> list1 = new ArrayList<>();
        list1.add(new Student());
        list1.add(new Person());

        List<?> list2 = new ArrayList<>();
        Object o = list2.get(0);
			

    }
}
```

经典案例

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.Period;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.function.*;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

/**
 * @Project : test1
 * @Author : Bohan Xiao
 * @Create : 2020-11-09
 **/
class Animal {}

class Dog extends Animal {}

class Cat extends Animal {}

class A<T> {
    T t;
    void set(T t) {
        this.t = t;
    }

    T get() {
        return t;
    }
}

public class Test {
    public static <T> void print1(A<? extends Animal> obj) {
        obj.set(new Dog()); //Line 22 报错 因为用的是 extend 限制了消费者 支持生产者
        obj.get();// 不报错
        System.out.println(obj.get().getClass());
    }

    public static <T> void print2(A<? super Dog> obj) {
        obj.set(new Dog()); //Line 27 不报错 super 支持消费者  限制生产者
        Dog object = obj.get(); // 报错
        Object object1 = obj.get(); // 不报错 因为object满足下届要求 object比较特殊 属于所有类起点 一定成立
        obj.get();// 不会报错 默认返回Object类型 因为无法确定 所以默认用起点
        System.out.println(obj.get().getClass());
    }

    public static void main(String[] args) {
        A<Dog> obj = new A<>();
        print1(obj); //Line 33
        print2(obj); //Line 34
    }
}
```

经典例题2

```java
import java.util.function.Supplier;
 
class Document {
    void printAuthor() {
        System.out.println("Document-Author");
    }
}
 
class RFP extends Document {
    @Override
    void printAuthor() {
        System.out.println("RFP-Author");
    }
}
 
public class Test {
    public static void main(String[] args) {
        check(Document::new);
        check(RFP::new);
    }
    
    private static void check(______________ supplier) {
        supplier.get().printAuthor();
    }
}

/**
要求： 返回
Document-Author
RFP-Author

Supplier<Document>  可以
Supplier<? extends Document> 可以
Supplier<? super Document> // check(RFP::new); 编译无法通过 因为RFP不是Document的父类
Supplier<RFP> //  check(Document::new); 编译错误
Supplier<? extends RFP> //  check(Document::new); 编译错误
Supplier<? super RFP> // 编译不通过 因为都是 >= RFP的  但是由于PECE super的生产者默认返回都是Object 无法找到PrintAuthor方法
Supplier // 默认用Object初始化 无法找到PrintAuthor方法

**/
```

### 8.11 通配符总结

**边界只是针对初始化的时候 调用其他操作的时候是其他规则**

下面说的添加 则是消费者方法 获取则是提供者方法

#### 8.11.1 super 下边界

* 只可以当做Object取 
* 只可以添加边界类 或者边界的子类（这样 无论是什么是什么都可以放入 因为无论放入的是什么都满足是可能的集合类型的子类）

```java
 public static void main(String[] args) {
        
        List<? super IOException> exceptions = new ArrayList<Exception>();
        exceptions.add(new Exception()); // 编译错误
        exceptions.add(new IOException());
        exceptions.add(new FileNotFoundException());
       	Object object = exceptions.get(0);

    }
```

#### 8.11.2 extends

* 不可以添加任何
* 可以获取 默认返回边界，**但是边界往上一样可以**

```java
 public static void main(String[] args) {

        List<? extends IOException> exceptions = new ArrayList<IOException>();
        exceptions.add(new Exception()); // 编译错误
        Exception exception = exceptions.get(0);
    }
```

#### 8.11.3  ？

等同于 <? extends Object>

* 不可以添加
* 可以以Object 来取

## 散碎

### 1.1  == 不可以用来对比Siblings （共同继承一个类）

### 1.2 witch 只接受常数为case 不可以枚举

### 1.3 匿名类或者接口里面声明的无法被外部调用 因为他相当于是子类  声明却被声明成为父类 所以只能调用父类里面的方法

```java
 public static void main(String[] args) {
        Message msg = new Message() {
            public void PrintMessage() {
                System.out.println("HELLO!");
            }
        };
        msg.PrintMessage(); // 编译错误 因为Message里面并没有这个方法，但是如果重写是可以的
    }
```

### 1.4 反序列化的时候 如果父类没有实现序列化接口 那么需要调用父类的无参构造器来初始化父类中的继承属性 如果没有 --Runtime Exception 如果没有初始化继承的属性 那么去默认值

### 1.5 在写匿名类的时候 要注意 access modifier 

```java
class Point {
    private int x;
    private int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public String toString() {
        return "Point(" + x + ", " + y + ")";
    }
}
 
public class TestPoint {
    public static void main(String [] args) {
        List<Point> points = new ArrayList<>();
        points.add(new Point(4, 5));
        points.add(new Point(6, 7));
        points.add(new Point(2, 2));
        
        Collections.sort(points, new Comparator<Point>() {
            @Override
            public int compare(Point o1, Point o2) {
                return o1.x - o2.x; // 报错 因为没有权限 是private 不可以在外部类（TestPoint）调用
            }
        });
    }
}
```

### 1.6 匿名内部类的创建需要考虑构造器的调用

```java
enum ShapeType {
    CIRCLE, SQUARE, RECTANGLE;
}

abstract class Shape {
    private ShapeType type = ShapeType.SQUARE; //default ShapeType

    Shape(ShapeType type) {
        this.type = type;
    }

    public ShapeType getType() {
        return type;
    }

    abstract void draw();
}

public class Test {
    public static void main(String[] args) {
        Shape shape = new Shape() { // 这里会报错 因为没有空参构造器
            @Override
            void draw() {
                System.out.println("Drawing a " + getType());
            }
        };
        shape.draw();
    }
}
```

### 1.7 switch 传入 null 会空指针

### 1.8 协变返回类型 

子类覆盖父类方法，返回类型可以是，父类返回类型的子类

```java
public class Task extends _______________ {
    @Override
    protected Long compute() {
        return null;
    }
}
// RecursiveTask<Object>
// RecursiveTask
// RecursiveTask<Long>
// RecursiveTask<Number>
// 都可以 只要是返回的是声明的子类
```

### 1.9 Switch case 只接受常数

```java
public class Test {
    enum TrafficLight {
        RED, YELLOW, GREEN;
    }
    
    public static void main(String[] args) {
        TrafficLight tl = TrafficLight.valueOf(args[1]);
        switch(tl) {
            case TrafficLight.RED: // 三个CASE 编译错误 正确写法应该是RED
                System.out.println("STOP");
                break;
            case TrafficLight.YELLOW:
                System.out.println("SLOW");
                break;
            case TrafficLight.GREEN:
                System.out.println("GO");
                break;
        }
    }
}
```

### 1.10 Optional.of(null)  --- 空指针

### 1.11 重载不能只改变return类型 需要改变的是参数

```java
public Object toString() { // 报错 没有这种改法
     return name + ", " + age;
}

public Object toString(String a) { // 正确
      return name + ", " + age;
}
```

### 1.12 静态方法不可以被重写！！

### 1.13 和Stream 不同 optional 和自己的 primetive 类之间没有继承关系 optional类都是final的

### 1.14 function类型和stream & optional不同 都是apply 并且不继承Function

```java
@FunctionalInterface
public interface LongFunction<R> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    R apply(long value);
}
```

### 1.15 函数式接口一样支持向上转型

```java
 class Person1{
    static Integer age;

   static public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

     public static void main(String [] args) {
         DoubleSupplier a = Person1::getAge;
     }
}
```

### 1.11 Optional 如果调用了 orElse打印 那么就是真实值 而不是Optional

```java
public static void main(String[] args) {
        Optional<String> stu = Optional.of("stu");
        System.out.println(stu.orElse("aaa")); // stu
    }
```

### 1.12 int 可以 和 char 转换  但是 int 不可以和 Character 转换

```java
UnaryOperator<Character> operator = c -> c + 1; // 编译错误
UnaryOperator<Character> operator = c -> (char)c + 1; //  但是可以这样 先手动转换为char 因为 int可以转换为char 然后会默认转型到Character
```

### 1.14 String 的 compare默认是重写是 如果都一样 那么会比较长度

### 1.15 parallelSetAll

```java
public static void main(String[] args){
         String[] list = {"1","2","3"};
         Arrays.parallelSetAll(list,x->Integer.toString(x)+list[x]); // x 是序号
          System.out.println(list[0]);        
}
```

### 1.16千万要留意 Integer::toString 

### 1.17 listIterator 不同于普通迭代器 是双向的

创建的时候如果添加了参数 那么就是从第几个开始（可以after last）

```java
public static void main(String[] args) {
    List<String> list = Arrays.asList("T", "S", "R", "I", "F");
    ListIterator<String> iter = list.listIterator(5);// 一共只有五个元素（0-4） 指针可以指到after 第五个 = 5 
    while(iter.hasPrevious()) {
        System.out.print(iter.previous());
    }
}
```

### 1.18 抽象类里完全不可以有private 不仅仅是在类声明

```java
abstract class Animal {
    public static void vaccinate() {
        System.out.println("Vaccinating...");
    }
 
    private abstract void treating(); // 这里也会编译错误
}
 
public class Test {
    public static void main(String[] args) {
        Animal.vaccinate();
    }
}
```

### 1.19 private 针对的是class 而不是object

```java
class A
{
  private int i;
  public void modifyOther(A a1)
  {
    a1.i = 20;  // 依然可以访问 因为实际上还是在A类内部
  }
}
```

### 1.20 HashCode 的设计思想

* 其中只1是 如果equals相等 那么 Hashcode应该也要相等

  ```java
  import java.util.*;
  public class Info
  {
     String s1, s2, s3;
     public Info(String a, String b, String c)
     {
        s1=a; s2=b; s3=c;
     }
     public boolean equals(Object obj)
     {
        if(! (obj instanceof Info) ) return false;
        Info i = (Info) obj;
        return (s1+s2+s3).equals(i.s1+i.s2+i.s3);
     }
     public int hashCode()
     {
        return s1.hashCode();
     }
     public static void main(String[] args)
     {
        HashMap map = new HashMap();
        Info i1 = new Info("aaa", "aaa", "aaa");
        Info i2 = new Info("aaa", "bbb", "ccc");
        map.put(i1, "hello"); //1
        map.put(i2, "world"); //2
     }
  }// 这里 hashcode 和equals的关联太少 这样可能存在equals相等但是hashcode不等的情况
  ```

### 1.21 如果class没有constructor 那么他的默认构造器和他自己的access modifier 相同

### 1.22 序列化的 序列化ID的作用就是如果在反序列化的过程前 class有所改变 那么可以使用序列化ID进行匹配 如果相同 那么新添加的属性则会被赋予java默认 否则会报错

### 1.23 instanceof

instanceof 需要会编译错误 如果某个类完全不可能来自这里 比如说

```java
s instanceof java.util.Date // 这里如果s 是 声明的String 那么会编译错误
  // 因为无论如何转换 String也无法转换为Date 类似catch Exception
```



# 例题

```java
public class A
{
      protected int i = 10;
      public int getI() { return i; }
}

//in file B.java
package p2;
import p1.*;
public class B extends p1.A
{
   public void process(A a)
   {
      a.i = a.i*2;
   }
   public static void main(String[] args)
   {
      A a = new B();
      B b = new B();
      b.process(a);
      System.out.println( a.getI() ); // 编译错误
    
   }
}
/**
	题目太特么阴了
	重点 i 是 protected 包外只有继承的子类可以访问
	a被声明为A 那么就不是包外继承的子类了 所以无法访问 如果声明为B就享受这个权限了那么就可以访问
**/
```

```java

Enthuware Mobile Test Studio
 
Home
Certifications
Test
Review
Progress
Notes
 
Objective-wise Tests - 01 - Java Class Design : 2020-11-20 13:42
Q 14 of 15
Mark	
Java Class Design - Inheritance and Polymorphism
enthuware.ocpjp.v8.2.1334
Consider the following code:
class A
{
   A() {  print();   }
   private void print() { System.out.println("A"); }
}
class B extends A
{
   int i =   Math.round(3.5f);
   public static void main(String[] args)
   {
      A a = new B();
      a.print();
   }
   void print() { System.out.println(i); }
}
/**
和上一题类似 这里a被声明为A 那么就没有权限访问private
**/
```

```java
interface Eatable{
    int types = 10;
}
class Food implements Eatable {
    public static int types = 20;
}
public class Fruit extends Food implements Eatable{  //LINE1
    
    public static void main(String[] args) {
        types = 30; //LINE 2
      // Food.types = 30; 可以
      // Eabtable.types = 30; 不可以
        System.out.println(types); //LINE 3
      // 读的话两个都可以
    }
}// 2 3 都报错  因为无法确定是哪个  接口中的默认是  static final 所以不可以改变
```

```java
class A
{
   int i;
   public A(int x) { this.i = x; }
}
class B extends A
{
   int j;
   public B(int x, int y) { super(x);  this.j = y; }
}
// 如果父类中有需要初始化的属性 且没有无参构造 那么就必须由子类手动创建 
// B(int y ) { super(y*2 ); j = y; } 可以被添加到classB
// B(int z ) { this(z, z); } 可以
// B(int y ) { j = y; } 不可以  因为编译器默认调用父类空参  但是没有--编译错误
```

