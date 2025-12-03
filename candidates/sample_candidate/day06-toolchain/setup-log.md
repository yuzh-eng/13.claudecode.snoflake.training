# 迁移工具链配置日志

## 学员信息
- 姓名：张三
- 日期：2025-01-22
- 任务：Day 06 - 迁移工具链配置

---

## 1. Ora2Pg 安装与配置

### 1.1 安装过程

**系统环境：**
- 操作系统：Ubuntu 22.04 LTS
- Perl 版本：5.34.0
- Oracle Client：19.3

**安装步骤：**
```bash
# 1. 安装依赖
sudo apt-get update
sudo apt-get install perl libdbi-perl libdbd-pg-perl

# 2. 下载 Ora2Pg
wget https://github.com/darold/ora2pg/archive/v23.2.tar.gz
tar -xzf v23.2.tar.gz
cd ora2pg-23.2

# 3. 编译安装
perl Makefile.PL
make
sudo make install

# 4. 验证安装
ora2pg --version
# 输出：Ora2Pg v23.2
```

**遇到的问题：**
- **问题1：** 缺少 `DBD::Oracle` 模块
  - **解决方案：** `sudo cpan DBD::Oracle`
  - **耗时：** 20 分钟

### 1.2 配置文件

**配置文件位置：** `/etc/ora2pg/ora2pg.conf`

```conf
# Oracle 数据库连接
ORACLE_HOME     /usr/lib/oracle/19.3/client64
ORACLE_DSN      dbi:Oracle:host=oracle-server;sid=ORCL;port=1521
ORACLE_USER     migration_user
ORACLE_PWD      SecurePassword123

# Snowflake 输出配置
OUTPUT          /home/migration/output
OUTPUT_DIR      /home/migration/output
TYPE            TABLE
SCHEMA          HR

# 数据类型映射
DATA_TYPE       NUMBER(p,s):NUMBER(p,s)
DATA_TYPE       VARCHAR2:VARCHAR
DATA_TYPE       CLOB:VARCHAR(16777216)

# 性能优化
PARALLEL_TABLES 4
JOBS            8
```

### 1.3 测试运行

```bash
# 导出表结构
ora2pg -c /etc/ora2pg/ora2pg.conf -t TABLE -o tables.sql

# 导出数据
ora2pg -c /etc/ora2pg/ora2pg.conf -t COPY -o data.sql

# 检查生成的文件
ls -lh /home/migration/output/
# tables.sql - 356 KB
# data.sql   - 12.5 MB
```

---

## 2. SnowConvert 配置

### 2.1 工具获取

**版本：** SnowConvert for Oracle v3.5.0
**安装方式：** Docker 容器

```bash
# 拉取 Docker 镜像
docker pull mobilize/snowconvert:oracle-3.5.0

# 运行容器
docker run -it --name snowconvert \
  -v /home/migration/source:/source \
  -v /home/migration/output:/output \
  mobilize/snowconvert:oracle-3.5.0 bash
```

### 2.2 转换配置

**配置文件：** `snowconvert-config.json`

```json
{
  "sourceType": "oracle",
  "targetType": "snowflake",
  "inputPath": "/source",
  "outputPath": "/output",
  "options": {
    "convertPLSQL": true,
    "optimizeQueries": true,
    "generateComments": true,
    "preserveFormatting": false
  },
  "typeMapping": {
    "NUMBER": "NUMBER",
    "VARCHAR2": "VARCHAR",
    "CLOB": "VARCHAR",
    "BLOB": "BINARY"
  }
}
```

### 2.3 转换示例

```bash
# 转换单个 SQL 文件
docker exec snowconvert snowconvert \
  --config /source/snowconvert-config.json \
  --input /source/procedure.sql \
  --output /output/procedure_converted.sql

# 批量转换
docker exec snowconvert snowconvert \
  --config /source/snowconvert-config.json \
  --input /source/*.sql \
  --output /output/
```

**转换结果：**
- 源文件：25 个 PL/SQL 存储过程
- 转换成功：22 个（88%）
- 需要手动修复：3 个（12%）

---

