# 🚀 Java 迁移开发工程师 - 极速上手指南
**项目背景**: 信用风险系统 Oracle → Snowflake 迁移
**培训周期**: 4周 (2025-12-01 至 2025-12-27)
**技术栈**: Java, Oracle SQL, Snowflake SQL, Git

---

## 🛠️ 环境准备（培训前完成）

### 重要说明
本培训使用 **完全免费** 的环境，无需任何费用。请在 **Day 1 之前** 完成以下环境配置。

---

### 1. Snowflake 免费试用账号

**注册步骤**:
1. 访问 [Snowflake 试用注册页面](https://signup.snowflake.com/)
2. 填写注册信息：
   - **Email**: 使用你的工作邮箱或个人邮箱
   - **Edition**: 选择 **Enterprise**（试用版包含所有功能）
   - **Cloud Provider**: 推荐选择 **AWS**（任何云平台均可）
   - **Region**: 选择距离你最近的区域（如：AWS Asia Pacific (Singapore) 或 AWS US East (N. Virginia)）
3. 点击 "CONTINUE" 并激活邮箱链接
4. 设置账号信息：
   - **Username**: 你的用户名（如：`zhang_san`）
   - **Password**: 设置强密码（至少 8 位，包含大小写字母和数字）
5. 完成注册后，记录以下信息（保存到 `candidates/{your_name}/credentials.txt`，**注意：不要提交到 Git！**）：
   ```
   Snowflake Account Locator: <your_account>.snowflakecomputing.com
   Username: <your_username>
   Password: <your_password>
   ```

**试用限制**:
- **免费期限**: 30 天
- **免费额度**: $400 USD 的使用额度（对于学习练习完全足够）
- **限制**: 试用期结束后，账号会被暂停（但可以联系 Snowflake 延期或重新注册）

**创建测试数据库和 Schema**:
登录 Snowflake Web UI 后，在 Worksheet 中执行：
```sql
-- 创建测试数据库
CREATE DATABASE MIGRATION_TRAINING;

-- 创建 Schema
CREATE SCHEMA MIGRATION_TRAINING.PRACTICE;

-- 切换上下文
USE DATABASE MIGRATION_TRAINING;
USE SCHEMA PRACTICE;

-- 创建测试表（示例）
CREATE TABLE EMPLOYEES (
    EMPLOYEE_ID NUMBER(10),
    FIRST_NAME VARCHAR(50),
    LAST_NAME VARCHAR(50),
    EMAIL VARCHAR(100),
    HIRE_DATE DATE,
    SALARY NUMBER(10,2)
);

-- 插入测试数据
INSERT INTO EMPLOYEES VALUES
(1, 'John', 'Doe', 'john.doe@example.com', '2023-01-15', 75000.00),
(2, 'Jane', 'Smith', 'jane.smith@example.com', '2023-03-20', 82000.00),
(3, 'Bob', 'Johnson', 'bob.j@example.com', '2023-06-10', 68000.00);

-- 验证数据
SELECT * FROM EMPLOYEES;
```

**配置 JDBC 连接信息**（用于 Java 程序）:
```
JDBC URL: jdbc:snowflake://<your_account>.snowflakecomputing.com/?db=MIGRATION_TRAINING&schema=PRACTICE&warehouse=COMPUTE_WH
Driver Class: net.snowflake.client.jdbc.SnowflakeDriver
Username: <your_username>
Password: <your_password>
```

---

### 2. Oracle 数据库免费环境

**方案 A: Oracle Database XE (推荐用于 Windows/Linux 本地开发)**

**下载与安装**:
1. 访问 [Oracle Database XE 下载页面](https://www.oracle.com/database/technologies/xe-downloads.html)
2. 下载 **Oracle Database 21c Express Edition**（免费版本）
   - Windows: `OracleXE213_Win64.zip` (~2.5 GB)
   - Linux: `oracle-database-xe-21c-1.0-1.ol7.x86_64.rpm`
3. 安装步骤（Windows 示例）：
   - 解压 ZIP 文件
   - 运行 `setup.exe`
   - 设置 **SYS 和 SYSTEM 密码**（如：`OraclePass123`）
   - 安装完成后，Oracle 服务会自动启动

**配置**:
安装完成后，在命令行中执行（Windows: 使用 SQL*Plus）：
```bash
# 连接到 Oracle（Windows 用户可使用 SQL*Plus）
sqlplus system/OraclePass123@localhost:1521/XE
```

在 SQL*Plus 中执行：
```sql
-- 创建测试用户
CREATE USER migration_user IDENTIFIED BY MigrationPass123;
GRANT CONNECT, RESOURCE, DBA TO migration_user;
GRANT UNLIMITED TABLESPACE TO migration_user;

-- 连接到新用户
CONNECT migration_user/MigrationPass123@localhost:1521/XE;

-- 创建测试表（对应 Snowflake 的 EMPLOYEES 表）
CREATE TABLE EMPLOYEES (
    EMPLOYEE_ID NUMBER(10),
    FIRST_NAME VARCHAR2(50),
    LAST_NAME VARCHAR2(50),
    EMAIL VARCHAR2(100),
    HIRE_DATE DATE,
    SALARY NUMBER(10,2)
);

-- 插入测试数据
INSERT INTO EMPLOYEES VALUES (1, 'John', 'Doe', 'john.doe@example.com', TO_DATE('2023-01-15', 'YYYY-MM-DD'), 75000.00);
INSERT INTO EMPLOYEES VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', TO_DATE('2023-03-20', 'YYYY-MM-DD'), 82000.00);
INSERT INTO EMPLOYEES VALUES (3, 'Bob', 'Johnson', 'bob.j@example.com', TO_DATE('2023-06-10', 'YYYY-MM-DD'), 68000.00);
COMMIT;

-- 验证数据
SELECT * FROM EMPLOYEES;
```

**JDBC 连接信息**:
```
JDBC URL: jdbc:oracle:thin:@localhost:1521:XE
Driver Class: oracle.jdbc.OracleDriver
Username: migration_user
Password: MigrationPass123
```

---

**方案 B: Oracle Cloud Free Tier (推荐用于无法本地安装的场景)**

**注册步骤**:
1. 访问 [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
2. 点击 "Start for free" 并注册账号（需要信用卡验证，但不会扣费）
3. 登录后，导航到 **Database → Autonomous Database**
4. 点击 "Create Autonomous Database"：
   - **Workload Type**: Transaction Processing
   - **Deployment Type**: Shared Infrastructure
   - **Database Name**: `MigrationDB`
   - **Admin Password**: 设置密码（如：`OracleCloud123#`）
   - 勾选 **"Always Free"** 选项（确保使用免费层）
5. 等待数据库创建完成（约 5 分钟）

**连接配置**:
1. 在 Autonomous Database 详情页，点击 **"DB Connection"**
2. 下载 **Wallet 文件**（`wallet_MigrationDB.zip`）
3. 解压 Wallet 文件到本地目录（如：`C:\oracle_wallet\`）
4. 获取连接字符串（在 DB Connection 页面复制 **TNS Names**）

**JDBC 连接信息**:
```
JDBC URL: jdbc:oracle:thin:@MigrationDB_high?TNS_ADMIN=C:/oracle_wallet/
Driver Class: oracle.jdbc.OracleDriver
Username: ADMIN
Password: OracleCloud123#
```

**在 Oracle Cloud 中创建测试表**:
使用 **SQL Developer Web**（在 Autonomous Database 详情页点击 "Database Actions" → "SQL"）:
```sql
-- 创建测试表
CREATE TABLE EMPLOYEES (
    EMPLOYEE_ID NUMBER(10),
    FIRST_NAME VARCHAR2(50),
    LAST_NAME VARCHAR2(50),
    EMAIL VARCHAR2(100),
    HIRE_DATE DATE,
    SALARY NUMBER(10,2)
);

-- 插入测试数据
INSERT INTO EMPLOYEES VALUES (1, 'John', 'Doe', 'john.doe@example.com', TO_DATE('2023-01-15', 'YYYY-MM-DD'), 75000.00);
INSERT INTO EMPLOYEES VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', TO_DATE('2023-03-20', 'YYYY-MM-DD'), 82000.00);
INSERT INTO EMPLOYEES VALUES (3, 'Bob', 'Johnson', 'bob.j@example.com', TO_DATE('2023-06-10', 'YYYY-MM-DD'), 68000.00);
COMMIT;
```

**免费限制**:
- **Always Free**: 2 个 Oracle Autonomous Database（每个 1 OCPU + 20 GB 存储）
- **无时间限制**：只要不超过配额，可以永久免费使用

---

### 3. Notion 文档管理配置

**注册与设置**:
1. 访问 [Notion 官网](https://www.notion.so/)
2. 点击 "Get Notion free" 并注册账号（使用邮箱或 Google 账号）
3. 选择 **Personal** 计划（免费，无限页面和块）

**创建培训工作空间**:
1. 登录后，点击左侧 **"+ New Page"**
2. 创建主页面：**"Java 迁移工程师培训 - 4 周计划"**
3. 在主页面下创建以下子页面结构：

```
📚 Java 迁移工程师培训
├── 📖 培训计划总览
├── 📅 Week 1: Oracle/Snowflake SQL 基础
│   ├── Day 1: 环境搭建
│   ├── Day 2: SQL 语法对比
│   ├── Day 3: 数据类型映射
│   ├── Day 4: 函数转换
│   └── Day 5: 周总结
├── 📅 Week 2: 迁移工具链与流程
│   ├── Day 6-10: (同上结构)
├── 📅 Week 3: 复杂场景处理
│   ├── Day 11-15: (同上结构)
├── 📅 Week 4: Mini-Project
│   ├── Day 16-20: (同上结构)
├── 📝 学习笔记
│   ├── SQL 转换速查表
│   ├── 常见错误案例库
│   └── 工具使用指南
├── 📊 作业提交记录
└── 🔗 资源链接
    ├── Snowflake 文档
    ├── Oracle 文档
    └── Git 仓库链接
```

**创建每日作业模板**:
在 Notion 中创建一个 **Database**（表格视图），用于跟踪每日作业：

| Day | 日期 | 任务 | 状态 | PR 链接 | 笔记 |
|-----|------|------|------|---------|------|
| Day 1 | 12.02 | 环境搭建 | ✅ 已完成 | [PR #1](url) | 遇到 JDBC 连接问题，已解决 |
| Day 2 | 12.03 | SQL 对比 | 🔄 进行中 | - | - |
| ... | ... | ... | ... | ... | ... |

**Notion 模板示例**（每日学习笔记）:
```markdown
# Day X: [任务名称]

## 🎯 今日目标
- [ ] 目标 1
- [ ] 目标 2

## 📚 学习内容
### 知识点 1
- 笔记...

### 知识点 2
- 笔记...

## 💡 遇到的问题与解决方案
1. **问题**: ...
   - **原因**: ...
   - **解决方案**: ...

## 📝 今日作业
- [x] 作业 1 - [PR 链接](url)
- [ ] 作业 2

## 🔗 参考资料
- [Snowflake 文档](url)
- [Stack Overflow 答案](url)

## 💭 反思与总结
- 今天学到的最重要的东西：...
- 明天需要重点关注的：...
```

**Notion 与 Git 集成**:
- 在每个 Notion 页面中，添加 **PR 链接**（指向 GitHub/GitLab）
- 在 Git 仓库的 README.md 中，添加 Notion 工作空间链接

**免费限制**:
- **Personal 计划**: 完全免费
- **页面数量**: 无限制
- **协作**: 最多 10 位访客（对于个人学习足够）

---

### 4. 其他必备工具

**4.1 Java 开发环境**
- **JDK 17+**（推荐 JDK 17 或 21）
  - 下载：[Oracle JDK](https://www.oracle.com/java/technologies/downloads/) 或 [OpenJDK](https://adoptium.net/)
  - 配置环境变量 `JAVA_HOME` 和 `PATH`

**4.2 IntelliJ IDEA**
- 下载 **Community Edition**（免费）：[IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
- 推荐插件：
  - Database Tools（内置，用于连接 Oracle/Snowflake）
  - Maven/Gradle（构建工具）
  - SonarLint（代码质量检查）

**4.3 Git 版本控制**
- 下载：[Git 官网](https://git-scm.com/downloads)
- 配置用户信息：
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your.email@example.com"
  ```
- 生成 SSH Key（用于 GitHub/GitLab）：
  ```bash
  ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
  # 将 ~/.ssh/id_rsa.pub 添加到 GitHub/GitLab
  ```

**4.4 Maven 依赖管理**
在 Java 项目中创建 `pom.xml`，添加 JDBC 驱动依赖：
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.migration</groupId>
    <artifactId>oracle-snowflake-migration</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Snowflake JDBC Driver -->
        <dependency>
            <groupId>net.snowflake</groupId>
            <artifactId>snowflake-jdbc</artifactId>
            <version>3.14.4</version>
        </dependency>

        <!-- Oracle JDBC Driver -->
        <dependency>
            <groupId>com.oracle.database.jdbc</groupId>
            <artifactId>ojdbc8</artifactId>
            <version>21.9.0.0</version>
        </dependency>

        <!-- JUnit 5 (单元测试) -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.9.3</version>
            <scope>test</scope>
        </dependency>

        <!-- SLF4J + Logback (日志) -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.14</version>
        </dependency>
    </dependencies>
</project>
```

---

### 5. 环境验证清单

在开始 Day 1 之前，请确认以下所有项目：

- [ ] **Snowflake 账号**: 能登录 Snowflake Web UI，成功创建 `MIGRATION_TRAINING` 数据库
- [ ] **Oracle 数据库**: 能通过 SQL*Plus 或 SQL Developer Web 连接并查询 `EMPLOYEES` 表
- [ ] **Notion 工作空间**: 已创建培训主页面和子页面结构
- [ ] **Java JDK**: 在命令行运行 `java -version` 显示版本 17+
- [ ] **IntelliJ IDEA**: 已安装并能创建新的 Maven 项目
- [ ] **Git**: 在命令行运行 `git --version` 显示版本信息，SSH Key 已配置
- [ ] **JDBC 驱动**: Maven `pom.xml` 能成功下载 Snowflake 和 Oracle JDBC 驱动

**环境验证测试程序**（可选）:
创建 `EnvironmentTest.java` 并运行：
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class EnvironmentTest {
    public static void main(String[] args) {
        // 测试 Snowflake 连接
        System.out.println("=== Testing Snowflake Connection ===");
        testSnowflake();

        // 测试 Oracle 连接
        System.out.println("\n=== Testing Oracle Connection ===");
        testOracle();
    }

    private static void testSnowflake() {
        String url = "jdbc:snowflake://<your_account>.snowflakecomputing.com/?db=MIGRATION_TRAINING&schema=PRACTICE&warehouse=COMPUTE_WH";
        String user = "<your_username>";
        String password = "<your_password>";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT CURRENT_VERSION()")) {
            if (rs.next()) {
                System.out.println("✅ Snowflake connection successful!");
                System.out.println("   Version: " + rs.getString(1));
            }
        } catch (Exception e) {
            System.out.println("❌ Snowflake connection failed: " + e.getMessage());
        }
    }

    private static void testOracle() {
        String url = "jdbc:oracle:thin:@localhost:1521:XE"; // 或 Oracle Cloud 连接字符串
        String user = "migration_user"; // 或 ADMIN（Oracle Cloud）
        String password = "MigrationPass123";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM V$VERSION WHERE ROWNUM = 1")) {
            if (rs.next()) {
                System.out.println("✅ Oracle connection successful!");
                System.out.println("   Version: " + rs.getString(1));
            }
        } catch (Exception e) {
            System.out.println("❌ Oracle connection failed: " + e.getMessage());
        }
    }
}
```

如果两个测试都显示 ✅，说明环境配置成功！

---

### 6. 故障排查（常见问题）

**问题 1: Snowflake JDBC 连接超时**
- **原因**: 防火墙或网络限制
- **解决方案**: 检查防火墙设置，确保允许访问 `*.snowflakecomputing.com:443`

**问题 2: Oracle XE 安装失败（Windows）**
- **原因**: 端口 1521 被占用
- **解决方案**: 停止其他使用 1521 端口的服务，或在安装时修改默认端口

**问题 3: Maven 依赖下载失败**
- **原因**: 网络问题或 Maven 镜像配置
- **解决方案**: 配置国内镜像（如阿里云 Maven 镜像）：
  ```xml
  <!-- settings.xml -->
  <mirrors>
      <mirror>
          <id>aliyun</id>
          <mirrorOf>central</mirrorOf>
          <url>https://maven.aliyun.com/repository/public</url>
      </mirror>
  </mirrors>
  ```

**问题 4: Oracle Cloud Wallet 文件连接失败**
- **原因**: `TNS_ADMIN` 路径配置错误
- **解决方案**: 确保 JDBC URL 中的路径使用绝对路径，并使用正斜杠 `/`（如：`C:/oracle_wallet/`）

---

## ⚠️ 核心原则

### Read First, Ask Later (15分钟原则)
遇到问题时，请按以下顺序处理：
1. 查阅 **Snowflake 官方文档** ([docs.snowflake.com](https://docs.snowflake.com))
2. 搜索 **内部迁移知识库** (`wiki/migration-playbook/`)
3. 使用 AI 辅助工具（ChatGPT/Claude）
4. **超过 15 分钟无果再向导师求助**

### Output Oriented (交付物导向)
- **只有提交了代码 PR 或技术文档，才算完成了工作**
- 禁止出现"今天阅读了XXX"这种无法验收的任务
- 所有作业必须走 **Git PR 流程**，并邀请 Mentor Review

### Zero-Defect Mindset (零缺陷思维)
- 这是一个 **700人月规模的"代码工厂"项目**，任何一个 SQL 语法错误都可能影响金融数据准确性
- 每次提交前必须通过：**本地单元测试 + 数据验证脚本**
- 养成 **Checklist 习惯**：修改一处，检查三遍

---

## 📂 提交规范

### 仓库结构
```
migration-training/
├── candidates/{your_name}/          # 你的个人作业目录
│   ├── day01-environment/
│   ├── day02-sql-basics/
│   ├── ...
│   └── mini-project/                # 第四周综合项目
├── wiki/
│   ├── migration-playbook/          # 迁移标准流程文档
│   ├── sql-conversion-guide/        # Oracle→Snowflake SQL 转换指南
│   └── common-pitfalls/             # 常见错误案例库
└── shared-utils/                    # 团队共享的迁移工具脚本
```

### Git 分支命名规范
```bash
feature/{your_name}/day{XX}-{task_name}
# 示例: feature/zhangsan/day03-datatype-mapping
```

### PR 提交要求
1. **Commit Message 格式**:
   ```
   [Day XX] 任务简述

   - 完成内容1
   - 完成内容2
   - 验证结果截图（如适用）
   ```
2. **PR Description** 必须包含：
   - 完成了哪些作业
   - 遇到的主要问题及解决方案
   - 自测结果（测试用例通过率）

---

## 📅 第一周：Oracle/Snowflake SQL 基础与差异

### Day 1 (12.02 周一): 环境验证与 Git 工作流
**🎯 目标**: 验证开发环境配置，跑通第一个 Oracle ↔ Snowflake 连接测试，掌握 Git 提交规范。

**⚠️ 前置条件**:
请确认已完成 **"🛠️ 环境准备"** 章节的所有配置（Snowflake、Oracle、Notion、Java、Git）。如未完成，请先返回完成环境配置。

**📚 学习内容**:
1. 复习环境配置（快速检查）：
   - Snowflake Web UI 能正常登录
   - Oracle 数据库能通过 SQL*Plus 或 SQL Developer Web 连接
   - IntelliJ IDEA 能创建 Maven 项目
   - Git 配置完成（`git config --list` 查看）

2. 阅读文档：
   - [Snowflake 快速入门](https://quickstarts.snowflake.com/)
   - [Git 提交规范](https://www.conventionalcommits.org/)

3. 在 Notion 中创建 Day 1 学习笔记页面（使用前面提供的模板）

**📝 实战作业**:

**作业 1: 运行环境验证测试程序**
1. 在 IntelliJ IDEA 中创建新的 Maven 项目：
   - 项目名称: `oracle-snowflake-migration-training`
   - Group ID: `com.migration`
   - Artifact ID: `migration-training`
   - 复制 **"环境准备"** 章节中的 `pom.xml` 依赖配置

2. 创建 `EnvironmentTest.java` 文件（代码见"环境准备"章节）：
   - 保存路径: `src/main/java/com/migration/EnvironmentTest.java`
   - 替换代码中的 `<your_account>`, `<your_username>`, `<your_password>` 为你的实际凭据
   - **重要**: 创建 `.gitignore` 文件，添加以下内容（避免提交敏感信息）：
     ```
     # 敏感信息
     **/credentials.txt
     **/application.properties

     # IDE 文件
     .idea/
     *.iml
     target/

     # 编译文件
     *.class
     ```

3. 运行 `EnvironmentTest.java`，确保两个测试都显示 ✅

4. 将测试输出截图保存到 `candidates/{your_name}/day01-environment/environment-test-result.png`

**作业 2: 编写第一个 Oracle ↔ Snowflake 对比程序**
创建 `DatabaseComparisonTest.java`，同时连接 Oracle 和 Snowflake，对比查询结果：

```java
package com.migration;

import java.sql.*;

public class DatabaseComparisonTest {
    // Oracle 连接信息
    private static final String ORACLE_URL = "jdbc:oracle:thin:@localhost:1521:XE";
    private static final String ORACLE_USER = "migration_user";
    private static final String ORACLE_PASSWORD = "MigrationPass123";

    // Snowflake 连接信息
    private static final String SNOWFLAKE_URL = "jdbc:snowflake://<your_account>.snowflakecomputing.com/?db=MIGRATION_TRAINING&schema=PRACTICE&warehouse=COMPUTE_WH";
    private static final String SNOWFLAKE_USER = "<your_username>";
    private static final String SNOWFLAKE_PASSWORD = "<your_password>";

    public static void main(String[] args) {
        System.out.println("=== Oracle vs Snowflake Comparison Test ===\n");

        // 测试 1: 查询 EMPLOYEES 表的行数
        System.out.println("Test 1: Row Count Comparison");
        compareRowCount();

        // 测试 2: 查询 EMPLOYEES 表的数据
        System.out.println("\nTest 2: Data Comparison");
        compareData();

        // 测试 3: 对比当前时间函数
        System.out.println("\nTest 3: Current Timestamp Function");
        compareTimestamp();
    }

    private static void compareRowCount() {
        try (Connection oracleConn = DriverManager.getConnection(ORACLE_URL, ORACLE_USER, ORACLE_PASSWORD);
             Connection snowflakeConn = DriverManager.getConnection(SNOWFLAKE_URL, SNOWFLAKE_USER, SNOWFLAKE_PASSWORD)) {

            // Oracle 查询
            Statement oracleStmt = oracleConn.createStatement();
            ResultSet oracleRs = oracleStmt.executeQuery("SELECT COUNT(*) FROM EMPLOYEES");
            oracleRs.next();
            int oracleCount = oracleRs.getInt(1);

            // Snowflake 查询
            Statement snowflakeStmt = snowflakeConn.createStatement();
            ResultSet snowflakeRs = snowflakeStmt.executeQuery("SELECT COUNT(*) FROM EMPLOYEES");
            snowflakeRs.next();
            int snowflakeCount = snowflakeRs.getInt(1);

            System.out.println("  Oracle Row Count: " + oracleCount);
            System.out.println("  Snowflake Row Count: " + snowflakeCount);
            System.out.println("  Match: " + (oracleCount == snowflakeCount ? "✅ YES" : "❌ NO"));

            oracleRs.close();
            snowflakeRs.close();
            oracleStmt.close();
            snowflakeStmt.close();
        } catch (SQLException e) {
            System.out.println("  ❌ Error: " + e.getMessage());
        }
    }

    private static void compareData() {
        try (Connection oracleConn = DriverManager.getConnection(ORACLE_URL, ORACLE_USER, ORACLE_PASSWORD);
             Connection snowflakeConn = DriverManager.getConnection(SNOWFLAKE_URL, SNOWFLAKE_USER, SNOWFLAKE_PASSWORD)) {

            // Oracle 查询
            Statement oracleStmt = oracleConn.createStatement();
            ResultSet oracleRs = oracleStmt.executeQuery("SELECT EMPLOYEE_ID, FIRST_NAME, LAST_NAME FROM EMPLOYEES ORDER BY EMPLOYEE_ID");

            System.out.println("  Oracle Data:");
            while (oracleRs.next()) {
                System.out.println("    " + oracleRs.getInt("EMPLOYEE_ID") + " | " +
                                   oracleRs.getString("FIRST_NAME") + " " +
                                   oracleRs.getString("LAST_NAME"));
            }

            // Snowflake 查询
            Statement snowflakeStmt = snowflakeConn.createStatement();
            ResultSet snowflakeRs = snowflakeStmt.executeQuery("SELECT EMPLOYEE_ID, FIRST_NAME, LAST_NAME FROM EMPLOYEES ORDER BY EMPLOYEE_ID");

            System.out.println("\n  Snowflake Data:");
            while (snowflakeRs.next()) {
                System.out.println("    " + snowflakeRs.getInt("EMPLOYEE_ID") + " | " +
                                   snowflakeRs.getString("FIRST_NAME") + " " +
                                   snowflakeRs.getString("LAST_NAME"));
            }

            oracleRs.close();
            snowflakeRs.close();
            oracleStmt.close();
            snowflakeStmt.close();
        } catch (SQLException e) {
            System.out.println("  ❌ Error: " + e.getMessage());
        }
    }

    private static void compareTimestamp() {
        try (Connection oracleConn = DriverManager.getConnection(ORACLE_URL, ORACLE_USER, ORACLE_PASSWORD);
             Connection snowflakeConn = DriverManager.getConnection(SNOWFLAKE_URL, SNOWFLAKE_USER, SNOWFLAKE_PASSWORD)) {

            // Oracle: SYSDATE
            Statement oracleStmt = oracleConn.createStatement();
            ResultSet oracleRs = oracleStmt.executeQuery("SELECT SYSDATE FROM DUAL");
            oracleRs.next();
            System.out.println("  Oracle SYSDATE: " + oracleRs.getTimestamp(1));

            // Snowflake: CURRENT_TIMESTAMP()
            Statement snowflakeStmt = snowflakeConn.createStatement();
            ResultSet snowflakeRs = snowflakeStmt.executeQuery("SELECT CURRENT_TIMESTAMP()");
            snowflakeRs.next();
            System.out.println("  Snowflake CURRENT_TIMESTAMP(): " + snowflakeRs.getTimestamp(1));

            oracleRs.close();
            snowflakeRs.close();
            oracleStmt.close();
            snowflakeStmt.close();
        } catch (SQLException e) {
            System.out.println("  ❌ Error: " + e.getMessage());
        }
    }
}
```

运行程序并截图保存结果。

**作业 3: 初始化 Git 仓库并提交**
1. 在项目根目录初始化 Git 仓库：
   ```bash
   cd oracle-snowflake-migration-training
   git init
   git add .
   git commit -m "[Day 01] 初始化项目 - 环境验证测试"
   ```

2. 创建 Day 1 作业分支：
   ```bash
   git checkout -b feature/{your_name}/day01-environment
   ```

3. 在 `candidates/{your_name}/day01-environment/` 目录下创建以下文件：
   - `README.md`: 记录环境配置过程和遇到的问题
   - `environment-checklist.md`: 填写"环境准备"章节中的验证清单
   - `environment-test-result.png`: 环境测试截图
   - `database-comparison-result.png`: 数据库对比测试截图

4. 在 Notion 中更新 Day 1 学习笔记：
   - 记录环境配置过程中的问题与解决方案
   - 总结 Oracle 和 Snowflake 连接的差异
   - 添加代码片段和截图

5. 提交所有作业：
   ```bash
   git add candidates/{your_name}/day01-environment/
   git commit -m "[Day 01] 完成环境验证与数据库对比测试

   - 完成 EnvironmentTest.java，验证 Oracle 和 Snowflake 连接
   - 完成 DatabaseComparisonTest.java，对比两个数据库的查询结果
   - 记录环境配置问题与解决方案
   "
   ```

6. 推送到远程仓库（如果有）并创建 Pull Request：
   ```bash
   git push -u origin feature/{your_name}/day01-environment
   ```

**作业 4: 编写环境配置文档**
在 `candidates/{your_name}/day01-environment/README.md` 中记录：

```markdown
# Day 1: 环境验证与配置总结

## 环境清单
- **操作系统**: [Windows 11 / macOS / Linux]
- **JDK 版本**: [java -version 输出]
- **IntelliJ IDEA 版本**: [版本号]
- **Git 版本**: [git --version 输出]
- **Maven 版本**: [mvn --version 输出]

## Snowflake 配置
- **Account Locator**: [xxx.snowflakecomputing.com]
- **Database**: MIGRATION_TRAINING
- **Schema**: PRACTICE
- **Warehouse**: COMPUTE_WH
- **JDBC Driver Version**: 3.14.4

## Oracle 配置
- **方案**: [Oracle XE 本地安装 / Oracle Cloud Free Tier]
- **版本**: [Oracle Database 21c XE / Autonomous Database]
- **JDBC Driver Version**: 21.9.0.0

## 遇到的问题与解决方案

### 问题 1: [问题描述]
- **现象**: ...
- **原因**: ...
- **解决方案**: ...

### 问题 2: [问题描述]
- **现象**: ...
- **原因**: ...
- **解决方案**: ...

## 测试结果
- ✅ EnvironmentTest.java 测试通过
- ✅ DatabaseComparisonTest.java 测试通过
- ✅ Oracle 和 Snowflake 的 EMPLOYEES 表数据一致

## 参考资料
- [Snowflake JDBC 文档](https://docs.snowflake.com/en/user-guide/jdbc.html)
- [Oracle JDBC 文档](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/)
```

**✅ 验收标准**:
- [ ] `EnvironmentTest.java` 和 `DatabaseComparisonTest.java` 都能成功运行，输出显示 ✅
- [ ] Oracle 和 Snowflake 的 EMPLOYEES 表行数一致（都是 3 行）
- [ ] Git 仓库初始化完成，`.gitignore` 配置正确（不提交敏感信息）
- [ ] PR 的 Commit Message 符合规范（参考 Conventional Commits）
- [ ] `README.md` 文档记录了至少 2 个遇到的问题及解决方案
- [ ] Notion 中的 Day 1 学习笔记更新完整

---

### Day 2 (12.03 周二): Oracle SQL 回顾 + Snowflake SQL 语法对比
**🎯 目标**: 掌握 Oracle 和 Snowflake 的 SQL 语法核心差异。

**📚 学习内容**:
1. 阅读：[Snowflake SQL 命令参考](https://docs.snowflake.com/en/sql-reference-commands.html)
2. 对比学习（重点）：
   - **FROM 子句**: Oracle 的 DUAL 表 vs Snowflake 的省略 FROM
   - **字符串拼接**: `||` 在两者中的行为差异
   - **NULL 处理**: `NVL` vs `COALESCE` vs `IFNULL`
   - **日期函数**: `SYSDATE` vs `CURRENT_TIMESTAMP()`
3. 观看内部录屏：`wiki/videos/sql-dialect-differences.mp4`（占位符）

**📝 实战作业**:
1. 创建对比练习文档 `candidates/{your_name}/day02-sql-basics/SQL_Comparison.md`，包含：
   - 10 个 Oracle SQL 示例
   - 对应的 Snowflake SQL 等价写法
   - 语法差异说明（用表格呈现）

   示例格式：
   | Oracle SQL | Snowflake SQL | 差异说明 |
   |------------|---------------|----------|
   | `SELECT SYSDATE FROM DUAL;` | `SELECT CURRENT_TIMESTAMP();` | Snowflake 无需 FROM DUAL，且推荐使用 CURRENT_TIMESTAMP() |

2. 编写 Java 测试程序 `SqlDialectTest.java`：
   - 同时连接 Oracle 测试库和 Snowflake 沙盒
   - 执行相同逻辑的查询（如：查询当前日期、字符串拼接）
   - 打印两者的查询结果，验证等价性

3. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 对比文档包含至少 10 对 SQL 示例
- [ ] Java 测试程序成功执行，且输出结果正确
- [ ] 文档排版规范（使用 Markdown 表格）

---

### Day 3 (12.04 周三): 数据类型映射与转换 🐛 Bug Bash
**🎯 目标**: 掌握 Oracle 和 Snowflake 的数据类型映射规则，修复真实案例中的类型转换错误。

**📚 学习内容**:
1. 阅读：[Snowflake 数据类型](https://docs.snowflake.com/en/sql-reference/data-types.html)
2. 重点对比：
   - **数值类型**: `NUMBER` vs `NUMBER(38,0)` vs `DECIMAL` vs `FLOAT`
   - **字符类型**: `VARCHAR2` vs `VARCHAR` (Snowflake 默认 16MB 上限)
   - **日期类型**: `DATE` vs `TIMESTAMP_NTZ` vs `TIMESTAMP_LTZ`
   - **特殊类型**: `CLOB`/`BLOB` → `VARIANT`/`BINARY`
3. 学习内部文档：`wiki/sql-conversion-guide/datatype-mapping.md`（占位符）

**📝 实战作业** (Bug Bash):
1. 从 `shared-utils/buggy-migration-examples/day03/` 中获取 5 个包含数据类型错误的迁移案例（Mentor 提供）
2. 每个案例包含：
   - 原 Oracle 表结构 (DDL)
   - 错误的 Snowflake 迁移 DDL
   - 导致的数据丢失或精度问题说明
3. 你的任务：
   - 修复每个案例的 Snowflake DDL
   - 编写数据验证 SQL，证明修复后数据一致
   - 在 `candidates/{your_name}/day03-datatype-mapping/bugfix-report.md` 中记录：
     - 错误原因分析
     - 修复方案
     - 验证结果截图

4. **额外挑战**（可选）：
   编写一个 Java 工具类 `DatatypeMapper.java`，输入 Oracle 数据类型，自动输出推荐的 Snowflake 数据类型。

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 5 个案例全部修复正确
- [ ] 每个案例都有数据验证 SQL 和结果截图
- [ ] Bug 修复报告格式规范，分析深入
- [ ] （可选）Java 工具类通过单元测试

---

### Day 4 (12.05 周四): 函数与语法差异转换
**🎯 目标**: 熟练转换 Oracle 特有函数到 Snowflake 等价实现。

**📚 学习内容**:
1. 高频差异函数对照表（必须掌握）：

   | 功能 | Oracle | Snowflake |
   |------|--------|-----------|
   | 字符串截取 | `SUBSTR(str, pos, len)` | `SUBSTRING(str, pos, len)` 或 `SUBSTR` |
   | 条件判断 | `DECODE(col, val1, res1, ...)` | `CASE WHEN ... END` 或 `IFF()` |
   | NULL 替换 | `NVL(col, default)` | `COALESCE(col, default)` 或 `IFNULL()` |
   | 日期计算 | `ADD_MONTHS(date, n)` | `DATEADD(MONTH, n, date)` |
   | 排名函数 | `ROWNUM` | `ROW_NUMBER() OVER (...)` |
   | 层级查询 | `CONNECT BY` | 递归 CTE (`WITH RECURSIVE`) |

2. 阅读：[Snowflake 函数参考](https://docs.snowflake.com/en/sql-reference/functions-all.html)
3. 特别注意：
   - Snowflake **不支持** `ROWNUM`，必须用窗口函数
   - `CONNECT BY` 需改写为递归 CTE（较复杂）

**📝 实战作业**:
1. 转换练习 - 在 `candidates/{your_name}/day04-function-conversion/exercises.sql` 中完成：

   **练习 1**: 转换 DECODE
   ```sql
   -- Oracle
   SELECT employee_id,
          DECODE(department_id, 10, 'Admin', 20, 'Sales', 'Other') AS dept_name
   FROM employees;

   -- 改写为 Snowflake (请在此处填写)
   ```

   **练习 2**: 转换 ROWNUM
   ```sql
   -- Oracle: 查询前 10 条记录
   SELECT * FROM orders WHERE ROWNUM <= 10;

   -- 改写为 Snowflake (请在此处填写)
   ```

   **练习 3**: 转换 ADD_MONTHS
   ```sql
   -- Oracle: 计算 3 个月后的日期
   SELECT order_id, ADD_MONTHS(order_date, 3) AS due_date
   FROM orders;

   -- 改写为 Snowflake (请在此处填写)
   ```

   **练习 4**: 转换 CONNECT BY（挑战）
   ```sql
   -- Oracle: 查询组织层级
   SELECT employee_id, manager_id, LEVEL
   FROM employees
   START WITH manager_id IS NULL
   CONNECT BY PRIOR employee_id = manager_id;

   -- 改写为 Snowflake 递归 CTE (请在此处填写)
   ```

   **练习 5**: 综合转换
   从 `shared-utils/practice-queries/oracle-complex-query.sql` 中获取一个包含多个 Oracle 特有函数的复杂查询，完整转换为 Snowflake 语法。

2. 编写验证脚本 `validation.sql`：
   - 在 Oracle 和 Snowflake 上分别执行原始查询和转换后的查询
   - 对比结果集是否一致（可使用 `MINUS` 或 `EXCEPT`）

3. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 5 个练习全部完成，语法正确
- [ ] 验证脚本证明转换后结果与 Oracle 一致
- [ ] 练习 4（递归 CTE）能正确处理层级关系

---

### Day 5 (12.06 周五): 第一周总结与最佳实践分享 📢
**🎯 目标**: 沉淀本周知识，反向输出，形成团队共享文档。

**📝 实战作业**:
1. 在 `wiki/knowledge-sharing/{your_name}/` 目录下创建 `Week1_Learning_Log.md`，包含：
   - **本周 Top 3 技术坑点**：
     - 坑点描述
     - 复现步骤
     - 解决方案
     - 预防措施

   - **Oracle → Snowflake 转换速查表**：
     - 整理本周学到的所有函数/语法对照（建议用表格）
     - 标注"高风险"转换项（如 `ROWNUM`、`CONNECT BY`）

   - **个人工具箱**：
     - 分享你编写的任何自动化脚本或工具类
     - 说明使用场景和使用方法

2. **Code Review**：
   - 随机抽取另一位同期新人的 Day 3 或 Day 4 作业进行 Review
   - 在其 PR 上留下至少 3 条有价值的评论（不能是"LGTM"这种空话）
   - 在自己的总结文档中记录：从别人代码中学到了什么

3. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 总结文档结构完整，内容有深度
- [ ] 速查表覆盖至少 15 个函数/语法对照
- [ ] 完成对他人代码的 Code Review，评论有建设性

---

## 📅 第二周：迁移工具链与标准化流程

### Day 6 (12.09 周一): 迁移工具链配置与使用
**🎯 目标**: 掌握团队标准迁移工具链，实现半自动化代码转换。

**📚 学习内容**:
1. 团队工具链介绍（假设场景）：
   - **Schema Converter**: 自动转换 Oracle DDL → Snowflake DDL
   - **SQL Translator**: 批量转换 SQL 文件中的方言差异
   - **Data Validator**: 对比 Oracle 和 Snowflake 表数据一致性
2. 阅读内部文档：
   - `shared-utils/tools/README.md`（工具使用手册）
   - `wiki/migration-playbook/tool-chain-setup.md`（配置指南）
3. 观看演示视频：`wiki/videos/tool-demo.mp4`（占位符）

**📝 实战作业**:
1. 配置工具链环境：
   - 安装并配置 Schema Converter、SQL Translator、Data Validator
   - 在 `candidates/{your_name}/day06-toolchain/setup-log.md` 中记录配置过程和遇到的问题

2. 工具实战：
   - 从 `shared-utils/sample-schemas/` 中获取 3 个 Oracle 表的 DDL
   - 使用 **Schema Converter** 自动转换为 Snowflake DDL
   - 手工检查转换结果，修正工具未处理的问题（如索引、分区）
   - 在 Snowflake 沙盒中执行 DDL，创建表

3. 编写工具评测报告 `tool-evaluation.md`：
   - 工具的准确率（哪些能自动处理，哪些需要人工介入）
   - 使用过程中的痛点
   - 改进建议

4. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 工具链成功配置，能正常运行
- [ ] 3 个表在 Snowflake 中成功创建
- [ ] 评测报告客观详实，有具体数据支撑

---

### Day 7 (12.10 周二): 接口修改标准流程实战
**🎯 目标**: 掌握"代码工厂"式的接口修改标准流程，提高修改效率和准确性。

**📚 学习内容**:
1. 阅读：`wiki/migration-playbook/interface-modification-SOP.md`（标准操作流程）
2. 核心流程（假设）：
   ```
   Step 1: 识别需修改的接口（通过 Grep 搜索 Oracle JDBC 连接）
   Step 2: 备份原代码（创建分支）
   Step 3: 修改数据源配置（Oracle → Snowflake）
   Step 4: 调整 SQL 语句（根据方言差异）
   Step 5: 修改结果集处理逻辑（如数据类型映射）
   Step 6: 编写/更新单元测试
   Step 7: 执行测试并记录结果
   Step 8: 提交 PR 并填写 Checklist
   ```
3. 学习 **Checklist 模板**：`wiki/templates/interface-modification-checklist.md`

**📝 实战作业**:
1. 从 `shared-utils/sample-code/legacy-java-app/` 中获取一个遗留 Java 应用（包含 3-5 个 Oracle 数据库调用接口）

2. 按照标准流程，迁移其中 **2 个接口** 到 Snowflake：
   - 接口 1：简单 SELECT 查询（如：查询用户信息）
   - 接口 2：包含 JOIN 和聚合函数的复杂查询（如：统计报表）

3. 对每个接口完成以下交付物：
   - **修改后的 Java 代码**（保存在 `candidates/{your_name}/day07-interface-migration/`）
   - **修改前后对比文档** `diff-report.md`（使用 `git diff` 或手工整理）
   - **单元测试代码**（至少覆盖正常场景和边界场景）
   - **填写 Checklist** `interface-modification-checklist.md`（基于模板）

4. **提交 Pull Request**，在 PR Description 中：
   - 说明修改了哪些文件
   - 列出 SQL 语句的主要变更点
   - 附上单元测试通过的截图

**✅ 验收标准**:
- [ ] 2 个接口全部迁移完成，代码能编译运行
- [ ] 单元测试全部通过（覆盖率 ≥ 80%）
- [ ] Checklist 填写完整，无遗漏项
- [ ] Diff 报告清晰标注了所有 SQL 修改点

---

### Day 8 (12.11 周三): 数据库连接替换实战 🐛 Bug Bash
**🎯 目标**: 掌握 JDBC 连接池配置差异，修复连接泄漏和配置错误。

**📚 学习内容**:
1. 对比 Oracle JDBC 和 Snowflake JDBC：
   - **驱动类**: `oracle.jdbc.OracleDriver` vs `net.snowflake.client.jdbc.SnowflakeDriver`
   - **连接 URL 格式**:
     Oracle: `jdbc:oracle:thin:@host:port:SID`
     Snowflake: `jdbc:snowflake://account.region.snowflakecomputing.com/?db=DB&schema=SCHEMA`
   - **连接池参数**: `maxPoolSize`、`connectionTimeout`、`idleTimeout` 在两者中的默认值差异
2. 阅读：[Snowflake JDBC Driver 文档](https://docs.snowflake.com/en/user-guide/jdbc.html)
3. 学习常见问题：`wiki/common-pitfalls/jdbc-connection-issues.md`

**📝 实战作业** (Bug Bash):
1. 从 `shared-utils/buggy-migration-examples/day08/` 中获取 5 个包含 JDBC 连接错误的代码示例（Mentor 提供）

2. 每个示例的典型问题（示例）：
   - **Bug 1**: 连接池配置错误，导致 Snowflake 连接超时
   - **Bug 2**: 未关闭 `ResultSet` 和 `Statement`，导致连接泄漏
   - **Bug 3**: 使用了 Oracle 特有的连接参数（如 `oracle.net.encryption_client`），在 Snowflake 中无效
   - **Bug 4**: 错误的事务隔离级别设置
   - **Bug 5**: 混用 Oracle 和 Snowflake 连接池，导致配置冲突

3. 你的任务：
   - 定位每个 Bug 的根本原因
   - 修复代码并验证（需要能实际运行）
   - 在 `candidates/{your_name}/day08-jdbc-bugfix/bugfix-report.md` 中记录：
     - Bug 现象（错误日志）
     - 根因分析
     - 修复代码 Diff
     - 验证结果（如：连接池监控截图、内存泄漏测试结果）

4. **编写最佳实践文档** `jdbc-best-practices.md`：
   - 总结 JDBC 连接管理的 5 条黄金法则
   - 提供一个"安全"的连接池配置模板（HikariCP 或 DBCP）

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 5 个 Bug 全部修复，代码能正常运行
- [ ] Bug 修复报告有错误日志和验证截图
- [ ] 最佳实践文档实用性强，有代码示例

---

### Day 9 (12.12 周四): 单元测试与回归测试
**🎯 目标**: 为迁移后的代码编写高质量单元测试，确保功能正确性。

**📚 学习内容**:
1. 单元测试框架：JUnit 5 + Mockito
2. 数据库测试策略：
   - **方案 1**: 使用 H2/HSQLDB 内存数据库（适合简单场景）
   - **方案 2**: Mock JDBC 连接（适合复杂场景）
   - **方案 3**: 使用 Snowflake 测试环境（真实但慢）
3. 阅读：`wiki/testing-guide/unit-test-guidelines.md`
4. 学习测试数据管理：如何准备和清理测试数据

**📝 实战作业**:
1. 为 Day 7 迁移的 2 个接口补充完整的单元测试：
   - **测试用例设计**：在 `candidates/{your_name}/day09-unit-test/test-cases.md` 中列出：
     - 正常场景（Happy Path）
     - 边界场景（空结果、大数据量）
     - 异常场景（连接失败、SQL 错误）

   - **测试代码**：保存在 `candidates/{your_name}/day09-unit-test/`
     - 使用 JUnit 5 注解（`@Test`, `@BeforeEach`, `@AfterEach`）
     - 使用 Mockito Mock 数据库连接（或使用真实 Snowflake 测试环境）
     - 断言覆盖：返回值、异常类型、日志输出

2. **回归测试**：
   - 编写一个自动化脚本 `regression-test.sh`（或 `.bat`），能一键运行所有测试
   - 生成测试报告（HTML 或文本格式）

3. **测试覆盖率**：
   - 使用 JaCoCo 生成代码覆盖率报告
   - 目标：行覆盖率 ≥ 80%，分支覆盖率 ≥ 70%
   - 将覆盖率报告截图保存到 `coverage-report.png`

4. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 每个接口至少有 5 个测试用例
- [ ] 所有测试用例通过
- [ ] 代码覆盖率达到目标（≥ 80%）
- [ ] 回归测试脚本能正常运行

---

### Day 10 (12.13 周五): 错误预防清单与分享 📢
**🎯 目标**: 总结第二周实战经验,建立个人"防错体系"。

**📝 实战作业**:
1. 编写 **迁移错误预防清单** `wiki/knowledge-sharing/{your_name}/Week2_Error_Prevention_Checklist.md`，包含：

   **Section 1: 代码修改前检查（Pre-Modification Checklist）**
   - [ ] 是否已备份原代码（创建 Git 分支）？
   - [ ] 是否已阅读相关接口的业务逻辑文档？
   - [ ] 是否已识别所有需修改的 SQL 语句？
   - [ ] 是否已查阅 SQL 方言转换指南？
   - [ ] （补充至少 5 条）

   **Section 2: 代码修改中检查（During-Modification Checklist）**
   - [ ] 数据库连接 URL 是否正确替换？
   - [ ] JDBC 驱动类是否更新？
   - [ ] SQL 语句中是否还有 Oracle 特有函数（如 `DECODE`、`NVL`）？
   - [ ] 数据类型映射是否正确（尤其是 `NUMBER`、`DATE`）？
   - [ ] （补充至少 5 条）

   **Section 3: 代码修改后检查（Post-Modification Checklist）**
   - [ ] 代码是否能编译通过？
   - [ ] 单元测试是否全部通过？
   - [ ] 是否已进行数据验证（对比 Oracle 和 Snowflake 结果）？
   - [ ] 是否已检查连接池配置（防止连接泄漏）？
   - [ ] （补充至少 5 条）

2. **案例库建设**：
   - 在 `wiki/common-pitfalls/{your_name}/` 创建 `Week2_Common_Mistakes.md`
   - 记录本周你犯过的错误或差点犯的错误（至少 3 个）
   - 每个错误包含：
     - 错误代码示例
     - 错误原因
     - 正确代码示例
     - 如何通过 Checklist 避免

3. **工具分享**：
   - 如果你编写了任何自动化脚本（如批量替换工具、验证脚本），分享到 `shared-utils/contributed-tools/{your_name}/`
   - 编写使用说明 `README.md`

4. **团队分享会**（可选，如果团队组织）：
   - 准备 5 分钟的演讲 PPT 或 Markdown 文档，分享本周最大的收获

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 错误预防清单包含至少 15 条检查项（3 个阶段各 ≥5 条）
- [ ] 案例库至少包含 3 个真实错误案例，分析深入
- [ ] 分享的工具（如有）能正常运行，文档完整

---

## 📅 第三周：复杂场景处理

### Day 11 (12.16 周一): 存储过程迁移
**🎯 目标**: 掌握 Oracle PL/SQL 存储过程到 Snowflake JavaScript/SQL 存储过程的迁移方法。

**📚 学习内容**:
1. 阅读：[Snowflake 存储过程](https://docs.snowflake.com/en/sql-reference/stored-procedures.html)
2. 核心差异：
   - Oracle PL/SQL vs Snowflake JavaScript Stored Procedure
   - Oracle PL/SQL vs Snowflake Snowflake Scripting（SQL 存储过程，类似 PL/SQL）
   - 变量声明、游标、异常处理、动态 SQL 的语法差异
3. 迁移策略选择：
   - **策略 1**: 将 PL/SQL 逻辑改写为 Java 应用层代码（推荐用于复杂逻辑）
   - **策略 2**: 使用 Snowflake JavaScript 存储过程（适合简单脚本）
   - **策略 3**: 使用 Snowflake Scripting（适合需要保留存储过程的场景）
4. 阅读案例：`wiki/migration-playbook/stored-procedure-migration-examples.md`

**📝 实战作业**:
1. 从 `shared-utils/sample-code/oracle-stored-procedures/` 中获取 3 个存储过程：
   - **SP 1**: 简单存储过程（参数传递 + 基本 CRUD）
   - **SP 2**: 包含游标和循环的存储过程
   - **SP 3**: 包含动态 SQL 和异常处理的存储过程

2. 迁移任务：
   - **SP 1**: 改写为 Snowflake Scripting 存储过程
   - **SP 2**: 改写为 Snowflake JavaScript 存储过程
   - **SP 3**: 改写为 Java 应用层代码

3. 对每个存储过程提供：
   - **迁移方案说明** `migration-plan.md`（为什么选择这种策略？）
   - **迁移后的代码**（Snowflake SQL/JavaScript 或 Java）
   - **功能验证测试**（对比 Oracle 和 Snowflake 执行结果）
   - **性能对比**（可选，如果方便测试）

4. 编写 **存储过程迁移指南** `stored-procedure-migration-guide.md`：
   - 总结 3 种迁移策略的适用场景
   - 列出常见语法转换规则（如：`CURSOR` → `RESULTSET`）
   - 提供决策树（帮助后续迁移选择策略）

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 3 个存储过程全部迁移完成，能正常执行
- [ ] 功能验证测试通过（结果与 Oracle 一致）
- [ ] 迁移指南实用性强，有清晰的决策树

---

### Day 12 (12.17 周二): 复杂查询优化
**🎯 目标**: 识别并优化迁移后的性能瓶颈查询。

**📚 学习内容**:
1. Snowflake 查询性能优化基础：
   - 集群键（Clustering Key）的作用
   - 查询剪枝（Partition Pruning）
   - 结果集缓存（Result Cache）
2. 阅读：[Snowflake 查询性能优化](https://docs.snowflake.com/en/user-guide/performance-query.html)
3. 使用 Snowflake Query Profile 分析查询计划：
   - 如何读懂 Query Profile
   - 识别常见性能问题（如：全表扫描、数据膨胀）
4. Oracle vs Snowflake 优化差异：
   - Oracle 的索引 vs Snowflake 的聚簇
   - Oracle 的 Hint vs Snowflake 的查询优化器（自动优化）

**📝 实战作业**:
1. 从 `shared-utils/sample-queries/slow-queries/` 中获取 3 个慢查询（Mentor 提供）：
   - **Query 1**: 复杂多表 JOIN（5+ 表）
   - **Query 2**: 大数据量聚合（GROUP BY + COUNT/SUM）
   - **Query 3**: 嵌套子查询（Subquery）

2. 优化任务：
   - 在 Snowflake 中执行原始查询，记录执行时间和 Query Profile
   - 分析性能瓶颈（使用 Query Profile）
   - 应用优化策略（如：改写 JOIN 顺序、添加 Clustering Key、改写子查询为 CTE）
   - 记录优化后的执行时间和 Query Profile

3. 对每个查询提供：
   - **优化前的 Query Profile 截图**
   - **性能瓶颈分析报告**（哪个步骤最慢？为什么？）
   - **优化后的 SQL 代码**
   - **优化后的 Query Profile 截图**
   - **性能提升对比**（如：从 120 秒降至 15 秒）

4. 编写 **查询优化技巧总结** `query-optimization-tips.md`：
   - Snowflake 查询优化的 5 大技巧
   - 常见反模式（Anti-patterns）
   - 快速诊断工具（如 Query Profile 的关键指标）

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 3 个查询全部优化完成，性能有显著提升（≥ 30%）
- [ ] 每个查询有完整的 Query Profile 对比
- [ ] 优化技巧总结有深度，不是泛泛而谈

---

### Day 13 (12.18 周三): 数据验证与对比
**🎯 目标**: 掌握自动化数据验证方法，确保迁移后数据一致性。

**📚 学习内容**:
1. 数据验证策略：
   - **行数验证**（Row Count）
   - **列和验证**（Checksum/Hash）
   - **抽样对比**（Sample Comparison）
   - **全量对比**（Full Data Diff，适用于小表）
2. 工具使用：
   - 使用团队的 **Data Validator** 工具
   - 编写自定义 SQL 对比脚本
3. 处理常见差异：
   - 浮点数精度差异
   - 时区差异（Oracle `DATE` vs Snowflake `TIMESTAMP_NTZ`）
   - NULL 值处理差异
4. 阅读：`wiki/migration-playbook/data-validation-guide.md`

**📝 实战作业**:
1. 从 `shared-utils/sample-schemas/migrated-tables/` 中获取 5 个已迁移的表（Oracle 和 Snowflake 均有数据）

2. 验证任务：
   - **Level 1**: 行数验证
     ```sql
     -- Oracle
     SELECT COUNT(*) FROM table_name;
     -- Snowflake
     SELECT COUNT(*) FROM table_name;
     ```

   - **Level 2**: 列和验证（使用 `HASH` 或 `CHECKSUM`）
     ```sql
     -- Oracle
     SELECT SUM(ORA_HASH(column1 || column2 || ...)) FROM table_name;
     -- Snowflake
     SELECT SUM(HASH(column1, column2, ...)) FROM table_name;
     ```

   - **Level 3**: 抽样对比（随机抽取 1000 行对比明细）

   - **Level 4**: 全量对比（适用于小表 < 10 万行）
     - 使用 `EXCEPT` 或 `MINUS` 找出差异行
     - 分析差异原因（数据类型、精度、NULL 处理）

3. 对每个表提供：
   - **验证报告** `validation-report.md`：
     - 行数对比结果
     - 列和对比结果
     - 差异行数量和示例（如有）
     - 差异根因分析
   - **自动化验证脚本** `validate_table.sql` 或 `validate_table.py`

4. **编写数据验证自动化框架**（挑战任务）：
   - 输入：表名列表
   - 输出：验证报告（HTML 或 CSV）
   - 功能：自动执行 4 个 Level 的验证，汇总结果

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 5 个表全部完成验证，报告详实
- [ ] 如有差异，根因分析清晰
- [ ] 自动化脚本能正常运行（至少支持 Level 1 和 Level 2）
- [ ] （可选）自动化框架能处理多表验证

---

### Day 14 (12.19 周四): 性能测试基础 🐛 Bug Bash
**🎯 目标**: 掌握基础性能测试方法，识别并修复性能瓶颈。

**📚 学习内容**:
1. 性能测试类型：
   - **基准测试（Baseline）**: 记录迁移前 Oracle 的性能指标
   - **对比测试（Comparison）**: 迁移后 Snowflake 的性能对比
   - **负载测试（Load Testing）**: 模拟并发场景
2. 关键性能指标：
   - 查询响应时间（Query Latency）
   - 吞吐量（Throughput, QPS）
   - 资源消耗（CPU/Memory/IO）
3. 工具使用：
   - Snowflake Query History
   - JMeter 或 Gatling（模拟并发）
4. 阅读：`wiki/testing-guide/performance-testing-guide.md`

**📝 实战作业** (Bug Bash):
1. 从 `shared-utils/buggy-migration-examples/day14/` 中获取 3 个性能问题案例（Mentor 提供）：
   - **Case 1**: 迁移后查询速度下降 10 倍（原因：缺少 Clustering Key）
   - **Case 2**: 批量插入性能差（原因：逐行插入未改为批量）
   - **Case 3**: 并发查询导致连接池耗尽（原因：连接池配置不当）

2. 你的任务：
   - 复现性能问题（执行查询/代码，记录性能指标）
   - 使用 Query Profile 或 JProfiler 等工具定位瓶颈
   - 修复问题（如：添加 Clustering Key、优化批量插入、调整连接池）
   - 验证修复效果（性能提升 ≥ 50%）

3. 对每个案例提供：
   - **问题复现报告**（性能指标、错误日志）
   - **根因分析**（为什么慢？）
   - **修复方案**（代码 Diff 或配置变更）
   - **性能对比图表**（修复前后的响应时间对比）

4. **编写性能测试模板** `performance-test-template.md`：
   - 定义标准的性能测试流程（5-7 步）
   - 提供性能指标记录表格模板
   - 列出常见性能瓶颈及快速诊断方法

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 3 个性能问题全部修复，性能提升显著
- [ ] 每个案例有性能对比数据和图表
- [ ] 性能测试模板实用性强，可复用

---

### Day 15 (12.20 周五): 常见坑点总结分享 📢
**🎯 目标**: 系统总结三周学习成果，形成团队知识资产。

**📝 实战作业**:
1. 编写 **Oracle → Snowflake 迁移常见坑点手册** `wiki/knowledge-sharing/{your_name}/Migration_Pitfalls_Handbook.md`，包含：

   **Part 1: SQL 语法坑点（至少 10 个）**
   | 坑点 | Oracle 写法 | 错误的 Snowflake 写法 | 正确的 Snowflake 写法 | 说明 |
   |------|-------------|----------------------|---------------------|------|
   | ROWNUM | `WHERE ROWNUM <= 10` | 直接照搬 | `LIMIT 10` 或 `ROW_NUMBER()` | Snowflake 不支持 ROWNUM |
   | ... | ... | ... | ... | ... |

   **Part 2: 数据类型坑点（至少 5 个）**
   - 示例：`NUMBER` 不指定精度时，Oracle 默认 `NUMBER(38,127)`，Snowflake 默认 `NUMBER(38,0)`

   **Part 3: 性能优化坑点（至少 5 个）**
   - 示例：Oracle 的 B-Tree 索引在 Snowflake 中不存在，需用 Clustering Key 替代

   **Part 4: JDBC 连接坑点（至少 3 个）**
   - 示例：Snowflake JDBC URL 必须包含 `db` 和 `schema` 参数

2. **制作速查卡片** `Quick_Reference_Card.pdf` 或 `.md`（单页 A4 纸大小）：
   - 涵盖最常用的 20 个 Oracle→Snowflake 转换规则
   - 可打印出来贴在工位上

3. **贡献到团队案例库**：
   - 将你本周遇到的所有错误（包括已修复和未修复的）提交到 `wiki/common-pitfalls/`
   - 每个错误包含：错误代码、错误原因、解决方案、预防措施

4. **准备团队分享（可选）**：
   - 如果团队组织周五分享会，准备 10 分钟的演讲：
     - 主题："我踩过的 5 个最痛的坑"
     - 包含真实案例和解决过程

5. **提交 Pull Request**

**✅ 验收标准**:
- [ ] 坑点手册包含至少 23 个坑点（SQL 10 + 数据类型 5 + 性能 5 + JDBC 3）
- [ ] 速查卡片简洁实用，排版清晰
- [ ] 案例库至少新增 3 个案例

---

## 📅 第四周：Mini-Project - 完整模块迁移

### Day 16-18 (12.23-12.25): Mini-Project 实施
**🎯 目标**: 独立完成一个完整业务模块的 Oracle → Snowflake 迁移，模拟真实项目流程。

**📚 项目背景**:
Mentor 将为你分配一个 **真实但经过简化的信用风险计算模块**（假设场景），包含：
- **3-5 张数据表**（客户信息、交易记录、风险评分等）
- **5-10 个 Java 接口**（查询、统计、报表生成）
- **1-2 个存储过程**（风险计算逻辑）
- **单元测试和集成测试**（需要一起迁移）

**📝 项目要求**:

**Phase 1: 需求分析与方案设计（Day 16 上午）**
1. 阅读项目文档：`shared-utils/mini-project/{your_name}/requirements.md`
2. 绘制架构图：
   - 原系统架构（Oracle + Java）
   - 目标系统架构（Snowflake + Java）
   - 标注需迁移的组件
3. 编写 **迁移方案设计文档** `migration-design.md`：
   - 迁移范围（哪些表、哪些接口、哪些存储过程）
   - 迁移策略（数据迁移、代码迁移、测试迁移）
   - 风险评估（可能遇到的问题及缓解措施）
   - 时间计划（Day 16-18 的任务分解）

**Phase 2: 数据库迁移（Day 16 下午 - Day 17 上午）**
1. **Schema 迁移**：
   - 将 Oracle DDL 转换为 Snowflake DDL
   - 使用工具辅助 + 人工检查
   - 在 Snowflake 中创建表结构

2. **数据迁移**：
   - 使用 Snowflake 的 `COPY INTO` 或团队工具迁移数据
   - 执行数据验证（行数、列和、抽样对比）
   - 记录验证结果

3. **存储过程迁移**：
   - 将 1-2 个存储过程迁移到 Snowflake（或改写为 Java）
   - 功能验证测试

**Phase 3: 应用层迁移（Day 17 下午 - Day 18 上午）**
1. **接口迁移**：
   - 修改 5-10 个 Java 接口的数据库连接
   - 调整 SQL 语句（方言转换）
   - 修改结果集处理逻辑（如数据类型映射）

2. **单元测试迁移**：
   - 更新测试用例（适配 Snowflake）
   - 补充缺失的测试
   - 确保测试覆盖率 ≥ 80%

3. **集成测试**：
   - 端到端测试（模拟真实业务流程）
   - 性能测试（对比 Oracle 和 Snowflake）

**Phase 4: 文档与交付（Day 18 下午）**
1. 编写 **迁移总结报告** `migration-summary.md`：
   - 完成情况（哪些完成了，哪些未完成）
   - 遇到的主要问题及解决方案
   - 数据验证结果
   - 性能对比结果
   - 经验教训（Lessons Learned）

2. 准备 **演示 Demo**：
   - 能演示至少 2 个核心功能（迁移前后对比）
   - 展示数据验证结果
   - 展示性能对比图表

3. **提交 Pull Request**：
   - 包含所有代码、文档、测试
   - PR Description 详细说明迁移内容

**📂 交付物清单**:
- [ ] 迁移方案设计文档 `migration-design.md`
- [ ] Snowflake DDL 脚本（所有表）
- [ ] 数据验证报告 `data-validation-report.md`
- [ ] 迁移后的 Java 代码（所有接口）
- [ ] 单元测试代码（覆盖率 ≥ 80%）
- [ ] 集成测试脚本和结果
- [ ] 性能测试报告 `performance-comparison.md`
- [ ] 迁移总结报告 `migration-summary.md`
- [ ] 演示 Demo（截图或录屏）

**✅ 验收标准**:
- [ ] 所有表成功迁移到 Snowflake，数据验证通过
- [ ] 所有接口成功迁移，功能正确
- [ ] 单元测试全部通过，覆盖率 ≥ 80%
- [ ] 集成测试通过，端到端功能正确
- [ ] 文档完整，逻辑清晰
- [ ] Demo 能成功演示核心功能

---

### Day 19 (12.26 周四): Code Review 与质量检查
**🎯 目标**: 通过严格的 Code Review，发现并修复潜在问题，提升代码质量。

**📝 实战作业**:
1. **自我 Code Review**：
   - 使用 **Code Review Checklist**（`wiki/templates/code-review-checklist.md`）逐项检查你的 Mini-Project 代码
   - 重点检查：
     - [ ] SQL 语句是否有 Oracle 残留语法？
     - [ ] 数据类型映射是否正确？
     - [ ] JDBC 连接是否正确关闭（无泄漏）？
     - [ ] 异常处理是否完善？
     - [ ] 日志输出是否合理（不过多、不过少）？
     - [ ] 代码风格是否符合团队规范？
     - [ ] 测试用例是否覆盖边界场景？
   - 记录发现的问题并修复，填写 `self-review-report.md`

2. **交叉 Code Review**：
   - 邀请 Mentor 或另一位同期新人 Review 你的代码
   - 在 PR 上回应所有 Review 意见（接受或解释）
   - 修复所有 Critical 和 High 级别的问题

3. **代码质量检查**：
   - 运行静态代码分析工具（如 SonarQube、Checkstyle）
   - 修复所有 Critical 问题，降低 Code Smell 数量
   - 将分析报告截图保存到 `code-quality-report.png`

4. **最终测试**：
   - 重新运行所有单元测试和集成测试
   - 确保测试通过率 100%
   - 执行回归测试（确保修复没有引入新问题）

5. **提交最终版本 Pull Request**

**✅ 验收标准**:
- [ ] 自我 Review 发现并修复至少 3 个问题
- [ ] 所有 Code Review 意见已处理
- [ ] 静态代码分析无 Critical 问题
- [ ] 所有测试通过率 100%

---

### Day 20 (12.27 周五): 成果展示与总结 🎉
**🎯 目标**: 向团队展示四周学习成果，完成知识沉淀。

**📝 实战作业**:

**Part 1: 成果展示（15-20 分钟演讲）**
准备演示材料（PPT 或 Markdown），包含：
1. **项目背景**：信用风险模块迁移概述
2. **迁移过程**：
   - 数据库迁移（表、数据、存储过程）
   - 应用层迁移（接口、测试）
   - 遇到的主要挑战及解决方案
3. **成果展示**：
   - **Live Demo**：演示 2 个核心功能
   - 数据验证结果（行数、列和、抽样对比）
   - 性能对比图表（Oracle vs Snowflake）
4. **数据亮点**：
   - 迁移代码行数、修改文件数
   - 测试覆盖率
   - 性能提升百分比（如有）
5. **经验教训**：
   - 技术层面：3 个最大的坑点
   - 流程层面：哪些流程有效，哪些需要改进

**Part 2: 四周学习总结报告**
编写 `wiki/knowledge-sharing/{your_name}/4-Week_Learning_Summary.md`，包含：
1. **技能成长轨迹**：
   - Week 1: SQL 方言差异（掌握 15+ 函数转换规则）
   - Week 2: 迁移工具链与标准流程（完成 5+ 接口迁移）
   - Week 3: 复杂场景处理（存储过程、性能优化）
   - Week 4: 端到端项目实战（完整模块迁移）

2. **量化成果**：
   - 提交 PR 数量
   - 编写代码行数
   - 修复 Bug 数量
   - 编写文档篇数
   - 贡献到团队知识库的案例数量

3. **知识沉淀**：
   - 整理一份 **个人迁移工具箱清单**（脚本、工具类、Checklist）
   - 整理一份 **个人速查手册**（SQL 转换、数据类型、JDBC 配置）

4. **后续学习计划**：
   - 哪些知识点还需要深入？
   - 下一步想学习什么技术？

**Part 3: 反馈与建议**
填写 **培训反馈问卷** `training-feedback.md`：
1. 培训内容的有效性（1-5 分评价）
2. 培训节奏是否合适（太快/太慢/刚好）
3. 哪些环节最有价值？
4. 哪些环节需要改进？
5. 对导师的建议

**Part 4: 最终提交**
- **提交 Pull Request**（包含所有文档和演示材料）
- 将演示 PPT/Markdown 上传到 `candidates/{your_name}/final-presentation/`
- 将四周学习总结报告提交到 `wiki/knowledge-sharing/{your_name}/`

**✅ 验收标准**:
- [ ] 演示准备充分，能流畅讲解
- [ ] Live Demo 成功运行，无报错
- [ ] 四周学习总结报告详实，有深度
- [ ] 培训反馈问卷填写完整

---

## 🏆 最终验收标准 (Definition of Done)

### 技术能力验收
- [ ] 能独立完成 Oracle → Snowflake 的表迁移（DDL + 数据）
- [ ] 能独立完成 Java 接口的数据库连接替换和 SQL 转换
- [ ] 掌握至少 20 个 Oracle→Snowflake SQL 函数/语法转换规则
- [ ] 能编写数据验证脚本，确保迁移后数据一致性
- [ ] 能使用 Query Profile 分析性能问题并进行基础优化
- [ ] 单元测试覆盖率 ≥ 80%

### 流程规范验收
- [ ] 所有作业通过 Git PR 流程提交，无直接 push 到 main 分支
- [ ] Commit Message 符合团队规范
- [ ] 每次提交前使用 Checklist 自查
- [ ] 能独立进行 Code Review，提出有价值的意见

### 文档产出验收
- [ ] 至少贡献 5 篇知识分享文档到团队 Wiki
- [ ] 至少记录 10 个真实错误案例到案例库
- [ ] 编写至少 1 个可复用的自动化脚本/工具

### 团队协作验收
- [ ] 完成至少 3 次 Code Review（Review 他人代码）
- [ ] 参加所有周五知识分享会（4 次）
- [ ] 能主动向团队分享踩坑经验

### Mini-Project 验收
- [ ] 数据迁移成功率 100%（数据验证通过）
- [ ] 功能迁移成功率 ≥ 90%（核心功能正常）
- [ ] 测试通过率 100%
- [ ] 能成功演示端到端业务流程

---

## 📞 求助渠道

### 技术问题
1. **优先级 1**: 查阅 Snowflake 官方文档和内部 Wiki（15 分钟原则）
2. **优先级 2**: 在团队 Slack/Teams 频道提问（附上错误日志和已尝试的解决方案）
3. **优先级 3**: 预约 Mentor 1-on-1 答疑（每周最多 2 次，每次 30 分钟）

### 流程问题
- 联系项目协调员（Project Coordinator）
- 参考 `wiki/migration-playbook/FAQ.md`

### 环境问题
- 联系 IT 支持团队
- 在 Slack `#infra-support` 频道报障

---

## 🎯 成功标志

**4 周后，你应该能够**：
1. **独立迁移**：在导师最小干预下，完成一个中等复杂度模块的完整迁移
2. **质量保证**：提交的代码通过 Code Review，无 Critical 缺陷
3. **知识输出**：能向新人讲解 Oracle → Snowflake 迁移的核心要点
4. **团队贡献**：你的工具/文档被团队其他成员复用

**Welcome to the Migration Team! Let's build quality migrations together! 🚀**
