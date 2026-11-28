---
title: Mybatis
date: 2022-01-09 14:23:46
categories: 
  - 后端             #分类
tags:                       #标签
  - Java
  - Mybatis
---

# MyBatis笔记

环境：

- JDK 1.8
- MySQL 5.7
- Maven 3.6.1
- IDEA

回顾：JDBC、MySQL、Java 基础、Maven、Junit。

> SSM 框架：配置文件的。最好的方式：看官网文档。

---

## 1、简介

### 1.1、什么是 Mybatis

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。

MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。

MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

MyBatis 本是 apache 的一个开源项目 iBatis，2010 年这个项目由 apache software foundation 迁移到了 google code，并且改名为 MyBatis。2013 年 11 月迁移到 Github。

如何获得 Mybatis？

Maven 仓库：

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.2</version>
</dependency>
```

- Github：https://github.com/mybatis/mybatis-3/releases
- 中文文档：https://mybatis.org/mybatis-3/zh/index.html

### 1.2、持久化

数据持久化：持久化就是将程序的数据在持久状态和瞬时状态转化的过程。

- 内存：断电即失。
- 数据库（Jdbc）、io 文件持久化。
- 生活：冷藏、罐头。

为什么需要持久化？

- 有一些对象，不能让它丢掉。
- 内存太贵了。

### 1.3、持久层

Dao 层、Service 层、Controller 层……完成持久化工作的代码块，层界限十分明显。

### 1.4、为什么需要 Mybatis？

帮助程序猿将数据存入到数据库中，方便。传统的 JDBC 代码太复杂了，简化、框架、自动化。不用 Mybatis 也可以，更容易上手，技术没有高低之分。

优点：

- 简单易学。
- 灵活。
- SQL 和代码的分离，提高了可维护性。
- 提供映射标签，支持对象与数据库的 orm 字段关系映射。
- 提供对象关系映射标签，支持对象关系组件维护。
- 提供 xml 标签，支持编写动态 sql。

> 最重要的一点：使用的人多！

---

## 2、第一个 Mybatis 程序

思路：搭建环境 --> 导入 Mybatis --> 编写代码 --> 测试！

### 2.1、搭建环境

搭建数据库：

```sql
CREATE DATABASE `mybatis`;

USE `mybatis`;

CREATE TABLE `user`(
  `id` INT(20) NOT NULL PRIMARY KEY,
  `name` VARCHAR(30) DEFAULT NULL,
  `pwd` VARCHAR(30) DEFAULT NULL
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user`(`id`,`name`,`pwd`) VALUES
(1,'狂神','123456'),
(2,'张三','123456'),
(3,'李四','123890');
```

新建项目：新建一个普通的 maven 项目，删除 src 目录，导入 maven 依赖：

```xml
<!-- 导入依赖 -->
<dependencies>
    <!-- mysql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <!-- mybatis -->
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>
    <!-- junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

### 2.2、创建一个模块

编写 mybatis 的核心配置文件 `mybatis-config.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- configuration 核心配置文件 -->
<configuration>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

</configuration>
```

编写 mybatis 工具类：

```java
// sqlSessionFactory --> sqlSession
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            // 使用 Mybatis 第一步：获取 sqlSessionFactory 对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    // 既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了。
    // SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。
    public static SqlSession  getSqlSession(){
        return sqlSessionFactory.openSession();
    }

}
```

### 2.3、编写代码

实体类：

```java
package com.kuang.pojo;

// 实体类
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

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
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

Dao 接口：

```java
public interface UserDao {
    List<User> getUserList();
}
```

接口实现类由原来的 UserDaoImpl 转变为一个 Mapper 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace=绑定一个对应的 Dao/Mapper 接口 -->
<mapper namespace="com.kuang.dao.UserDao">

    <!-- select 查询语句 -->
    <select id="getUserList" resultType="com.kuang.pojo.User">
        select * from mybatis.user
    </select>

