# Bug 修复报告 - 数据类型转换错误

## Bug 基本信息

- **Bug ID**: MIGRATION-2025-001
- **发现日期**: 2025-01-17
- **修复人**: 张三
- **严重级别**: P1（数据准确性问题）
- **影响范围**: 订单金额计算模块

---

## 1. Bug 描述

### 1.1 现象

在迁移 `ORDER_SUMMARY` 表后，执行金额统计查询时发现：
- Oracle 中查询结果：`总金额 = 12,345,678.90`
- Snowflake 中查询结果：`总金额 = 12,345,678.00`

**金额丢失了小数部分！**

### 1.2 复现步骤

```sql
-- Oracle 查询（正确）
SELECT SUM(order_amount) AS total_amount
FROM ORDER_SUMMARY
WHERE order_date = DATE '2025-01-15';
-- 结果：12345678.90

-- Snowflake 查询（错误）
SELECT SUM(order_amount) AS total_amount
FROM ORDER_SUMMARY
WHERE order_date = DATE '2025-01-15';
-- 结果：12345678.00（小数部分丢失）
```

---

## 2. 原因分析

### 2.1 数据类型定义对比

**Oracle 原始表结构：**
```sql
CREATE TABLE ORDER_SUMMARY (
    order_id NUMBER(10),
    order_amount NUMBER(12, 2),  -- 总共 12 位，小数点后 2 位
    order_date DATE
);
```

**Snowflake 迁移后表结构（错误版本）：**
```sql
CREATE TABLE ORDER_SUMMARY (
    order_id NUMBER(10),
    order_amount NUMBER,  -- ❌ 错误：省略了精度和标度
    order_date DATE
);
```

### 2.2 根本原因

在 Snowflake 中：
- `NUMBER` 不指定精度时，默认为 `NUMBER(38, 0)`
  - 精度 38：最多 38 位数字
  - 标度 0：**小数点后 0 位**（相当于整数）
- 插入数据时，小数部分被自动截断（不是四舍五入）

**验证测试：**
```sql
-- 在 Snowflake 中创建测试表
CREATE TABLE test_number (
    val1 NUMBER,        -- 默认 NUMBER(38, 0)
    val2 NUMBER(12, 2)  -- 明确指定精度
);

-- 插入测试数据
INSERT INTO test_number VALUES (123.45, 123.45);

-- 查询结果
SELECT * FROM test_number;
-- 结果：val1 = 123, val2 = 123.45
```

### 2.3 错误传播路径

1. **DDL 生成工具错误**：自动化迁移工具将 `NUMBER(12, 2)` 简化为 `NUMBER`
2. **Code Review 遗漏**：PR 审查时未仔细检查数据类型定义
3. **测试数据不足**：初始测试数据都是整数，未发现小数丢失问题
4. **数据验证缺失**：未执行聚合函数对比验证

---

## 3. 修复方案

### 3.1 修复 DDL

**正确的表结构：**
```sql
CREATE TABLE ORDER_SUMMARY (
    order_id NUMBER(10, 0),
    order_amount NUMBER(12, 2),  -- ✅ 明确指定精度和标度
    order_date DATE
);
```

### 3.2 数据修复步骤

```sql
-- Step 1: 备份现有表
CREATE TABLE ORDER_SUMMARY_BACKUP AS SELECT * FROM ORDER_SUMMARY;

-- Step 2: 创建新表（正确的数据类型）
CREATE TABLE ORDER_SUMMARY_NEW (
    order_id NUMBER(10, 0),
    order_amount NUMBER(12, 2),
    order_date DATE
);

-- Step 3: 从 Oracle 重新导入数据
-- 使用 Snowflake COPY INTO 命令
COPY INTO ORDER_SUMMARY_NEW
FROM @oracle_stage/order_summary.csv
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1);

-- Step 4: 验证数据一致性
SELECT COUNT(*) AS oracle_count FROM ORDER_SUMMARY@oracle_db;
SELECT COUNT(*) AS snowflake_count FROM ORDER_SUMMARY_NEW;

SELECT SUM(order_amount) AS oracle_sum FROM ORDER_SUMMARY@oracle_db;
SELECT SUM(order_amount) AS snowflake_sum FROM ORDER_SUMMARY_NEW;

-- Step 5: 替换旧表
DROP TABLE ORDER_SUMMARY;
ALTER TABLE ORDER_SUMMARY_NEW RENAME TO ORDER_SUMMARY;
```

### 3.3 验证结果

