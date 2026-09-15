---
title: VScode+GitHubCopilot的agent的简单教程
date: 2025-09-17T04:05:49+08:00
draft: false
categories:
  - 技术分享
tags:
  - Vs code
  - Java
  - GitHub Copilot
---

下面我将以 **Java 开发学生成绩管理系统** 为例，带你完成 VS Code + GitHub Copilot Agent 的完整实操 Demo。这个 Demo 会覆盖 Java 项目的核心场景（Maven 结构、面向对象、异常处理、文件 IO），让你直观感受到 Copilot Agent 如何端到端完成 Java 开发任务。

------

## 前置条件（必看）

1. VS Code 版本 ≥ 1.90.0（2025+），已安装：

   - GitHub Copilot 插件（需订阅 Copilot Pro/Enterprise，Agent 功能仅付费版可用）
   - Java 相关插件：Extension Pack for Java（微软官方）、Maven for Java

   

2. 本地已配置 JDK 8+、Maven（确保 `java -version` 和 `mvn -v` 能正常输出）

3. Copilot Agent 入口：VS Code 右侧边栏「Copilot」→ 切换到「Agent」标签（不是普通 Chat）

------

## 完整 Demo：Java 学生成绩管理系统

### 步骤 1：初始化 Maven 项目（Agent 可自动完成，也可手动）

1. 新建文件夹 `java-score-manager`，用 VS Code 打开该文件夹；

2. 打开 Copilot Agent 面板，输入**初始化指令**（Agent 会自动创建 Maven 目录结构）：

   ```bash
   帮我初始化一个标准的 Maven 结构的 Java 项目，用于开发学生成绩管理系统：
   1. 项目坐标：groupId=com.example，artifactId=score-manager，version=1.0.0
2. JDK 版本：1.8
   3. 自动创建 src/main/java、src/main/resources、src/test/java 目录
4. 生成完整的 pom.xml，无需额外依赖（基础功能仅用 JDK 原生 API）
   ```xml

3. 发送指令后，Agent 会先规划步骤，确认后自动创建 `pom.xml` 和目录结构：

   ```bash
   执行步骤：
   Step 1: 创建 pom.xml 文件，配置 Maven 坐标和 JDK 版本
   Step 2: 创建标准的 Maven 目录结构
   Step 3: 验证目录结构合法性
### 步骤 2：核心需求指令（让 Agent 完成完整开发）

在 Agent 面板输入**详细的功能需求**（越具体，Agent 生成的代码越贴合）：

```
基于当前的 Maven 项目，帮我实现 Java 学生成绩管理系统，要求：
1. 核心功能：
   - 添加学生成绩（包含姓名、科目、分数）
   - 查询指定学生的所有成绩
   - 计算指定科目的班级平均分
   - 导出所有成绩到 src/main/resources/scores.csv 文件
2. 代码结构：
   - 实体类：com.example.entity.StudentScore（包含 name、subject、score 属性，getter/setter、toString）
   - 服务类：com.example.service.ScoreManagerService（实现所有核心功能，处理业务逻辑）
   - 异常类：com.example.exception.ScoreIllegalException（自定义异常，处理分数＜0 或 ＞100 的情况）
   - 主类：com.example.Main（控制台交互入口，提供菜单选择）
3. 额外要求：
   - 所有类添加详细注释，符合 Java 编码规范
   - 处理常见异常（分数非数字、学生不存在、文件写入失败等）
   - 控制台交互友好，输出清晰的提示信息
```



### 步骤 3：查看 Agent 自动生成的代码

#### 1. pom.xml（自动创建在根目录）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>score-manager</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- 无额外依赖，仅用 JDK 原生 API -->
    </dependencies>
</project>
```

#### 2. 自定义异常类：`src/main/java/com/example/exception/ScoreIllegalException.java`

```java
package com.example.exception;

/**
 * 自定义异常：分数不合法（＜0 或 ＞100）
 * @author Copilot Agent
 */
public class ScoreIllegalException extends RuntimeException {
    public ScoreIllegalException() {
        super();
    }

    public ScoreIllegalException(String message) {
        super(message);
    }
}
```

#### 3. 实体类：`src/main/java/com/example/entity/StudentScore.java`