## 3. 自研工具配置

### 3.1 DDL 转换工具

**工具名称：** `oracle-to-snowflake-ddl-converter`
**位置：** `shared-utils/tools/ddl-converter/`

**配置文件：** `config.yaml`

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
  connection:
    account: xyz12345
    user: migration_user
    password: ${SNOWFLAKE_PASSWORD}
    warehouse: COMPUTE_WH
    database: MIGRATION_DB
    schema: PUBLIC

conversion_rules:
  preserve_case: false
  add_comments: true
  include_constraints: true
  type_mappings:
    NUMBER: NUMBER
    VARCHAR2: VARCHAR
    DATE: DATE
    TIMESTAMP: TIMESTAMP_NTZ
    CLOB: VARCHAR(16777216)
```

**运行示例：**
```bash
cd shared-utils/tools/ddl-converter

# 转换单个表
python ddl_converter.py \
  --config config.yaml \
  --table EMPLOYEES \
  --output employees.sql

# 批量转换
python ddl_converter.py \
  --config config.yaml \
  --schema HR \
  --output-dir ./output/
```

### 3.2 数据导出工具

**工具名称：** `oracle-data-exporter`
**位置：** `shared-utils/tools/data-exporter/`

**使用示例：**
```bash
# 导出为 CSV（用于 COPY INTO）
python data_exporter.py \
  --source oracle \
  --table EMPLOYEES \
  --format csv \
  --output employees.csv \
  --batch-size 10000

# 导出为 Parquet（更高效）
python data_exporter.py \
  --source oracle \
  --table EMPLOYEES \
  --format parquet \
  --output employees.parquet
```

---

## 4. 工具对比评测

### 4.1 DDL 转换对比

| 工具 | 转换准确率 | 速度 | 易用性 | 推荐场景 |
|------|-----------|------|--------|---------|
| Ora2Pg | 85% | ⭐⭐⭐ | ⭐⭐⭐ | 简单表结构 |
| SnowConvert | 92% | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 复杂存储过程 |
| 自研工具 | 88% | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 批量迁移 |

### 4.2 数据导出对比

| 工具 | 性能 | 数据完整性 | 大数据支持 | 推荐场景 |
|------|------|-----------|-----------|---------|
| Ora2Pg | ⭐⭐⭐ | 95% | 中等 | 中小型表 |
| Oracle Data Pump | ⭐⭐⭐⭐⭐ | 100% | 优秀 | 大型表 |
| 自研工具 | ⭐⭐⭐⭐ | 98% | 优秀 | 增量导出 |

### 4.3 综合评价

**最佳实践组合：**
1. **DDL 转换：** SnowConvert（复杂逻辑） + 自研工具（批量处理）
2. **数据导出：** Oracle Data Pump（全量） + 自研工具（增量）
3. **代码转换：** SnowConvert（PL/SQL） + 人工 Review

---

## 5. 遇到的问题和解决方案

### 问题 1：Ora2Pg 内存溢出

**现象：**
```
Out of memory during large table conversion!
```

**原因：** 默认一次性加载整张表到内存

**解决方案：**
```conf
# 在 ora2pg.conf 中添加
DATA_LIMIT  100000  # 每次最多处理 10 万行
PARALLEL_TABLES  2  # 降低并行度
```

### 问题 2：SnowConvert 许可证限制

**现象：** 转换文件数量超过限制

**解决方案：** 联系供应商申请临时许可证扩展

### 问题 3：自研工具字符编码错误

**现象：** 中文字符显示为乱码

**解决方案：**
```python
# 在导出脚本中指定编码
with open('output.csv', 'w', encoding='utf-8') as f:
    writer = csv.writer(f)
    # ...
```

---

## 6. 配置检查清单

- [x] Ora2Pg 安装并验证
- [x] SnowConvert Docker 环境配置
- [x] 自研 DDL 转换工具配置
- [x] 自研数据导出工具配置
- [x] 所有工具连接测试通过
- [x] 性能测试完成
- [x] 编写工具使用文档

---

**配置完成日期：** 2025-01-22
**验证人：** 李导师
