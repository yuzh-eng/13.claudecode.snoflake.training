# Mini-Project 需求文档

## 项目概述

**项目名称：** CUSTOMER_ORDERS 模块迁移

**项目目标：** 将 Oracle 数据库中的客户订单模块完整迁移到 Snowflake

**预计时长：** 5 天（Day 15-20）

---

## 1. 迁移范围

### 1.1 数据库对象

**表（5 张）：**
1. `CUSTOMERS` - 客户信息表
2. `ORDERS` - 订单主表
3. `ORDER_ITEMS` - 订单明细表
4. `PRODUCTS` - 产品信息表
5. `CATEGORIES` - 产品分类表

**视图（2 个）：**
1. `CUSTOMER_ORDER_SUMMARY` - 客户订单汇总视图
2. `TOP_CUSTOMERS` - 最有价值客户视图（VIP）

**存储过程（3 个）：**
1. `calculate_order_total` - 计算订单总额
2. `process_refund` - 处理退款
3. `generate_invoice` - 生成发票

**函数（1 个）：**
1. `get_discount_rate` - 获取客户折扣率

### 1.2 数据量

| 表名 | 行数 | 数据大小 |
|------|------|---------|
| CUSTOMERS | 50,000 | 25 MB |
| ORDERS | 500,000 | 200 MB |
| ORDER_ITEMS | 1,200,000 | 500 MB |
| PRODUCTS | 10,000 | 5 MB |
| CATEGORIES | 50 | < 1 MB |

**总数据量：** ~730 MB

---

## 2. 功能需求

### 2.1 数据迁移要求

- ✅ 100% 数据准确性（无数据丢失）
- ✅ 保持数据完整性（主键、外键约束）
- ✅ 保持数据精度（小数位数）
- ✅ 保持 NULL 值处理

### 2.2 性能要求

| 指标 | 目标值 | 基准（Oracle） |
|------|--------|---------------|
| 单表查询响应时间 | ≤ 2 秒 | 5 秒 |
| JOIN 查询响应时间 | ≤ 5 秒 | 10 秒 |
| 聚合查询响应时间 | ≤ 10 秒 | 30 秒 |
| 并发支持 | 50 用户 | 20 用户 |

### 2.3 成本要求

- 运行成本不高于 Oracle（按相同查询量计算）
- 合理选择 Warehouse 大小

---

## 3. 技术要求

### 3.1 代码质量

- [ ] 所有 DDL 符合 Snowflake 最佳实践
- [ ] 所有 `NUMBER` 类型明确指定精度
- [ ] 所有存储过程使用 SQL Scripting
- [ ] Code Review 通过（≥ 2 个 Approve）

### 3.2 测试要求

- [ ] 单元测试覆盖率 ≥ 80%
- [ ] 所有存储过程有单元测试
- [ ] 数据验证 100% 通过
- [ ] 性能测试达标

### 3.3 文档要求

- [ ] 迁移方案设计文档
- [ ] 数据验证报告
- [ ] 性能测试报告
- [ ] 迁移总结报告

---

## 4. 详细需求

### 4.1 CUSTOMERS 表

**Oracle DDL：**
```sql
CREATE TABLE CUSTOMERS (
    customer_id NUMBER(10) PRIMARY KEY,
    customer_name VARCHAR2(100) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    phone VARCHAR2(20),
    address CLOB,
    credit_limit NUMBER(12, 2),
    customer_type VARCHAR2(20),  -- 'RETAIL' or 'BUSINESS'
    created_date DATE DEFAULT SYSDATE
);
```

**迁移要求：**
- `customer_id` 为主键
- `email` 唯一约束
- `address` 字段（CLOB）需要转换为 VARCHAR
- `credit_limit` 保持 2 位小数精度

### 4.2 ORDERS 表

**Oracle DDL：**
```sql
CREATE TABLE ORDERS (
    order_id NUMBER(10) PRIMARY KEY,
    customer_id NUMBER(10) NOT NULL,
    order_date DATE NOT NULL,
    total_amount NUMBER(12, 2),
    discount_amount NUMBER(10, 2),
    tax_amount NUMBER(10, 2),
    status VARCHAR2(20),  -- 'PENDING', 'COMPLETED', 'CANCELLED'
    created_at TIMESTAMP DEFAULT SYSTIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES CUSTOMERS(customer_id)
);
```

**迁移要求：**
- 外键约束必须保留
- 建议添加聚簇键（按 `order_date`）
- 金额字段保持 2 位小数

### 4.3 calculate_order_total 存储过程