</mapper>
```

### 2.4、测试

注意点：

```
org.apache.ibatis.binding.BindingException: Type interface com.kuang.dao.UserDao is not known to the MapperRegistry.
```

MapperRegistry 是什么？核心配置文件中注册 mappers。

junit 测试：

```java
@Test
public void test(){
    // 第一步：获得 SqlSession 对象
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    // 方式一：getMapper
    UserDao userDao = sqlSession.getMapper(UserDao.class);
    List<User> userList = userDao.getUserList();

    for (User user : userList) {
        System.out.println(user);
    }

    // 关闭 SqlSession
    sqlSession.close();
}
```

你们可能会遇到的问题：

- 配置文件没有注册。
- 绑定接口错误。
- 方法名不对。
- 返回类型不对。
- Maven 导出资源问题。

---

## 3、CRUD

### 1、namespace

namespace 中的包名要和 Dao/mapper 接口的包名一致！

### 2、select

选择，查询语句：

- `id`：就是对应的 namespace 中的方法名。
- `resultType`：Sql 语句执行的返回值。
- `parameterType`：参数类型。

编写接口：

```java
// 根据 ID 查询用户
User getUserById(int id);
```

编写对应的 mapper 中的 sql 语句：

```xml
<select id="getUserById" parameterType="int" resultType="com.kuang.pojo.User">
    select * from mybatis.user where id = #{id}
