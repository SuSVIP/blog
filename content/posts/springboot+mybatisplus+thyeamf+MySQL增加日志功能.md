---
title: Springboot+mybatisplus+thyeamf+MySQL增加日志功能
date: 2025-09-18T04:09:17+08:00
draft: false
categories:
  - 技术分享
tags:
  - Java
  - SpringBoot
  - MyBatis Plus
  - Thymeleaf
  - MySQL
---

基于 SpringBoot、MyBatis Plus、Thymeleaf 和 MySQL 的项目中增加日志功能，这是一个非常常见且实用的需求。日志功能通常包含**系统运行日志**（记录程序运行状态、错误等）和**业务操作日志**（记录用户关键操作）。

### 一、整体实现思路

1. **基础日志（系统日志）**：使用 SpringBoot 内置的 SLF4J + Logback 实现，配置日志输出格式、级别、存储路径等。

2. **业务日志（操作日志）**：

   - 设计 MySQL 日志表存储操作记录（用户、操作、时间、IP、模块等）。
   - 基于 MyBatis Plus 实现日志的 CRUD。
   - 用自定义注解 + AOP 实现日志的自动记录，避免重复代码。
   - 结合 Thymeleaf 实现日志的页面展示。

   

### 二、具体实现步骤

#### 1. 基础依赖（pom.xml）

确保项目引入以下核心依赖（已包含 SpringBoot、MyBatis Plus、Thymeleaf、MySQL 基础依赖的基础上，补充日志相关）：

```xml
<!-- SpringBoot 核心 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.15</version>
    <relativePath/>
</parent>

<dependencies>
    <!-- SpringBoot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- MyBatis Plus -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>
    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <!-- Thymeleaf -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <!-- AOP（用于业务日志） -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <!-- 工具类（获取IP等） -->
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.22</version>
    </dependency>
    <!-- Lombok（简化代码） -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

#### 2. 配置系统日志（Logback）

SpringBoot 默认使用 Logback，无需额外引入依赖，只需在 `resources` 下创建 `logback-spring.xml` 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 日志输出格式 -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
    <!-- 日志存储路径（可自定义） -->
    <property name="LOG_PATH" value="logs/app"/>

    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 文件输出（按天拆分，保留30天） -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 错误日志单独输出 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 日志级别：ERROR > WARN > INFO > DEBUG > TRACE -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

    <!-- 自定义包的日志级别（比如DAO层打印SQL） -->
    <logger name="com.baomidou.mybatisplus" level="DEBUG"/>
    <logger name="com.yourpackage.mapper" level="DEBUG"/>
</configuration>
```

#### 3. 实现业务操作日志

##### 3.1 设计 MySQL 日志表

```sql
CREATE TABLE `sys_oper_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `oper_user` varchar(50) NOT NULL COMMENT '操作人',
  `oper_module` varchar(50) NOT NULL COMMENT '操作模块（如：用户管理、订单管理）',
  `oper_type` varchar(20) NOT NULL COMMENT '操作类型（如：新增、修改、删除）',
  `oper_desc` varchar(200) DEFAULT NULL COMMENT '操作描述',
  `oper_ip` varchar(50) DEFAULT NULL COMMENT '操作IP',
  `oper_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '操作时间',
  `request_url` varchar(200) DEFAULT NULL COMMENT '请求URL',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统操作日志表';
```

##### 3.2 实体类（SysOperLog.java）

```java
package com.yourpackage.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("sys_oper_log")
public class SysOperLog {
    /** 主键ID */
    @TableId(type = IdType.AUTO)
    private Long id;
    /** 操作人 */
    private String operUser;
    /** 操作模块 */
    private String operModule;
    /** 操作类型（新增、修改、删除） */
    private String operType;
    /** 操作描述 */
    private String operDesc;
    /** 操作IP */
    private String operIp;
    /** 操作时间 */
    private LocalDateTime operTime;
    /** 请求URL */
    private String requestUrl;
}
```

##### 3.3 Mapper 层（SysOperLogMapper.java）

```java
package com.yourpackage.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.yourpackage.entity.SysOperLog;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface SysOperLogMapper extends BaseMapper<SysOperLog> {
}
```

##### 3.4 自定义日志注解（OperLog.java）

```java
package com.yourpackage.annotation;

import java.lang.annotation.*;

/**
 * 自定义操作日志注解
 */
@Target(ElementType.METHOD) // 注解作用在方法上
@Retention(RetentionPolicy.RUNTIME) // 运行时生效
@Documented
public @interface OperLog {
    /** 操作模块 */
    String module() default "";
    /** 操作类型 */
    String type() default "";
    /** 操作描述 */
    String desc() default "";
}
```

##### 3.5 AOP 切面实现日志记录（OperLogAspect.java）

