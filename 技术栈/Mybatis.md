# Mybatis

## 0. 依赖导入

```xml
    		<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
```

## 1. 配置和基本使用

0. pom中配置资源过滤

   ```xml
       <!-- build 中配置resouces, 防止资源导出失败 -->
       <build>
           <resources>
               <resource>
                   <directory>src/main/resources</directory>
                   <includes>
                       <include>**/*.properties</include>
                       <include>**/*.xml</include>
                   </includes>
                   <filtering>true</filtering>
               </resource>
               <resource>
                   <directory>src/main/java</directory>
                   <includes>
                       <include>**/*.properties</include>
                       <include>**/*.xml</include>
                   </includes>
                   <filtering>true</filtering>
               </resource>
           </resources>
       </build>
   ```

1. mybatis-config.xml XML核心配置

   * 写完了后记得回来注册

   ```xml
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   
   <!-- mybatis 核心配置 -->
   <configuration>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"></transactionManager>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=Utf-8"/>
                   <property name="username" value="root"/>
                   <property name="password" value="root"/>
               </dataSource>
           </environment>
       </environments>
     <!-- 注册 mapper -->
         <mappers>
           <mapper resource="com/bohan/dao/UserMapper.xml"></mapper>
       </mappers>
   </configuration>
   ```

2. MybatisUtils 创建mybatis工具类

   ```java
   package com.bohan.utils;
   
   import org.apache.ibatis.io.Resources;
   import org.apache.ibatis.session.SqlSession;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.apache.ibatis.session.SqlSessionFactoryBuilder;
   
   import java.io.IOException;
   import java.io.InputStream;
   
   //sqlSessionFactory --> sqlSession
   public class MybatisUtils {
   
       private static SqlSessionFactory sqlSessionFactory;
   
       static{
           try {
               //1.获取sqlSessionFactory对象
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
   
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   
       // 用工厂模式创建sqlSession
       public static SqlSession getSqlSession(){
           return sqlSessionFactory.openSession();
       }
   }
   
   ```

3. UserMapper 接口

   ```java
   package com.bohan.dao;
   
   import com.bohan.pojo.User;
   
   import java.util.List;
   
   public interface UserMapper {
       List<User> findAll();
   }
   
   ```

4. UserMapper.xml 相当于实现类 namespace用来连接mapper接口 id是对应方法名字

   ```xml
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.bohan.dao.UserMapper">
       <select id="findAll" resultType="com.bohan.pojo.User">
           select * from user
       </select>
   </mapper>
   ```

5. 注册（在主配置文件里面）并且测试

   ```java
   package com.bohan.dao;
   
   import com.bohan.pojo.User;
   import com.bohan.utils.MybatisUtils;
   import org.apache.ibatis.session.SqlSession;
   import org.junit.Test;
   
   import java.util.List;
   
   public class UserDaoTest {
   
       @Test
       public void test(){
           // 获取sqlSession
           SqlSession sqlSession = MybatisUtils.getSqlSession();
   
           //方式1 使用getMapper
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           List<User> users = mapper.findAll();
           for (User user: users){
               System.out.println(user);
           }
   
           //关闭
           sqlSession.close();
       }
   }
   
   ```

6. 总结 一共5个文件

   * **Mybatis-config.xml (主配置文件)**
     1. 连接池
     2. 注册mapper
   * **MybatisUtils** 
     1. 导入配置文件
     2. 负责创建sqlSession（工厂模式）
   * UserMapper (接口)
   * UserMapper.xml（sql）
   * Test

7. 注意点

   1. SqlSessionFactoryBuilder 是单例模式 不关闭一直存在
   2. 注册mapper的时候用'/' 而不是'.'。

## 2. CRUD

1. **注意点**

   * 增删改一定要提交事务 否则不会保存到数据库

2. 代码

   ```java
    @Test
       public void select(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user = mapper.selectUserById(1);
           System.out.println(user);
       }
   
       @Test
       public void add(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user1 = new User();
           user1.setId(5);
           user1.setName("bohan");
           user1.setPwd("11111");
           mapper.addUser(user1);
           sqlSession.commit();
           System.out.println(mapper.selectUserById(5));
           sqlSession.close();
   
       }
   
       @Test
       public void update(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user1 = new User();
           user1.setId(5);
           user1.setName("gai");
           user1.setPwd("11111");
           mapper.updateUser(user1);
           sqlSession.commit();
           System.out.println(mapper.selectUserById(5));
           sqlSession.close();
       }
   
       @Test
       public void delete(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           mapper.deleteUser(1);
   
           sqlSession.commit();
           sqlSession.close();
       }
   ```

   ```xml
       <select id="selectUserById" parameterType="int" resultType="com.bohan.pojo.User">
           select * from user where id = #{id}
       </select>
   
       <insert id="addUser" parameterType="com.bohan.pojo.User">
           insert into user (id, name, pwd) values (#{id}, #{name}, #{pwd})
       </insert>
   
       <update id="updateUser" parameterType="com.bohan.pojo.User">
           update user set name=#{name}, pwd=#{pwd} where id=#{id}
       </update>
   
       <delete id="deleteUser" parameterType="int">
           delete from user where id=#{id}
       </delete>
   ```



