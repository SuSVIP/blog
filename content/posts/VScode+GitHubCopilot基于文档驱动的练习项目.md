---
title: VScode+GitHubCopilot基于文档驱动的练习项目
date: 2025-09-16T04:03:28+08:00
draft: false
categories:
  - 技术分享
tags:
  - Vs code
  - Java
  - GitHub Copilot
---

### 🎯 一、项目整体规划

我们要在 `Java-Basics-Practice` 文件夹下，搭建一个**基于文档驱动（SDD）+ GitHub Copilot** 的 Java 知识点练习项目，完全遵循大模型编程方法论。

#### 1. 项目核心目标

- 按 Java 知识点分模块练习（基础语法、面向对象、集合、IO、并发等）
- 用 `SKILL.md` + Prompt 模板约束 Copilot 行为
- 用 `Spec-Driven Development` 流程：先写文档 → 再生成代码 → 再验收测试
- 所有代码和文档都纳入 Git 管理

------

### 📂 二、完整目录结构（推荐）

plaintext

```
Java-Basics-Practice/
├── .gitignore                 # Git 忽略文件
├── README.md                  # 项目总览（给人看的说明书）
├── SKILL.md                   # Copilot 技能配置（AI 的“大脑”）
├── prompts/                   # 提示词模板库
│   ├── copilot.md             # GitHub Copilot 专属 Prompt 模板
│   └── sdd.md                 # 基于文档开发的 Prompt 模板
├── docs/                      # 所有需求/规格文档（SDD 核心：文档即源码）
│   ├── 01-basic-syntax/       # 知识点 1：基础语法
│   │   └── spec.md            # 该知识点的详细规格文档
│   ├── 02-oop/                # 知识点 2：面向对象
│   │   └── spec.md
│   ├── 03-collection/         # 知识点 3：集合框架
│   │   └── spec.md
│   └── ...                    # 后续可扩展更多知识点模块
└── src/                       # Java 源代码（由文档编译生成）
    ├── main/
    │   └── java/
    │       └── com/
    │           └── example/
    │               ├── basics/     # 基础语法代码
    │               ├── oop/        # 面向对象代码
    │               └── collection/ # 集合代码
    └── test/
        └── java/
            └── com/
                └── example/
                    ├── basics/     # 基础语法测试
                    ├── oop/        # 面向对象测试
                    └── collection/ # 集合测试
```

------

### 📝 三、核心文件内容详解

#### 1. `.gitignore`（标准 Java 项目忽略规则）

```gitignore
# Compiled class file
*.class

# Log file
*.log
logs/

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
replay_pid*

# Maven
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

# IntelliJ IDEA
.idea/
*.iws
*.iml
*.ipr

# VS Code
.vscode/

# OS
.DS_Store
Thumbs.db
```

------

#### 2. `README.md`（项目总览）

```markdown
# Java-Basics-Practice

基于 **文档驱动开发（SDD）+ GitHub Copilot** 的 Java 基础知识点练习项目。

## 核心理念
- 文档即源码：所有需求先写 `spec.md`，再由 Copilot 生成代码
- Skill 驱动：用 `SKILL.md` 约束 Copilot 的编码风格、输出格式和工作流
- 渐进式学习：按知识点分模块，每个模块包含「文档 → 代码 → 测试」

## 目录说明
- `docs/`: 所有知识点的规格文档（SDD 核心）
- `src/main/java/`: 业务代码
- `src/test/java/`: 单元测试代码
- `prompts/`: Prompt 模板库
- `SKILL.md`: Copilot 技能配置

## 快速开始
1.  安装 VS Code + GitHub Copilot + Extension Pack for Java
2.  打开本项目，在 `docs/[知识点]/spec.md` 中编写需求
3.  用 `prompts/sdd.md` 模板唤起 Copilot，生成对应代码
4.  运行测试验证结果
```

------

#### 3. `SKILL.md`（Copilot 的 “大脑”）



```markdown
# Java-Basics-Practice Skill

## 角色定义
你是一位资深 Java 教学工程师，专注于 Java 基础知识点教学与代码示例生成。

## 目标与范围
- 为 `Java-Basics-Practice` 项目生成**符合 Java 编码规范**的基础代码示例
- 严格遵循 **Spec-Driven Development**：只根据 `docs/[模块]/spec.md` 生成代码，不凭空猜测
- 每个知识点模块必须包含：**业务代码 + JUnit 5 单元测试**

## 工作方式
1.  先读取 `spec.md`，确认需求、输入输出、约束条件
2.  生成代码前先给出**简要设计方案**（类结构、核心方法）
3.  再生成完整代码（含中文注释，解释「Why」而非仅「How」）
4.  最后生成对应的 JUnit 5 测试用例
5.  代码必须可直接编译运行，无语法错误

## 禁止事项
- 禁止在没有 `spec.md` 支撑的情况下直接生成代码
- 禁止一次修改超过 3 个文件
- 禁止忽略异常处理和边界情况
- 禁止使用未说明的第三方库

## 输出格式
- 代码使用 ```java ``` 代码块包裹
- 测试代码使用 ```java ``` 代码块包裹
- 关键逻辑必须加中文注释，说明设计思路
- 最终给出「运行说明」和「测试验证步骤」

## 项目特定习惯
- 包名统一使用 `com.example.[模块名]`（如 `com.example.basics`）
- 测试类名 = 业务类名 + `Test`（如 `StringUtilTest.java`）
- 日志框架：使用 `System.out` 或 JUnit 5 断言，不引入额外日志库
- 编码：UTF-8，缩进 4 空格
```

