# Week 2 错误预防清单

## 学员信息
- 姓名：张三
- 周期：Week 2（2025-01-22 至 2025-01-26）
- 导师：李导师

---

## 1. 数据类型相关错误

### 1.1 NUMBER 类型精度丢失

**问题：**
```sql
-- ❌ 错误：未指定精度
CREATE TABLE orders (
    amount NUMBER  -- 默认 NUMBER(38, 0)，小数丢失！
);
```

**正确做法：**
```sql
-- ✅ 正确：明确指定精度
CREATE TABLE orders (
    order_id NUMBER(10, 0),     -- 整数
    amount NUMBER(12, 2),        -- 金额（2位小数）
    tax_rate NUMBER(5, 4)        -- 税率（4位小数）
);
```

**预防措施：**
- [ ] DDL 中所有 `NUMBER` 类型都指定精度和标度
- [ ] Code Review 时重点检查数据类型定义
- [ ] 使用自动化脚本检查 DDL

---

### 1.2 VARCHAR 长度不足

**问题：**
```sql
-- ❌ 错误：长度可能不够
CREATE TABLE employees (
    name VARCHAR(20)  -- 中文名字可能超长
);
```

**正确做法：**
```sql
-- ✅ 正确：预留足够长度
CREATE TABLE employees (
    name VARCHAR(100)  -- 预留足够空间
);
```

**预防措施：**
- [ ] 分析源数据最大长度
- [ ] 预留 20-30% 的冗余空间
- [ ] 长文本字段使用 `VARCHAR(16777216)`

---

### 1.3 日期类型选择错误

**问题：**
```sql
-- ❌ 错误：Oracle TIMESTAMP 直接映射为 TIMESTAMP
CREATE TABLE events (
    event_time TIMESTAMP  -- 默认 TIMESTAMP_NTZ，可能丢失时区信息
);
```

**正确做法：**
```sql
-- ✅ 正确：根据业务需求选择
CREATE TABLE events (
    event_date DATE,                    -- 仅日期
    event_time TIMESTAMP_NTZ,          -- 无时区时间戳
    scheduled_time TIMESTAMP_TZ        -- 带时区时间戳
);
```

**预防措施：**
- [ ] 理解 Snowflake 的 3 种时间戳类型
- [ ] 根据业务需求选择合适类型
- [ ] 统一使用 UTC 时区存储

---

## 2. SQL 语法相关错误

### 2.1 OUTER JOIN 语法错误

**问题：**
```sql
-- ❌ 错误：Snowflake 不支持 (+) 语法
SELECT e.name, d.dept_name
FROM employees e, departments d
WHERE e.dept_id = d.dept_id(+);  -- 语法错误！
```

**正确做法：**
```sql
-- ✅ 正确：使用 ANSI JOIN 语法
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

**预防措施：**
- [ ] 将所有 `(+)` 改写为 ANSI OUTER JOIN
- [ ] 使用自动化工具检测 `(+)` 语法
- [ ] Code Review 时检查 JOIN 语法

---

### 2.2 ROWNUM 分页错误

**问题：**
```sql
-- ❌ 错误：Snowflake 不支持 ROWNUM
SELECT * FROM employees WHERE ROWNUM <= 10;  -- 语法错误！
```

**正确做法：**
```sql
-- ✅ 正确：使用 LIMIT
SELECT * FROM employees LIMIT 10;

-- 或使用 OFFSET（分页）
SELECT * FROM employees LIMIT 10 OFFSET 20;
```

**预防措施：**
- [ ] 将所有 `ROWNUM` 改写为 `LIMIT`
- [ ] 使用自动化工具检测 `ROWNUM`
- [ ] 统一分页实现

---

### 2.3 DUAL 表遗留

**问题：**
```sql
-- ⚠️ 可以运行但不推荐
SELECT SYSDATE FROM DUAL;  -- Snowflake 支持但不推荐
```

**正确做法：**
```sql
-- ✅ 正确：删除 FROM DUAL
SELECT CURRENT_TIMESTAMP();  -- 更简洁
```

**预防措施：**
- [ ] 删除所有不必要的 `FROM DUAL`
- [ ] Code Review 时检查

---

## 3. 存储过程相关错误

### 3.1 OUT 参数未转换

**问题：**
```sql
-- ❌ 错误：Snowflake 不支持 OUT 参数
CREATE PROCEDURE get_salary(emp_id IN NUMBER, salary OUT NUMBER) ...
```

**正确做法：**
```sql
-- ✅ 正确：改为 RETURNS
CREATE PROCEDURE get_salary(emp_id NUMBER)
RETURNS NUMBER
LANGUAGE SQL
AS
$$
DECLARE
    v_salary NUMBER;
BEGIN
    SELECT salary INTO :v_salary FROM employees WHERE employee_id = :emp_id;
    RETURN v_salary;
END;
$$;
```

**预防措施：**
- [ ] 所有 OUT 参数改为 RETURNS
- [ ] 更新调用方代码
- [ ] 测试所有存储过程

---

### 3.2 变量引用缺少冒号

**问题：**
```sql
-- ❌ 错误：变量引用缺少冒号
SELECT salary INTO v_salary FROM employees;
```

**正确做法：**
```sql
-- ✅ 正确：变量引用加冒号
SELECT salary INTO :v_salary FROM employees;

