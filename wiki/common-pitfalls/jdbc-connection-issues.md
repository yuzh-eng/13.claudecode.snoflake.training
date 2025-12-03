# JDBC 连接常见问题

## 1. 连接超时

### 1.1 问题描述

```
SnowflakeSQLException: Timeout waiting for connection from pool
```

### 1.2 原因

- 连接池配置不合理（最大连接数过小）
- Snowflake Warehouse 暂停导致连接建立慢
- 网络延迟过高

### 1.3 解决方案

```java
// 增加连接超时时间
HikariConfig config = new HikariConfig();
config.setConnectionTimeout(30000);  // 30 秒
config.setMaximumPoolSize(20);       // 增加连接池大小

// 启用 Keep-Alive
config.addDataSourceProperty("client_session_keep_alive", "true");
```

---

## 2. 连接被关闭

### 2.1 问题描述

```
Connection is closed
```

### 2.2 原因

- 连接空闲时间过长被 Snowflake 关闭
- 未启用 Keep-Alive

### 2.3 解决方案

```java
// 启用 Keep-Alive
config.addDataSourceProperty("client_session_keep_alive", "true");

// 设置合理的空闲超时
config.setIdleTimeout(600000);  // 10 分钟
```

---

## 3. SSL/TLS 错误

### 3.1 问题描述

```
javax.net.ssl.SSLHandshakeException: PKIX path building failed
```

### 3.2 原因

- Java 证书库缺少 Snowflake 证书
- 网络环境有中间代理

### 3.3 解决方案

```bash
# 导入 Snowflake 证书
keytool -import -alias snowflake -file snowflake.cer -keystore $JAVA_HOME/lib/security/cacerts
```

---

## 4. 性能问题

### 4.1 批量操作慢

**问题：** 逐条插入性能差

**解决方案：**
```java
// ✅ 使用批量插入
PreparedStatement ps = conn.prepareStatement("INSERT INTO employees VALUES (?, ?)");
for (Employee emp : employees) {
    ps.setInt(1, emp.getId());
    ps.setString(2, emp.getName());
    ps.addBatch();
}
ps.executeBatch();
```

### 4.2 大数据量查询

**问题：** 查询返回数据量过大导致 OOM

**解决方案：**
```java
// 使用流式查询
Statement stmt = conn.createStatement();
stmt.setFetchSize(1000);  // 设置批量获取大小
ResultSet rs = stmt.executeQuery("SELECT * FROM large_table");
```

---

## 5. 认证问题

### 5.1 MFA 认证失败

**问题：** 启用 MFA 后 JDBC 连接失败

**解决方案：**
- 使用 Key Pair 认证替代密码认证
- 配置 SSO 集成

```java
// Key Pair 认证
Properties props = new Properties();
props.put("privateKey", privateKeyString);
props.put("user", "username");
Connection conn = DriverManager.getConnection(jdbcUrl, props);
```

---

**文档版本:** v1.0
**最后更新:** 2025-01-24