```java
package com.example.entity;

/**
 * 学生成绩实体类
 * @author Copilot Agent
 */
public class StudentScore {
    // 学生姓名
    private String name;
    // 科目名称
    private String subject;
    // 成绩分数
    private double score;

    public StudentScore() {
    }

    public StudentScore(String name, String subject, double score) {
        this.name = name;
        this.subject = subject;
        this.score = score;
    }

    // Getter 和 Setter 方法
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSubject() {
        return subject;
    }

    public void setSubject(String subject) {
        this.subject = subject;
    }

    public double getScore() {
        return score;
    }

    public void setScore(double score) {
        this.score = score;
    }

    @Override
    public String toString() {
        return "StudentScore{" +
                "name='" + name + '\'' +
                ", subject='" + subject + '\'' +
                ", score=" + score +
                '}';
    }
}
```

#### 4. 服务类（核心逻辑）：`src/main/java/com/example/service/ScoreManagerService.java`

```java
package com.example.service;

import com.example.entity.StudentScore;
import com.example.exception.ScoreIllegalException;

import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

/**
 * 成绩管理服务类，实现核心业务逻辑
 * @author Copilot Agent
 */
public class ScoreManagerService {
    // 存储所有学生成绩
    private List<StudentScore> scoreList = new ArrayList<>();

    /**
     * 添加学生成绩
     * @param name 学生姓名
     * @param subject 科目
     * @param score 分数（0-100）
     * @throws ScoreIllegalException 分数不合法时抛出
     */
    public void addScore(String name, String subject, double score) {
        // 分数合法性校验
        if (score < 0 || score > 100) {
            throw new ScoreIllegalException("分数不合法！分数必须在 0-100 之间");
        }
        // 避免空值
        if (name == null || name.trim().isEmpty() || subject == null || subject.trim().isEmpty()) {
            throw new IllegalArgumentException("姓名和科目不能为空！");
        }
        scoreList.add(new StudentScore(name, subject, score));
    }

    /**
     * 查询指定学生的所有成绩
     * @param name 学生姓名
     * @return 该学生的所有成绩列表
     */
    public List<StudentScore> queryScoreByName(String name) {
        return scoreList.stream()
                .filter(score -> score.getName().equals(name))
                .collect(Collectors.toList());
    }

    /**
     * 计算指定科目的班级平均分
     * @param subject 科目名称
     * @return 该科目的平均分（无数据时返回 0）
     */
    public double calculateSubjectAverage(String subject) {
        List<StudentScore> subjectScores = scoreList.stream()
                .filter(score -> score.getSubject().equals(subject))
                .collect(Collectors.toList());
        
        if (subjectScores.isEmpty()) {
            return 0.0;
        }
        
        double total = subjectScores.stream()
                .mapToDouble(StudentScore::getScore)
                .sum();
        
        return Math.round(total / subjectScores.size() * 100.0) / 100.0; // 保留两位小数
    }

    /**
     * 导出所有成绩到 CSV 文件
     * @param filePath 文件路径（如 src/main/resources/scores.csv）
     * @throws IOException 文件写入失败时抛出
     */
    public void exportToCsv(String filePath) throws IOException {
        try (FileWriter writer = new FileWriter(filePath)) {
            // 写入 CSV 表头
            writer.write("姓名,科目,分数\n");
            // 写入所有成绩数据
            for (StudentScore score : scoreList) {
                writer.write(String.format("%s,%s,%.2f\n", 
                        score.getName(), score.getSubject(), score.getScore()));
            }
        }
    }
}
```

#### 5. 主类（控制台交互）：`src/main/java/com/example/Main.java`

