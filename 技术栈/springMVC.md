# SpringMVC

##  1.回顾servlet

### 1.1实现一个servlet

从普通的maven项目创建

#### 1.1.1导包

基本都认识 不做介绍

```xml

<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
```

#### 1.1.2 设置项目为web项目 

右键项目--> 添加框架支持 --> Web

添加成功后悔出现web包并且 **蓝色圆点**

![image-20200417090636187](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200417090636187.png)



#### 1.1.3 所需文件

* HelloServlet 继承**HttpServlet** 实现**doGet** 和 **doPost**

  ```java
  package com.bohan.servlet;
  
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  public class HelloServlet extends HttpServlet {
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //获取前端参数
          String method = req.getParameter("method");
          if(method.equals("add")){
              req.getSession().setAttribute("msg", "add");
          }
          if(method.equals("delete")){
              req.getSession().setAttribute("msg", "delete");
          }
          //调用业务层
  
          //视图转发或者重定向
          req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req, resp);
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  
  ```

* 跳转的视图

  ```jsp
  <%--
    Created by IntelliJ IDEA.
    User: bohanxiao
    Date: 2020-04-17
    Time: 08:52
    To change this template use File | Settings | File Templates.
  --%>
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  	${msg}
  </body>
  </html>
  
  ```

* 配置servlet web.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
      
    <!-- 绑定servlet的名字和类 -->
      <servlet>
          <servlet-name>hello</servlet-name>
          <servlet-class>com.bohan.servlet.HelloServlet</servlet-class>
      </servlet>
    <!-- 绑定名字和前端call的api名字 -->
      <servlet-mapping>
          <servlet-name>hello</servlet-name>
          <url-pattern>/hello</url-pattern>
      </servlet-mapping>
    <!-- 设置主页 -->
      <welcome-file-list>
          <welcome-file>index.jsp</welcome-file>
      </welcome-file-list>
  </web-app>
  ```

* 测试 配置TomCat（配置里面添加，部署）直接在url上拼接api + 参数



## 2. 简介

* 底层基于servlet
* 功能强大 简单灵活



## 3. 原理讲解例子

这个例子只是用于讲解原理实际开发用注解十分简单。

### 3.1 所需文件

#### 3.1.1 HelloController（controller）

* 实现Controller接口

* 这里实现接受参数后的业务处理和视图的封装

* 返回的**ModelAndView** 是数据+视图

  ```java
  package com.bohan;
  
  import org.springframework.web.servlet.ModelAndView;
  import org.springframework.web.servlet.mvc.Controller;
  
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  
  public class HelloController implements Controller {
      public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
  
          ModelAndView mv = new ModelAndView();
          //业务代码
          String result = "HelloSpringMVC";
          mv.addObject("msg", result);
          //视图挑战
          mv.setViewName("test");
  
          return mv;
      }
  }
  
  ```

#### 3.1.2 spring-servlet.xml （spring配置文件）

* 类似mybatis 和 spring 都有自己的配置文件

* 配置三个核心

  * 处理器映射器 BeanNameUrlHandlerMapping

  * 处理适配器 SimpleControllerHandlerAdapter

  * 视图解析器 InternalResourceViewResolver

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:mvc="http://www.springframework.org/schema/mvc"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    
    
        <!-- 处理器映射器 -->
        <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
        <!-- 处理适配器 -->
        <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
        <!-- 视图解析器 -->
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver" >
            <property name="prefix" value="/WEB-INF/jsp/"/>
            <property name="suffix" value=".jsp"/>
         </bean>
    <!-- 注册 托管给 spring -->
        <bean id="/hello" class="com.bohan.HelloController"/>
    </beans>
    ```

  * 这三个部分是固定代码

* 注册配置好的servlet

#### 3.1.3 test.jsp(view)

```jsp
<%--
  Created by IntelliJ IDEA.
  User: bohanxiao
  Date: 2020-04-17
  Time: 09:43
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>

```



