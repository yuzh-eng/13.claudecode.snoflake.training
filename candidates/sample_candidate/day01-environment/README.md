# Day 01 - 环境准备

## 培训学员

- 姓名：余志豪
- 日期：2025-12-02

## 环境配置过程

## 环境清单
- **操作系统**: Windows 11
- **JDK 版本**: 
  - Java 25（最初使用）
  - Java 17（最终用于运行 Snowflake JDBC）
- **IntelliJ IDEA 版本**: IntelliJ IDEA Community Edition 2025.2.5
- **Git 版本**: （待你填入：git --version 的输出）
- **Maven 版本**: （待你填入：mvn --version 的输出）

## Snowflake 配置
- **Account Locator**: （你公司 Snowflake 的域名，例如：xxx.ap-northeast-1.snowflakecomputing.com）
- **Database**: MIGRATION_TRAINING
- **Schema**: PRACTICE
- **Warehouse**: COMPUTE_WH
- **JDBC Driver Version**: 3.13.30

## Oracle 配置
- **方案**: Oracle XE 本地安装
- **版本**: Oracle Database 21c XE
- **JDBC Driver Version**: 21.9.0.0


# 遇到的问题与解决方案

## 问题 1：Snowflake JDBC 无法运行（MFA 阻挡登录）
- **现象**:  
  Snowflake 报错  
MFA authentication is required, but none of your current MFA methods are supported for programmatic authentication


- **原因**: 账户启用了 MFA，而 JDBC 不能使用 MFA 登录
- **解决方案**:  
- 管理员设置 `MINS_TO_BYPASS_MFA` 或关闭 MFA  
- 程序成功连接 Snowflake

## 问题 2：Java 版本冲突（UnsupportedClassVersionError）
- **现象**:
class file version 69.0, this Java Runtime only recognizes up to 61.0


- **原因**:
- 使用 Java 25 编译，但运行时切换到 Java 17
- **解决方案**:
- 统一使用 Java 17（兼容 Snowflake JDBC）
- 或修改 IntelliJ 的 Project SDK / Run JDK

---

## 问题 3：Snowflake JDBC 报错 MemoryUtil / add-opens
- **现象**:
Failed to initialize MemoryUtil.
module java.base does not "opens java.nio" to unnamed module


- **原因**: Snowflake JDBC（内部使用 Apache Arrow）需要访问 java.nio.Buffer 内部字段，而 Java 17 默认禁止
- **解决方案**:
- 在 IntelliJ Run Configuration → VM Options 添加：  
  ```
  --add-opens=java.base/java.nio=ALL-UNNAMED
  ```
- 成功运行 Snowflake 查询

---
## 问题 4：Snowflake 域名解析错误 → 指向错误的 AWS 区域
- **现象**

nslookup dqkoolq-lt05012.ap-northeast-1.snowflakecomputing.com 返回：

af5513c7....elb.us-west-2.amazonaws.com


也就是说 我的东京（ap-northeast-1）Snowflake 实例被错误解析到美国西区（us-west-2）

导致 JDBC 证书校验失败：

Certificate doesn't match any of the subject alternative names
- **原因**

我的网络（NTT FLET’S + IPv6 + DS-Lite）使用了 运营商 NAT + IPv6 DNS，DNS 把 Snowflake 域名解析到 AWS 美国区。

- **解决方案**

关闭 WiFi 网卡的 IPv6：

Disable-NetAdapterBinding -Name "Wi-Fi" -ComponentID ms_tcpip6


并将 DNS 改为 IPv4：

8.8.8.8

8.8.4.4

重新 ipconfig /all 后 IPv6 DNS 已被移除 → 解析恢复正常。

✔ 解决证书 SAN mismatch 问题

## 问题 5：成功解析后又报 MFA 错误 → 需要使用 KEY PAIR 登录
- **现象**

JDBC 提示：

MFA authentication is required, but none of your current MFA methods are supported for programmatic authentication.

- **原因**


我使用的是 UI 密码登录 + MFA，但是 JDBC 无法使用 DUO / Authenticator 等交互式 MFA。

- **解决方案**

必须切换成 Key Pair Authentication

流程：

生成 RSA 私钥（登录用）

将公钥上传到 Snowflake 用户

JDBC 使用私钥连接

## 问题 6：公钥被 Snowflake 拒绝 → Invalid Public Key
### 问题：Snowflake 拒绝公钥

#### 现象
New public key rejected by current policy. Reason: 'Invalid Public key'


#### 原因
我最初生成的公钥格式为：

ssh-rsa AAAA...



这是 **OpenSSH 格式**，Snowflake **不接受**该格式。

Snowflake 需要的格式是 **PEM / PKCS8**，如下所示：

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqh...
-----END PUBLIC KEY-----




#### 解决方案（正确生成方法）

**1. 生成 PEM 格式私钥**

