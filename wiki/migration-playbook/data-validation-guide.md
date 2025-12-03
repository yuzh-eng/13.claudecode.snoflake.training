# 数据验证指南

## 1. 验证概述

数据迁移后必须进行全面验证，确保数据的完整性、准确性和一致性。

---

## 2. 验证维度

### 2.1 行数验证

**目标：** 确保记录数一致

```sql
-- Oracle
SELECT COUNT(*) AS row_count FROM employees;

-- Snowflake
SELECT COUNT(*) AS row_count FROM employees;

-- 对比结果
-- 如果不一致，检查是否有数据过滤条件
```

### 2.2 列值验证

**目标：** 验证关键字段的值

```sql
-- 数值列求和
SELECT SUM(salary) AS total_salary FROM employees;

-- 最小值/最大值
SELECT MIN(hire_date) AS earliest, MAX(hire_date) AS latest FROM employees;

-- NULL 值统计
SELECT COUNT(*) AS null_count FROM employees WHERE email IS NULL;
```

### 2.3 精度验证

**目标：** 检查小数精度是否丢失

```sql
-- 检查小数位数
SELECT order_id, amount,
       LENGTH(SUBSTR(TO_CHAR(amount), INSTR(TO_CHAR(amount), '.') + 1)) AS decimal_places
FROM orders
WHERE amount != FLOOR(amount)
LIMIT 10;
```

### 2.4 唯一性验证

**目标：** 验证主键和唯一约束

```sql
-- 检查重复主键
SELECT employee_id, COUNT(*) AS cnt
FROM employees
GROUP BY employee_id
HAVING COUNT(*) > 1;
```

---

## 3. 自动化验证脚本

### 3.1 Python 验证脚本

**validate_migration.py**

```python
import snowflake.connector
import cx_Oracle

def compare_row_counts(oracle_conn, snowflake_conn, table_name):
    """比较行数"""
    oracle_cursor = oracle_conn.cursor()
    oracle_cursor.execute(f"SELECT COUNT(*) FROM {table_name}")
    oracle_count = oracle_cursor.fetchone()[0]

    snowflake_cursor = snowflake_conn.cursor()
    snowflake_cursor.execute(f"SELECT COUNT(*) FROM {table_name}")
    snowflake_count = snowflake_cursor.fetchone()[0]

    if oracle_count == snowflake_count:
        print(f"✅ {table_name}: Row count match ({oracle_count})")
        return True
    else:
        print(f"❌ {table_name}: Row count mismatch")
        print(f"   Oracle: {oracle_count}, Snowflake: {snowflake_count}")
        return False

def compare_checksums(oracle_conn, snowflake_conn, table_name, numeric_cols):
    """比较数值列的校验和"""
    results = {}

    for col in numeric_cols:
        # Oracle
        oracle_cursor = oracle_conn.cursor()
        oracle_cursor.execute(f"SELECT SUM({col}) FROM {table_name}")
        oracle_sum = oracle_cursor.fetchone()[0]

        # Snowflake
        snowflake_cursor = snowflake_conn.cursor()
        snowflake_cursor.execute(f"SELECT SUM({col}) FROM {table_name}")
        snowflake_sum = snowflake_cursor.fetchone()[0]

        match = abs(oracle_sum - snowflake_sum) < 0.01  # 允许 0.01 的误差
        results[col] = {
            'oracle': oracle_sum,
            'snowflake': snowflake_sum,
            'match': match
        }

        if match:
            print(f"✅ {table_name}.{col}: Checksum match")
        else:
            print(f"❌ {table_name}.{col}: Checksum mismatch")
            print(f"   Oracle: {oracle_sum}, Snowflake: {snowflake_sum}")

    return results

# 使用示例
oracle_conn = cx_Oracle.connect('user/password@oracle-server:1521/ORCL')
snowflake_conn = snowflake.connector.connect(
    account='xyz12345',
    user='migration_user',
    password='password',
    warehouse='COMPUTE_WH',
    database='MIGRATION_DB'
)

compare_row_counts(oracle_conn, snowflake_conn, 'EMPLOYEES')
compare_checksums(oracle_conn, snowflake_conn, 'ORDERS', ['order_amount', 'discount'])
```

---

## 4. 验证报告模板

### 4.1 报告结构

```markdown
# 数据验证报告

## 表：EMPLOYEES

### 行数验证
- Oracle: 150,234 行
- Snowflake: 150,234 行
- 状态: ✅ 一致

### 列值验证
| 列名 | Oracle 总和 | Snowflake 总和 | 状态 |
|------|------------|---------------|------|
| salary | 12,345,678.90 | 12,345,678.90 | ✅ |
| bonus | 567,890.50 | 567,890.50 | ✅ |

### 精度验证
- 小数精度: ✅ 保持 2 位小数
- NULL 值数量: ✅ 一致（23 个）

### 唯一性验证
- 主键重复: ✅ 无重复
- 唯一约束: ✅ 通过
```

---

## 5. 常见问题

### 5.1 行数不一致

**可能原因:**
- 迁移过程中有增量数据写入
- 数据过滤条件不同
- 软删除记录处理不一致

**解决方案:**
- 停止 Oracle 写入，重新迁移
- 导出迁移时间戳前的数据快照

### 5.2 校验和不一致

**可能原因:**
- 数据类型精度丢失（NUMBER → NUMBER(38,0)）
- 浮点数舍入误差
- 字符编码问题

**解决方案:**
- 检查数据类型定义
- 使用抽样对比检查具体差异行

---

## 6. Checklist

- [ ] 所有表的行数验证通过
- [ ] 关键数值列的校验和验证通过
- [ ] 主键和唯一约束验证通过
- [ ] 外键关系验证通过
- [ ] 日期字段验证通过
- [ ] NULL 值统计验证通过
- [ ] 生成验证报告并归档

---

**文档版本:** v1.1
**最后更新:** 2025-01-29
**维护人:** 数据团队
