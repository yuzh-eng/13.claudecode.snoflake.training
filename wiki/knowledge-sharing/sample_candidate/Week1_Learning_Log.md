# Week 1 学习总结

## 学员信息
- 姓名：张三
- 周期：2025-01-15 至 2025-01-19（第 1 周）
- 导师：李导师

---

## 1. 本周学习内容概览

| 日期 | 主题 | 状态 |
|------|------|------|
| Day 01 | 环境准备 | ✅ 完成 |
| Day 02 | SQL 语法对比 | ✅ 完成 |
| Day 03 | 数据类型映射 | ✅ 完成 |
| Day 04 | Bug Bash | ✅ 完成 |
| Day 05 | 周总结与知识分享 | ✅ 完成 |

---

## 2. 关键学习点

### 2.1 Oracle vs Snowflake 核心差异

#### SQL 语法差异

**最重要的发现：**
1. **DUAL 表**：Snowflake 不需要 `FROM DUAL`
2. **JOIN 语法**：Snowflake 不支持 `(+)` 外连接，必须使用 ANSI JOIN
3. **ROWNUM**：Snowflake 使用 `LIMIT/OFFSET` 替代

**示例代码：**
```sql
-- Oracle（旧语法）
SELECT * FROM employees e, departments d
WHERE e.department_id = d.department_id(+)
AND ROWNUM <= 10;

-- Snowflake（标准语法）
SELECT * FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
LIMIT 10;
```

#### 数据类型差异

**关键陷阱：**
- Oracle `NUMBER` 默认 `NUMBER(38, 127)`
- Snowflake `NUMBER` 默认 `NUMBER(38, 0)` ⚠️ **会丢失小数！**

**解决方案：**
- 始终明确指定精度和标度：`NUMBER(12, 2)`

---

## 3. 完成的任务和成果

### Day 01 - 环境准备

**成果：**
- ✅ 配置 Snowflake 账户（启用 MFA）
- ✅ 安装开发工具（DBeaver, SnowSQL）
- ✅ 完成 Hello World 程序
- ✅ 提交第一个 PR（获得 2 个 Approve）

**文档输出：**
- `candidates/zhang-san/day01-environment/README.md`
- `candidates/zhang-san/day01-environment/environment-checklist.md`

---

### Day 02 - SQL 语法对比

**成果：**
- ✅ 完成 10 组 SQL 对比练习
- ✅ 理解 JOIN、日期函数、字符串函数差异
- ✅ 编写对比文档

**关键收获：**
- `SYSDATE` → `CURRENT_TIMESTAMP()`
- `NVL` → `COALESCE`（更标准）
- `SUBSTR` 函数兼容，但推荐使用 `SUBSTRING`

**文档输出：**
- `candidates/zhang-san/day02-sql-basics/SQL_Comparison.md`

---

### Day 03 - 数据类型映射

**成果：**
- ✅ 修复 NUMBER 精度丢失 Bug
- ✅ 学习数据类型映射规则
- ✅ 编写 Bug 修复报告

**Bug 修复案例：**
- **问题：** 订单金额小数丢失（`123.45` → `123.00`）
- **原因：** `NUMBER` 类型未指定精度
- **解决：** 改为 `NUMBER(12, 2)`

**文档输出：**
- `candidates/zhang-san/day03-datatype-mapping/bugfix-report.md`

---

### Day 04 - Bug Bash

**成果：**
- ✅ 修复 3 个数据类型相关 Bug
- ✅ 提交 3 个 PR（全部通过 Review）

**Bug 清单：**
1. `EMPLOYEES` 表 `salary` 列精度丢失
2. `ORDERS` 表 `discount_rate` 列精度丢失
3. `PRODUCTS` 表 `weight` 列精度丢失

---

### Day 05 - 周总结与知识分享

**成果：**
- ✅ 编写 Week 1 学习总结（本文档）
- ✅ 准备知识分享 PPT（15 分钟）
- ✅ 在团队会议上分享学习心得

**分享主题：** "Oracle NUMBER 类型迁移陷阱及解决方案"

---

## 4. 遇到的困难和解决方案

### 困难 1：MFA 设置失败

**问题描述：** 扫描二维码后，验证码始终错误

**解决过程：**
1. 查阅 Snowflake MFA 文档
2. 发现手机时间未同步
3. 启用"自动设置时间"后问题解决

