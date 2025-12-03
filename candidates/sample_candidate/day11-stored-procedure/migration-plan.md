# 存储过程迁移方案说明

## 任务信息
- 学员：张三
- 日期：2025-01-29
- 目标存储过程：`calculate_employee_bonus`

---

## 1. 原始存储过程分析

### 1.1 Oracle 存储过程

```sql
CREATE OR REPLACE PROCEDURE calculate_employee_bonus(
    p_employee_id IN NUMBER,
    p_year IN NUMBER,
    p_bonus OUT NUMBER
) AS
    v_salary NUMBER;
    v_performance_rating NUMBER;
    v_department_budget NUMBER;
BEGIN
    -- 获取员工薪资
    SELECT salary INTO v_salary
    FROM employees
    WHERE employee_id = p_employee_id;

    -- 获取绩效评级
    SELECT rating INTO v_performance_rating
    FROM performance_reviews
    WHERE employee_id = p_employee_id
      AND review_year = p_year;

    -- 获取部门预算
    SELECT bonus_budget INTO v_department_budget
    FROM department_budgets
    WHERE department_id = (
        SELECT department_id
        FROM employees
        WHERE employee_id = p_employee_id
    )
    AND budget_year = p_year;

    -- 计算奖金
    IF v_performance_rating >= 4 THEN
        p_bonus := v_salary * 0.15;  -- 优秀员工 15%
    ELSIF v_performance_rating >= 3 THEN
        p_bonus := v_salary * 0.10;  -- 良好员工 10%
    ELSE
        p_bonus := v_salary * 0.05;  -- 一般员工 5%
    END IF;

    -- 检查部门预算限制
    IF p_bonus > v_department_budget THEN
        p_bonus := v_department_budget;
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        p_bonus := 0;
        DBMS_OUTPUT.PUT_LINE('Employee or review not found');
    WHEN OTHERS THEN
        RAISE;
END;
/
```

### 1.2 复杂度分析

| 特性 | 使用情况 | 迁移难度 |
|------|---------|---------|
| 游标 | 未使用 | - |
| OUT 参数 | 1个 | ⭐⭐⭐ 中等 |
| 动态 SQL | 未使用 | - |
| 异常处理 | 使用 | ⭐⭐ 简单 |
| 嵌套查询 | 使用 | ⭐ 简单 |
| PL/SQL 特有函数 | DBMS_OUTPUT | ⭐⭐ 简单 |

**总体难度：** ⭐⭐⭐ 中等

---

## 2. 迁移策略选择

### 2.1 可选方案

| 方案 | 描述 | 优点 | 缺点 | 选择 |
|------|------|------|------|------|
| **方案A** | 转换为 Snowflake SQL Scripting | 逻辑保持一致 | OUT 参数需要改为 RETURNS | ✅ **推荐** |
| **方案B** | 转换为 JavaScript UDF | 灵活性高 | 学习成本高，调试困难 | ❌ |
| **方案C** | 重写为 SQL 函数 | 性能好 | 复杂逻辑难以实现 | ❌ |

### 2.2 选择理由

**选择方案A：Snowflake SQL Scripting**

1. **语法相似度高：** Snowflake SQL Scripting 与 PL/SQL 语法相近，迁移工作量小
2. **可维护性好：** 团队已熟悉 SQL 语法
3. **性能良好：** Snowflake 原生支持，性能优于 JavaScript UDF
4. **调试方便：** 支持 RETURN 语句和异常处理

---

## 3. 迁移方案设计

### 3.1 接口变更

**变更点：**
1. **OUT 参数 → RETURNS**：将 `p_bonus OUT NUMBER` 改为 `RETURNS NUMBER`
2. **异常处理**：将 `EXCEPTION WHEN` 改为 Snowflake 的异常处理语法
3. **输出语句**：移除 `DBMS_OUTPUT`（可选：改为日志表记录）

### 3.2 Snowflake 存储过程

```sql
CREATE OR REPLACE PROCEDURE calculate_employee_bonus(
    p_employee_id NUMBER,
    p_year NUMBER
)
RETURNS NUMBER
LANGUAGE SQL
AS
$$
DECLARE
    v_salary NUMBER;
    v_performance_rating NUMBER;
    v_department_budget NUMBER;
    v_bonus NUMBER DEFAULT 0;
    v_department_id NUMBER;
BEGIN
    -- 获取员工薪资和部门ID
    SELECT salary, department_id
    INTO :v_salary, :v_department_id
    FROM employees
    WHERE employee_id = :p_employee_id;

    -- 获取绩效评级
    SELECT rating INTO :v_performance_rating
    FROM performance_reviews
    WHERE employee_id = :p_employee_id
      AND review_year = :p_year;

    -- 获取部门预算
    SELECT bonus_budget INTO :v_department_budget
    FROM department_budgets
    WHERE department_id = :v_department_id
      AND budget_year = :p_year;

    -- 计算奖金
    IF (v_performance_rating >= 4) THEN
        v_bonus := v_salary * 0.15;  -- 优秀员工 15%
    ELSEIF (v_performance_rating >= 3) THEN
        v_bonus := v_salary * 0.10;  -- 良好员工 10%
    ELSE
        v_bonus := v_salary * 0.05;  -- 一般员工 5%
    END IF;

    -- 检查部门预算限制
    IF (v_bonus > v_department_budget) THEN
        v_bonus := v_department_budget;
    END IF;

    RETURN v_bonus;

EXCEPTION
    WHEN STATEMENT_ERROR THEN
        -- 数据未找到或其他错误
        RETURN 0;
    WHEN OTHER THEN
        RAISE;
END;
$$;
```

