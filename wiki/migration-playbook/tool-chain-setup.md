# 迁移工具链配置指南

## 1. 工具概览

### 1.1 必备工具清单

| 工具类别 | 工具名称 | 版本要求 | 用途 |
|---------|---------|---------|------|
| DDL 转换 | Ora2Pg | ≥ 23.0 | 表结构转换 |
| DDL 转换 | SnowConvert | ≥ 3.5 | 存储过程转换 |
| 数据导出 | Oracle Data Pump | ≥ 19c | 数据导出 |
| 数据加载 | SnowSQL | ≥ 1.2 | 数据加载到 Snowflake |
| 自研工具 | DDL Converter | ≥ 2.1 | 批量 DDL 转换 |
| 自研工具 | Data Exporter | ≥ 1.8 | 增量数据导出 |

---

## 2. Ora2Pg 配置

### 2.1 安装步骤

```bash
# Ubuntu/Debian
sudo apt-get install ora2pg

# RHEL/CentOS
sudo yum install ora2pg

# 验证安装
ora2pg --version
```

### 2.2 配置文件模板

**位置:** `/etc/ora2pg/ora2pg.conf`

```conf
# Oracle 连接配置
ORACLE_HOME     /usr/lib/oracle/19.3/client64
ORACLE_DSN      dbi:Oracle:host=oracle-server;sid=ORCL;port=1521
ORACLE_USER     migration_user
ORACLE_PWD      ${ORACLE_PASSWORD}

# 输出配置
OUTPUT          output.sql
OUTPUT_DIR      /home/migration/output
TYPE            TABLE
SCHEMA          HR

# 数据类型映射
DATA_TYPE       NUMBER(p,s):NUMBER(p,s)
DATA_TYPE       VARCHAR2:VARCHAR
DATA_TYPE       CLOB:VARCHAR(16777216)
DATA_TYPE       BLOB:BINARY(8388608)

# 性能配置
PARALLEL_TABLES 4
JOBS            8
```

---

## 3. SnowConvert 配置

### 3.1 Docker 安装

```bash
# 拉取镜像
docker pull mobilize/snowconvert:oracle-3.5.0

# 运行容器
docker run -d \
  --name snowconvert \
  -v /path/to/source:/source \
  -v /path/to/output:/output \
  mobilize/snowconvert:oracle-3.5.0
```

### 3.2 配置文件

**snowconvert-config.json**

```json
{
  "sourceType": "oracle",
  "targetType": "snowflake",
  "inputPath": "/source",
  "outputPath": "/output",
  "options": {
    "convertPLSQL": true,
    "optimizeQueries": true
  }
}
```

---

## 4. SnowSQL 配置

### 4.1 安装

```bash
# macOS
brew install snowsql

# Linux
curl -O https://sfc-repo.snowflakecomputing.com/snowsql/bootstrap/1.2/linux_x86_64/snowsql-1.2.28-linux_x86_64.bash
bash snowsql-1.2.28-linux_x86_64.bash
```

### 4.2 配置文件

**~/.snowsql/config**

```ini
[connections]
accountname = xyz12345
username = migration_user
password = ${SNOWFLAKE_PASSWORD}
dbname = MIGRATION_DB
schemaname = PUBLIC
warehousename = COMPUTE_WH

[options]
exit_on_error = True
auto_completion = True
```

---

## 5. 自研工具配置

### 5.1 DDL 转换器

**config.yaml**

```yaml
source:
  database: oracle
  connection:
    host: oracle-server
    port: 1521
    sid: ORCL
    user: migration_user
    password: ${ORACLE_PASSWORD}

target:
  database: snowflake
  account: xyz12345
  warehouse: COMPUTE_WH
```

**使用示例:**

```bash
python ddl_converter.py \
  --config config.yaml \
  --schema HR \
  --output-dir ./output/
```

---

## 6. 验证检查清单

- [ ] 所有工具安装成功
- [ ] 可以连接 Oracle 数据库
- [ ] 可以连接 Snowflake 数据库
- [ ] 配置文件环境变量已设置
- [ ] 测试转换运行成功

---

**文档版本:** v1.0
**最后更新:** 2025-01-22