------

#### 4. `prompts/copilot.md`（Copilot 专属 Prompt 模板）



```markdown
# GitHub Copilot Prompt 模板

## 基础指令
请读取本项目的 `SKILL.md`，严格遵循其中的角色定义、工作方式和输出格式。

## 当前任务
我正在学习 Java 知识点：`{知识点名称}`，对应的规格文档是 `docs/{知识点目录}/spec.md`。

请完成以下工作：
1.  读取 `docs/{知识点目录}/spec.md`，理解需求
2.  生成符合规范的 Java 业务代码
3.  生成对应的 JUnit 5 单元测试代码
4.  给出运行和验证步骤

## 约束
- 代码必须放在 `src/main/java/com/example/{模块名}/` 下
- 测试代码必须放在 `src/test/java/com/example/{模块名}/` 下
- 所有类、方法必须加中文注释
- 测试必须覆盖正常场景和边界异常场景
```

------

#### 5. `prompts/sdd.md`（文档驱动开发 Prompt 模板）

```markdown
# SDD 工作流 Prompt

我将遵循 **Spec-Driven Development** 流程，请你作为我的 AI 代码生成助手。

## 当前阶段：AI 编译（Implementation）
我已经在 `docs/{模块名}/spec.md` 中完成了需求定义和人工确认。

请你：
1.  严格按照 `spec.md` 中的 Steps、接口定义和数据流生成代码
2.  代码结构必须与文档完全一致
3.  输出 Diff 格式或完整文件内容
4.  同时生成对应的单元测试

## 关键提醒
- 如果文档描述不清晰，**必须先向我提问澄清**，禁止猜测
- 代码修改后，必须同步更新 `spec.md` 中的相关描述
- 永远保持「文档是唯一可信源」
```

------

#### 6. 示例：`docs/01-basic-syntax/spec.md`（第一个知识点文档）

```markdown
# 知识点：Java 基础语法

## 意图
学习 Java 基础语法：变量、数据类型、运算符、流程控制（if/for/while）。

## 需求
1.  实现一个 `BasicSyntaxDemo` 类，包含以下方法：
    - `calculate(int a, int b)`：实现加减乘除运算，返回结果字符串
    - `checkEven(int num)`：判断数字是否为偶数，返回 boolean
    - `printNumbers(int n)`：打印 1~n 的数字，用 for 循环实现
2.  所有方法必须处理输入为负数的情况，抛出 `IllegalArgumentException`
3.  编写 JUnit 5 测试用例，覆盖：
    - 正常输入场景
    - 负数输入异常场景
    - 边界值（n=0, n=1）

## 步骤
1.  创建 `com.example.basics.BasicSyntaxDemo` 类
2.  实现 `calculate` 方法，支持 `+ - * /` 四种运算
3.  实现 `checkEven` 方法
4.  实现 `printNumbers` 方法
5.  创建 `com.example.basics.BasicSyntaxDemoTest` 测试类
6.  编写测试方法验证所有场景

## 约束
- 不使用任何第三方库
- 代码必须兼容 Java 8+
- 方法名、变量名遵循驼峰命名法
```

------

### 🚀 四、VS Code + GitHub Copilot 实操流程

#### 1. 环境准备

1. 安装 VS Code
2. 安装插件：
   - **Extension Pack for Java**（Java 核心支持）
   - **GitHub Copilot**（AI 代码补全）
   - **GitHub Copilot Chat**（对话式 AI）
3. 打开 `Java-Basics-Practice` 文件夹

#### 2. 初始化项目

1. 创建上述所有文件和文件夹

2. 初始化 Git：

   ```bash
   git init
   git add .
   git commit -m "feat: init project structure"
   ```

   

#### 3. 第一个知识点开发（SDD 流程）

1. **Phase 1：文档生成**

   - 在 `docs/01-basic-syntax/spec.md` 中写好需求
   - 人工 review 并确认文档

   

2. **Phase 2：AI 编译**

   - 打开 Copilot Chat，粘贴 `prompts/sdd.md` 模板
   - 替换 `{模块名}` 为 `01-basic-syntax`
   - 让 Copilot 生成 `BasicSyntaxDemo.java` 和 `BasicSyntaxDemoTest.java`

   

3. **Phase 3：文档验收**

   - 运行测试：`mvn test` 或 VS Code 测试面板
   - 如果测试失败，回到 `spec.md` 修改描述，再让 Copilot 重新生成
   - 确保文档和代码始终一致

   

#### 4. 扩展新知识点

