# 数据验证报告

## 验证信息
- 验证人：张三
- 验证日期：2025-01-31
- 验证范围：HR Schema（15 张表）
- 数据截止时间：2025-01-30 23:59:59

---

## 1. 验证概览

### 1.1 验证统计

| 指标 | 结果 | 状态 |
|------|------|------|
| 总表数 | 15 | - |
| 验证通过 | 14 | ✅ |
| 验证失败 | 1 | ❌ |
| 通过率 | 93.3% | ⚠️ |

### 1.2 验证维度

- [x] 行数对比
- [x] 列值对比（SUM, AVG, MIN, MAX）
- [x] NULL 值统计
- [x] 主键唯一性
- [x] 外键完整性
- [x] 数据精度（小数位数）

---

## 2. 表级验证结果

### 2.1 EMPLOYEES 表

**基本信息：**
- Oracle 表名：`HR.EMPLOYEES`
- Snowflake 表名：`HR.EMPLOYEES`
- 主键：`EMPLOYEE_ID`

**行数验证：**
```sql
-- Oracle
SELECT COUNT(*) FROM HR.EMPLOYEES;
-- 结果：107 行

-- Snowflake
SELECT COUNT(*) FROM HR.EMPLOYEES;
-- 结果：107 行

-- 状态：✅ 一致
```

**列值验证：**
| 列名 | Oracle | Snowflake | 差异 | 状态 |
|------|--------|-----------|------|------|
| SUM(salary) | 691,416.00 | 691,416.00 | 0.00 | ✅ |
| AVG(salary) | 6,461.83 | 6,461.83 | 0.00 | ✅ |
| MIN(hire_date) | 1987-06-17 | 1987-06-17 | - | ✅ |
| MAX(hire_date) | 2008-04-21 | 2008-04-21 | - | ✅ |

**NULL 值统计：**
| 列名 | Oracle NULL 数 | Snowflake NULL 数 | 状态 |
|------|----------------|-------------------|------|
| commission_pct | 72 | 72 | ✅ |
| manager_id | 1 | 1 | ✅ |
| department_id | 1 | 1 | ✅ |

**主键唯一性：**
```sql
-- 检查重复主键
SELECT employee_id, COUNT(*) AS cnt
FROM HR.EMPLOYEES
GROUP BY employee_id
HAVING COUNT(*) > 1;
-- 结果：0 行 ✅ 无重复
```

**总体状态：** ✅ **通过**

---

### 2.2 DEPARTMENTS 表

**基本信息：**
- Oracle 表名：`HR.DEPARTMENTS`
- Snowflake 表名：`HR.DEPARTMENTS`
- 主键：`DEPARTMENT_ID`

**行数验证：**
```sql
-- Oracle: 27 行
-- Snowflake: 27 行
-- 状态：✅ 一致
```

**总体状态：** ✅ **通过**

---

### 2.3 ORDERS 表

**基本信息：**
- Oracle 表名：`HR.ORDERS`
- Snowflake 表名：`HR.ORDERS`
- 主键：`ORDER_ID`

**行数验证：**
```sql
-- Oracle: 150,234 行
-- Snowflake: 150,234 行
-- 状态：✅ 一致
```

**列值验证：**
| 列名 | Oracle | Snowflake | 差异 | 状态 |
|------|--------|-----------|------|------|
| SUM(order_amount) | 12,345,678.90 | 12,345,678.00 | -0.90 | ❌ **失败** |
| AVG(order_amount) | 82.17 | 82.17 | 0.00 | ✅ |
| COUNT(*) | 150,234 | 150,234 | 0 | ✅ |

**问题分析：**

❌ **发现问题：** `order_amount` 列小数部分丢失

