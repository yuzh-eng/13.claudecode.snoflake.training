# Mini-Project 迁移方案设计文档

## 项目信息
- 项目名称：CUSTOMER_ORDERS 模块迁移
- 负责人：张三
- 开始日期：2025-02-03
- 计划完成：2025-02-07（5天）

---

## 1. 项目概述

### 1.1 迁移范围

**涉及数据库对象：**
- 表：5 张（customers, orders, order_items, products, categories)
- 视图：2 个（customer_order_summary, top_customers）
- 存储过程：3 个（calculate_order_total, process_refund, generate_invoice）
- 函数：1 个（get_discount_rate）

**数据量：**
- customers: 50,000 行
- orders: 500,000 行
- order_items: 1,200,000 行
- products: 10,000 行
- categories: 50 行

---

## 2. 技术架构

### 2.1 表结构设计

#### CUSTOMERS 表

**Oracle DDL:**
```sql
CREATE TABLE customers (
    customer_id NUMBER(10) PRIMARY KEY,
    customer_name VARCHAR2(100) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    phone VARCHAR2(20),
    address CLOB,
    credit_limit NUMBER(12, 2),
    created_date DATE DEFAULT SYSDATE
);
```

**Snowflake DDL:**
```sql
CREATE TABLE customers (
    customer_id NUMBER(10, 0) PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    address VARCHAR(16777216),  -- CLOB → VARCHAR
    credit_limit NUMBER(12, 2),
    created_date DATE DEFAULT CURRENT_DATE()
);
```

#### ORDERS 表

**Snowflake DDL:**
```sql
CREATE TABLE orders (
    order_id NUMBER(10, 0) PRIMARY KEY,
    customer_id NUMBER(10, 0) NOT NULL,
    order_date DATE NOT NULL,
    total_amount NUMBER(12, 2),
    status VARCHAR(20),
    created_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
)
CLUSTER BY (order_date);  -- 添加聚簇键提升查询性能
```

---

## 3. 存储过程迁移

### 3.1 calculate_order_total

**迁移策略：** SQL Scripting

**Oracle 版本：**
```sql
CREATE OR REPLACE PROCEDURE calculate_order_total(
    p_order_id IN NUMBER,
    p_total OUT NUMBER
) AS
BEGIN
    SELECT SUM(quantity * unit_price * (1 - discount_rate))
    INTO p_total
    FROM order_items
    WHERE order_id = p_order_id;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        p_total := 0;
END;
```

**Snowflake 版本：**
```sql
CREATE OR REPLACE PROCEDURE calculate_order_total(
    p_order_id NUMBER
)
RETURNS NUMBER
LANGUAGE SQL
AS
$$
DECLARE
    v_total NUMBER DEFAULT 0;
BEGIN
    SELECT SUM(quantity * unit_price * (1 - discount_rate))
    INTO :v_total
    FROM order_items
    WHERE order_id = :p_order_id;
    RETURN v_total;
EXCEPTION
    WHEN STATEMENT_ERROR THEN
        RETURN 0;
END;
$$;
```

---

## 4. 数据迁移策略

### 4.1 迁移方式

| 表名 | 数据量 | 迁移方式 | 预计时间 |
|------|--------|---------|---------|
| customers | 50K | COPY INTO (CSV) | 2 分钟 |
| orders | 500K | COPY INTO (Parquet) | 5 分钟 |
| order_items | 1.2M | COPY INTO (Parquet) | 10 分钟 |
| products | 10K | COPY INTO (CSV) | 1 分钟 |
| categories | 50 | INSERT INTO | 10 秒 |

### 4.2 迁移步骤

```bash
# Step 1: 导出 Oracle 数据
python data_exporter.py --table customers --format csv
python data_exporter.py --table orders --format parquet
python data_exporter.py --table order_items --format parquet

# Step 2: 上传到 Snowflake Stage
snowsql -q "PUT file://customers.csv @my_stage"
snowsql -q "PUT file://orders.parquet @my_stage"

# Step 3: 加载数据
snowsql -q "COPY INTO customers FROM @my_stage/customers.csv"
snowsql -q "COPY INTO orders FROM @my_stage/orders.parquet"

# Step 4: 验证数据
python data_validator.py --table customers
python data_validator.py --table orders
```

---

## 5. 性能优化

### 5.1 聚簇键设计

```sql
ALTER TABLE orders CLUSTER BY (order_date);
ALTER TABLE order_items CLUSTER BY (order_id);
```

### 5.2 物化视图

```sql
CREATE MATERIALIZED VIEW customer_order_summary AS
SELECT c.customer_id,
       c.customer_name,
       COUNT(o.order_id) AS order_count,
       SUM(o.total_amount) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

---

## 6. 测试计划

### 6.1 单元测试

- [ ] 所有存储过程单元测试
- [ ] 视图查询结果验证
- [ ] 函数返回值验证

### 6.2 集成测试

- [ ] 完整业务流程测试
- [ ] 外键约束验证
- [ ] 触发器逻辑验证

### 6.3 性能测试

- [ ] 查询响应时间测试
- [ ] 并发测试（50 用户）
- [ ] 大数据量测试

---

## 7. 风险评估

| 风险 | 严重程度 | 缓解措施 |
|------|---------|---------|
| 数据精度丢失 | 高 | 严格验证小数位数 |
| 存储过程接口变化 | 中 | 提前通知下游系统 |
| 性能不达标 | 低 | 充分性能测试 |

---

## 8. 上线计划

**时间表：**
- Day 1: DDL 创建 + 数据迁移
- Day 2: 存储过程迁移 + 单元测试
- Day 3: 数据验证 + 集成测试
- Day 4: 性能测试 + 优化
- Day 5: Code Review + 文档整理

**上线检查清单：**
- [ ] 所有测试通过
- [ ] 文档完整
- [ ] Code Review 通过
- [ ] 导师批准

---

**文档版本：** v1.0
**最后更新：** 2025-02-03
**作者：** 张三
