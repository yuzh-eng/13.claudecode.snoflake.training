# Oracle → Snowflake 迁移培训项目

## 项目简介

本项目为 Oracle → Snowflake 数据库迁移培训项目，提供完整的 4 周培训计划和示例文档。

---

## 📚 培训计划

### 培训对象

- Java 迁移工程师（有 Oracle 经验，需要学习 Snowflake）

### 培训周期

- **时长：** 4 周（20 个工作日）
- **方式：** 自学 + 导师指导
- **输出导向：** 每天必须有代码或文档输出

### 培训内容

| 周次 | 主题 | 关键内容 |
|------|------|---------|
| **Week 1** | 基础入门 | 环境配置、SQL 对比、数据类型映射、Bug Bash |
| **Week 2** | 工具与流程 | 工具链、接口修改、JDBC、单元测试 |
| **Week 3** | 高级技术 | 存储过程、查询优化、数据验证、性能测试 |
| **Week 4** | 实战项目 | Mini-Project（完整迁移流程） |

---

## 📁 项目结构

```
.
├── Java_Migration_Engineer_Onboarding_4weeks.md  # 完整培训计划
├── training-feedback.md                          # 培训反馈问卷
├── candidates/                                   # 学员工作目录
│   └── sample_candidate/                         # 示例学员
│       ├── day01-environment/                    # Day 1: 环境准备
│       ├── day02-sql-basics/                     # Day 2: SQL 对比
│       ├── day03-datatype-mapping/               # Day 3: 数据类型映射
│       ├── day06-toolchain/                      # Day 6: 工具链配置
│       ├── day08-jdbc-bugfix/                    # Day 8: JDBC 连接
│       ├── day09-unit-test/                      # Day 9: 单元测试
│       ├── day11-stored-procedure/               # Day 11: 存储过程
│       ├── day12-query-optimization/             # Day 12: 查询优化
│       ├── day13-data-validation/                # Day 13: 数据验证
│       ├── day14-performance-testing/            # Day 14: 性能测试
│       └── day15-20-mini-project/                # Day 15-20: Mini-Project
├── wiki/                                         # 知识库
│   ├── sql-conversion-guide/                     # SQL 转换指南
│   │   └── datatype-mapping.md                   # 数据类型映射
│   ├── migration-playbook/                       # 迁移手册
│   │   ├── tool-chain-setup.md                   # 工具链配置
│   │   ├── interface-modification-SOP.md         # 接口修改 SOP
│   │   └── data-validation-guide.md              # 数据验证指南
│   ├── templates/                                # 模板文件
│   │   ├── interface-modification-checklist.md   # 接口修改清单
│   │   └── code-review-checklist.md              # Code Review 清单
│   ├── common-pitfalls/                          # 常见问题
│   │   └── jdbc-connection-issues.md             # JDBC 连接问题
│   ├── testing-guide/                            # 测试指南
│   │   └── unit-test-guidelines.md               # 单元测试指南
│   └── knowledge-sharing/                        # 知识分享
│       └── sample_candidate/                     # 示例学员分享
│           ├── Week1_Learning_Log.md             # Week 1 学习总结
│           ├── Week2_Error_Prevention_Checklist.md  # Week 2 错误预防
│           └── 4-Week_Learning_Summary.md        # 4 周总结
└── shared-utils/                                 # 共享工具
    ├── tools/                                    # 迁移工具
    │   └── README.md                              # 工具使用手册
    └── mini-project/                             # Mini-Project 需求
        └── sample_candidate/
            └── requirements.md                    # 项目需求文档
```

---

## 🎯 培训核心原则

### 1. 低管理成本

- 自学为主，导师指导为辅
- 详细的文档和示例
- 自动化工具和脚本

### 2. 输出导向

- **每天必须有输出：**
  - 代码 PR 或
  - 文档（设计文档、总结报告、Bug 报告）
- **严禁被动学习：**
  - ❌ "今天阅读了文档"
  - ✅ "今天编写了 SQL 对比文档"

### 3. 15 分钟规则

- 遇到问题先独立研究 15 分钟
- 培养独立解决问题的能力
- 15 分钟后仍无法解决再求助

### 4. Git 工作流

- 所有任务必须提交 PR
- Commit message 符合规范
- Code Review 必须通过

---

## 📖 学习路径

### Week 1: 基础入门

**Day 01 - 环境准备**
- Snowflake 账户配置
- 开发工具安装（DBeaver, SnowSQL）
- Hello World 程序
- Git 工作流验证

**Day 02 - SQL 语法对比**
- Oracle vs Snowflake SQL 差异
- JOIN、日期函数、字符串函数对比
- 编写对比文档

**Day 03 - 数据类型映射**
- NUMBER, VARCHAR, DATE 类型映射
- 修复数据类型精度 Bug
- 编写 Bug 修复报告

**Day 04 - Bug Bash**
- 修复数据类型相关 Bug
- 提交 PR

**Day 05 - 周总结**
- 编写 Week 1 学习总结
- 知识分享（15 分钟）

### Week 2: 工具与流程

**Day 06 - 工具链配置**
- Ora2Pg, SnowConvert, 自研工具
- 编写工具评测报告

**Day 07 - 接口修改 SOP**
- 学习接口修改流程
- 实践接口修改

**Day 08 - JDBC 连接**
- 修复 JDBC 连接问题
- 编写最佳实践文档

**Day 09 - 单元测试**
- 编写单元测试（覆盖率 ≥ 80%）
- 学习 Mock 和断言

**Day 10 - 周总结**
- 编写 Week 2 学习总结
- 错误预防清单

### Week 3: 高级技术