**修复前后对比：**
| 指标 | 修复前 | 修复后 | Oracle 原始值 |
|------|--------|--------|---------------|
| 总记录数 | 150,234 | 150,234 | 150,234 |
| 总金额 | 12,345,678.00 | 12,345,678.90 | 12,345,678.90 |
| 平均金额 | 82.00 | 82.17 | 82.17 |

✅ **修复后数据与 Oracle 完全一致**

---

## 4. 预防措施

### 4.1 代码规范

**建议：强制指定 NUMBER 类型的精度和标度**

```sql
-- ❌ 禁止：省略精度
order_amount NUMBER

-- ✅ 推荐：明确指定
order_amount NUMBER(12, 2)

-- ✅ 也可以：整数类型明确标注
order_id NUMBER(10, 0)  -- 或使用 INTEGER
```

### 4.2 Code Review Checklist

在 PR Review 时必须检查：
- [ ] 所有 `NUMBER` 类型是否指定了精度和标度？
- [ ] 小数类型的标度是否与 Oracle 一致？
- [ ] 是否有金额、百分比等需要小数精度的字段？

### 4.3 自动化验证脚本

**DDL 检查脚本：**
```python
import re

def check_number_precision(ddl_file):
    """检查 DDL 中是否有未指定精度的 NUMBER 类型"""
    with open(ddl_file, 'r') as f:
        content = f.read()

    # 匹配 NUMBER 类型但未指定精度的情况
    pattern = r'\bNUMBER\b(?!\s*\()'
    matches = re.findall(pattern, content, re.IGNORECASE)

    if matches:
        print(f"❌ 警告：发现 {len(matches)} 处未指定精度的 NUMBER 类型")
        return False
    else:
        print("✅ 所有 NUMBER 类型都已指定精度")
        return True

# 使用示例
check_number_precision('order_summary.sql')
```

### 4.4 数据验证流程

**迁移后必须执行的验证：**

```sql
-- 1. 行数对比
SELECT 'Oracle' AS source, COUNT(*) AS row_count FROM ORDER_SUMMARY@oracle_db
UNION ALL
SELECT 'Snowflake' AS source, COUNT(*) AS row_count FROM ORDER_SUMMARY;

-- 2. 数值列求和对比（检查精度丢失）
SELECT 'Oracle' AS source,
       SUM(order_amount) AS total,
       AVG(order_amount) AS avg_amount,
       MIN(order_amount) AS min_amount,
       MAX(order_amount) AS max_amount
FROM ORDER_SUMMARY@oracle_db
UNION ALL
SELECT 'Snowflake' AS source,
       SUM(order_amount) AS total,
       AVG(order_amount) AS avg_amount,
       MIN(order_amount) AS min_amount,
       MAX(order_amount) AS max_amount
FROM ORDER_SUMMARY;

-- 3. 抽样对比（检查小数位数）
SELECT * FROM ORDER_SUMMARY@oracle_db WHERE ROWNUM <= 100
MINUS
SELECT * FROM ORDER_SUMMARY LIMIT 100;
```

---

## 5. 经验总结

### 5.1 关键学习点

1. **Snowflake 的 NUMBER 默认行为**
   - 不指定精度时默认为 `NUMBER(38, 0)`
   - 与 Oracle 不同（Oracle `NUMBER` 默认为 `NUMBER(38, 127)`）

2. **数据类型映射规则**
   - Oracle `NUMBER(p, s)` → Snowflake `NUMBER(p, s)`（保持一致）
   - Oracle `NUMBER` → Snowflake `NUMBER(38, 0)` 或 `FLOAT`（需根据实际情况选择）

3. **验证的重要性**
   - DDL 验证：检查表结构定义
   - 数据验证：检查数据准确性
   - 聚合验证：检查计算结果一致性

### 5.2 工具改进

**已更新的自动化工具：**
- 在 DDL 生成工具中添加了精度检查
- 在 CI/CD 流程中添加了数据类型验证
- 更新了 Code Review Checklist

---

## 6. 相关文档

- [Snowflake NUMBER 类型文档](https://docs.snowflake.com/en/sql-reference/data-types-numeric.html#number)
- [Oracle NUMBER 类型文档](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Data-Types.html#GUID-A53B378A-D8B5-4BF0-8FFB-F3A11A0A68E5)
- 内部文档：`wiki/sql-conversion-guide/datatype-mapping.md`

---

**修复状态：** ✅ 已完成并通过验证
**PR 链接：** https://github.com/company/snowflake-migration/pull/1235
**导师审核：** 已通过（李导师，2025-01-17）