**耗时：** 15 分钟

**学习点：** 遇到问题先查文档，再独立研究 15 分钟（15 分钟规则）

---

### 困难 2：理解 Snowflake 时间戳类型

**问题描述：** Snowflake 有 3 种时间戳类型，不知道如何选择

**解决过程：**
1. 阅读官方文档对比差异
2. 请教导师：业务场景如何选择
3. 总结最佳实践

**学习点：**
- `TIMESTAMP_NTZ`：最常用（对应 Oracle `TIMESTAMP`）
- `TIMESTAMP_TZ`：需要存储时区时使用
- `TIMESTAMP_LTZ`：需要本地化显示时使用

**耗时：** 30 分钟

---

### 困难 3：第一次 Code Review 被拒

**问题描述：** 提交的 DDL 中 `NUMBER` 类型未指定精度

**解决过程：**
1. 认真阅读 Reviewer 的评论
2. 学习数据类型映射规则
3. 修复后重新提交

**学习点：**
- Code Review 是学习的好机会
- 要主动理解 Reviewer 的建议，而不是简单修改

**耗时：** 20 分钟

---

## 5. 技能提升

### 5.1 技术技能

| 技能 | 初始水平 | 当前水平 | 进步 |
|------|---------|---------|------|
| Snowflake SQL | ⭐ | ⭐⭐⭐ | +2 |
| 数据类型映射 | ⭐ | ⭐⭐⭐⭐ | +3 |
| DBeaver 使用 | ⭐⭐ | ⭐⭐⭐⭐ | +2 |
| Git 工作流 | ⭐⭐⭐ | ⭐⭐⭐⭐ | +1 |

### 5.2 软技能

- ✅ 学会使用 15 分钟规则（独立研究后再求助）
- ✅ 提升了文档编写能力
- ✅ 学会了有效的 Code Review 沟通

---

## 6. 下周计划

### Week 2 目标

| 日期 | 主题 | 预期成果 |
|------|------|---------|
| Day 06 | 工具链配置 | 配置 Ora2Pg, SnowConvert |
| Day 07 | 接口修改 | 完成 1 个接口修改练习 |
| Day 08 | JDBC 连接 | 修复 JDBC 连接问题 |
| Day 09 | 单元测试 | 单元测试覆盖率 ≥ 80% |
| Day 10 | 周总结 | Week 2 学习总结 |

### 学习重点

1. **工具链熟练度**：掌握 Ora2Pg 和 SnowConvert
2. **接口修改流程**：理解 SOP 和 Checklist
3. **JDBC 最佳实践**：连接池配置、性能优化
4. **测试驱动开发**：先写测试，再写代码

---

## 7. 反思和改进

### 7.1 做得好的地方

- ✅ 严格遵守 15 分钟规则，培养独立解决问题的能力
- ✅ 所有任务都有详细文档记录
- ✅ 主动分享学习心得，帮助团队

### 7.2 需要改进的地方

- ⚠️ 单元测试编写还不够熟练（下周重点提升）
- ⚠️ Code Review 时需要更仔细检查数据类型
- ⚠️ 时间管理可以更好（Day 03 任务完成较晚）

### 7.3 导师反馈

> "张三本周表现优秀，学习态度积极，文档质量高。建议下周加强单元测试能力，多练习 Mock 和断言的使用。"
> —— 李导师，2025-01-19

---

## 8. 知识点速查卡

### 快速参考

```sql
-- Oracle → Snowflake 常用替换

-- 1. FROM DUAL
Oracle:    SELECT 1 FROM DUAL;
Snowflake: SELECT 1;

-- 2. OUTER JOIN
Oracle:    WHERE a.id = b.id(+)
Snowflake: LEFT JOIN b ON a.id = b.id

-- 3. ROWNUM
Oracle:    WHERE ROWNUM <= 10
Snowflake: LIMIT 10

-- 4. SYSDATE
Oracle:    SELECT SYSDATE FROM DUAL;
Snowflake: SELECT CURRENT_TIMESTAMP();

-- 5. NVL
Oracle:    SELECT NVL(col, 0) FROM table;
Snowflake: SELECT COALESCE(col, 0) FROM table;
```

---

**总结完成日期：** 2025-01-19
**导师审核：** ✅ 已通过
**下周计划审核：** ✅ 已批准
