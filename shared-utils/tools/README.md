# 迁移工具使用手册

## 工具清单

本目录包含 Oracle → Snowflake 迁移过程中使用的自研工具。

### 1. DDL 转换器 (ddl-converter)

**功能：** 将 Oracle DDL 自动转换为 Snowflake DDL

**位置：** `ddl-converter/`

**主要特性：**
- 自动转换数据类型（`VARCHAR2` → `VARCHAR`, `NUMBER` 精度处理等）
- 批量处理整个 Schema
- 生成转换报告
- 支持自定义映射规则

**使用示例：**
```bash
cd ddl-converter

# 转换单个表
python ddl_converter.py \
  --config config.yaml \
  --table EMPLOYEES \
  --output employees.sql

# 批量转换整个 Schema
python ddl_converter.py \
  --config config.yaml \
  --schema HR \
  --output-dir ./output/

# 查看转换报告
python ddl_converter.py \
  --config config.yaml \
  --schema HR \
  --report-only
```

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
    CLOB: VARCHAR(16777216)
```

---

### 2. 数据导出器 (data-exporter)

**功能：** 从 Oracle 导出数据到文件（CSV, Parquet, JSON）

**位置：** `data-exporter/`

**主要特性：**
- 支持多种导出格式
- 增量导出（基于时间戳）
- 自动分片（大表拆分为多个文件）
- 并行导出（多线程）

**使用示例：**
```bash
cd data-exporter

# 导出为 CSV
python data_exporter.py \
  --source oracle \
  --table EMPLOYEES \
  --format csv \
  --output employees.csv \
  --batch-size 10000

# 导出为 Parquet（推荐，压缩率高）
python data_exporter.py \
  --source oracle \
  --table EMPLOYEES \
  --format parquet \
  --output employees.parquet

# 增量导出（基于时间戳列）
python data_exporter.py \
  --source oracle \
  --table ORDERS \
  --format csv \
  --output orders_incremental.csv \
  --incremental \
  --timestamp-column updated_at \
  --since "2025-01-01 00:00:00"

# 并行导出（多线程）
python data_exporter.py \
  --source oracle \
  --table LARGE_TABLE \
  --format parquet \
  --output large_table.parquet \
  --parallel 4
```

---

### 3. 数据加载器 (data-loader)

**功能：** 将数据加载到 Snowflake

**位置：** `data-loader/`

**主要特性：**
- 使用 Snowflake COPY INTO 高性能加载
- 自动创建 Stage
- 错误处理和重试
- 加载进度监控

**使用示例：**
```bash
cd data-loader

# 加载 CSV 文件
python data_loader.py \
  --target snowflake \
  --table EMPLOYEES \
  --file employees.csv \
  --format csv

# 加载 Parquet 文件
python data_loader.py \
  --target snowflake \
  --table EMPLOYEES \
  --file employees.parquet \
  --format parquet

# 批量加载目录下所有文件
python data_loader.py \
  --target snowflake \
  --table EMPLOYEES \
  --directory ./data/ \
  --format csv \
  --pattern "employees_*.csv"
```

---

### 4. 数据验证器 (data-validator)

**功能：** 验证迁移后的数据一致性

**位置：** `data-validator/`

**主要特性：**
- 行数对比
- 校验和对比（SUM, AVG, MIN, MAX）
- 抽样对比
- 生成验证报告

**使用示例：**
```bash
cd data-validator

# 验证单个表
python data_validator.py \
  --source oracle \
  --target snowflake \
  --table EMPLOYEES \
  --report validation_report.md

# 批量验证 Schema
python data_validator.py \
  --source oracle \
  --target snowflake \
  --schema HR \
  --report validation_report.md

# 仅生成报告（不执行验证）
python data_validator.py \
  --report-file validation_report.json \
  --output-format markdown
```

---

## 安装依赖

### 所有工具的通用依赖

```bash
# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt
```

**requirements.txt:**
```
cx_Oracle==8.3.0
snowflake-connector-python==3.0.0
pandas==2.0.0
pyarrow==12.0.0
pyyaml==6.0
click==8.1.0
```

---

## 环境变量配置

**创建 `.env` 文件：**

```bash
# Oracle 连接
ORACLE_HOST=oracle-server
ORACLE_PORT=1521
ORACLE_SID=ORCL
ORACLE_USER=migration_user
ORACLE_PASSWORD=your_password_here

# Snowflake 连接
SNOWFLAKE_ACCOUNT=xyz12345
SNOWFLAKE_USER=migration_user
SNOWFLAKE_PASSWORD=your_password_here
SNOWFLAKE_WAREHOUSE=COMPUTE_WH
SNOWFLAKE_DATABASE=MIGRATION_DB
SNOWFLAKE_SCHEMA=PUBLIC
```

**加载环境变量：**
```bash
# Linux/macOS
export $(cat .env | xargs)

# Windows PowerShell
Get-Content .env | ForEach-Object { $var = $_.Split('='); [Environment]::SetEnvironmentVariable($var[0], $var[1]) }
```

---

## 常见问题

### Q1: 导出大表时内存不足

**解决方案：**
- 使用 `--batch-size` 参数控制每批处理的行数
- 使用 `--parallel` 启用并行导出并减少单线程内存占用

### Q2: Snowflake 加载速度慢

**解决方案：**
- 使用 Parquet 格式（比 CSV 快 2-3 倍）
- 增加 Warehouse 大小
- 使用 `--parallel` 参数并行加载

### Q3: 数据验证发现不一致

**解决方案：**
- 检查数据类型映射是否正确（特别是 `NUMBER` 类型）
- 使用 `--sample` 参数抽样检查具体差异行
- 查看验证报告的详细日志

---

## 工具开发规范

### 代码风格

- 使用 PEP 8 风格
- 使用 Type Hints
- 添加 Docstring

### 测试

- 每个工具都有单元测试（`tests/` 目录）
- 运行测试：`pytest tests/`

### 文档

- 每个工具都有详细的 README
- 提供使用示例和常见问题解答

---

## 贡献指南

欢迎提交 Bug 报告和功能请求！

**提交流程：**
1. Fork 仓库
2. 创建功能分支：`git checkout -b feature/your-feature`
3. 提交代码：`git commit -m "feat: add your feature"`
4. 推送分支：`git push origin feature/your-feature`
5. 创建 Pull Request

---

**文档版本:** v2.1
**最后更新:** 2025-01-22
**维护团队:** 数据迁移团队