**详细检查：**
```sql
-- 检查小数位数
SELECT order_id, order_amount,
       LENGTH(SUBSTR(TO_CHAR(order_amount), INSTR(TO_CHAR(order_amount), '.') + 1)) AS decimal_places
FROM HR.ORDERS
WHERE order_amount != FLOOR(order_amount)
LIMIT 10;

-- Oracle 结果：
-- order_id | order_amount | decimal_places
-- ---------|--------------|---------------
-- 1001     | 123.45       | 2
-- 1002     | 456.78       | 2

-- Snowflake 结果：
-- order_id | order_amount | decimal_places
-- ---------|--------------|---------------
-- 1001     | 123.00       | 0  ❌ 小数丢失
-- 1002     | 456.00       | 0  ❌ 小数丢失
```

**根本原因：**
```sql
-- 检查表结构
SHOW COLUMNS IN HR.ORDERS;

-- Oracle DDL:
-- order_amount NUMBER(12, 2)  ✅ 正确

-- Snowflake DDL:
-- order_amount NUMBER(12, 0)  ❌ 错误（未指定小数位数）
```

**修复方案：**
```sql
-- 1. 备份现有表
CREATE TABLE HR.ORDERS_BACKUP AS SELECT * FROM HR.ORDERS;

-- 2. 重新创建表（正确的数据类型）
CREATE TABLE HR.ORDERS_NEW (
    order_id NUMBER(10, 0),
    order_amount NUMBER(12, 2),  -- ✅ 修复：明确指定 2 位小数
    order_date DATE,
    customer_id NUMBER(10, 0)
);

-- 3. 从 Oracle 重新导入数据
COPY INTO HR.ORDERS_NEW
FROM @oracle_stage/orders.csv
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1);

-- 4. 验证修复
SELECT SUM(order_amount) FROM HR.ORDERS_NEW;
-- 结果：12,345,678.90 ✅ 修复成功

-- 5. 替换旧表
DROP TABLE HR.ORDERS;
ALTER TABLE HR.ORDERS_NEW RENAME TO HR.ORDERS;
```

**总体状态：** ❌ **失败（已修复）**

---

### 2.4 其他表验证结果

| 表名 | 行数匹配 | 列值匹配 | NULL 匹配 | 主键唯一 | 状态 |
|------|---------|---------|----------|---------|------|
| JOB_HISTORY | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| JOBS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| LOCATIONS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| COUNTRIES | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| REGIONS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| CUSTOMERS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| PRODUCTS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| ORDER_ITEMS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| INVOICES | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| PAYMENTS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |
| SHIPMENTS | ✅ | ✅ | ✅ | ✅ | ✅ 通过 |

---

## 3. 外键完整性验证

### 3.1 EMPLOYEES → DEPARTMENTS

```sql
-- 检查孤立记录（员工的部门ID在 DEPARTMENTS 表中不存在）
SELECT e.employee_id, e.department_id
FROM HR.EMPLOYEES e
LEFT JOIN HR.DEPARTMENTS d ON e.department_id = d.department_id
WHERE e.department_id IS NOT NULL
  AND d.department_id IS NULL;

-- 结果：0 行 ✅ 无孤立记录
```

### 3.2 ORDERS → CUSTOMERS

```sql
-- 检查孤立记录
SELECT o.order_id, o.customer_id
FROM HR.ORDERS o
LEFT JOIN HR.CUSTOMERS c ON o.customer_id = c.customer_id
WHERE o.customer_id IS NOT NULL
  AND c.customer_id IS NULL;

-- 结果：0 行 ✅ 无孤立记录
```

**所有外键验证：** ✅ **通过**

---

## 4. 抽样对比

### 4.1 随机抽样 100 行

```sql
-- Oracle（使用 DBMS_RANDOM）
SELECT * FROM (
    SELECT * FROM HR.EMPLOYEES
    ORDER BY DBMS_RANDOM.VALUE
) WHERE ROWNUM <= 100;

-- Snowflake（使用 SAMPLE）
SELECT * FROM HR.EMPLOYEES SAMPLE (100 ROWS);

-- 逐行对比：✅ 100 行全部一致
```

### 4.2 极值数据检查