```java
package com.example;

import com.example.entity.StudentScore;
import com.example.exception.ScoreIllegalException;
import com.example.service.ScoreManagerService;

import java.io.IOException;
import java.util.List;
import java.util.Scanner;

/**
 * 学生成绩管理系统入口类
 * @author Copilot Agent
 */
public class Main {
    public static void main(String[] args) {
        ScoreManagerService service = new ScoreManagerService();
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("===== 学生成绩管理系统 =====");
        while (true) {
            // 打印菜单
            System.out.println("\n请选择操作：");
            System.out.println("1. 添加学生成绩");
            System.out.println("2. 查询学生成绩");
            System.out.println("3. 计算科目平均分");
            System.out.println("4. 导出成绩到 CSV");
            System.out.println("5. 退出系统");
            System.out.print("输入操作编号（1-5）：");
            
            String choice = scanner.next();
            scanner.nextLine(); // 吸收换行符
            
            switch (choice) {
                case "1":
                    // 添加成绩
                    try {
                        System.out.print("输入学生姓名：");
                        String name = scanner.nextLine();
                        System.out.print("输入科目名称：");
                        String subject = scanner.nextLine();
                        System.out.print("输入成绩（0-100）：");
                        double score = scanner.nextDouble();
                        
                        service.addScore(name, subject, score);
                        System.out.println("成绩添加成功！");
                    } catch (ScoreIllegalException e) {
                        System.out.println("添加失败：" + e.getMessage());
                    } catch (IllegalArgumentException e) {
                        System.out.println("添加失败：" + e.getMessage());
                    } catch (Exception e) {
                        System.out.println("添加失败：输入格式错误，请输入数字分数！");
                    }
                    break;
                    
                case "2":
                    // 查询成绩
                    System.out.print("输入要查询的学生姓名：");
                    String queryName = scanner.nextLine();
                    List<StudentScore> scores = service.queryScoreByName(queryName);
                    
                    if (scores.isEmpty()) {
                        System.out.println("未找到该学生的成绩！");
                    } else {
                        System.out.println("===== " + queryName + " 的成绩 =====");
                        for (StudentScore s : scores) {
                            System.out.println("科目：" + s.getSubject() + "，分数：" + s.getScore());
                        }
                    }
                    break;
                    
                case "3":
                    // 计算平均分
                    System.out.print("输入要计算平均分的科目：");
                    String subject = scanner.nextLine();
                    double average = service.calculateSubjectAverage(subject);
                    
                    if (average == 0.0) {
                        System.out.println("该科目暂无成绩数据！");
                    } else {
                        System.out.println(subject + " 科目平均分：" + average);
                    }
                    break;
                    
                case "4":
                    // 导出 CSV
                    try {
                        String filePath = "src/main/resources/scores.csv";
                        service.exportToCsv(filePath);
                        System.out.println("成绩已成功导出到：" + filePath);
                    } catch (IOException e) {
                        System.out.println("导出失败：" + e.getMessage());
                    }
                    break;
                    
                case "5":
                    // 退出系统
                    System.out.println("感谢使用，系统退出！");
                    scanner.close();
                    System.exit(0);
                    break;
                    
                default:
                    System.out.println("输入错误，请选择 1-5 之间的编号！");
            }
        }
    }
}
```

### 步骤 4：让 Agent 验证并运行代码

1. 在 Agent 面板输入指令，让 Agent 帮你检查并给出运行步骤：

   ```bash
请检查当前项目的所有代码是否有语法错误或逻辑问题，并给出详细的运行步骤
   ```

   

2. Agent 会自动分析代码，然后给出**运行步骤**：

   ```bash
   代码验证结果：无语法错误，核心逻辑完整，异常处理覆盖全面。
   运行步骤：
   Step 1: 打开 VS Code 终端，进入项目根目录
   Step 2: 编译代码：mvn clean compile
   Step 3: 运行主类：mvn exec:java -Dexec.mainClass="com.example.Main"
   Step 4: 按控制台提示测试各功能（添加、查询、计算、导出）
```
   


### 步骤 5：手动运行验证

1. 打开 VS Code 终端（`Ctrl+``），执行编译命令：

   ```bash
mvn clean compile
```

   

2. 运行主类：

   ```bash
mvn exec:java -Dexec.mainClass="com.example.Main"
   ```

   

3. 测试功能（示例）：

   - 选择 1 → 输入姓名「张三」、科目「数学」、分数「95」→ 提示添加成功；
   - 选择 2 → 查询「张三」→ 显示数学 95 分；
   - 选择 3 → 计算「数学」平均分 → 显示 95.0；
   - 选择 4 → 导出 CSV → 在 `src/main/resources` 下生成 `scores.csv`；
   - 选择 5 → 退出系统。

   

### 步骤 6：让 Agent 迭代优化（可选）

如果想优化功能，直接给 Agent 指令即可，比如：

```bash
请优化当前系统：
1. 添加「删除指定学生的指定科目成绩」功能
2. 在控制台显示所有学生的所有成绩（新增菜单选项 6）
3. 优化 CSV 导出格式，添加导出时间
```

Agent 会自动修改 `ScoreManagerService.java` 和 `Main.java`，无需你手动改一行代码。

------

### 总结

1. **核心流程**：初始化 Maven 项目 → 给 Agent 提详细的 Java 需求（包含包结构、功能、规范）→ Agent 自动生成完整代码 → 验证运行 → 迭代优化。
2. **关键技巧**：给 Agent 的指令要明确「包路径、类职责、异常处理、交互方式」，Java 项目需指定 JDK 版本、Maven 坐标等规范。
3. **核心优势**：Agent 能感知 Maven 项目上下文，自动拆分「实体类→服务类→主类」的开发步骤，还能处理异常、生成注释，符合 Java 编码规范。
