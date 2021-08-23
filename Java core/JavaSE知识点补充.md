# 泛型

---

## 1. 泛型类和接口

- 两者差不多 类和接口里面有一个属性参数类型不明。

## 2. 泛型方法

- 方法上声明泛型
- **泛型类中的泛型只能适用于非静态方法，如果需要给静态方法设置泛型，此时使用泛型方法**

```java
    public static <T> DataResult success(T data){
      // 因为要使用静态方法且有泛型参数 所以必须使用泛型方法
        return new <T>DataResult(data);
    }
```



# 内部类

---

一个类的 某一中属性过于复杂不适合用属性来描述，而这个属性也只有这个类能使用（没必要暴露给其他类）那么就需要创建内部类

## 1. 分类

### 1.1 成员内部类

#### 1.1.1 非静态成员内部类

* 需要先创建 主类 然后再通过主类创建

  ```java
  // normal
  innerClass aClass = new innerClass();
  bird bird = aClass.new bird();
  ```

#### 1.1.2 静态成员内部类

* 通过 class.innerclass var = new class.innerclass(); 来创建

  ```java
  // static
  innerClass.Dog dag = new innerClass.Dog();
  ```

### 1.2 局部内部类

* 开发中很少用到

#### 1.2.1 使用案例

一个方法需要返回一个实现了comparator接口的类

```java
import java.util.Comparator;

public class InnerClass02 {

    public Comparator getComparable(){
        
        // 局部内部类
        class MyComparable implements Comparator{
        
            // 实现一个接口的匿名类
            @Override
            public int compare(Object o1, Object o2) {
                return 0;
            }
        }

        return new MyComparable();

    }
}
```

### 1.3 分布

```java
public class innerClass {

    // 静态成员内部类
    static class Dog{

    }

    // 非静态成员内部类
    class bird{

    }

    public void methods(){
        
       // 局部内部类 
        class AA{
            
        }
    }


}
```

# Lambda 表达式

---

类似箭头函数 只是 = 变成了 -



# TimeUnit

TimeUnit是java.util.concurrent包下面的一个**枚举类**，表示给定单元粒度的时间段。

```java
TimeUnit.DAYS          //天
TimeUnit.HOURS         //小时
TimeUnit.MINUTES       //分钟
TimeUnit.SECONDS       //秒
TimeUnit.MILLISECONDS 
```



1. 互相转换

   ```cpp
   public long toMillis(long d)    //转化成毫秒
   public long toSeconds(long d)  //转化成秒
   public long toMinutes(long d)  //转化成分钟
   public long toHours(long d)    //转化成小时
   public long toDays(long d)   
   ```

2. 例子

   ```csharp
   public class TimeUnitTest {
   
       public static void main(String[] args) {
           //convert 1 day to 24 hour
           System.out.println(TimeUnit.DAYS.toHours(1));
           //convert 1 hour to 60*60 second.
           System.out.println(TimeUnit.HOURS.toSeconds(1));
           //convert 3 days to 72 hours.
           System.out.println(TimeUnit.HOURS.convert(3, TimeUnit.DAYS));
       }
   
   }
   ```

3. 可以用来延时 代替 Thread.sleep()

   ```csharp
   public class TimeUnitTest {
   
       public static void main(String[] args) {
           //convert 1 day to 24 hour
           System.out.println(TimeUnit.DAYS.toHours(1));
           //convert 1 hour to 60*60 second.
           System.out.println(TimeUnit.HOURS.toSeconds(1));
           //convert 3 days to 72 hours.
           System.out.println(TimeUnit.HOURS.convert(3, TimeUnit.DAYS));
       }
   
   }
   ```

