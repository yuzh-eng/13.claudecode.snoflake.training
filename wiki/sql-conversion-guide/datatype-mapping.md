# Oracle → Snowflake 数据类型映射指南

## 1. 数值类型

| Oracle | Snowflake | 说明 | 示例 |
|--------|-----------|------|------|
| `NUMBER` | `NUMBER(38, 0)` | Oracle 默认 (38, 127)，Snowflake 默认 (38, 0) | ⚠️ **需要明确指定精度** |
| `NUMBER(p)` | `NUMBER(p, 0)` | 整数，精度 p | `NUMBER(10)` → `NUMBER(10, 0)` |
| `NUMBER(p, s)` | `NUMBER(p, s)` | 定点数 | `NUMBER(10, 2)` → `NUMBER(10, 2)` ✅ |
| `INTEGER` | `NUMBER(38, 0)` | 整数 | `INTEGER` → `NUMBER(38, 0)` |
| `FLOAT` | `FLOAT` | 浮点数 | ✅ 兼容 |
| `BINARY_FLOAT` | `FLOAT` | 32 位浮点数 | `BINARY_FLOAT` → `FLOAT` |
| `BINARY_DOUBLE` | `DOUBLE` | 64 位浮点数 | `BINARY_DOUBLE` → `DOUBLE` |

### 关键注意事项

⚠️ **NUMBER 类型陷阱**

```sql
-- Oracle（默认精度很高）
CREATE TABLE test (amount NUMBER);  -- 默认 NUMBER(38, 127)
INSERT INTO test VALUES (123.456);  -- ✅ 可以存储小数

-- Snowflake（默认为整数）
CREATE TABLE test (amount NUMBER);  -- 默认 NUMBER(38, 0)
INSERT INTO test VALUES (123.456);  -- ❌ 小数部分被截断 -> 123
```

**最佳实践：**
```sql
-- ✅ 始终明确指定精度和标度
CREATE TABLE test (
    order_id NUMBER(10, 0),      -- 整数
    amount NUMBER(12, 2),         -- 金额（2位小数）
    ratio NUMBER(5, 4)            -- 比率（4位小数）
);
```

---

## 2. 字符串类型

| Oracle | Snowflake | 说明 | 示例 |
|--------|-----------|------|------|
| `CHAR(n)` | `CHAR(n)` | 定长字符串 | ✅ 兼容 |
| `VARCHAR2(n)` | `VARCHAR(n)` | 变长字符串 | ⚠️ 注意类型名称变化 |
| `NCHAR(n)` | `CHAR(n)` | Unicode 定长字符串 | Snowflake 原生支持 UTF-8 |
| `NVARCHAR2(n)` | `VARCHAR(n)` | Unicode 变长字符串 | Snowflake 原生支持 UTF-8 |
| `CLOB` | `VARCHAR(16777216)` | 大文本（最大 16 MB） | ⚠️ Snowflake 无 CLOB 类型 |
| `NCLOB` | `VARCHAR(16777216)` | Unicode 大文本 | 同上 |
| `LONG` | `VARCHAR(16777216)` | 旧式大文本 | ⚠️ Oracle 已不推荐使用 |

### 字符串类型转换示例

```sql
-- Oracle
CREATE TABLE employees (
    employee_id NUMBER(6),
    first_name VARCHAR2(20),
    last_name VARCHAR2(25),
    bio CLOB
);

-- Snowflake（迁移后）
CREATE TABLE employees (
    employee_id NUMBER(6, 0),
    first_name VARCHAR(20),
    last_name VARCHAR(25),
    bio VARCHAR(16777216)  -- CLOB → VARCHAR
);
```

---

## 3. 日期和时间类型

| Oracle | Snowflake | 说明 | 示例 |
|--------|-----------|------|------|
| `DATE` | `DATE` | 日期（无时间） | ✅ 兼容 |
| `TIMESTAMP` | `TIMESTAMP_NTZ` | 时间戳（无时区） | ⚠️ Snowflake 有 3 种时间戳类型 |
| `TIMESTAMP WITH TIME ZONE` | `TIMESTAMP_TZ` | 时间戳（带时区） | ✅ 兼容 |
| `TIMESTAMP WITH LOCAL TIME ZONE` | `TIMESTAMP_LTZ` | 本地时区时间戳 | ⚠️ 行为略有不同 |

### 时间戳类型详解

**Snowflake 的 3 种时间戳类型：**

1. **TIMESTAMP_NTZ** (No Time Zone) - 默认
   - 不包含时区信息
   - Oracle `TIMESTAMP` 对应此类型

2. **TIMESTAMP_TZ** (Time Zone)
   - 包含时区偏移量（如 `2025-01-15 14:30:00 +08:00`）
   - Oracle `TIMESTAMP WITH TIME ZONE` 对应此类型

3. **TIMESTAMP_LTZ** (Local Time Zone)
   - 存储为 UTC，显示时转换为会话时区
   - Oracle `TIMESTAMP WITH LOCAL TIME ZONE` 对应此类型

**迁移示例：**
```sql
-- Oracle
CREATE TABLE events (
    event_id NUMBER,
    event_date DATE,
    created_at TIMESTAMP,
    scheduled_at TIMESTAMP WITH TIME ZONE
);

-- Snowflake
CREATE TABLE events (
    event_id NUMBER(38, 0),
    event_date DATE,
    created_at TIMESTAMP_NTZ,             -- Oracle TIMESTAMP
    scheduled_at TIMESTAMP_TZ             -- Oracle TIMESTAMP WITH TIME ZONE
);
```

