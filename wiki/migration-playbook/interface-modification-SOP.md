# 接口修改标准操作流程 (SOP)

## 1. 概述

本文档定义了在迁移过程中修改数据库接口时的标准操作流程，确保修改的安全性和可追溯性。

---

## 2. 适用场景

以下情况需要遵循本 SOP：
- 修改表结构（增删改列）
- 修改存储过程签名
- 修改视图定义
- 修改函数接口
- 修改触发器逻辑

---

## 3. 操作流程

### 步骤 1：需求分析

- [ ] 明确修改原因（Bug 修复、性能优化、功能增强）
- [ ] 评估影响范围（调用方、依赖关系）
- [ ] 确定修改方案（兼容性、性能影响）

### 步骤 2：设计评审

- [ ] 编写设计文档（包含修改前后对比）
- [ ] 提交 Design Review（至少 1 位 Senior 审核）
- [ ] 评估回滚方案

### 步骤 3：代码实现

- [ ] 创建功能分支：`feature/{your-name}-{task-name}`
- [ ] 实现修改（DDL/DML/代码）
- [ ] 编写单元测试（覆盖率 ≥ 80%）
- [ ] 本地测试通过

### 步骤 4：代码审查

- [ ] 提交 Pull Request
- [ ] 填写 `interface-modification-checklist.md`
- [ ] 至少 2 位 Reviewer 批准
- [ ] CI/CD 检查通过

### 步骤 5：测试验证

- [ ] 在测试环境部署
- [ ] 执行集成测试
- [ ] 执行性能测试
- [ ] 验证回滚流程

### 步骤 6：生产部署

- [ ] 编写部署脚本
- [ ] 在维护窗口期执行
- [ ] 监控关键指标
- [ ] 确认功能正常

### 步骤 7：文档更新

- [ ] 更新 API 文档
- [ ] 更新团队 Wiki
- [ ] 通知下游团队
- [ ] 归档变更记录

---

## 4. 示例：修改存储过程接口

### 4.1 修改前

**Oracle 存储过程：**
```sql
CREATE OR REPLACE PROCEDURE get_employee_info(
    emp_id IN NUMBER,
    emp_name OUT VARCHAR2,
    emp_salary OUT NUMBER
) AS
BEGIN
    SELECT first_name, salary
    INTO emp_name, emp_salary
    FROM employees
    WHERE employee_id = emp_id;
END;
```

### 4.2 修改后

**Snowflake 存储过程：**
```sql
CREATE OR REPLACE PROCEDURE get_employee_info(emp_id NUMBER)
RETURNS TABLE (emp_name VARCHAR, emp_salary NUMBER, emp_dept VARCHAR)
LANGUAGE SQL
AS
$$
BEGIN
    LET result RESULTSET := (
        SELECT first_name AS emp_name,
               salary AS emp_salary,
               department_name AS emp_dept  -- ✅ 新增部门字段
        FROM employees e
        JOIN departments d ON e.department_id = d.department_id
        WHERE e.employee_id = :emp_id
    );
    RETURN TABLE(result);
END;
$$;
```

### 4.3 修改说明文档

**diff-report.md:**

```markdown
## 接口变更

### 参数变化
- ✅ 新增返回字段：`emp_dept`（部门名称）
- ⚠️ 返回方式变化：OUT 参数 → TABLE 返回

### 兼容性影响
- ❌ 不兼容：调用方需要修改代码
- 受影响的调用方：
  - Java 应用：EmployeeService.java
  - Python 脚本：employee_report.py

### 迁移步骤
1. 更新 Java 代码，使用 ResultSet 接收返回值
2. 更新 Python 代码，使用 cursor.fetchall()
```

### 4.4 Checklist 填写

**interface-modification-checklist.md:**

- [x] 已评估影响范围
- [x] 已通知下游团队
- [x] 已编写单元测试
- [x] 已更新 API 文档
- [x] 已验证回滚流程

---

## 5. 注意事项

### 5.1 兼容性原则

- ✅ 优先选择兼容性方案（向后兼容）
- ⚠️ 如必须破坏兼容性，提前至少 1 周通知下游
- ❌ 禁止在生产环境直接删除字段或接口

### 5.2 回滚策略

- 所有修改必须可回滚
- 回滚脚本必须提前准备并测试
- 如果无法回滚，必须经过 Manager 批准

### 5.3 沟通机制

- 修改前：邮件通知 + Wiki 公告
- 修改中：Slack 实时同步进度
- 修改后：发布 Release Notes

---

## 6. 常见问题

**Q: 如果修改影响了多个下游系统怎么办？**

A:
1. 召开评审会议，邀请所有下游团队
2. 协调统一的发布时间窗口
3. 准备 Feature Flag，支持灰度发布

**Q: 如果紧急 Bug 需要跳过某些步骤？**

A:
1. 必须经过 Tech Lead 或 Manager 批准
2. 事后补充缺失的文档和测试
3. 在下一个 Sprint 补充完整流程

---

**文档版本:** v1.2
**最后更新:** 2025-01-23
**维护人:** 架构团队
