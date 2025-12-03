# 环境准备验证清单

## 培训学员信息
- 姓名：张三
- 完成日期：2025-01-15
- 审核人：李导师

---

## 1. Snowflake 账户访问

- [x] 已收到账户邀请邮件
- [x] 成功设置初始密码
- [x] 已启用 MFA（多因素认证）
- [x] 可以登录 Snowflake Web UI
- [x] 可以访问指定的数据库和 Schema
- [x] 可以使用 Warehouse 执行查询

**验证截图：** `screenshots/snowflake-login.png`

**账户信息：**
```
Account: xyz12345
Username: ZHANG_SAN
Default Role: DEVELOPER
Default Warehouse: COMPUTE_WH
Default Database: MIGRATION_DB
```

---

## 2. 本地开发工具

### 2.1 IDE 和编辑器
- [x] IntelliJ IDEA（版本：2024.1.1）
- [x] VS Code（可选，版本：1.85.0）
- [x] 已安装必要插件：
  - [x] SQL Language Support
  - [x] Git Integration
  - [x] Markdown Preview

### 2.2 数据库客户端
- [x] DBeaver（版本：23.3.0）
- [x] Oracle SQL Developer（版本：23.1）
- [x] SnowSQL CLI（版本：1.2.28）

**DBeaver 连接测试：**
- [x] Oracle 连接成功
- [x] Snowflake 连接成功
- [x] 可以执行简单查询

**SnowSQL 测试：**
```bash
$ snowsql -a xyz12345 -u ZHANG_SAN
Password: ****
zhang_san#COMPUTE_WH@MIGRATION_DB.PUBLIC> SELECT CURRENT_VERSION();
+-------------------+
| CURRENT_VERSION() |
|-------------------|
| 8.2.0             |
+-------------------+
```

### 2.3 版本控制工具
- [x] Git（版本：2.43.0）
- [x] 配置了 Git 用户信息
  ```bash
  git config --global user.name "Zhang San"
  git config --global user.email "zhang.san@company.com"
  ```
- [x] 配置了 SSH Key
- [x] 可以克隆团队仓库

---

## 3. 网络和权限

- [x] 可以访问公司 VPN
- [x] 可以访问内部 Wiki（wiki.company.com）
- [x] 可以访问 Notion 工作空间
- [x] 可以访问 Git 仓库
- [x] 可以访问 Snowflake 文档（docs.snowflake.com）
- [x] 可以访问 Oracle 文档（docs.oracle.com）

**VPN 配置：**
- 服务器：vpn.company.com
- 协议：OpenVPN
- 测试命令：`ping internal-wiki.company.local`

---

## 4. 文档和资源

- [x] 已加入团队 Slack/Teams 频道
- [x] 已获得 Notion 访问权限
- [x] 已阅读 `README.md`（项目根目录）
- [x] 已阅读 `CONTRIBUTING.md`
- [x] 已了解团队的代码规范和 Git 工作流

**重要文档链接：**
- [团队 Wiki](https://wiki.company.com/migration)
- [Notion 工作空间](https://notion.so/company/migration-team)
- [代码仓库](https://github.com/company/snowflake-migration)
- [培训计划](./Java_Migration_Engineer_Onboarding_4weeks.md)

---

## 5. Hello World 验证

- [x] 成功在 Oracle 中执行 Hello World 查询
- [x] 成功在 Snowflake 中执行 Hello World 查询
- [x] 创建了代码对比文档
- [x] 理解了基本的 SQL 语法差异

**执行结果：**
```sql
-- Oracle
SELECT 'Hello World from Oracle' FROM DUAL;
-- 结果：Hello World from Oracle

-- Snowflake
SELECT 'Hello World from Snowflake';
-- 结果：Hello World from Snowflake
```

---

## 6. Git 工作流验证

- [x] 成功创建功能分支
- [x] 提交符合规范的 Commit Message
- [x] 创建并提交了第一个 Pull Request
- [x] PR 通过了自动化检查（CI/CD）
- [x] PR 获得了导师的 Code Review

**第一个 PR：**
- PR 链接：`https://github.com/company/snowflake-migration/pull/1234`
- 标题：`docs: add day01 environment setup`
- 状态：✅ Merged

---

## 遇到的问题和解决方案

### 问题 1：MFA 设置失败
- **描述：** 扫描二维码后，验证码始终提示错误
- **原因：** 手机时间未同步
- **解决：** 启用手机"自动设置时间"功能
- **耗时：** 15 分钟（包括查找文档时间）

### 问题 2：DBeaver 连接 Snowflake 超时
- **描述：** 连接时提示 Connection timeout
- **原因：** 未连接公司 VPN
- **解决：** 连接 VPN 后重试
- **耗时：** 10 分钟

### 问题 3：Git Push 权限被拒绝
- **描述：** `git push` 时提示 Permission denied (publickey)
- **原因：** SSH Key 未添加到 GitHub
- **解决：** 生成 SSH Key 并添加到 GitHub 账户
- **耗时：** 20 分钟

---

## 审核签字

- [ ] 导师已验证所有环境配置正确
- [ ] 导师已审核 Hello World 代码
- [ ] 导师已审核第一个 PR

**导师签字：** ________________
**日期：** 2025-01-15