### 3.3 关键改动说明

| Oracle 语法 | Snowflake 语法 | 说明 |
|------------|---------------|------|
| `p_bonus OUT NUMBER` | `RETURNS NUMBER` | OUT 参数改为返回值 |
| `v_salary NUMBER;` | `v_salary NUMBER;` | ✅ 变量声明兼容 |
| `INTO v_salary` | `INTO :v_salary` | ⚠️ 需要加冒号 `:` |
| `ELSIF` | `ELSEIF` | ⚠️ 拼写差异 |
| `NO_DATA_FOUND` | `STATEMENT_ERROR` | 异常类型不同 |
| `DBMS_OUTPUT.PUT_LINE` | 移除 | Snowflake 无此函数 |

---

## 4. 调用方式变更

### 4.1 Oracle 调用方式

```sql
-- PL/SQL 块调用
DECLARE
    v_bonus NUMBER;
BEGIN
    calculate_employee_bonus(1001, 2025, v_bonus);
    DBMS_OUTPUT.PUT_LINE('Bonus: ' || v_bonus);
END;
/

-- Java 调用
CallableStatement cs = conn.prepareCall("{call calculate_employee_bonus(?, ?, ?)}");
cs.setInt(1, 1001);
cs.setInt(2, 2025);
cs.registerOutParameter(3, Types.NUMERIC);
cs.execute();
double bonus = cs.getDouble(3);
```

### 4.2 Snowflake 调用方式

```sql
-- SQL 直接调用
CALL calculate_employee_bonus(1001, 2025);

-- 在查询中使用
SELECT employee_id,
       first_name,
       calculate_employee_bonus(employee_id, 2025) AS bonus_2025
FROM employees;
```

**Java 调用（改动）：**
```java
// ✅ Snowflake 调用方式
CallableStatement cs = conn.prepareCall("CALL calculate_employee_bonus(?, ?)");
cs.setInt(1, 1001);
cs.setInt(2, 2025);
ResultSet rs = cs.executeQuery();
if (rs.next()) {
    double bonus = rs.getDouble(1);  // 从结果集获取返回值
}
```

---

## 5. 测试计划

### 5.1 单元测试

```sql
-- 测试用例 1：优秀员工（评级 >= 4）
CALL calculate_employee_bonus(1001, 2025);
-- 预期：salary * 0.15

-- 测试用例 2：良好员工（评级 >= 3）
CALL calculate_employee_bonus(1002, 2025);
-- 预期：salary * 0.10

-- 测试用例 3：一般员工（评级 < 3）
CALL calculate_employee_bonus(1003, 2025);
-- 预期：salary * 0.05

-- 测试用例 4：超出部门预算
CALL calculate_employee_bonus(1004, 2025);
-- 预期：受部门预算限制

-- 测试用例 5：员工不存在
CALL calculate_employee_bonus(9999, 2025);
-- 预期：返回 0
```

### 5.2 对比验证

```sql
-- 在 Oracle 和 Snowflake 中运行相同测试数据
-- 对比结果是否一致
```

---

## 6. 风险评估

| 风险 | 严重程度 | 缓解措施 |
|------|---------|---------|
| OUT 参数调用方式变化 | ⭐⭐⭐ 高 | 提前通知下游系统，提供迁移指南 |
| 异常处理行为差异 | ⭐⭐ 中 | 充分测试异常场景 |
| 性能差异 | ⭐ 低 | 性能测试验证 |

---

## 7. 实施步骤

1. ✅ 分析原始存储过程
2. ✅ 设计迁移方案
3. ⏳ 编写 Snowflake 存储过程
4. ⏳ 单元测试
5. ⏳ 对比验证
6. ⏳ 性能测试
7. ⏳ 更新调用方代码
8. ⏳ Code Review
9. ⏳ 部署到测试环境
10. ⏳ 生产环境部署

---

**文档状态：** ✅ 设计阶段完成
**下一步：** 开始编码实现
**预计完成时间：** 2025-01-29