**功能：** 计算订单总额（含税和折扣）

**Oracle 实现：**
```sql
CREATE OR REPLACE PROCEDURE calculate_order_total(
    p_order_id IN NUMBER,
    p_total OUT NUMBER
) AS
    v_subtotal NUMBER;
    v_discount NUMBER;
    v_tax NUMBER;
BEGIN
    -- 计算小计
    SELECT SUM(quantity * unit_price) INTO v_subtotal
    FROM order_items
    WHERE order_id = p_order_id;

    -- 获取折扣和税
    SELECT discount_amount, tax_amount
    INTO v_discount, v_tax
    FROM orders
    WHERE order_id = p_order_id;

    -- 计算总额
    p_total := v_subtotal - v_discount + v_tax;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        p_total := 0;
END;
```

**迁移要求：**
- OUT 参数改为 RETURNS
- 使用 Snowflake SQL Scripting
- 保持业务逻辑一致

---

## 5. 验收标准

### 5.1 数据验证

- [ ] 所有表行数 100% 一致
- [ ] 所有数值列校验和一致（误差 < 0.01）
- [ ] NULL 值数量一致
- [ ] 主键无重复
- [ ] 外键无孤立记录

### 5.2 功能验证

- [ ] 所有存储过程执行成功
- [ ] 所有视图查询成功
- [ ] 业务逻辑与 Oracle 一致

### 5.3 性能验证

- [ ] 所有查询响应时间达标
- [ ] 并发测试通过（50 用户）
- [ ] 大数据量测试通过

### 5.4 文档验证

- [ ] 所有文档完整
- [ ] Code Review 通过
- [ ] 导师审核通过

---

## 6. 提交要求

### 6.1 代码提交

**目录结构：**
```
candidates/
└── {your_name}/
    └── day15-20-mini-project/
        ├── ddl/                    # 所有表结构 DDL
        ├── procedures/             # 存储过程代码
        ├── views/                  # 视图定义
        ├── functions/              # 函数定义
        ├── tests/                  # 单元测试
        ├── migration-design.md     # 方案设计
        ├── data-validation-report.md  # 数据验证报告
        ├── performance-comparison.md  # 性能对比报告
        └── migration-summary.md    # 总结报告
```

### 6.2 Git 规范

- 创建分支：`feature/{your-name}-mini-project`
- Commit message 符合规范
- 提交 Pull Request
- 获得 ≥ 2 个 Approve

---

## 7. 评分标准

| 评分项 | 权重 | 评分标准 |
|--------|------|---------|
| 数据准确性 | 30% | 100% 准确（30 分） |
| 性能表现 | 25% | 达标（25 分） |
| 代码质量 | 20% | Code Review 通过（20 分） |
| 测试覆盖率 | 15% | ≥ 80%（15 分） |
| 文档完整性 | 10% | 完整规范（10 分） |

**总分：** 100 分

**评级：**
- 90-100 分：优秀
- 80-89 分：良好
- 70-79 分：合格
- < 70 分：需改进

---

## 8. 时间规划建议

| 天数 | 任务 | 预计时间 |
|------|------|---------|
| Day 1 | DDL 创建 + 数据迁移 | 8 小时 |
| Day 2 | 存储过程迁移 + 单元测试 | 8 小时 |
| Day 3 | 数据验证 + 集成测试 | 8 小时 |
| Day 4 | 性能测试 + 优化 | 8 小时 |
| Day 5 | Code Review + 文档整理 | 8 小时 |

---

## 9. 参考资料

- 内部 Wiki：`wiki/migration-playbook/`
- 数据类型映射：`wiki/sql-conversion-guide/datatype-mapping.md`
- 存储过程迁移指南：`candidates/{your_name}/day11-stored-procedure/stored-procedure-migration-guide.md`
- 性能优化技巧：`candidates/{your_name}/day12-query-optimization/query-optimization-tips.md`

---

## 10. 常见问题

**Q: 是否需要迁移数据？**
A: 是的，需要迁移全部数据并验证准确性。

**Q: 性能不达标怎么办？**
A: 使用聚簇键、物化视图等优化方法，并选择合适的 Warehouse 大小。

**Q: 遇到技术难题怎么办？**
A: 先独立研究 15 分钟，然后可以请教导师或查阅文档。

**Q: 能否使用 JavaScript UDF？**
A: 推荐使用 SQL Scripting，除非有特殊需求。

---

**文档版本：** v1.0
**发布日期：** 2025-02-03
**维护人：** 培训团队
