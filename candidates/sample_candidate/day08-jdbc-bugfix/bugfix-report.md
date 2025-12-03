# JDBC 连接问题修复报告

## Bug 信息
- Bug ID: JDBC-2025-008
- 发现日期: 2025-01-24
- 修复人: 张三
- 严重程度: P0（生产环境连接失败）

---

## 1. 问题描述

**现象：** 迁移到 Snowflake 后，Java 应用启动时报连接错误

```
net.snowflake.client.jdbc.SnowflakeSQLException:
  JDBC driver encountered communication error.
  Message: Exception encountered for HTTP request:
  Timeout waiting for connection from pool
```

---

## 2. 根本原因

Oracle JDBC 和 Snowflake JDBC 的连接池配置参数不兼容：

**Oracle 配置（旧）：**
```properties
jdbc.url=jdbc:oracle:thin:@oracle-server:1521:ORCL
jdbc.maxPoolSize=50
jdbc.minPoolSize=10
jdbc.maxIdleTime=1800  # 30分钟
```

**Snowflake 配置（错误）：**
```properties
jdbc.url=jdbc:snowflake://xyz12345.snowflakecomputing.com
jdbc.maxPoolSize=50      # ❌ Snowflake 默认最大连接数较小
jdbc.minPoolSize=10
jdbc.maxIdleTime=1800    # ❌ 参数名不兼容
```

---

## 3. 解决方案

**正确的 Snowflake JDBC 配置：**
```properties
jdbc.url=jdbc:snowflake://xyz12345.snowflakecomputing.com
jdbc.user=MIGRATION_USER
jdbc.password=${SNOWFLAKE_PASSWORD}
jdbc.db=MIGRATION_DB
jdbc.schema=PUBLIC
jdbc.warehouse=COMPUTE_WH

# 连接池配置
jdbc.maxPoolSize=20              # 降低最大连接数
jdbc.minPoolSize=5
jdbc.maxLifetime=600000          # 10分钟（毫秒）
jdbc.connectionTimeout=30000     # 30秒超时

# Snowflake 特定参数
jdbc.client_session_keep_alive=true
jdbc.network_timeout=300000      # 5分钟网络超时
```

**代码修改：**
```java
// 修改前
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:snowflake://xyz12345.snowflakecomputing.com");
config.setMaximumPoolSize(50);

// 修改后
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:snowflake://xyz12345.snowflakecomputing.com");
config.setMaximumPoolSize(20);  // ✅ 降低连接数
config.setConnectionTimeout(30000);
config.setIdleTimeout(600000);
config.setMaxLifetime(1800000);
config.addDataSourceProperty("client_session_keep_alive", "true");  // ✅ 保持会话
```

---

## 4. 验证结果

**测试通过：** ✅ 应用成功启动，连接池稳定运行

---

**PR**: #1240
**审核人**: 李导师