**Day 11 - 存储过程迁移**
- PL/SQL → SQL Scripting
- OUT 参数转换
- 编写迁移指南

**Day 12 - 查询优化**
- 聚簇键、物化视图、Search Optimization
- 性能优化案例
- 编写优化技巧总结

**Day 13 - 数据验证**
- 行数对比、列值对比、外键验证
- 编写验证报告

**Day 14 - 性能测试**
- 并发测试、响应时间测试
- 编写性能测试报告

**Day 15 - 周总结**
- 编写 Week 3 学习总结

### Week 4: Mini-Project

**Day 16-20 - 实战项目**
- 完整迁移 CUSTOMER_ORDERS 模块
- 5 张表 + 3 个存储过程
- 数据验证 + 性能测试
- 编写完整文档

---

## 🚀 快速开始

### 1. 查看培训计划

```bash
# 查看完整培训计划
cat Java_Migration_Engineer_Onboarding_4weeks.md
```

### 2. 参考示例文档

```bash
# 查看示例学员的文档
ls candidates/sample_candidate/
```

### 3. 使用模板和工具

```bash
# 查看模板
ls wiki/templates/

# 查看工具手册
cat shared-utils/tools/README.md
```

---

## 📝 文档说明

### 示例文档列表

**Day 01-09 文档（Week 1-2）：**
- ✅ `day01-environment/README.md` - 环境配置记录
- ✅ `day01-environment/environment-checklist.md` - 环境验证清单
- ✅ `day02-sql-basics/SQL_Comparison.md` - SQL 对比文档
- ✅ `day03-datatype-mapping/bugfix-report.md` - Bug 修复报告
- ✅ `day06-toolchain/setup-log.md` - 工具配置日志
- ✅ `day06-toolchain/tool-evaluation.md` - 工具评测报告
- ✅ `day08-jdbc-bugfix/bugfix-report.md` - JDBC Bug 修复
- ✅ `day08-jdbc-bugfix/jdbc-best-practices.md` - JDBC 最佳实践
- ✅ `day09-unit-test/test-cases.md` - 测试用例设计

**Day 11-14 文档（Week 3）：**
- ✅ `day11-stored-procedure/migration-plan.md` - 存储过程迁移方案
- ✅ `day11-stored-procedure/stored-procedure-migration-guide.md` - 迁移指南
- ✅ `day12-query-optimization/query-optimization-tips.md` - 查询优化技巧
- ✅ `day13-data-validation/validation-report.md` - 数据验证报告
- ✅ `day14-performance-testing/performance-test-template.md` - 性能测试模板

**Mini-Project 文档（Week 4）：**
- ✅ `day15-20-mini-project/migration-design.md` - 迁移方案设计
- ✅ `day15-20-mini-project/migration-summary.md` - 迁移总结报告

**Wiki 知识库：**
- ✅ `wiki/sql-conversion-guide/datatype-mapping.md` - 数据类型映射权威指南
- ✅ `wiki/migration-playbook/tool-chain-setup.md` - 工具链配置指南
- ✅ `wiki/migration-playbook/interface-modification-SOP.md` - 接口修改 SOP
- ✅ `wiki/migration-playbook/data-validation-guide.md` - 数据验证指南
- ✅ `wiki/templates/interface-modification-checklist.md` - 接口修改检查清单
- ✅ `wiki/templates/code-review-checklist.md` - Code Review 检查清单
- ✅ `wiki/common-pitfalls/jdbc-connection-issues.md` - JDBC 常见问题
- ✅ `wiki/testing-guide/unit-test-guidelines.md` - 单元测试指南

**学习总结：**
- ✅ `wiki/knowledge-sharing/sample_candidate/Week1_Learning_Log.md` - Week 1 总结
- ✅ `wiki/knowledge-sharing/sample_candidate/Week2_Error_Prevention_Checklist.md` - Week 2 错误预防
- ✅ `wiki/knowledge-sharing/sample_candidate/4-Week_Learning_Summary.md` - 4 周总结

**工具和需求：**
- ✅ `shared-utils/tools/README.md` - 迁移工具使用手册
- ✅ `shared-utils/mini-project/sample_candidate/requirements.md` - Mini-Project 需求

---

## 🎓 成功案例

**示例学员：张三**

- **培训周期：** 2025-01-15 至 2025-02-07（4 周）
- **培训成果：**
  - ✅ 完成 1 个完整迁移项目（Mini-Project）
  - ✅ 修复 10+ 个 Bug
  - ✅ 编写 20+ 份技术文档
  - ✅ 性能优化提升 2.9 倍
  - ✅ 培训评级：优秀

---

## 📊 培训效果

**技能提升：**
- Snowflake SQL：⭐ → ⭐⭐⭐⭐⭐
- 数据类型映射：⭐ → ⭐⭐⭐⭐⭐
- 存储过程转换：⭐ → ⭐⭐⭐⭐⭐
- 性能优化：⭐ → ⭐⭐⭐⭐⭐

**项目能力：**
- 能够独立完成迁移项目
- 掌握完整的迁移流程
- 具备性能优化能力
- 文档编写规范

---

## 🤝 贡献

欢迎提交改进建议和新的示例文档！

**提交方式：**
1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m "feat: add your feature"`
4. 推送分支：`git push origin feature/your-feature`
5. 创建 Pull Request

---

## 📞 联系方式

- **培训负责人：** 数据迁移团队
- **导师：** 李导师
- **反馈渠道：** 提交 Issue 或 PR

---

## 📄 许可证

本项目仅供内部培训使用。

---

**版本：** v1.0
**最后更新：** 2025-02-07
**维护团队：** 数据迁移团队
