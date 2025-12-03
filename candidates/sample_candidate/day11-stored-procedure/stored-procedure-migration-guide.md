# 存储过程迁移指南

## 1. Oracle vs Snowflake 存储过程对比

### 1.1 语法差异总览

| 特性 | Oracle PL/SQL | Snowflake SQL Scripting | Snowflake JavaScript |
|------|---------------|------------------------|---------------------|
| 语言类型 | PL/SQL | SQL | JavaScript |
| 学习曲线 | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| 性能 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 调试难度 | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| 推荐度 | - | ⭐⭐⭐⭐⭐ | ⭐⭐ |

---

## 2. 参数处理

### 2.1 IN 参数（输入参数）

**Oracle:**
```sql
CREATE OR REPLACE PROCEDURE get_employee(
    p_emp_id IN NUMBER  -- IN 参数
) AS
BEGIN
    -- 使用 p_emp_id
END;
```

**Snowflake:**
```sql
CREATE OR REPLACE PROCEDURE get_employee(
    p_emp_id NUMBER  -- 默认为 IN 参数
)
LANGUAGE SQL
AS
$$
BEGIN
    -- 使用 :p_emp_id（注意冒号）
END;
$$;
```

### 2.2 OUT 参数（输出参数）

**Oracle:**
```sql
CREATE OR REPLACE PROCEDURE get_employee_name(
    p_emp_id IN NUMBER,
    p_name OUT VARCHAR2  -- OUT 参数
) AS
BEGIN
    SELECT first_name INTO p_name
    FROM employees
    WHERE employee_id = p_emp_id;
END;
```

**Snowflake（方案1：改为 RETURNS）：**
```sql
CREATE OR REPLACE PROCEDURE get_employee_name(
    p_emp_id NUMBER
)
RETURNS VARCHAR
LANGUAGE SQL
AS
$$
DECLARE
    v_name VARCHAR;
BEGIN
    SELECT first_name INTO :v_name
    FROM employees
    WHERE employee_id = :p_emp_id;
    RETURN v_name;
END;
$$;
```

**Snowflake（方案2：返回表）：**
```sql
CREATE OR REPLACE PROCEDURE get_employee_info(
    p_emp_id NUMBER
)
RETURNS TABLE (name VARCHAR, salary NUMBER)
LANGUAGE SQL
AS
$$
BEGIN
    LET result RESULTSET := (
        SELECT first_name AS name, salary
        FROM employees
        WHERE employee_id = :p_emp_id
    );
    RETURN TABLE(result);
END;
$$;
```

### 2.3 IN OUT 参数

**Oracle:**
```sql
CREATE OR REPLACE PROCEDURE update_counter(
    p_counter IN OUT NUMBER  -- IN OUT 参数
) AS
BEGIN
    p_counter := p_counter + 1;
END;
```

**Snowflake（不直接支持，改为 RETURNS）：**
```sql
CREATE OR REPLACE PROCEDURE update_counter(
    p_counter NUMBER
)
RETURNS NUMBER
LANGUAGE SQL
AS
$$
BEGIN
    RETURN :p_counter + 1;
END;
$$;
```

---

## 3. 变量声明和赋值

### 3.1 变量声明

**Oracle:**
```sql
DECLARE
    v_count NUMBER;
    v_name VARCHAR2(100);
    v_salary NUMBER := 50000;  -- 声明时赋值
BEGIN
    -- ...
END;
```

**Snowflake:**
```sql
DECLARE
    v_count NUMBER;
    v_name VARCHAR(100);
    v_salary NUMBER DEFAULT 50000;  -- 使用 DEFAULT
BEGIN
    -- ...
END;
```

### 3.2 变量赋值

**Oracle:**
```sql
-- 方式1：直接赋值
v_count := 10;

-- 方式2：从查询赋值
SELECT COUNT(*) INTO v_count FROM employees;
```