</select>
```

测试：

```java
@Test
public void getUserById() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user = mapper.getUserById(1);
    System.out.println(user);

    sqlSession.close();
}
```

### 3、Insert

```xml
<!-- 对象中的属性，可以直接取出来 -->
<insert id="addUser" parameterType="com.kuang.pojo.User">
    insert into mybatis.user (id, name, pwd) values (#{id},#{name},#{pwd});
</insert>
```

### 4、update

```xml
<update id="updateUser" parameterType="com.kuang.pojo.User">
    update mybatis.user set name=#{name},pwd=#{pwd}  where id = #{id} ;
</update>
```

### 5、Delete

```xml
<delete id="deleteUser" parameterType="int">
    delete from mybatis.user where id = #{id};
</delete>
```

> 注意点：增删改需要提交事务！

### 6、分析错误

- 标签不要匹配错。
- resource 绑定 mapper，需要使用路径！
- 程序配置文件必须符合规范！
- NullPointerException，没有注册到资源！
- 输出的 xml 文件中存在中文乱码问题！
- maven 资源没有导出问题！

### 7、万能 Map

假设，我们的实体类，或者数据库中的表，字段或者参数过多，我们应当考虑使用 Map！

```java
// 万能的 Map
int addUser2(Map<String,Object> map);
```

```xml
<!-- 对象中的属性，可以直接取出来    传递 map 的 key -->
<insert id="addUser" parameterType="map">
    insert into mybatis.user (id, pwd) values (#{userid},#{passWord});
</insert>
```

```java
@Test
public void addUser2() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    Map<String, Object> map = new HashMap<String, Object>();

    map.put("userid",5);
    map.put("passWord","2222333");

    mapper.addUser2(map);

    sqlSession.close();
}
```

- Map 传递参数，直接在 sql 中取出 key即可！【`parameterType="map"`】
- 对象传递参数，直接在 sql 中取对象的属性即可！【`parameterType="Object"`】
- 只有一个基本类型参数的情况下，可以直接在 sql 中取到！
- 多个参数用 Map，或者注解！

### 8、思考题

模糊查询怎么写？

Java 代码执行的时候，传递通配符 `%` `%`：

```java
List<User> userList = mapper.getUserLike("%李%");
```

在 sql 拼接中使用通配符！

```sql
select * from mybatis.user where name like "%"#{value}"%"
```

---

## 4、配置解析

### 1、核心配置文件

`mybatis-config.xml`，MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息：

```
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

### 2、环境配置（environments）

MyBatis 可以配置成适应多种环境。不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。学会使用配置多套运行环境！

Mybatis 默认的事务管理器就是 JDBC，连接池：POOLED。

### 3、属性（properties）

我们可以通过 properties 属性来实现引用配置文件。这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。【db.properties】

编写一个配置文件 `db.properties`：

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8
username=root
password=123456
```

在核心配置文件中引入：

```xml
<!-- 引入外部配置文件 -->
<properties resource="db.properties">
    <property name="username" value="root"/>
    <property name="pwd" value="11111"/>
</properties>
```

- 可以直接引入外部文件。
- 可以在其中增加一些属性配置。
- 如果两个文件有同一个字段，优先使用外部配置文件的！

### 4、类型别名（typeAliases）

类型别名是为 Java 类型设置一个短的名字。存在的意义仅在于用来减少类完全限定名的冗余。

```xml
<!-- 可以给实体类起别名 -->
<typeAliases>
    <typeAlias type="com.kuang.pojo.User" alias="User"/>
</typeAliases>
```

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：扫描实体类的包，它的默认别名就为这个类的类名，首字母小写！

```xml
<!-- 可以给实体类起别名 -->
<typeAliases>
    <package name="com.kuang.pojo"/>
</typeAliases>
```

- 在实体类比较少的时候，使用第一种方式。
- 如果实体类十分多，建议使用第二种。
- 第一种可以 DIY 别名，第二种则不行，如果非要改，需要在实体上增加注解：

```java
@Alias("user")
public class User {}
```

### 5、设置

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

### 6、其他配置

- typeHandlers（类型处理器）：https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers
- objectFactory（对象工厂）：https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory
- plugins 插件：mybatis-generator-core、mybatis-plus、通用 mapper。

### 7、映射器（mappers）

MapperRegistry：注册绑定我们的 Mapper 文件。

方式一：【推荐使用】

```xml
<!-- 每一个 Mapper.XML 都需要在 Mybatis 核心配置文件中注册！ -->
<mappers>
    <mapper resource="com/kuang/dao/UserMapper.xml"/>
</mappers>
```

方式二：使用 class 文件绑定注册：

```xml
<!-- 每一个 Mapper.XML 都需要在 Mybatis 核心配置文件中注册！ -->
<mappers>
    <mapper class="com.kuang.dao.UserMapper"/>
</mappers>
```

注意点：接口和他的 Mapper 配置文件必须同名！接口和他的 Mapper 配置文件必须在同一个包下！

方式三：使用扫描包进行注入绑定：

```xml
<!-- 每一个 Mapper.XML 都需要在 Mybatis 核心配置文件中注册！ -->
<mappers>
    <package name="com.kuang.dao"/>
</mappers>
```

注意点：接口和他的 Mapper 配置文件必须同名！接口和他的 Mapper 配置文件必须在同一个包下！

练习时间：

- 将数据库配置文件外部引入。
- 实体类别名。
- 保证 UserMapper 接口和 UserMapper.xml 改为一致！并且放在同一个包下！

### 8、生命周期和作用域

生命周期和作用域是至关重要的，因为错误的使用会导致非常严重的并发问题。

**SqlSessionFactoryBuilder**：一旦创建了 SqlSessionFactory，就不再需要它了，局部变量。

**SqlSessionFactory**：说白了就是可以想象为数据库连接池。SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。因此 SqlSessionFactory 的最佳作用域是应用作用域。最简单的就是使用单例模式或者静态单例模式。

**SqlSession**：连接到连接池的一个请求！SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。用完之后需要赶紧关闭，否则资源被占用！

这里面的每一个 Mapper，就代表一个具体的业务！

---

## 5、解决属性名和字段名不一致的问题

### 1、问题

数据库中的字段。新建一个项目，拷贝之前的，测试实体类字段不一致的情况：

```java
public class User {

    private int id;
    private String name;
    private String password;
}
```

测试出现问题。

```java
//   select * from mybatis.user where id = #{id}
// 类型处理器
//   select id,name,pwd from mybatis.user where id = #{id}
```

解决方法：起别名。

```xml
<select id="getUserById" resultType="com.kuang.pojo.User">
    select id,name,pwd as password from mybatis.user where id = #{id}
</select>
```

### 2、resultMap

结果集映射：

```
id   name   pwd
id   name   password
```

```xml
<!-- 结果集映射 -->
<resultMap id="UserMap" type="User">
    <!-- column 数据库中的字段，property 实体类中的属性 -->
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>

<select id="getUserById" resultMap="UserMap">
    select * from mybatis.user where id = #{id}
</select>
```

resultMap 元素是 MyBatis 中最重要最强大的元素。ResultMap 的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。

ResultMap 最优秀的地方在于，虽然你已经对它相当了解了，但是根本就不需要显式地用到它们。如果世界总是这么简单就好了。

---

## 6、日志

### 6.1、日志工厂

如果一个数据库操作出现了异常，我们需要排错，日志就是最好的助手！曾经：sout、debug；现在：日志工厂！

- SLF4J
- LOG4J【掌握】
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING
- STDOUT_LOGGING【掌握】
- NO_LOGGING

在 Mybatis 中具体使用哪个日志实现，在设置中设定！

`STDOUT_LOGGING` 标准日志输出。在 mybatis 核心配置文件中，配置我们的日志：

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

### 6.2、Log4j

什么是 Log4j？Log4j 是 Apache 的一个开源项目，通过使用 Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI 组件；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

先导入 log4j 的包：

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

`log4j.properties`：

```properties
# 将等级为 DEBUG 的日志信息输出到 console 和 file 这两个目的地，console 和 file 的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

# 控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

# 文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/kuang.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

# 日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

配置 log4j 为日志的实现：

```xml
<settings>
    <setting name="logImpl" value=""/>
</settings>
```

Log4j 的使用！直接测试运行刚才的查询。

简单使用：在要使用 Log4j 的类中，导入包 `import org.apache.log4j.Logger;`，日志对象，参数为当前类的 class：

```java
static Logger logger = Logger.getLogger(UserDaoTest.class);
```

日志级别：

```java
logger.info("info:进入了testLog4j");
logger.debug("debug:进入了testLog4j");
logger.error("error:进入了testLog4j");
```

---

## 7、分页

思考：为什么要分页？减少数据的处理量。

### 7.1、使用 Limit 分页

语法：

```sql
SELECT * from user limit startIndex,pageSize;
SELECT * from user limit 3;  -- [0,n]
```

使用 Mybatis 实现分页，核心 SQL。

接口：

```java
// 分页
List<User> getUserByLimit(Map<String,Integer> map);
```

Mapper.xml：

```xml
<!-- //分页 -->
<select id="getUserByLimit" parameterType="map" resultMap="UserMap">
    select * from  mybatis.user limit #{startIndex},#{pageSize}
</select>
```

测试：

```java
@Test
public void getUserByLimit() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    HashMap<String, Integer> map = new HashMap<String, Integer>();
    map.put("startIndex",1);
    map.put("pageSize",2);

    List<User> userList =  mapper.getUserByLimit(map);
    for (User user : userList) {
        System.out.println(user);
    }

    sqlSession.close();
}
```

### 7.2、RowBounds 分页

不再使用 SQL 实现分页。

接口：

```java
// 分页2
List<User> getUserByRowBounds();
```

mapper.xml：

```xml
<!-- 分页2 -->
<select id="getUserByRowBounds" resultMap="UserMap">
    select * from  mybatis.user
</select>
```

测试：

```java
@Test
public void getUserByRowBounds() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    // RowBounds 实现
    RowBounds rowBounds = new RowBounds(1, 2);

    // 通过 Java 代码层面实现分页
    List<User> userList = sqlSession.selectList("com.kuang.dao.UserMapper.getUserByRowBounds",null,rowBounds);

    for (User user : userList) {
        System.out.println(user);
    }

    sqlSession.close();
}
```

### 7.3、分页插件

![image-20261128162956366](../../static/Mybatis/image-20261128162956366.png)

PageHelper：[https://pagehelper.github.io](https://pagehelper.github.io)

了解即可，万一以后公司的架构师说要使用，你需要知道它是什么东西！

---

## 8、使用注解开发

### 8.1、面向接口编程

- 大家之前都学过面向对象编程，也学习过接口，但在真正的开发中，很多时候我们会选择面向接口编程。
- 根本原因：解耦，可拓展，提高复用；分层开发中，上层不用管具体的实现，大家都遵守共同的标准，使得开发变得容易，规范性更好。
- 在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的。在这种情况下，各个对象内部是如何实现自己的，对系统设计人员来讲就不那么重要了；而各个对象之间的协作关系则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是指按照这种思想来编程。

关于接口的理解：

- 接口从更深层次的理解，应是定义（规范，约束）与实现（名实分离的原则）的分离。
- 接口的本身反映了系统设计人员对系统的抽象理解。
- 接口应有两类：第一类是对一个个体的抽象，它可对应为一个抽象体（abstract class）；第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）。
- 一个体有可能有多个抽象面。抽象体与抽象面是有区别的。

三个面向区别：

- 面向对象是指，我们考虑问题时，以对象为单位，考虑它的属性及方法。
- 面向过程是指，我们考虑问题时，以一个具体的流程（事务过程）为单位，考虑它的实现。
- 接口设计与非接口设计是针对复用技术而言的，与面向对象（过程）不是一个问题，更多的体现就是对系统整体的架构。

### 8.2、使用注解开发

注解在接口上实现：

```java
@Select("select * from user")
List<User> getUsers();
```

需要在核心配置文件中绑定接口：

```xml
<!-- 绑定接口 -->
<mappers>
    <mapper class="com.kuang.dao.UserMapper"/>
</mappers>
```

测试。本质：反射机制实现；底层：动态代理！

### 8.3、CRUD

我们可以在工具类创建的时候实现自动提交事务！

```java
public static SqlSession  getSqlSession(){
    return sqlSessionFactory.openSession(true);
}
```

编写接口，增加注解：

```java
public interface UserMapper {

    @Select("select * from user")
    List<User> getUsers();

    // 方法存在多个参数，所有的参数前面必须加上 @Param("id") 注解
    @Select("select * from user where id = #{id}")
    User getUserByID(@Param("id") int id);

    @Insert("insert into user(id,name,pwd) values (#{id},#{name},#{password})")
    int addUser(User user);

    @Update("update user set name=#{name},pwd=#{password} where id = #{id}")
    int updateUser(User user);

    @Delete("delete from user where id = #{uid}")
    int deleteUser(@Param("uid") int id);
}
```

测试类：【注意：我们必须要将接口注册绑定到我们的核心配置文件中！】

关于 `@Param()` 注解：

- 基本类型的参数或者 String 类型，需要加上。
- 引用类型不需要加。
- 如果只有一个基本类型的话，可以忽略，但是建议大家都加上！
- 我们在 SQL 中引用的就是我们这里的 `@Param()` 中设定的属性名！

`#{}` 与 `${}` 区别。

---

## 9、Lombok

> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java. Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.

使用步骤：

1. 在 IDEA 中安装 Lombok 插件！
2. 在项目中导入 lombok 的 jar 包：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
</dependency>
```

3. 在实体类上加注解即可！

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
```

常用注解：

```
@Getter and @Setter
@FieldNameConstants
@ToString
@EqualsAndHashCode
@AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
@Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger
@Data
@Builder
@Singular
@Delegate
@Value
@Accessors
@Wither
@SneakyThrows
```

说明：

- `@Data`：无参构造，get、set、toString、hashcode、equals。
- `@AllArgsConstructor`
- `@NoArgsConstructor`
- `@EqualsAndHashCode`
- `@ToString`
- `@Getter`

---

## 10、多对一处理

多对一：多个学生，对应一个老师。对于学生这边而言，关联——多个学生，关联一个老师【多对一】；对于老师而言，集合，一个老师，有很多学生【一对多】。

SQL：

```sql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师');

CREATE TABLE `student` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  `tid` INT(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fktid` (`tid`),
  CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8


INSERT INTO `student`(`id`, `name`, `tid`) VALUES ('1', '小明', '1');
INSERT INTO `student`(`id`, `name`, `tid`) VALUES ('2', '小红', '1');
INSERT INTO `student`(`id`, `name`, `tid`) VALUES ('3', '小张', '1');
INSERT INTO `student`(`id`, `name`, `tid`) VALUES ('4', '小李', '1');
INSERT INTO `student`(`id`, `name`, `tid`) VALUES ('5', '小王', '1');
```

测试环境搭建：导入 lombok；新建实体类 Teacher、Student；建立 Mapper 接口；建立 Mapper.xml 文件；在核心配置文件中绑定注册我们的 Mapper 接口或者文件！【方式很多，随心选】；测试查询是否能够成功！

### 按照查询嵌套处理

```xml
<!--
    思路：
        1. 查询所有的学生信息
        2. 根据查询出来的学生的 tid，寻找对应的老师！  子查询
-->
<select id="getStudent" resultMap="StudentTeacher">
    select * from student
</select>

<resultMap id="StudentTeacher" type="Student">
    <result property="id" column="id"/>
    <result property="name" column="name"/>
    <!-- 复杂的属性，我们需要单独处理 对象： association  集合： collection -->
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
</resultMap>

<select id="getTeacher" resultType="Teacher">
    select * from teacher where id = #{id}
</select>
```

### 按照结果嵌套处理

```xml
<!-- 按照结果嵌套处理 -->
<select id="getStudent2" resultMap="StudentTeacher2">
    select s.id sid,s.name sname,t.name tname
    from student s,teacher t
    where s.tid = t.id;
</select>

<resultMap id="StudentTeacher2" type="Student">
    <result property="id" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="Teacher">
        <result property="name" column="tname"/>
    </association>
</resultMap>
```

回顾 Mysql 多对一查询方式：子查询、联表查询。

---

## 11、一对多处理

比如：一个老师拥有多个学生！对于老师而言，就是一对多的关系！

环境搭建：和刚才一样。

实体类：

```java
@Data
public class Student {

    private int id;
    private String name;
    private int tid;

}
```

```java
@Data
public class Teacher {
    private int id;
    private String name;

    // 一个老师拥有多个学生
    private List<Student> students;
}
```

### 按照结果嵌套处理

```xml
<!-- 按结果嵌套查询 -->
<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid, s.name sname, t.name tname,t.id tid
    from student s,teacher t
    where s.tid = t.id and t.id = #{tid}
</select>

<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <!-- 复杂的属性，我们需要单独处理 对象： association  集合： collection
        javaType="" 指定属性的类型！
        集合中的泛型信息，我们使用 ofType 获取
    -->
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```

### 按照查询嵌套处理

```xml
<select id="getTeacher2" resultMap="TeacherStudent2">
    select * from mybatis.teacher where id = #{tid}
</select>

<resultMap id="TeacherStudent2" type="Teacher">
    <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
</resultMap>

<select id="getStudentByTeacherId" resultType="Student">
    select * from mybatis.student where tid = #{tid}
</select>
```

### 小结

- 关联——association【多对一】
- 集合——collection【一对多】
- javaType & ofType
  - JavaType 用来指定实体类中属性的类型。
  - ofType 用来指定映射到 List 或者集合中的 pojo 类型，泛型中的约束类型！

注意点：

- 保证 SQL 的可读性，尽量保证通俗易懂。
- 注意一对多和多对一中，属性名和字段的问题！
- 如果问题不好排查错误，可以使用日志，建议使用 Log4j。
- 慢 SQL：1s、1000s。

面试高频：Mysql 引擎、InnoDB 底层原理、索引、索引优化！

---

## 12、动态 SQL

什么是动态 SQL：动态 SQL 就是指根据不同的条件生成不同的 SQL 语句。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

动态 SQL 元素和 JSTL 或基于类似 XML 的文本处理器相似。在 MyBatis 之前的版本中，有很多元素需要花时间了解。MyBatis 3 大大精简了元素种类，现在只需学习原来一半的元素便可。MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。

```
if
choose (when, otherwise)
trim (where, set)
foreach
```

### 搭建环境

```sql
CREATE TABLE `blog`(
  `id` varchar(50) NOT NULL COMMENT '博客id',
  `title` varchar(100) NOT NULL COMMENT '博客标题',
  `author` varchar(30) NOT NULL COMMENT '博客作者',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `views` int(30) NOT NULL COMMENT '浏览量'
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

创建一个基础工程：导包；编写配置文件；编写实体类；编写实体类对应 Mapper 接口和 Mapper.xml 文件。

```java
@Data
public class Blog {
    private int id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```

### IF

```xml
<select id="queryBlogIF" parameterType="map" resultType="blog">
    select * from mybatis.blog where 1=1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```

### choose (when, otherwise)

```xml
<select id="queryBlogChoose" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

### trim (where, set)

```xml
select * from mybatis.blog
<where>
    <if test="title != null">
        title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</where>
```

```xml
<update id="updateBlog" parameterType="map">
    update mybatis.blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author}
        </if>
    </set>
    where id = #{id}
</update>
```

所谓的动态 SQL，本质还是 SQL 语句，只是我们可以在 SQL 层面去执行一个逻辑代码：if、where、set、choose、when。

### SQL 片段

有的时候，我们可能会将一些功能的部分抽取出来，方便复用！

使用 SQL 标签抽取公共的部分：

```xml
<sql id="if-title-author">
    <if test="title != null">
        title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</sql>
```

在需要使用的地方使用 Include 标签引用即可：

```xml
<select id="queryBlogIF" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <include refid="if-title-author"></include>
    </where>
</select>
```

注意事项：最好基于单表来定义 SQL 片段！不要存在 where 标签。

### Foreach

```sql
select * from user where 1=1 and

<foreach item="id" collection="ids"
         open="(" separator="or" close=")">
    #{id}
</foreach>

(id=1 or id=2 or id=3)
```

```xml
<!--
    select * from mybatis.blog where 1=1 and (id=1 or id = 2 or id=3)

    我们现在传递一个万能的 map，这 map 中可以存在一个集合！
-->
<select id="queryBlogForeach" parameterType="map" resultType="blog">
    select * from mybatis.blog

    <where>
        <foreach collection="ids" item="id" open="and (" close=")" separator="or">
            id = #{id}
        </foreach>
    </where>

</select>
```

动态 SQL 就是在拼接 SQL 语句，我们只要保证 SQL 的正确性，按照 SQL 的格式去排列组合就可以了。

建议：现在 Mysql 中写出完整的 SQL，再对应的去修改成为我们的动态 SQL 实现通用即可！

---

## 13、缓存（了解）

### 13.1、简介

查询：连接数据库，耗资源！一次查询的结果，给他暂存在一个可以直接取到的地方！--> 内存：缓存。我们再次查询相同数据的时候，直接走缓存，就不用走数据库了。

什么是缓存 [ Cache ]？存在内存中的临时数据。将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库数据文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。

为什么使用缓存？减少和数据库的交互次数，减少系统开销，提高系统效率。

什么样的数据能使用缓存？经常查询并且不经常改变的数据。【可以使用缓存】

### 13.2、Mybatis 缓存

MyBatis 包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存，缓存可以极大的提升查询效率。

MyBatis 系统中默认定义了两级缓存：一级缓存和二级缓存。

- 默认情况下，只有一级缓存开启。（SqlSession 级别的缓存，也称为本地缓存）
- 二级缓存需要手动开启和配置，它是基于 namespace 级别的缓存。
- 为了提高扩展性，MyBatis 定义了缓存接口 Cache，我们可以通过实现 Cache 接口来自定义二级缓存。

### 13.3、一级缓存

一级缓存也叫本地缓存：SqlSession。与数据库同一次会话期间查询到的数据会放在本地缓存中。以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库。

测试步骤：开启日志！测试在一个 Session 中查询两次相同记录；查看日志输出。

缓存失效的情况：

- 查询不同的东西。
- 增删改操作，可能会改变原来的数据，所以必定会刷新缓存！
- 查询不同的 Mapper.xml。
- 手动清理缓存！

小结：一级缓存默认是开启的，只在一次 SqlSession 中有效，也就是拿到连接到关闭连接这个区间段！一级缓存就是一个 Map。

### 13.4、二级缓存

二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存。基于 namespace 级别的缓存，一个名称空间，对应一个二级缓存。

工作机制：一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中；新的会话查询信息，就可以从二级缓存中获取内容；不同的 mapper 查出的数据会放在自己对应的缓存（map）中。

步骤：

开启全局缓存：

```xml
<!-- 显式的开启全局缓存 -->
<setting name="cacheEnabled" value="true"/>
```

在要使用二级缓存的 Mapper 中开启：

```xml
<!-- 在当前 Mapper.xml 中使用二级缓存 -->
<cache/>
```

也可以自定义参数：

```xml
<!-- 在当前 Mapper.xml 中使用二级缓存 -->
<cache  eviction="FIFO"
        flushInterval="60000"
        size="512"
        readOnly="true"/>
```

测试。问题：我们需要将实体类序列化！否则就会报错！

```
Caused by: java.io.NotSerializableException: com.kuang.pojo.User
```

小结：

- 只要开启了二级缓存，在同一个 Mapper 下就有效。
- 所有的数据都会先放在一级缓存中。
- 只有当会话提交，或者关闭的时候，才会提交到二级缓存中！

### 13.5、缓存原理

（略）

### 13.6、自定义缓存 - ehcache

Ehcache 是一种广泛使用的开源 Java 分布式缓存，主要面向通用缓存。

要在程序中使用 ehcache，先要导包：

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```

在 mapper 中指定使用我们的 ehcache 缓存实现：

```xml
<!-- 在当前 Mapper.xml 中使用二级缓存 -->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

`ehcache.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/21/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <!--
       diskStore：为缓存路径，ehcache 分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
       user.home – 用户主目录
       user.dir  – 用户当前工作目录
       java.io.tmpdir – 默认临时文件路径
     -->
    <diskStore path="./tmpdir/Tmp_EhCache"/>

    <defaultCache
            eternal="false"
            maxElementsInMemory="10000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="259200"
            memoryStoreEvictionPolicy="LRU"/>

    <cache
            name="cloud_user"
            eternal="false"
            maxElementsInMemory="5000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            memoryStoreEvictionPolicy="LRU"/>
    <!--
       defaultCache：默认缓存策略，当 ehcache 找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
    -->
    <!--
       name：缓存名称。
       maxElementsInMemory：缓存最大数目
       maxElementsOnDisk：硬盘最大缓存个数。
       eternal：对象是否永久有效，一但设置了，timeout 将不起作用。
       overflowToDisk：是否保存到磁盘，当系统当机时
       timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当 eternal=false 对象不是永久有效时使用，可选属性，默认值是 0，也就是可闲置时间无穷大。
       timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当 eternal=false 对象不是永久有效时使用，默认是 0.，也就是对象存活时间无穷大。
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
       diskSpoolBufferSizeMB：这个参数设置 DiskStore（磁盘缓存）的缓存区大小。默认是 30MB。每个 Cache 都应该有自己的一个缓冲区。
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是 120 秒。
       memoryStoreEvictionPolicy：当达到 maxElementsInMemory 限制时，Ehcache 将会根据指定的策略去清理内存。默认策略是 LRU（最近最少使用）。你可以设置为 FIFO（先进先出）或是 LFU（较少使用）。
       clearOnFlush：内存数量最大时是否清除。
       memoryStoreEvictionPolicy：可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
       FIFO，first in first out，这个是大家最熟的，先进先出。
       LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个 hit 属性，hit 值最小的将会被清出缓存。
       LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
    -->

</ehcache>
```