```java
package com.yourpackage.aspect;

import cn.hutool.extra.servlet.ServletUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.yourpackage.annotation.OperLog;
import com.yourpackage.entity.SysOperLog;
import com.yourpackage.mapper.SysOperLogMapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.time.LocalDateTime;

/**
 * 操作日志切面
 */
@Slf4j
@Aspect
@Component
@RequiredArgsConstructor
public class OperLogAspect {

    private final SysOperLogMapper operLogMapper;

    // 切入点：匹配带有@OperLog注解的方法
    @Pointcut("@annotation(com.yourpackage.annotation.OperLog)")
    public void operLogPointCut() {}

    // 后置通知：方法正常执行后记录日志
    @AfterReturning(pointcut = "operLogPointCut()", returning = "result")
    public void saveOperLog(JoinPoint joinPoint, Object result) {
        try {
            // 1. 获取请求上下文
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();

            // 2. 获取注解信息
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Method method = signature.getMethod();
            OperLog operLogAnnotation = method.getAnnotation(OperLog.class);

            // 3. 构建日志实体
            SysOperLog operLog = new SysOperLog();
            operLog.setOperModule(operLogAnnotation.module()); // 操作模块
            operLog.setOperType(operLogAnnotation.type());     // 操作类型
            operLog.setOperDesc(operLogAnnotation.desc());     // 操作描述
            operLog.setOperIp(ServletUtil.getClientIP(request)); // 操作IP
            operLog.setRequestUrl(request.getRequestURI());    // 请求URL
            operLog.setOperTime(LocalDateTime.now());          // 操作时间
            // 这里假设你有用户登录功能，可替换为实际登录用户名
            operLog.setOperUser("admin"); // 临时写死，实际需从Token/Session获取

            // 4. 保存日志到数据库
            operLogMapper.insert(operLog);
        } catch (Exception e) {
            log.error("记录操作日志失败", e);
        }
    }
}
```

##### 3.6 日志查询接口（LogController.java）

```java
package com.yourpackage.controller;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.yourpackage.annotation.OperLog;
import com.yourpackage.entity.SysOperLog;
import com.yourpackage.mapper.SysOperLogMapper;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
@RequiredArgsConstructor
public class LogController {

    private final SysOperLogMapper operLogMapper;

    // 日志列表页面
    @GetMapping("/log/list")
    @OperLog(module = "日志管理", type = "查询", desc = "查看操作日志列表")
    public String logList(Model model,
                          @RequestParam(defaultValue = "1") Integer pageNum,
                          @RequestParam(defaultValue = "10") Integer pageSize) {
        // 分页查询日志
        IPage<SysOperLog> page = new Page<>(pageNum, pageSize);
        IPage<SysOperLog> logPage = operLogMapper.selectPage(page, null);
        
        model.addAttribute("logPage", logPage);
        return "log/list"; // 对应Thymeleaf模板路径：templates/log/list.html
    }
}
```

##### 3.7 Thymeleaf 日志展示页面（log/list.html）

```html
<!DOCTYPE html>
<html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>操作日志列表</title>
    <style>
        table {border-collapse: collapse; width: 90%; margin: 20px auto;}
        th, td {border: 1px solid #ccc; padding: 8px; text-align: center;}
        th {background-color: #f5f5f5;}
        .pagination {text-align: center; margin-top: 20px;}
        .pagination a {margin: 0 5px; text-decoration: none;}
    </style>
</head>
<body>
    <h2 style="text-align: center;">系统操作日志</h2>
    <table>
        <tr>
            <th>ID</th>
            <th>操作人</th>
            <th>操作模块</th>
            <th>操作类型</th>
            <th>操作描述</th>
            <th>操作IP</th>
            <th>操作时间</th>
            <th>请求URL</th>
        </tr>
        <tr th:each="log : ${logPage.records}">
            <td th:text="${log.id}"></td>
            <td th:text="${log.operUser}"></td>
            <td th:text="${log.operModule}"></td>
            <td th:text="${log.operType}"></td>
            <td th:text="${log.operDesc}"></td>
            <td th:text="${log.operIp}"></td>
            <td th:text="${#dates.format(log.operTime, 'yyyy-MM-dd HH:mm:ss')}"></td>
            <td th:text="${log.requestUrl}"></td>
        </tr>
    </table>

    <div class="pagination">
        <a th:href="@{/log/list(pageNum=1, pageSize=${logPage.size})}">首页</a>
        <a th:href="@{/log/list(pageNum=${logPage.current-1}, pageSize=${logPage.size})}" 
           th:if="${logPage.current > 1}">上一页</a>
        <span th:text="'当前第 ' + ${logPage.current} + ' 页，共 ' + ${logPage.pages} + ' 页'"></span>
        <a th:href="@{/log/list(pageNum=${logPage.current+1}, pageSize=${logPage.size})}" 
           th:if="${logPage.current < logPage.pages}">下一页</a>
        <a th:href="@{/log/list(pageNum=${logPage.pages}, pageSize=${logPage.size})}">尾页</a>
    </div>
</body>
</html>
```

#### 4. 配置 MyBatis Plus 分页插件（可选，用于日志分页）

```java
package com.yourpackage.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyBatisPlusConfig {

    // 分页插件
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

#### 5. 测试日志功能

1. 启动项目，访问 `http://localhost:8080/log/list`。
2. 操作后会在数据库 `sys_oper_log` 表中生成日志记录，同时页面会展示日志列表。
3. 系统日志会输出到控制台和 `logs/app` 目录下的文件中。

### 三、关键补充说明

1. **用户信息获取**：示例中 `operUser` 写死为 `admin`，实际项目中需从登录态（如 Session、Token、Spring Security/Spring Cloud Security）中获取当前用户名。
2. **日志级别调整**：可在 `logback-spring.xml` 中修改 `<root level="INFO">` 调整全局日志级别，或单独配置某个包的级别（如 DAO 层 DEBUG 打印 SQL）。
3. **日志脱敏**：如果日志中包含敏感信息（如手机号、身份证），需在保存前进行脱敏处理。
4. **日志清理**：可定时清理过期日志（如通过 MyBatis Plus 定时任务删除 3 个月前的日志），避免数据库表过大。

### 总结

1. **系统日志**：通过 Logback 配置实现，支持控制台 + 文件输出，可自定义格式和存储路径。
2. **业务日志**：基于自定义注解 + AOP 实现无侵入式记录，结合 MyBatis Plus 存储到 MySQL，Thymeleaf 展示。
3. 核心优势：AOP 方式避免了在业务代码中重复编写日志逻辑，日志记录与业务解耦，易维护、易扩展。