**Snowflake:**
```sql
-- 方式1：使用 LET
LET v_count := 10;

-- 方式2：从查询赋值（注意冒号）
SELECT COUNT(*) INTO :v_count FROM employees;

-- 方式3：使用 LET + 子查询
LET v_count := (SELECT COUNT(*) FROM employees);
```

---

## 4. 控制流语句

### 4.1 IF-THEN-ELSE

**Oracle:**
```sql
IF v_salary > 100000 THEN
    v_bonus := v_salary * 0.15;
ELSIF v_salary > 50000 THEN
    v_bonus := v_salary * 0.10;
ELSE
    v_bonus := v_salary * 0.05;
END IF;
```

**Snowflake:**
```sql
IF (v_salary > 100000) THEN
    v_bonus := v_salary * 0.15;
ELSEIF (v_salary > 50000) THEN  -- ⚠️ ELSEIF（不是 ELSIF）
    v_bonus := v_salary * 0.10;
ELSE
    v_bonus := v_salary * 0.05;
END IF;
```

### 4.2 CASE 语句

**Oracle:**
```sql
CASE v_grade
    WHEN 'A' THEN v_bonus := 5000;
    WHEN 'B' THEN v_bonus := 3000;
    ELSE v_bonus := 1000;
END CASE;
```

**Snowflake:**
```sql
CASE
    WHEN v_grade = 'A' THEN v_bonus := 5000;
    WHEN v_grade = 'B' THEN v_bonus := 3000;
    ELSE v_bonus := 1000;
END CASE;
```

### 4.3 循环

**Oracle - FOR 循环:**
```sql
FOR i IN 1..10 LOOP
    INSERT INTO test VALUES (i);
END LOOP;
```

**Snowflake - FOR 循环:**
```sql
-- Snowflake 不支持简单的数值 FOR 循环
-- 需要使用游标或其他方式

-- 游标循环
FOR record IN (SELECT * FROM employees) DO
    -- 处理每一行
END FOR;
```

**Oracle - WHILE 循环:**
```sql
WHILE v_count < 10 LOOP
    v_count := v_count + 1;
END LOOP;
```

**Snowflake - WHILE 循环:**
```sql
WHILE (v_count < 10) DO
    v_count := v_count + 1;
END WHILE;
```

---

## 5. 游标处理

### 5.1 显式游标

**Oracle:**
```sql
DECLARE
    CURSOR emp_cursor IS
        SELECT employee_id, first_name FROM employees;
    v_emp_id NUMBER;
    v_name VARCHAR2(100);
BEGIN
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor INTO v_emp_id, v_name;
        EXIT WHEN emp_cursor%NOTFOUND;
        -- 处理数据
    END LOOP;
    CLOSE emp_cursor;
END;
```

**Snowflake:**
```sql
DECLARE
    emp_cursor CURSOR FOR SELECT employee_id, first_name FROM employees;
    v_emp_id NUMBER;
    v_name VARCHAR;
BEGIN
    OPEN emp_cursor;
    FOR record IN emp_cursor DO
        v_emp_id := record.employee_id;
        v_name := record.first_name;
        -- 处理数据
    END FOR;
    CLOSE emp_cursor;
END;
```

### 5.2 FOR 游标循环（推荐）

**Oracle:**
```sql
FOR emp_rec IN (SELECT employee_id, first_name FROM employees) LOOP
    -- 直接使用 emp_rec.employee_id
END LOOP;
```

**Snowflake:**
```sql
FOR emp_rec IN (SELECT employee_id, first_name FROM employees) DO
    -- 直接使用 emp_rec.employee_id
END FOR;
```

---

## 6. 异常处理

### 6.1 基本异常处理

**Oracle:**
```sql
BEGIN
    -- 业务逻辑
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No data');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Too many rows');
    WHEN OTHERS THEN
        RAISE;
END;
```

