---
title: SpringMVC02：第一个MVC程序
date: 2026-11-28T15:29:37+08:00
draft: false
categories: 
  - 后端             #分类
tags:                       #标签
  - Java
  - SpringMVC
---

# SpringMVC02：第一个 MVC 程序

在上一节中，我们讲解了什么是 SpringMVC 以及它的执行原理！



现在我们来看看如何快速使用 SpringMVC 编写我们的程序吧！

---

## 配置版

1. 新建一个 Module，`springmvc-02-hello`，添加 web 的支持！
2. 确定导入了 SpringMVC 的依赖！
3. 配置 `web.xml`，注册 DispatcherServlet：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 1.注册 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 关联一个 springmvc 的配置文件：【servlet-name】-servlet.xml -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!-- 启动级别-1 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- / 匹配所有的请求；（不包括 .jsp） -->
    <!-- /* 匹配所有的请求；（包括 .jsp） -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

4. 编写 SpringMVC 的配置文件！名称：`springmvc-servlet.xml`：`[servletname]-servlet.xml`。说明，这里的名称要求是按照官方来的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

5. 添加处理映射器：

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

6. 添加处理器适配器：

```xml
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

7. 添加视图解析器：

```xml
<!-- 视图解析器：DispatcherServlet 给他的 ModelAndView -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
    <!-- 前缀 -->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!-- 后缀 -->
    <property name="suffix" value=".jsp"/>
</bean>
```

8. 编写我们要操作业务 Controller，要么实现 Controller 接口，要么增加注解；需要返回一个 ModelAndView，装数据，封视图：

```java
package com.kuang.controller;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

// 注意：这里我们先导入 Controller 接口
public class HelloController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        // ModelAndView 模型和视图
        ModelAndView mv = new ModelAndView();

        // 封装对象，放在 ModelAndView 中。Model
        mv.addObject("msg", "HelloSpringMVC!");
        // 封装要跳转的视图，放在 ModelAndView 中
        mv.setViewName("hello"); // : /WEB-INF/jsp/hello.jsp
        return mv;
    }
}
```

9. 将自己的类交给 Spring IOC 容器，注册 bean：

```xml
<!-- Handler -->
<bean id="/hello" class="com.kuang.controller.HelloController"/>
```

10. 写要跳转的 jsp 页面，显示 ModelAndView 存放的数据，以及我们的正常页面：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Kuangshen</title>
</head>
<body>
${msg}
</body>
</html>
```

11. 配置 Tomcat 启动测试！

可能遇到的问题：访问出现 404，排查步骤：

- 查看控制台输出，看一下是不是缺少了什么 jar 包；
- 如果 jar 包存在，显示无法输出，就在 IDEA 的项目发布中，添加 lib 依赖！
- 重启 Tomcat 即可解决！

小结：看这个估计大部分同学都能理解其中的原理了，但是我们实际开发才不会这么写，不然就疯了，还学这个玩意干嘛！我们来看个注解版实现，这才是 SpringMVC 的精髓，到底有多么简单，看这个图就知道了。

---

## 注解版

1. 新建一个 Module，`springmvc-03-hello-annotation`，添加 web 支持！
2. 由于 Maven 可能存在资源过滤的问题，我们将配置完善：

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

3. 在 `pom.xml` 文件引入相关的依赖：主要有 Spring 框架核心库、Spring MVC、servlet、JSTL 等。我们在父依赖中已经引入了！
4. 配置 `web.xml`，注意点：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 1.注册 servlet -->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 通过初始化参数指定 SpringMVC 配置文件的位置，进行关联 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!-- 启动顺序，数字越小，启动越早 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- 所有请求都会被 springmvc 拦截 -->
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

`/` 和 `/*` 的区别：

- `<url-pattern>/</url-pattern>` 不会匹配到 `.jsp`，只针对我们编写的请求；即：`.jsp` 不会进入 spring 的 DispatcherServlet 类。
- `<url-pattern>/*</url-pattern>` 会匹配 `*.jsp`，会出现返回 jsp 视图时再次进入 spring 的 DispatcherServlet 类，导致找不到对应的 controller 所以报 404 错。

注意 `web.xml` 版本问题，要最新版！

- 注册 DispatcherServlet
- 关联 SpringMVC 的配置文件
- 启动级别为 1
- 映射路径为 `/`【不要用 `/*`，会 404】

5. 添加 Spring MVC 配置文件

在 resource 目录下添加 `springmvc-servlet.xml` 配置文件，配置的形式与 Spring 容器配置基本类似，为了支持基于注解的 IOC，设置了自动扫描包的功能，具体配置信息如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 自动扫描包，让指定包下的注解生效，由 IOC 容器统一管理 -->
    <context:component-scan base-package="com.kuang.controller"/>
    <!-- 让 Spring MVC 不处理静态资源 -->
    <mvc:default-servlet-handler/>
    <!--
        支持 mvc 注解驱动
        在 spring 中一般采用 @RequestMapping 注解来完成映射关系
        要想使 @RequestMapping 注解生效
        必须向上下文中注册 DefaultAnnotationHandlerMapping
        和一个 AnnotationMethodHandlerAdapter 实例
        这两个实例分别在类级别和方法级别处理。
        而 annotation-driven 配置帮助我们自动完成上述两个实例的注入。
    -->
    <mvc:annotation-driven/>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          id="internalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!-- 后缀 -->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

在视图解析器中我们把所有的视图都存放在 `/WEB-INF/` 目录下，这样可以保证视图安全，因为这个目录下的文件，客户端不能直接访问。

- 让 IOC 的注解生效
- 静态资源过滤：HTML、JS、CSS、图片，视频……
- MVC 的注解驱动
- 配置视图解析器

6. 创建 Controller

编写一个 Java 控制类：`com.kuang.controller.HelloController`，注意编码规范：

```java
package com.kuang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/HelloController")
public class HelloController {

    // 真实访问地址：项目名/HelloController/hello
    @RequestMapping("/hello")
    public String sayHello(Model model) {
        // 向模型中添加属性 msg 与值，可以在 JSP 页面中取出并渲染
        model.addAttribute("msg", "hello,SpringMVC");
        // web-inf/jsp/hello.jsp
        return "hello";
    }
}
```

- `@Controller` 是为了让 Spring IOC 容器初始化时自动扫描到；
- `@RequestMapping` 是为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是 `/HelloController/hello`；
- 方法中声明 Model 类型的参数是为了把 Action 中的数据带到视图中；
- 方法返回的结果是视图的名称 `hello`，加上配置文件中的前后缀变成 `WEB-INF/jsp/hello.jsp`。

7. 创建视图层

在 `WEB-INF/jsp` 目录中创建 `hello.jsp`，视图可以直接取出并展示从 Controller 带回的信息；可以通过 EL 表示取出 Model 中存放的值，或者对象：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>SpringMVC</title>
</head>
<body>
${msg}
</body>
</html>
```

8. 配置 Tomcat 运行

配置 Tomcat，开启服务器，访问对应的请求路径！

OK，运行成功！

---

## 小结

实现步骤其实非常的简单：

1. 新建一个 web 项目；
2. 导入相关 jar 包；
3. 编写 `web.xml`，注册 DispatcherServlet；
4. 编写 springmvc 配置文件；
5. 接下来就是去创建对应的控制类，controller；
6. 最后完善前端视图和 controller 之间的对应；
7. 测试运行调试。

使用 springMVC 必须配置的三大件：

- 处理器映射器
- 处理器适配器
- 视图解析器

通常，我们只需要手动配置视图解析器，而处理器映射器和处理器适配器只需要开启注解驱动即可，而省去了大段的 xml 配置。

再来回顾下原理吧 ~