## 3. 模糊查询

使用通配符 %

```xml
    <select id="findUserByName" parameterType="java.lang.String" resultType="com.bohan.pojo.User">
        select * from user where name like "%"#{value}"%"
    </select>
```



## 4. 配置解析

### 4.1 核心配置文件 mybatis-config.xml

#### 4.1.1 用properties文件配置

properties文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=Utf-8
username=root
password=root
```

在mybatis-config中引用 然后就可以使用变量 类似map

```xml
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- mybatis 核心配置 -->
<configuration>
    <!-- 引入外部配置文件 -->
    <properties resource="db.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    <mappers>
        <mapper resource="com/bohan/dao/UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

#### 4.1.2 别名(typeAliases)

更短的名字 写在mybatis-config中

1. 一个一个写

   ```xml
       <typeAliases>
           <typeAlias type="com.bohan.pojo.User" alias="User"/>
       </typeAliases>
   ```

2. 扫描一个包（默认是实体类小写）实测大小写都可以

   ```xml
       <typeAliases>
           <package name="com.bohan.pojo"/>
       </typeAliases>
   ```

3. 注解 自定义

   ```java
   package com.bohan.pojo;
   
   import org.apache.ibatis.type.Alias;
   
   @Alias("hello")
   public class User {
       private int id;
       private String name;
       private String pwd;
   
       public User() {
       }
   
       public User(int id, String name, String pwd) {
           this.id = id;
           this.name = name;
           this.pwd = pwd;
       }
   
       public int getId() {
           return id;
       }
   
       public String getName() {
           return name;
       }
   
       public String getPwd() {
           return pwd;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public void setPwd(String pwd) {
           this.pwd = pwd;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   ", pwd='" + pwd + '\'' +
                   '}';
       }
   }
   ```

#### 4.1.3 mappers（引射器）

mapperRegistry:注册绑定mapper文件

方式1 通过resource

```xml
    <mappers>
        <mapper resource="com/bohan/dao/UserMapper.xml"></mapper>
    </mappers>
```

方式2 通过class

```xml
    <mappers>
        <mapper class="com.bohan.dao.UserMapper" />
    </mappers>
```

* **接口和mapper必须同名**
* **两个文件必须在同一包下**

方式3 通过pakage

```xml
    <mappers>
        <package name="com.bohan.dao" />
    </mappers>
```

* **接口和mapper必须同名**
* **两个文件必须在同一包下**

### 4.2 设置

#### 4.2.1 resultMap

结果集映射，相当于别名

```xml
    <resultMap id="UserMap" type="User">
        <result column="id" property="id" />
        <result column="name" property="name" />
        <result column="pwd" property="password" />
    </resultMap>

    <select id="findAll" resultMap="UserMap">
        select * from user
    </select>
```

设置类很多可以通过文档查询

## 5. 日志

### 5.1 日志工厂

在mybatis-config中配置

* SLF4J

  ```xml
      <settings>
          <setting name="logImpl" value="STDOUT_LOGGING"/>
      </settings>
  ```

  

* LOG4J

  1. 导包

     ```xml
             <dependency>
                 <groupId>log4j</groupId>
                 <artifactId>log4j</artifactId>
                 <version>1.2.17</version>
             </dependency>
     ```

  2. 配置文件 网上随便找 这里一个简单配置

     ```properties
     
     #设置日志的级别 ,多个以,分开(没有给出的，则不会被输出)
     log4j.rootLogger=DEBUG,console,file
     
     log4j.appender.console=org.apache.log4j.ConsoleAppender
     log4j.appender.console.Target = System.out
     log4j.appender.console.Threshold=DEBUG
     log4j.appender.console.layout = org.apache.log4j.PatternLayout
     log4j.appender.console.layout.ConversionPattern = [%c]- %m%n
     
     log4j.appender.file=org.apache.log4j.RollingFileAppender
     log4j.appender.file.File=./log/bohan.log
     log4j.appender.file.MaxFileSize=10mb
     log4j.appender.file.Threshold=Debug
     log4j.appender.file.layout=org.apache.log4j.PatternLayout
     log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-mm-dd}][%c]%m%n
     
     log4j.logger.org.mybatis=DEBUG
     log4j.logger.java.sql=DEBUG
     log4j.logger.java.sql.Statement=DEBUG
     log4j.logger.java.sql.ResultSet=DEBUG
     log4j.logger.java.sql.PreparedStatement=DEBUG
     ```

  3. 测试+调用

     ```java
     package com.bohan;
     
     import com.bohan.dao.UserMapper;
     import com.bohan.pojo.User;
     import com.bohan.utils.MybatisUtils;
     import org.apache.ibatis.session.SqlSession;
     import org.apache.log4j.Logger;
     import org.junit.Test;
     
     import java.util.List;
     
     public class UserTest {
     
         static Logger logger = Logger.getLogger(UserTest.class);
     
         @Test
         public void test(){
             SqlSession sqlSession = MybatisUtils.getSqlSession();
     
             UserMapper mapper = sqlSession.getMapper(UserMapper.class);
             logger.info("=====================================");
             List<User> users = mapper.findAll();
             for (User user:users){
                 System.out.println(user);
             }
         }
     
         @Test
         public void testLog4j(){
             logger.info("info:进入了testLog4j");
             logger.debug("debug");
             logger.error("error");
         }
     
     }
     ```



