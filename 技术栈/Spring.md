

doc地址： https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-value-element

# Spring

## 1. 优点

* 控制翻转（IOC）， 面向切片编程（AOP）
* 支持事务处理，对框架整合的支持

## 2.  快速开发

1. 导入包

   ```xml
           <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>5.2.4.RELEASE</version>
           </dependency>
   ```

   

2. 注入 xml文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd
           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">
   
       <!--无参构造的创建-->
       <bean id="hello" class="com.bohan.pojo.Hello">
           <!-- value 用来直接赋值  ref是用Ioc容器中有的bean进行赋值，因为有的时候
           一个bean中使用了另一个bean-->
           <property name="str" value="spring"></property>
       </bean>
   
       <!--有参构造的创建-->
       <bean id="user" class="com.bohan.pojo.User">
           <constructor-arg index="0" value="bohan" ></constructor-arg>
       </bean>
   </beans>
   ```

   

3. 获取容器

   ```java
   public class MyTest {
       public static void main(String[] args) {
           //获取spring的上下文对象
           ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
           Hello hello = (Hello) context.getBean("hello");
           System.out.println(hello.getStr());
       }
   }
   ```

## 4.  IOC创建对象的方式

1. 默认是无参构造创建

2. 常用三种赋值

   * 下标赋值

   * 类型

   * 参数名（常用）

     ```xml
         <bean id="user" class="com.bohan.pojo.User">
             <constructor-arg name="name" value="bohan" ></constructor-arg>
         </bean>
     ```

## 4. spring配置

### 4.1 alias 

别名（没啥用 bean里面有 name属性来创建别名）

```xml
<!-- alias的方法 -->
<alias name="user" alias="useruser"></alias>

<!-- name的方法 -->
    <bean id="user" class="com.bohan.pojo.User" name="bohan1 bohan2 bohan3">
        <constructor-arg name="name" value="bohan" ></constructor-arg>
    </bean>
```

### 4.2 import

* 导入其他的配置文件（合并）

* 内容相同会被自动去除

```xml
<import resource="beans.xml"></import>
```



## 5.  依赖注入

### 5.1 构造器注入

```xml
<bean id="Address" class="com.bohan.Address">
    <property name="address" value="Detroit"></property>
</bean>
```

### 5.2 Set注入

* 需要注意实体类中一定要有set方法

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="Address" class="com.bohan.Address"></bean>

    <bean id="student" class="com.bohan.Student">
        <!-- 基本数据类型 -->
        <property name="name" value="bohan"></property>

        <!-- 引用数据类型 -->
        <property name="address" ref="Address"></property>

        <!--数组-->
        <property name="books">
            <array>
                <value>11111</value>
                <value>22222</value>
                <value>33333</value>
                <value>44444</value>
                <value>55555</value>
            </array>
        </property>

        <!--list-->
        <property name="hobbys">
            <list>
                <value>list1</value>
                <value>list2</value>
                <value>list3</value>
                <value>list4</value>
            </list>
        </property>

        <!--map-->
        <property name="card">
            <map>
                <entry key="key1" value="value1"></entry>
                <entry key="key2" value="value2"></entry>
                <entry key="key3" value="value3"></entry>
                <entry key="key4" value="value4"></entry>
            </map>
        </property>

        <!--set-->
        <property name="games">
            <set>
                <value>1</value>
                <value>2</value>
                <value>3</value>
            </set>
        </property>

        <!--null-->
        <property name="wife">
            <null></null>
        </property>

        <!--properties-->
        <property name="info">
            <props>
                <prop key="key1">value1</prop>
                <prop key="key2">value2</prop>
                <prop key="key3">value3</prop>
            </props>
        </property>
    </bean>
</beans>

```

### 5.3 p,c命名空间注入 

* 需要导入约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">

		<!-- 举例p， 区别不大-->
    <bean id="user" class="com.bohan.User" p:name="bohan" p:age="18"/>
</beans>

```



## 6. bean的作用域



![image-20200401084706556](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200401084706556.png)

### 6.1 单例模式

只有一个实例

```xml
    <bean id="user" class="com.bohan.User" p:name="bohan" p:age="18" scope="singleton"/>
```

### 6.2 原型模式

可以有多个实例

```xml
    <bean id="user" class="com.bohan.User" p:name="bohan" p:age="18" scope="prototype"/>
```

### 6.3 剩下的只有在web中会遇到

## 7. 自动装配

### 7.1 xml

### 7.2 java

### 7.3 隐式装配

* 会自动装配 在容器中自动寻找

* ```xml
      <bean id="cat" class="com.bohan.pojo.Cat" />
      <bean id="dog" class="com.bohan.pojo.Dog" />
  
  
      <bean id="person" class="com.bohan.pojo.Person" autowire="byName">
  <!--        <property name="cat" ref="cat" />-->
  <!--        <property name="dog" ref="dog" />-->
          <property name="name" value="bohan"/>
      </bean>
  ```

* byName（根据id） byType(要求类型唯一， 根据class)

## 8. 注解装配

**jdk 1.5 spring2.5 开始支持**

* xml头文件

* ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
  
     <!-- 开启注解 -->
      <context:annotation-config></context:annotation-config>
  </beans>
  ```