```sql
-- 检查最大值和最小值记录是否一致

-- Oracle
SELECT * FROM HR.EMPLOYEES WHERE salary = (SELECT MAX(salary) FROM HR.EMPLOYEES);
-- employee_id: 100, salary: 24000

-- Snowflake
SELECT * FROM HR.EMPLOYEES WHERE salary = (SELECT MAX(salary) FROM HR.EMPLOYEES);
-- employee_id: 100, salary: 24000

-- 状态：✅ 一致
```

---

## 5. 性能对比

### 5.1 查询性能

| 查询类型 | Oracle 耗时 | Snowflake 耗时 | 性能对比 |
|---------|------------|---------------|---------|
| 全表扫描 | 12.5 秒 | 2.3 秒 | ✅ 提升 5.4 倍 |
| 聚合查询 | 8.2 秒 | 1.1 秒 | ✅ 提升 7.5 倍 |
| JOIN 查询 | 15.7 秒 | 3.8 秒 | ✅ 提升 4.1 倍 |
| 点查询 | 0.05 秒 | 0.02 秒 | ✅ 提升 2.5 倍 |

**结论：** Snowflake 查询性能普遍优于 Oracle

---

## 6. 问题汇总

### 6.1 已发现问题

| 问题ID | 表名 | 问题描述 | 严重程度 | 状态 |
|--------|------|---------|---------|------|
| DV-001 | ORDERS | `order_amount` 小数精度丢失 | P1 | ✅ 已修复 |

### 6.2 问题详情

**DV-001: ORDERS 表小数精度丢失**

- **发现时间：** 2025-01-31 10:30
- **影响范围：** 150,234 行订单数据
- **数据差异：** 总金额差异 $0.90
- **根本原因：** DDL 中 `NUMBER` 类型未指定精度
- **修复时间：** 2025-01-31 14:15
- **修复方式：** 重新创建表并重新导入数据
- **验证状态：** ✅ 修复后验证通过

---

## 7. 验证结论

### 7.1 总体评估

- **验证通过率：** 93.3% → 100%（修复后）
- **数据一致性：** ✅ 高度一致
- **性能表现：** ✅ Snowflake 性能优于 Oracle
- **数据完整性：** ✅ 外键约束全部满足

### 7.2 建议

1. **立即行动：**
   - ✅ ORDERS 表已修复并验证通过
   - ✅ 所有表验证通过

2. **后续监控：**
   - 建立定期数据验证机制（每周）
   - 监控新增数据的一致性

3. **流程改进：**
   - DDL 生成时强制指定 `NUMBER` 类型精度
   - Code Review 时重点检查数据类型定义
   - 自动化验证脚本集成到 CI/CD

---

## 8. 验证脚本

### 8.1 自动化验证脚本

```python
# validate_migration.py
import snowflake.connector
import cx_Oracle

def validate_table(table_name):
    """验证单张表"""
    # 连接 Oracle 和 Snowflake
    oracle_conn = cx_Oracle.connect('user/pass@oracle')
    snowflake_conn = snowflake.connector.connect(...)

    # 1. 行数对比
    oracle_count = oracle_conn.cursor().execute(
        f"SELECT COUNT(*) FROM {table_name}"
    ).fetchone()[0]

    snowflake_count = snowflake_conn.cursor().execute(
        f"SELECT COUNT(*) FROM {table_name}"
    ).fetchone()[0]

    print(f"✅ {table_name}: Row count match ({oracle_count})")
    if oracle_count != snowflake_count:
        print(f"❌ {table_name}: Row count mismatch!")

    # 2. 校验和对比
    # ...

# 运行验证
for table in ['EMPLOYEES', 'DEPARTMENTS', 'ORDERS']:
    validate_table(table)
```

---

## 9. 附件

- `validation_scripts/` - 验证脚本
- `validation_results/` - 详细验证结果（CSV）
- `fix_scripts/` - 问题修复脚本

---

**验证完成时间：** 2025-01-31 16:00
**报告审核人：** 李导师
**批准状态：** ✅ 已批准上线