**Snowflake:**
```sql
BEGIN
    -- 业务逻辑
EXCEPTION
    WHEN STATEMENT_ERROR THEN
        -- 对应 NO_DATA_FOUND 等错误
        RETURN -1;
    WHEN OTHER THEN
        RAISE;
END;
```

### 6.2 自定义异常

**Oracle:**
```sql
DECLARE
    invalid_salary EXCEPTION;
BEGIN
    IF v_salary < 0 THEN
        RAISE invalid_salary;
    END IF;
EXCEPTION
    WHEN invalid_salary THEN
        DBMS_OUTPUT.PUT_LINE('Invalid salary');
END;
```

**Snowflake（使用 EXCEPTION）:**
```sql
BEGIN
    IF (v_salary < 0) THEN
        RAISE EXCEPTION -20001, 'Invalid salary';
    END IF;
EXCEPTION
    WHEN OTHER THEN
        -- 处理异常
END;
```

---

## 7. 动态 SQL

### 7.1 EXECUTE IMMEDIATE

**Oracle:**
```sql
EXECUTE IMMEDIATE 'INSERT INTO ' || v_table_name || ' VALUES (:1, :2)'
    USING v_id, v_name;
```

**Snowflake:**
```sql
-- 方式1：使用 EXECUTE IMMEDIATE
EXECUTE IMMEDIATE
    'INSERT INTO ' || :v_table_name || ' VALUES (?, ?)'
    USING (v_id, v_name);

-- 方式2：使用字符串拼接（简单场景）
EXECUTE IMMEDIATE
    'INSERT INTO ' || :v_table_name ||
    ' VALUES (' || :v_id || ', \'' || :v_name || '\')';
```

---

## 8. 常用函数替换

| Oracle 函数 | Snowflake 替换 | 说明 |
|------------|---------------|------|
| `DBMS_OUTPUT.PUT_LINE` | 移除或使用日志表 | Snowflake 无此功能 |
| `SYSDATE` | `CURRENT_TIMESTAMP()` | 需要加括号 |
| `NVL(a, b)` | `COALESCE(a, b)` | 更标准 |
| `SUBSTR(str, pos, len)` | `SUBSTR(str, pos, len)` | ✅ 兼容 |
| `INSTR(str, substr)` | `POSITION(substr IN str)` | 参数顺序不同 |
| `LENGTH(str)` | `LENGTH(str)` | ✅ 兼容 |
| `UPPER(str)` | `UPPER(str)` | ✅ 兼容 |
| `TRIM(str)` | `TRIM(str)` | ✅ 兼容 |

---

## 9. 最佳实践

### 9.1 变量引用加冒号

```sql
-- ❌ 错误（会被当作列名）
SELECT salary INTO v_salary FROM employees;

-- ✅ 正确
SELECT salary INTO :v_salary FROM employees;
```

### 9.2 条件表达式加括号

```sql
-- ❌ 不推荐
IF v_count > 10 THEN

-- ✅ 推荐
IF (v_count > 10) THEN
```

### 9.3 优先使用 SQL 而非过程逻辑

```sql
-- ❌ 不推荐：使用循环
FOR record IN (SELECT * FROM employees) DO
    UPDATE employees SET bonus = salary * 0.1
    WHERE employee_id = record.employee_id;
END FOR;

-- ✅ 推荐：使用 SQL
UPDATE employees SET bonus = salary * 0.1;
```

---

## 10. 迁移检查清单

- [ ] OUT 参数已改为 RETURNS
- [ ] 变量引用已添加冒号 `:`
- [ ] `ELSIF` 已改为 `ELSEIF`
- [ ] `LOOP ... END LOOP` 已改为 `DO ... END FOR/WHILE`
- [ ] 异常类型已更新
- [ ] `DBMS_OUTPUT` 已移除
- [ ] 动态 SQL 语法已更新
- [ ] 所有测试用例通过

---

**文档版本:** v1.0
**最后更新:** 2025-01-29
**作者:** 张三