* 然后使用 @Autowired来自动注入 （byName）

  * 如果自动转配环境比较复杂 可以使用 @Qualifier 来配合使用 指定一个唯一的（id）



## 9. 注解开发

### 9.1 简单案例

1. applicationContext.xml 头文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
   
       <!--告知spring在创建容器时要扫描的包,配置此项所需标签不在beans的约束中,而在一个名为context的名称空间和约束中-->
       <context:component-scan base-package=""></context:component-scan>
   </beans>
   
   ```

   

2. java 文件

   ```java
   package com.bohan.pojo;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Component;
   
   @Component
   public class User {
       @Value("bohan")
       private String name ;
     
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "name='" + name + '\'' +
                   '}';
       }
   }
   
   ```

   3. 使用和之前一样。 默认bean的名字是class的小写
   4. @Componet @Service @ Repository @Controller 都一样 只是用于分类
   5. 作用域 @Scope("singleton")

## 10. xml 和 注解

* xml更万能 适用于任何场合 维护起来简单方便
* 注解不是自己的类使用不了 维护相对复杂
* xml 一般用来管理bean
* 注解只负责完成属性的注入



## 11. 纯java配置方法

1. java中 使用 @Configuration + @Bean

   ```java
   package com.bohan.config;
   
   import com.bohan.pojo.User;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class UserConfig {
   
       @Bean
       public User getUser(){
           return new User();
       }
   }
   
   ```

2. ```java
   import com.bohan.config.UserConfig;
   import com.bohan.pojo.User;
   import org.junit.Test;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.annotation.AnnotationConfigApplicationContext;
   
   public class MyTest {
   
       @Test
       public void test(){
         //这里使用了annotaion 并且导入配置的class
          ApplicationContext context = new AnnotationConfigApplicationContext(UserConfig.class);
           User user = (User) context.getBean("getUser");
           System.out.println(user.getName());
       }
   }
   ```

3. 补充 实体类中的配置

   ```java
   package com.bohan.pojo;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Component;
   
   @Component
   public class User {
       @Value("bohan111")
       private String name;
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "name='" + name + '\'' +
                   '}';
       }
   }
   ```



## 12. 动态代理

23种设计模式笔记

## 13. AOP

其实主要的就是在不干扰业务逻辑的情况下 增强逻辑

### 13.1 依赖包

```xml
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.4</version>
        </dependency>
```

### 13.2 方法1 使用原生的 spring接口

* 通过配置AOP约束在xml中

* ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
  
      <!-- 注册 bean -->
      <bean id="userService" class="service.UserServiceImpl"></bean>
      <bean id="log" class="service.log.Log"></bean>
      <bean id="afterLog" class="service.log.AfterLog"></bean>
  
  
      <!--    方式1 使用spring API接口-->
  
      <!-- 配置 aop:需要导入aop的约束 -->
      <aop:config>
          <!-- 创建一个切入点 -->
        	<!-- 表示UserServiceImpl下的所有方法都会被拦截 -->
          <aop:pointcut id="pointcut" expression="execution(* service.UserServiceImpl.*(..) )"/>
  
          <!--执行环绕增强 增强和切入点匹配-->
          <aop:advisor advice-ref="log" pointcut-ref="pointcut" />
          <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut" />
      </aop:config>
  </beans>
  
  ```
```
  
* 所需要的增强函数 需要实现不同的接口 有很多种这里举例两种

  ```java
  package service.log;
  
  import org.springframework.aop.MethodBeforeAdvice;
  
  import java.lang.reflect.Method;
  
  public class Log implements MethodBeforeAdvice {
  
  
      /**
       *
       * @param method 要执行的目标对象的方法 谁被调用就是谁
       * @param objects 参数
       * @param o 目标对象
       * @throws Throwable
       */
     //这里的method就是被pointcut设置的拦截的方法
      public void before(Method method, Object[] objects, Object o) throws Throwable {
          System.out.println(o.getClass().getName() + "的" + method.getName() + "被执行了");
      }
  }
  