#### 3.1.4 配置web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <!-- 配置 dispatchServlet: springMVC的核心 -->
    
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!-- 配置绑定的配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>

        <!--配置启动级别 -->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!-- / 和 /* 是有区别的
    /: 值匹配请求 不匹配jsp
    /*: 包括jsp界面
    -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

基本都是固定代码 只是需要绑定一下配置文件 -- **springmvc-servlet.xml**

### 3.2 **原理重点**

![IMG_1764](/Users/bohanxiao/Downloads/IMG_1764.PNG)

* **虚线**是需要自己配置，**实线**是内部处理好的
* 虚线对应的4个步骤
  * Controller 业务处理 根据需求调用Service层
  * Controller 封装model数据（vo
  * 封装完毕后返回给前端等待view页面使用
  * ViewResolver来识别处理对应的view

#### 3.2.1 总体步骤分析

1. 用户访问 url + 参数，被**Dispatcher**拦截
2. 分析url里面访问的api，提交给**HandlerMapping**
3. **HandlerMapping**根据url找到正确的**controller** **HandlerExcution**
4. 返回给**Despatcher**
5. 将获得的**Controller**映射路径和参数传递给 **HandlerAdapter** 来处理
6. **HandlerAdapter** 调用对应的 **Controller** （虚线1）
7. **Controller**处理好业务封装好返回的数据和跳转的视图 **ModelAndView**，并且返回给 **HandlerAdapter**
8. **HandlerAdapter** 返回给 **Dispatcher**
9. **Dispatcher** 访问 **ViewResolver** 来解析view 包括路径拼接
10. 返回给**Dispatcher**
11. 调用对应的 view
12. 返回给用户



## 4. 实际开发（annotation）

### 4.1 配置所需变化 

#### 4.1.1 三大核心只需要配置视图，固定代码处理视图解析器**小概率**可能需要**修改**

* 处理器映射器 BeanNameUrlHandlerMapping
* 处理适配器 SimpleControllerHandlerAdapter
* **视图解析器 InternalResourceViewResolver**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--自动扫描包 和过滤静态资源 开启注解交给IOC统一管理-->
    <context:component-scan base-package="com.bohan.controller"/>
    <mvc:default-servlet-handler/>

    <!-- 替代前两个控制器 -->
    <mvc:annotation-driven />

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

#### 4.1.2 Controller

```java
package com.bohan.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloController {

    @RequestMapping("/hello")
    public String hello(Model model){

        model.addAttribute("msg", "hello,SpringMVC annotation");
        return "hello"; //这个字符串会被视图解析器chuli
    }
}
```

需要注意的是 controller是视图解析 如果使用restController则会返回字符串通常用json

### 4.2 实际案例

#### 4.2.1导包

```xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
```

#### 4.2.2 配置

* 添加项目架构支持

  右键项目--> 添加框架支持 --> Web

  添加成功后悔出现web包并且 **蓝色圆点**

  ![image-20200417090636187](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200417090636187.png)

* 部署到Tomcat

* 创建lib包导入所有依赖（ide问题）



#### 4.2.3 代码

* Controller

  ```java
  package com.bohan.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.RequestMapping;
  
  @Controller
  public class HelloController {
  
      @RequestMapping("/hello")
      public String hello(Model model){
  
          model.addAttribute("msg", "hello,SpringMVC annotation");
          return "hello"; //这个字符串会被视图解析器chuli
      }
  }
  ```

* springmvc-servlet.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/mvc
          http://www.springframework.org/schema/mvc/spring-mvc.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
  
      <!--自动扫描包 和过滤静态资源 开启注解交给IOC统一管理-->
      <context:component-scan base-package="com.bohan.controller"/>
      <mvc:default-servlet-handler/>
  
      <!-- 替代一大堆控制器 -->
      <mvc:annotation-driven />
  
      <!-- 视图解析器 -->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
          <property name="prefix" value="/WEB-INF/jsp/" />
          <property name="suffix" value=".jsp"/>
      </bean>
  
  </beans>
  ```

* Web.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
  
      <!-- 配置 dispatchServlet: springMVC的核心 -->
  
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  
          <!-- 配置绑定的配置文件 -->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springmvc-servlet.xml</param-value>
          </init-param>
  
          <!--配置启动级别 -->
          <load-on-startup>1</load-on-startup>
      </servlet>
  
      <!-- / 和 /* 是有区别的
      /: 值匹配请求 不匹配jsp
      /*: 包括jsp界面
      -->
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
  </web-app>
  ```

* Hello.jsp

  ```jsp
  <%--
    Created by IntelliJ IDEA.
    User: bohanxiao
    Date: 2020-04-17
    Time: 11:04
    To change this template use File | Settings | File Templates.
  --%>
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  ${msg}
  </body>
  </html>
  ```

### 4.2.3 重定位和转发

默认就是转发，需要重定位只需要再前面加上 **redirect:**

```java
return "redirect:hello";
```

### 4.2.4 url上的参数问题

* 最好只要是需要前端手动传入的参数都用*@RequestParam*。
* 如果同名，则不需要 如果不同名则需要在*@RequestParam*里面写入前端传入的名字的。
* 前端传过来的参数可以用一个object来接收，会自动匹配前端传来的参数和object里面的属性进行匹配，有的则有 没有的则null。

### 4.3.5 中文乱码

#### 4.3.5.1使用过滤器解决

```java
package com.bohan.filter;

import javax.servlet.*;
import java.io.IOException;

public class EncodingFilter implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");

        filterChain.doFilter(servletRequest,servletResponse);
    }

    public void destroy() {

    }
}

```



```xml
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>com.bohan.filter.EncodingFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

#### 4.3.5.2 springmvc 自带的过滤器

```xml
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

## 5. JSON

### 5.1 统一处理json乱码

springmvc-servlet.xml 里面添加

```xml
    <!-- 处理json乱码 -->
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <!-- 处理请求返回json字符串的中文乱码问题 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="utf-8"/>
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                        <property name="failOnEmptyBeans" value="false" />
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```



### 5.2 Jackson 

导包

```xml
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.10.2</version>
        </dependency>
```



```java
package com.bohan.controller;

import com.bohan.pojo.User;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.ObjectWriter;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class UserController {

    @ResponseBody //不走视图解析器 直接返回字符串
    @RequestMapping("j1")
    public String jon1(){

        User user = new User(1,"bohan","male");
        
        //jackson转换
        ObjectMapper mapper = new ObjectMapper();

        String str = null;
        try {
            str = mapper.writeValueAsString(user);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        return str;
    }
}

```

### 5.3 Fastjson

导包

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.66</version>
        </dependency>
```

更简单

```java
String str = JSON.toJSONString(userList);
String str = JSON.parseObject(str,User.class);
```

