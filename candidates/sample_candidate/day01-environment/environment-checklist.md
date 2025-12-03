# 环境准备验证清单

## 培训学员信息
- 姓名：余志豪
- 完成日期：2025-12-02
- 审核人：魏导师

---

## 1. Snowflake 账户访问

- [√] 已收到账户邀请邮件
- [√] 成功设置初始密码
- [√] 已启用 MFA（多因素认证）
- [√] 可以登录 Snowflake Web UI
- [√] 可以访问指定的数据库和 Schema
- [√] 可以使用 Warehouse 执行查询

**验证截图：** ![`snowflake-login.png`](snowflake-login.png)

**账户信息：**
```
Account: QX45382
Username: YUZHIHAO
Default Role: DEVELOPER
Default Warehouse: COMPUTE_WH
Default Database: MIGRATION_DB
```

---

## 2. 本地开发工具

### 2.1 IDE 和编辑器
- [√] IntelliJ IDEA（版本：2025.1.1）
- [√] VS Code（可选，版本：1.85.0）
- [√] 已安装必要插件
- [√] SQL Language Support
- [√] Git Integration
- [√] Markdown Preview

### 2.2 数据库客户端
- [√] DBeaver（版本：25.3.0）
- [×] Oracle SQL Developer（版本：23.1）
- [√] SnowSQL CLI（版本：1.4.5）

**DBeaver 连接测试：**
- [√] Oracle 连接成功
- [√] Snowflake 连接成功
- [√] 可以执行简单查询

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
- [√] Git（版本：2.25.3）
- [√] 配置了 Git 用户信息
  <!-- ```bash
  git config --global user.name "Zhang San"
  git config --global user.email "zhang.san@company.com"
  ``` -->
- [√] 配置了 SSH Key
- [√] 可以克隆团队仓库

---

## 3. 网络和权限

<!-- - [√] 可以访问公司 VPN -->
- [√] 可以访问内部 Wiki（wiki.company.com）
- [√] 可以访问 Notion 工作空间
- [√] 可以访问 Git 仓库
- [√] 可以访问 Snowflake 文档（docs.snowflake.com）
- [√] 可以访问 Oracle 文档（docs.oracle.com）

<!-- **VPN 配置：**
- 服务器：vpn.company.com
- 协议：OpenVPN
- 测试命令：`ping internal-wiki.company.local` -->

---

## 4. 文档和资源

- [√] 已加入团队 Slack/Teams 频道
- [√] 已获得 Notion 访问权限
- [√] 已阅读 `README.md`（项目根目录）
- [√] 已阅读 `CONTRIBUTING.md`
- [√] 已了解团队的代码规范和 Git 工作流

<!-- **重要文档链接：**
- [团队 Wiki](https://wiki.company.com/migration)
- [Notion 工作空间](https://notion.so/company/migration-team)
- [代码仓库](https://github.com/company/snowflake-migration)
- [培训计划](./Java_Migration_Engineer_Onboarding_4weeks.md) -->

---

## 5. Hello World 验证

- [√] 成功在 Oracle 中执行 Hello World 查询
- [√] 成功在 Snowflake 中执行 Hello World 查询
- [√] 创建了代码对比文档
- [√] 理解了基本的 SQL 语法差异
<!-- 
**执行结果：**
```sql
-- Oracle
SELECT 'Hello World from Oracle' FROM DUAL;
-- 结果：Hello World from Oracle

-- Snowflake
SELECT 'Hello World from Snowflake';
-- 结果：Hello World from Snowflake
```

--- -->

## 6. Git 工作流验证

- [√] 成功创建功能分支
- [√] 提交符合规范的 Commit Message
- [√] 创建并提交了第一个 Pull Request
<!-- - [x] PR 通过了自动化检查（CI/CD）
- [x] PR 获得了导师的 Code Review -->

<!-- **第一个 PR：**
- PR 链接：`https://github.com/company/snowflake-migration/pull/1234`
- 标题：`docs: add day01 environment setup`
- 状态：✅ Merged

--- -->

<!-- ## 遇到的问题和解决方案

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
- **耗时：** 20 分钟 -->

---

## 审核签字

- [ ] 导师已验证所有环境配置正确
- [ ] 导师已审核 Hello World 代码
- [ ] 导师已审核第一个 PR

**导师签字：** ________________
**日期：** 2025-01-15