ssh-keygen -t rsa -b 4096 -m PEM -f snowflake_rsa_key
2. 使用 openssl 生成 PKCS8 公钥

openssl rsa -in snowflake_rsa_key -pubout -out snowflake_rsa_key.pub.pem
3. 将公钥内容写入 Snowflake 用户


ALTER USER PC010 SET RSA_PUBLIC_KEY='-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----';
已验证：以上生成的公钥格式 Snowflake 可以成功接受

---

## 问题 7：Snowflake JDBC 报错 MemoryUtil / add-opens
现象
Failed to initialize MemoryUtil.
module java.base does not "opens java.nio" to unnamed module

原因

Snowflake JDBC（内部使用 Apache Arrow）需要访问 java.nio.Buffer 的内部字段
但 Java 17+ 默认禁止非法反射 → 需要手动开放模块。

## 解决方案

在 IntelliJ → Run Configuration → VM Options 添加：

--add-opens=java.base/java.nio=ALL-UNNAMED


之后 Snowflake JDBC 程序成功运行。

## IntelliJ 配置中遇到的额外问题
- 找不到 VM Options  
- 原因：创建的是错误的运行配置类型  
- 解决方案：  
- 使用 “+” → **应用程序（Application）** 创建新的运行配置  
- 才会出现 VM Options 项  
- 正确填写 Main class & VM Options

## 学习收获

1. 理解了 Snowflake 域名解析机制，以及为什么 IPv6 / ISP 会导致域名被解析到错误区域（如解析到 us-west-2 → 导致证书 SAN mismatch）。
2. 学会了在 Windows 中通过 **禁用 IPv6 + 设置 IPv4 DNS** 的方式修复 Snowflake 的域名解析问题。
3. 掌握了 Snowflake JDBC 在启用 MFA 时的限制，明确了程序必须使用 **Key Pair Authentication** 才能在启用 MFA 的情况下连接。
4. 理解了 **OpenSSH 公钥格式与 Snowflake 支持格式不兼容** 的原因，并学会使用 **PEM / PKCS8** 格式生成可被 Snowflake 接受的公钥。
5. 了解了 Java 17 模块化导致 Arrow / Snowflake JDBC 的 MemoryUtil 反射权限问题，并掌握通过  
   `--add-opens=java.base/java.nio=ALL-UNNAMED`  
   的方式解决反射访问限制。
6. 熟悉了从 **密钥生成 → 公钥格式转换 → Snowflake 用户绑定 → JDBC 私钥连接** 的完整流程。
7. 更清晰理解了 Snowflake 的安全策略（MFA、Key Pair、证书校验）以及它们与客户端行为之间的对应关系。
8. 理解了 Git 环境配置全流程，包括安装 Git、配置 SSH Key、添加到 GitHub 并成功实现 SSH 免密登录。
9. 熟练掌握了在 Windows 上排查 Git、SSH、PATH 环境变量等常见问题的方法。
10. 掌握了 IntelliJ IDEA 的运行配置（Run Configuration）创建方法，并理解 Application 类型与 VM Options 的作用。
11. 理解了 Java 版本兼容性（class file version）问题，学会区分 **编译 JDK 与运行 JDK 的关系**。
12. 学习了 Snowflake JDBC 与 Java Module System 的关系，理解了为什么需要使用 `--add-opens` 来解决 Apache Arrow 在 JDK17 上的反射访问限制。
13. 理解了 Snowflake MFA 对 JDBC 程序的影响，并知道如何通过 **禁用 MFA 或开启 bypass** 来允许程序正常连接。
14. 解决了 Oracle XE Listener 监听地址问题，对 **PDB / CDB 架构、用户创建位置与连接字符串** 的关系有了清晰认识。
    

## 后续计划

明天开始学习 SQL 语法对比，重点关注：

- **JOIN 语法差异**  
  - 比较 Oracle 传统外连接写法 `(+)` 与 Snowflake 标准 ANSI JOIN 的差异  
  - 理解 INNER / LEFT / RIGHT / FULL JOIN 在两个平台的执行逻辑是否一致  
  - 掌握 Snowflake 对 JOIN 条件、过滤顺序的处理特点

- **聚合函数差异**  
  - 对比 Oracle 与 Snowflake 在 `COUNT(DISTINCT)`, `LISTAGG`, `GROUP BY` 等函数上的差异  
  - 理解 Snowflake `LISTAGG` 在超长字符串场景的行为  
  - 回顾窗口函数（分析函数）如 `RANK()`, `ROW_NUMBER()` 在两个平台的相似点与差异点

- **字符串处理函数差异**  
  - 对比常见函数：`SUBSTR` vs `SUBSTRING`, `INSTR` vs `CHARINDEX`  
  - 理解两者在字符串拼接 `||` 时对 NULL 的不同处理方式  
  - 熟悉 Snowflake 的 `REGEXP_*`、`SPLIT`、`TRY_*` 系列函数