```

* ```java
  package service.log;
  
  import org.springframework.aop.AfterReturningAdvice;
  
  import java.lang.reflect.Method;
  
  public class AfterLog implements AfterReturningAdvice {
      //可以拿到返回值
      public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
          System.out.println("执行了" + method.getName() + "方法， 返回了" + o);
      }
  }
  ```

* 调用和结果

  ```java
  
  import org.springframework.context.ApplicationContext;
  import org.springframework.context.support.ClassPathXmlApplicationContext;
  import service.UserService;
  import service.UserServiceImpl;
  
  public class Mytest {
  
      public static void main(String[] args) {
          ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
          //要注意这里代理的是接口 不要返回代理类
          UserService userService = context.getBean("userService", UserService.class);
  
          userService.update();
      }
  }
  
  ```

  ![image-20200402131022479](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200402131022479.png)

### 13.3 方法2 用一个AOP类来实现增强

* 设置特定的函数 并且通过xml配置分配到切入点中（不需要实现各种增强接口）

  ```xml
  <!-- 方式2 自定义类 -->
  <!-- 注入增强类 -->
  <bean id="diy" class="service.diy.DivPointCut" />
  
  <!-- 配置 -->
  <aop:config>
      <!-- 切入面 ref 要引用的类-->
      <aop:aspect ref="diy">
          <!-- 切入点 -->
          <aop:pointcut id="point" expression="execution(* service.UserServiceImpl.*(..) )"/>
          <!-- 细节 -->
          <aop:before method="before" pointcut-ref="point" />
          <aop:after method="after" pointcut-ref="point" />
      </aop:aspect>
  </aop:config>
  ```

* ```java
  package service.diy;
  
  public class DivPointCut {
  
      public void before(){
          System.out.println("方法执行前");
      }
  
      public void after(){
          System.out.println("方法执行后");
      }
  }
  ```

### 13.4 方法3 注解

* ```java
  package service.diy;
  
  
  import org.aspectj.lang.annotation.Aspect;
  import org.aspectj.lang.annotation.Before;
  
  @Aspect
  public class AnnotationPointCut {
  
      @Before("execution(* service.UserServiceImpl.*(..) )")
      public void before(){
          System.out.println("执行前");
      }
  }
  
  ```

* ```xml
      <!-- 方式3 -->
      <!-- 注入 -->
      <bean id="annotationPointCut" class="service.diy.AnnotationPointCut" />
  
      <!-- 开启注解 -->
      <aop:aspectj-autoproxy/>
  ```

## 14. 整合mybatis

几个需要了解的东西

* sqlSessionFactory --在这里配置数据源和一些配置信息 类似mybatis
* sqlSessionTemplate-- 当mybatis中的sqlSession用 读取mapper来创建sqlSession连接 **线程安全**

#### 14.1导包

```xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>

        <!-- 数据库 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.10</version>
        </dependency>
    </dependencies>
```

#### 14.2 快速例子

1. spring的配置文件中编写

   * sqlSessionFactory 数据源配置信息

   * sqlSessionTemplate 读取配置信息（其实就是mybatis中的 sqlsession）

     * mybatis的配置文件在这里读取
     * 也可以在spring中完成一些mybatis中的配置

   * 注册mapper的实现类（稍后编写）

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans.xsd">
     
     
         <!-- DataSource：使用spring 的数据源替代mybatis的配置 c3p0 dbcp druid
             这里使用spring提供的JDBC ： org.springframework.jdbc
          -->
         <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
             <property name="driverClassName" value="com.mysql.jdbc.Driver" />
             <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=Utf-8"/>
             <property name="username" value="root"/>
             <property name="password" value="root"/>
         </bean>
     
         <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
             <property name="dataSource" ref="dataSource"/>
             <!-- 绑定mybatis的配置文件 -->
             <property name="configLocation" value="classpath:mybatis-config.xml" />
           	<!-- 绑定mapper的位置 -->
             <property name="mapperLocations" value="classpath:com/bohan/dao/*.xml" />
         </bean>
         <!-- sqlSessionTemplate 就是我们使用的 sqlSession -->
         <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
             <!-- 没有set方法 所以只能用构造器注入 -->
             <constructor-arg index="0" ref="sqlSessionFactory"/>
         </bean>
     
         <bean id="userMapper" class="com.bohan.dao.UserMapperImpl">
             <property name="sqlSession" ref="sqlSession"/>
         </bean>
     </beans>
     
     ```

     

2. 给mapper加实现类 在里面 并且注册到spring中

   ```java
   package com.bohan.dao;
   
   import com.bohan.pojo.User;
   import org.mybatis.spring.SqlSessionTemplate;
   
   import java.util.List;
   
   public class UserMapperImpl  implements UserMapper{
   
       private SqlSessionTemplate sqlSession;
   
   
       public void setSqlSession(SqlSessionTemplate sqlSession){
           this.sqlSession = sqlSession;
       }
   
       public List<User> findAll() {
           return sqlSession.getMapper(UserMapper.class).findAll() ;
       }
   }
   ```

3. 测试

   ```java
   @Test
       public void testSpringMybatis(){
           ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-dao.xml");
           UserMapper userMapper = context.getBean("userMapper", UserMapper.class);
   
           List<User> users = userMapper.findAll();
           for (User user : users) {
               System.out.println(user);
           }
       }
   ```

#### 14.3 SqlSessionDaoSupport

是一个抽象类 可以通过继承来使用，可以用来直接生成sqlsession 使用（getSqlSession()）

```java
package com.bohan.dao;

import com.bohan.pojo.User;
import org.mybatis.spring.support.SqlSessionDaoSupport;

import java.util.List;

public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper{


    public List<User> findAll() {

        return getSqlSession().getMapper(UserMapper.class).findAll() ;
    }
}

```

需要注意的是虽然没有使用sqlSessionTemplate 但是需要注册并且不能改名

```xml
   <bean id="userMapper2" class="com.bohan.dao.UserMapperImpl2" >
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>
```



## 15. 声明式事务

* 要么都成功要么都失败
* 一致性和完整性

![image-20200414212709870](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414212709870.png)

## 