- 复制 `docs/01-basic-syntax/` 为 `docs/02-oop/`
- 修改 `spec.md` 为面向对象相关需求
- 重复 SDD 流程，生成对应代码和测试

------

### ✅ 四、示例代码输出（由 Copilot 生成）

#### `src/main/java/com/example/basics/BasicSyntaxDemo.java`

```java
package com.example.basics;

/**
 * Java 基础语法演示类
 * 包含加减乘除运算、奇偶判断、数字打印等基础方法
 */
public class BasicSyntaxDemo {

    /**
     * 计算两个数的加减乘除结果
     * @param a 第一个数
     * @param b 第二个数
     * @return 运算结果字符串，格式为 "a + b = result"
     * @throws IllegalArgumentException 如果输入为负数
     */
    public String calculate(int a, int b) {
        if (a < 0 || b < 0) {
            throw new IllegalArgumentException("输入数字不能为负数");
        }
        StringBuilder sb = new StringBuilder();
        sb.append(a).append(" + ").append(b).append(" = ").append(a + b).append("\n");
        sb.append(a).append(" - ").append(b).append(" = ").append(a - b).append("\n");
        sb.append(a).append(" * ").append(b).append(" = ").append(a * b).append("\n");
        if (b != 0) {
            sb.append(a).append(" / ").append(b).append(" = ").append((double) a / b).append("\n");
        } else {
            sb.append(a).append(" / 0 = 除数不能为0").append("\n");
        }
        return sb.toString();
    }

    /**
     * 判断数字是否为偶数
     * @param num 待判断的数字
     * @return true-偶数，false-奇数
     * @throws IllegalArgumentException 如果输入为负数
     */
    public boolean checkEven(int num) {
        if (num < 0) {
            throw new IllegalArgumentException("输入数字不能为负数");
        }
        return num % 2 == 0;
    }

    /**
     * 打印 1~n 的所有数字
     * @param n 最大数字
     * @throws IllegalArgumentException 如果 n < 1
     */
    public void printNumbers(int n) {
        if (n < 1) {
            throw new IllegalArgumentException("n 必须大于等于 1");
        }
        for (int i = 1; i <= n; i++) {
            System.out.println(i);
        }
    }
}
```

#### `src/test/java/com/example/basics/BasicSyntaxDemoTest.java`



|   中文章节名    |        英文目录名        |
| :-------------: | :----------------------: |
|  01 - 基础语法  |     01-basic-syntax      |
|  02 - 面向对象  |          02-oop          |
|  03 - 集合框架  |      03-collection       |
|  04 - 异常处理  |  04-exception-handling   |
|    05-IO 流     |       05-io-stream       |
|   06 - 多线程   |    06-multithreading     |
| 07 - 注解与反射 | 07-annotation-reflection |
|  08 - 网络编程  |  08-network-programming  |
| 09 - 数据库连接 |  09-database-connection  |
|  10 - 设计模式  |    10-design-pattern     |



```java
package com.example.basics;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class BasicSyntaxDemoTest {

    private final BasicSyntaxDemo demo = new BasicSyntaxDemo();

    @Test
    void calculate_ValidNumbers_ReturnsResult() {
        String result = demo.calculate(10, 2);
        assertTrue(result.contains("10 + 2 = 12"));
        assertTrue(result.contains("10 - 2 = 8"));
        assertTrue(result.contains("10 * 2 = 20"));
        assertTrue(result.contains("10 / 2 = 5.0"));
    }

    @Test
    void calculate_NegativeInput_ThrowsException() {
        assertThrows(IllegalArgumentException.class, () -> demo.calculate(-5, 3));
        assertThrows(IllegalArgumentException.class, () -> demo.calculate(5, -3));
    }

    @Test
    void checkEven_EvenNumber_ReturnsTrue() {
        assertTrue(demo.checkEven(4));
        assertTrue(demo.checkEven(0));
    }

    @Test
    void checkEven_OddNumber_ReturnsFalse() {
        assertFalse(demo.checkEven(3));
        assertFalse(demo.checkEven(7));
    }

    @Test
    void checkEven_NegativeInput_ThrowsException() {
        assertThrows(IllegalArgumentException.class, () -> demo.checkEven(-4));
    }

    @Test
    void printNumbers_ValidN_PrintsNumbers() {
        assertDoesNotThrow(() -> demo.printNumbers(5));
    }

    @Test
    void printNumbers_InvalidN_ThrowsException() {
        assertThrows(IllegalArgumentException.class, () -> demo.printNumbers(0));
        assertThrows(IllegalArgumentException.class, () -> demo.printNumbers(-1));
    }
}
```

------

### 💡 五、关键优势

1. **文档即源码**：所有需求都沉淀在 `spec.md`，换模型 / 新人也能快速复现
2. **Skill 约束**：`SKILL.md` 让 Copilot 始终符合你的编码规范和工作流
3. **可复用性**：Prompt 模板和目录结构可直接复用给其他语言 / 项目
4. **渐进式进化**：每次任务后可更新 `SKILL.md`，让 Copilot 越来越懂你

------