---

## 4. 二进制类型

| Oracle | Snowflake | 说明 | 示例 |
|--------|-----------|------|------|
| `RAW(n)` | `BINARY(n)` | 定长二进制 | ✅ 兼容 |
| `LONG RAW` | `BINARY(8388608)` | 变长二进制（最大 8 MB） | ⚠️ Oracle 已不推荐使用 |
| `BLOB` | `BINARY(8388608)` | 大二进制对象 | ⚠️ Snowflake 无 BLOB 类型 |

---

## 5. 特殊类型

| Oracle | Snowflake | 说明 | 示例 |
|--------|-----------|------|------|
| `ROWID` | `VARCHAR(18)` | 行标识符 | Snowflake 无原生支持 |
| `UROWID` | `VARCHAR(4000)` | 通用行标识符 | 同上 |
| `XMLType` | `VARIANT` | XML 数据 | ⚠️ 需要转换为 JSON/VARIANT |
| - | `VARIANT` | 半结构化数据（JSON, Avro, Parquet） | ✅ Snowflake 特有 |
| - | `OBJECT` | JSON 对象 | ✅ Snowflake 特有 |
| - | `ARRAY` | 数组 | ✅ Snowflake 特有 |

---

## 6. 自动化转换脚本

### 6.1 类型映射函数

```python
def map_oracle_to_snowflake(oracle_type):
    """
    将 Oracle 数据类型转换为 Snowflake 数据类型
    """
    type_mappings = {
        r'NUMBER\((\d+),(\d+)\)': r'NUMBER(\1,\2)',      # NUMBER(p,s) → NUMBER(p,s)
        r'NUMBER\((\d+)\)': r'NUMBER(\1,0)',             # NUMBER(p) → NUMBER(p,0)
        r'NUMBER': 'NUMBER(38,0)',                       # NUMBER → NUMBER(38,0) ⚠️
        r'VARCHAR2\((\d+)\)': r'VARCHAR(\1)',            # VARCHAR2(n) → VARCHAR(n)
        r'NVARCHAR2\((\d+)\)': r'VARCHAR(\1)',           # NVARCHAR2(n) → VARCHAR(n)
        r'CHAR\((\d+)\)': r'CHAR(\1)',                   # CHAR(n) → CHAR(n)
        r'CLOB': 'VARCHAR(16777216)',                    # CLOB → VARCHAR(16MB)
        r'BLOB': 'BINARY(8388608)',                      # BLOB → BINARY(8MB)
        r'DATE': 'DATE',                                 # DATE → DATE
        r'TIMESTAMP': 'TIMESTAMP_NTZ',                   # TIMESTAMP → TIMESTAMP_NTZ
        r'TIMESTAMP WITH TIME ZONE': 'TIMESTAMP_TZ',     # TIMESTAMP WITH TIME ZONE → TIMESTAMP_TZ
        r'RAW\((\d+)\)': r'BINARY(\1)',                  # RAW(n) → BINARY(n)
    }

    import re
    for pattern, replacement in type_mappings.items():
        if re.match(pattern, oracle_type, re.IGNORECASE):
            return re.sub(pattern, replacement, oracle_type, flags=re.IGNORECASE)

    return oracle_type  # 未匹配则返回原类型

# 使用示例
print(map_oracle_to_snowflake('NUMBER(10,2)'))  # 输出：NUMBER(10,2)
print(map_oracle_to_snowflake('VARCHAR2(100)')) # 输出：VARCHAR(100)
print(map_oracle_to_snowflake('CLOB'))          # 输出：VARCHAR(16777216)
```

---

## 7. 常见陷阱和解决方案

### 陷阱 1：NUMBER 类型精度丢失

**问题：**
```sql
-- Oracle
CREATE TABLE orders (amount NUMBER);
INSERT INTO orders VALUES (123.45);  -- ✅ 可以存储

-- Snowflake（错误迁移）
CREATE TABLE orders (amount NUMBER);  -- 默认 NUMBER(38, 0)
INSERT INTO orders VALUES (123.45);   -- ❌ 变成 123（小数丢失）
```

**解决方案：**
```sql
-- ✅ 始终明确指定精度
CREATE TABLE orders (amount NUMBER(12, 2));
```

### 陷阱 2：CLOB/BLOB 大小限制

**问题：** Snowflake 的 VARCHAR/BINARY 有最大长度限制

**解决方案：**
- 如果数据量 ≤ 16 MB：使用 `VARCHAR(16777216)` 或 `BINARY(8388608)`
- 如果数据量 > 16 MB：考虑存储到外部文件系统（S3），数据库仅保存引用

### 陷阱 3：时区处理

**问题：** Oracle 和 Snowflake 的时区行为不完全一致

**解决方案：**
- 统一使用 UTC 时区存储
- 在应用层处理时区转换

---

## 8. 验证 Checklist

迁移后必须验证：

- [ ] 所有 `NUMBER` 类型都指定了精度和标度
- [ ] `VARCHAR2` 已改为 `VARCHAR`
- [ ] `CLOB` 已改为 `VARCHAR(16777216)`
- [ ] `TIMESTAMP` 已改为 `TIMESTAMP_NTZ`
- [ ] 金额字段使用 `NUMBER(p, 2)` 格式
- [ ] 日期字段验证时区处理正确

---

**文档版本:** v2.1
**最后更新:** 2025-01-17
**维护人:** 数据库团队