-- 或使用 LET
LET v_salary := (SELECT salary FROM employees);
```

**预防措施：**
- [ ] 所有变量引用加冒号 `:`
- [ ] Code Review 时检查

---

### 3.3 ELSIF vs ELSEIF

**问题：**
```sql
-- ❌ 错误：Snowflake 使用 ELSEIF
IF condition1 THEN
    ...
ELSIF condition2 THEN  -- 拼写错误！
    ...
END IF;
```

**正确做法：**
```sql
-- ✅ 正确：使用 ELSEIF
IF (condition1) THEN
    ...
ELSEIF (condition2) THEN
    ...
END IF;
```

**预防措施：**
- [ ] 将所有 `ELSIF` 改为 `ELSEIF`
- [ ] 语法检查工具

---

## 4. JDBC 连接相关错误

### 4.1 连接池配置不合理

**问题：**
```java
// ❌ 错误：连接数过多
config.setMaximumPoolSize(100);  // Snowflake 连接数有限
```

**正确做法：**
```java
// ✅ 正确：合理配置连接池
config.setMaximumPoolSize(20);  // 根据 Warehouse 大小调整
config.setMinimumIdle(5);
config.setConnectionTimeout(30000);
config.addDataSourceProperty("client_session_keep_alive", "true");
```

**预防措施：**
- [ ] 根据 Warehouse 大小设置连接数
- [ ] 启用 Keep-Alive
- [ ] 监控连接池使用情况

---

### 4.2 未处理 OUT 参数变化

**问题：**
```java
// ❌ 错误：仍然使用 OUT 参数方式调用
CallableStatement cs = conn.prepareCall("{call get_salary(?, ?)}");
cs.registerOutParameter(2, Types.NUMERIC);  // 错误！
```

**正确做法：**
```java
// ✅ 正确：使用 ResultSet 获取返回值
CallableStatement cs = conn.prepareCall("CALL get_salary(?)");
cs.setInt(1, 1001);
ResultSet rs = cs.executeQuery();
if (rs.next()) {
    double salary = rs.getDouble(1);
}
```

**预防措施：**
- [ ] 更新所有存储过程调用代码
- [ ] 测试所有 JDBC 调用

---

## 5. 数据验证相关错误

### 5.1 未验证小数精度

**问题：**
- 只验证行数和总和
- 未检查小数位数

**正确做法：**
```sql
-- ✅ 验证小数精度
SELECT order_id, amount,
       LENGTH(SUBSTR(TO_CHAR(amount), INSTR(TO_CHAR(amount), '.') + 1)) AS decimal_places
FROM orders
WHERE amount != FLOOR(amount)
LIMIT 100;
```

**预防措施：**
- [ ] 检查小数位数
- [ ] 抽样对比具体数据
- [ ] 验证聚合函数结果

---

### 5.2 未验证 NULL 值

**问题：**
- 未检查 NULL 值数量

**正确做法：**
```sql
-- ✅ 验证 NULL 值统计
SELECT COUNT(*) AS null_count
FROM employees
WHERE commission_pct IS NULL;
```

**预防措施：**
- [ ] 对比 NULL 值数量
- [ ] 检查可空列

---

## 6. 性能相关错误

### 6.1 未添加聚簇键

**问题：**
- 大表未添加聚簇键
- 查询全表扫描

**正确做法：**
```sql
-- ✅ 添加聚簇键
ALTER TABLE orders CLUSTER BY (order_date);
```

**预防措施：**
- [ ] 大表（>1TB）添加聚簇键
- [ ] 选择频繁过滤的列

---

### 6.2 Warehouse 大小选择不当

**问题：**
- 使用过大或过小的 Warehouse

**正确做法：**
- 根据查询复杂度选择合适大小
- 使用 Auto-Suspend 节省成本

**预防措施：**
- [ ] 性能测试选择合适大小
- [ ] 监控 Warehouse 使用率

---

## 7. 文档相关错误

### 7.1 缺少修改说明文档

**问题：**
- 接口修改未编写文档
- 下游团队不知道变更

**正确做法：**
- 编写修改前后对比文档
- 提前通知下游团队

**预防措施：**
- [ ] 填写 interface-modification-checklist.md
- [ ] 编写迁移指南
- [ ] 邮件或 Slack 通知

---

## 8. 总结

### 8.1 Top 5 错误

1. **NUMBER 类型精度丢失**（最常见！）
2. **OUT 参数未转换**
3. **(+) JOIN 语法未改写**
4. **变量引用缺少冒号**
5. **数据验证不充分**

### 8.2 预防措施总结

**技术措施：**
- ✅ 使用自动化脚本检查 DDL
- ✅ 编写 Code Review Checklist
- ✅ 编写数据验证脚本
- ✅ 单元测试覆盖率 ≥ 80%

**流程措施：**
- ✅ 严格执行 Code Review
- ✅ 填写 Checklist
- ✅ 充分测试和验证
- ✅ 文档完整

---

**文档版本：** v1.0
**最后更新：** 2025-01-26
**作者：** 张三
**审核：** 李导师