## 6.分页

### 6.1 使用limit

第一个参数是从第几页开始，第二个是每页几个。

```sql
select * from user limit 2,2;
```



基本使用

UserMapper

```java
    List<User> getUserByLimit(Map<String,Integer> map);
```

Xml

```xml
    <select id="getUserByLimit" parameterType="map" resultType="user">
        select * from user limit #{startIndex},#{pageSize};
    </select>
```

测试

```java
    @Test
    public void getUserByLimit(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        HashMap<String, Integer> map = new HashMap<String, Integer>();
        map.put("startIndex", 1);
        map.put("pageSize",2);


        List<User> users = mapper.getUserByLimit(map);
        for (User user : users){
            System.out.println(user);
        }

        sqlSession.close();
    }
```

## 7.注解开发

### 7.1基本使用

1. 接口上直接写sql

   ```java
       @Select("select * from user")
       List<User> findAll();
   ```

2. 绑定

   ```xml
       <mappers>
           <mapper class="com.bohan.dao.UserMapper"/>
       </mappers>
   ```



### 7.2本质

通过反射拿到类和所有属性 和注解上的sql。

### 7.3 注意点

* 如果有多个参数 所有参数必须加@Param 这个注解后面跟着的值就是sql里面的变量

  ```java
  @Select("select * from user where id = #{id}")
  User getUserById(@Param("id") int id)
  ```

  



## 8. 多表查询

### 8.1 多对1 association

*  嵌套查询
* 学生class里面组合teacher

![image-20200414110803176](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414110803176.png)

### 8.2 一对多 collection

* teacher class 里面放List<Student>

  ![image-20200414112609530](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414112609530.png)



## 9. 动态SQL

### 9.1 if 

案例需求：搜索的时候 如果没有传入搜索关键字则查询全部

![image-20200414115244993](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414115244993.png)

如果不进行判断 那么title 和author的值为空会报错 而不会跳过

#### 9.1.1 注意点

这里需要注意的是 where 1=1 实际上是不规范的 但是如果不这么写 那么下面都用if的话 则第一个没有and但是后面的都有and 那么如果第一个if是false那么后面直接拼接变成 where and title=#{title} 这样会报错 所以有**where**的诞生（**set**也类似）都是用来删除逗号或者 and的

![image-20200414120213786](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414120213786.png)

![image-20200414124946953](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414124946953.png)

### 9.2 choose（when other）

类似swich 多种情况选择一种的执行的

![image-20200414120414688](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414120414688.png)



### 9.3 SQL片段

处理冗余 复用片段提取

![image-20200414182617861](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414182617861.png)

注意事项：

* 最好使用单表
* 不要存在where标签在片段里面

### 9.4 Foreach

主要内容还是用来拼接遍历并且拼接SQL

![image-20200414183427649](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200414183427649.png)

* ids 是map中的一个list 然后从里面取出id 用后面的来拼接



## 10 缓存

### 10.1一级缓存（默认开启 无法关闭）

sqlSession级别的缓存 在一次sqlSession开启和close之间可以使用。相当于一个map

#### 10.1.1缓存失效

* 增删改操作可能会改变原来的数据 所以会刷新缓存
* 手动青村也会让缓存失效

### 10.2二级缓存（手动开启）

在namespace范围内有效 也就是一个mapper文件里面的所有sql都可以访问缓存

#### 10.2.1开启方法

1. 开启缓存 在设置中

   ```xml
   <settings>
     <!-- 其实默认是开启的 -->
   	<setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. 在mapper中添加 下面是例子

```xml
<cache
       eviction="FIFO"
       flushInterval="60000"
       size="521"
       readOnly="true"/>
```

#### 10.2.2配置信息

见文档缓存

#### 10.2.3工作机制

当一级缓存失效后才会放入二级缓存中

查询的时候 先从二级缓存中查找，然后查找一级缓存，最后查找数据库

### 10.3自定义缓存

是一种分布式缓存 貌似没什么人用