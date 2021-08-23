# Stream

---

## 1. API 相关

### 1.1 Stream接口中包括了 min max 但是没有其他数学类的比如sum，并且intStream或者doublestream这样的是重写了min 和max 的 但是Stream需要传入 comparator

### 1.2 Optional会在如果出现空值的时候返回  min max  reduce 和那些判断类比如（findAny） 但是sum不会。 average永远是Optional double

### 1.3 findAny 会提前终止，注意打印

```java
OptionalInt first = stream.peek(System.out::print).findAny();
//  22
```



### 1.4 primitive version 调用 match 类数据需要的是 DoublePredicte 不是 predicte<double>

### 1.5 flatMap 的参数的返回值**必须**是stream

```java
     	 Stream<String> stream = Stream.of("SE","EE","ME");
       Stream<Integer> list = stream.flatMap(s -> Stream.of(s.length()));// 这种就不行
       System.out.println(list.collect(Collectors.toSet()));



				Stream<Double> stream = Stream.of(12.1,12.5,12.9);
        Stream list = stream.flatMap(d -> Stream.of(d.intValue())); //这种就可以
        System.out.println(list.collect(Collectors.toSet()));
```



# Optional

---

## 1. API 相关

### 1.1 primitive version 的get 对应是 getAsXXX

### 1.2 如果value为空 调用get会抛出异常

## 2. 创建

### 2.1 创建空

```java
Optional ops = Optional.empty();
```



# 函数式接口

---

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

# Locale 

---

## **1 ResourceBundle** 

查找顺序

1. bundleName + "_" + localeLanguage + "_" + localeCountry + "_" + localeVariant
2. bundleName + "_" + localeLanguage + "_" + localeCountry
3. bundleName + "_" + localeLanguage
4. bundleName + "_" + defaultLanguage + "_" + defaultCountry + "_" + defaultVariant
5. bundleName + "_" + defaultLanguage + "_" + defaultCountry
6. bundleName + "_" + defaultLanguage
7. bundleName

注意不会有 base + Country 必须有语言才考虑国家！

**最后会检查 default**

### 1.1  **ResourceBundle** 会从上向下遍历到base

```java
Locale locale = new Locale("de");
ResourceBundle rb = ResourceBundle.getBundle("SRBundel", locale);
Enumeration bundleKeys = rb.getKeys();
// 这里会先遍历 SRBundel_de 然后遍历 SRBundel的 key 如果有子包有重复key取high level
// 如果 key不重复 则都输出
```

### 1.2  如果没有匹配的 则找base

## 2. 各种支持

### 2.1 instant 

- `NANO_OF_SECOND`
- `MICRO_OF_SECOND`
- `MILLI_OF_SECOND`
- `INSTANT_SECONDS`

# 时间类型

---

**不可变类型！！！！**

## 1. Duration

基于时间 秒 纳秒

### 1.1 support

* Seconds 
* Nanos

## 2.Period

基于日期

### 2.1 api

#### 2.1.1 withDays 等不是static 只有of 和一些计算是static 别的类也类似

### 2.2 不转换性

主要是计数 不会发生自动转换尽管数据超过了 比如 40天 不会变成一个月10天

```java
public static void main(String[] args) throws IOException {
    Period p = Period.ofDays(2);
    p = p.multipliedBy(30).normalized();
    System.out.println(p);
    // P60D
}
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

## 6 Dataformat

### 6.1 创建

抽象类 不可以 new

* getInstance 不可以
* getDateInstance 可以带参数

## 7. APIs

### 7.1  format() 返回值是String 不是 LocalDate or LocalDateTime

# Lambda

---

条件 一个**接口**仅有**一个**待实现的方法 

* 不可以初始化抽象类

# 枚举类

---

## 1. 注意点

* 常数必须写在最前面 ---否则**编译报错**

## 2. 顺序

枚举类实现了 Comparable  排序方法是声明顺序

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



# **inner** class

---

## 1 注意点



## 2. 成员内部类

可以静态或者非静态 和**正常的类一样** 该有啥都可以用 也可以再添加内部类

* 可以被 final修饰
* 可以被 abstract 修饰

### 2.2  调用outer的东西

如果需要通过 outerclass.this.(method or fields)

## 3. 局部内部类

* 不可以添加任何 access modifier

* JDK7 inner class 可以访问 static 和 final修饰的变量（在外部类中）

* JDK8 之后inner class 有权限访问的是 final修饰 或者 **实际上是final的**（effectively final）即初始化后从来没有修改过的

```java
 public class Whiz{
        
           int x = 10;
          
           public static void method(int c,int i){
                      
                       class Test{
                                       public void in(){
                                       // here
                                         // 这里只可以访问i 因为i从来没有被修改过
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



# JDBC

---

## 1. JdbcRowSet

高级版 ResultSet **数据连接必须在线** 和普通版一样

### 1.1 创建

使用 实现类（**JdbcRowSetImpl**）

* 直接创建
* 接收ResultSet 必须是可以滚动的
* 接收Connection

使用 **RowSetProvider**

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

## 2  CachedRowSet

**离线模式** 短暂接触数据源 内容保存在内存中 获取方式和 RS类似 

```java
crs.populate(rs); // 转换
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

# Files

---

## 1. APIs

### 1.1 读取

* lines 返回值 Stream<String> 参数是path 也可以加字符集
* readAllLines 返回值是List<String> 参数是 path 也可以加字符集

### 1.2 读取属性

Files.readAttributes(path, PosixFileAttributes.class);

# Collection

---

## 1. List

### 1.1 removeAll 

接收Collection 移除List里面参数List的所有elements

## 2. Map

### 2.1 replaceAll

接收 Bifunction 两个参数一个返回值

2.2

## 3. Queue

### 3.1 removeLastOccurrence

移除最后一个出现的啥啥啥



# Path

---

## 1 APIs

### 1.1  拼接例子

root 代表磁盘 subpath 代表后面的以\\\分割的

```java
Path path1 = Paths.get("F:\\Whizlabs\\java\\NIO");
Path path2 = Paths.get("c:\\output");
Path path  = Paths.get(path2.getRoot().toString(), path1.subpath(0,3).toString());
System.out.print(path.toString());
```

# 断言

---

## 1. enable 和 disable

```bash
java -ea -da:Lab Whiz # -da: 表示disable的 
```
