# Oracle 数据库基本操作与备份恢复指南


Oracle 数据库基本操作与备份恢复指南

<!--more-->

## 1. 命令环境与基本概念

本文的命令分为三类：

| 命令类型            | 运行位置                               | 常见提示符或示例   |
| ------------------- | -------------------------------------- | ------------------ |
| 操作系统命令        | Windows CMD、PowerShell 或 Linux shell | `C:\>`、`PS>`、`$` |
| SQL / SQL*Plus 命令 | SQL*Plus                               | `SQL>`             |
| RMAN 命令           | RMAN 客户端                            | `RMAN>`            |

不要把不同环境的命令混在同一提示符中执行。例如，`lsnrctl`、`sqlplus` 和 `rman` 是操作系统程序；`SELECT`、`ALTER USER` 是 SQL；`BACKUP DATABASE` 是 RMAN 命令。

### 常见术语

- **实例（instance）**：Oracle 内存结构和后台进程的集合。
- **数据库（database）**：由数据文件、控制文件和联机重做日志等组成的物理文件集合。
- **监听器（listener）**：负责接受客户端连接请求并将请求转交给数据库服务；停止监听器通常不会关闭已运行的数据库实例。
- **服务名（service name）**：客户端连接使用的逻辑服务标识，不一定与 SID 相同。
- **归档模式（ARCHIVELOG）**：将已切换的重做日志保存为归档日志，是进行介质恢复和时间点恢复的重要基础。
- **SYSDBA**：高权限管理连接，不应作为应用程序或日常查询账户使用。

## 2. 登录与连接

### 2.1 操作系统认证登录

在数据库服务器本机，以属于 Oracle OSDBA 管理组的操作系统账户打开命令行：

```cmd
REM 本地 OS 认证连接；通常要求账户属于 Oracle 安装配置的 OSDBA 组。
REM ORACLE_SID 应指向要连接的实例；根据实际环境替换 ORCL。
set ORACLE_SID=ORCL

REM 启动 SQL*Plus，并以 SYSDBA 身份建立本地管理连接。
sqlplus / as sysdba
```

Windows 下常见的 Oracle OSDBA 组名为 `ORA_DBA` 或带有 Oracle Home 标识的组名，实际名称取决于安装方式。Linux/Unix 环境通常使用 `dba` 等安装时配置的组。

```bash
# 设置当前 shell 使用的实例 SID；替换为实际实例名。
export ORACLE_SID=ORCL

# 本地 OS 认证登录 SQL*Plus；当前操作系统用户必须获准使用 SYSDBA。
sqlplus / as sysdba
```

### 2.2 使用数据库账户登录

推荐让 SQL*Plus 交互式提示输入密码，避免密码出现在 shell 历史、进程列表或脚本中：

```text
-- 在操作系统命令行运行；程序会提示输入密码。
sqlplus app_user@//dbhost.example.com:1521/orclpdb
```

也可以先启动 SQL*Plus，再用 `CONNECT` 连接：

```sql
-- 在 SQL*Plus 中连接到指定服务；输入命令后按提示输入密码。
CONNECT app_user@//dbhost.example.com:1521/orclpdb
```

`//主机:端口/服务名` 是 Easy Connect 格式。默认监听端口常见为 `1521`，但应以现场配置为准。传统的 `username/password@database` 格式会把密码暴露在命令文本中，不建议用于交互或自动化。

### 2.3 连接到指定数据库服务

```text
-- 在操作系统命令行运行；使用 SQL*Plus 交互式输入密码。
sqlplus app_user@//dbhost.example.com:1521/service_name
```

```sql
-- 在 SQL*Plus 中切换到另一个服务；执行后按提示输入密码。
CONNECT app_user@//dbhost.example.com:1521/service_name
```

连接失败时，依次核对主机名解析、网络路由、防火墙端口、监听器状态、服务名和账户状态。客户端能连上监听器并不一定表示数据库服务可用。

## 3. 查看数据库状态

以下 SQL 在 SQL*Plus 中运行。`V$` 视图通常需要相应的动态性能视图访问权限；管理员可在授权范围内查询。

```sql
-- 显示当前会话连接的数据库用户名。
SELECT USER FROM dual;

-- 显示当前实例的 SID、状态和启动时间。
SELECT instance_name, status, startup_time FROM v$instance;

-- 显示数据库名称、打开模式和归档模式。
SELECT name, open_mode, log_mode FROM v$database;

-- 显示当前会话连接的服务名；结果受服务配置与权限影响。
SELECT sys_context('USERENV', 'SERVICE_NAME') AS service_name FROM dual;
```

SQL*Plus 中可使用以下客户端命令：

```sql
-- 显示当前连接的数据库用户名。
SHOW USER

-- 显示 SQL*Plus 版本与连接信息。
SHOW RELEASE
```

## 4. 搜索用户、表和其他对象

### 4.1 查找用户账户

```sql
-- 查询当前用户有权看到的账户；需要访问 DBA_USERS 视图的权限。
SELECT username, account_status, default_tablespace, profile
FROM dba_users
ORDER BY username;

-- 查询当前用户有权访问的用户列表，不需要 DBA_USERS 权限。
SELECT username
FROM all_users
ORDER BY username;

-- 查看当前连接用户。
SELECT USER FROM dual;
```

`DBA_USERS` 中可能存在认证相关的内部字段或哈希字段；不要将凭据材料导出、传播或尝试反推出明文密码。账户密码无法通过正常查询还原为明文。

### 4.2 查找表和列

```sql
-- 查找当前用户拥有的表。
SELECT table_name
FROM user_tables
ORDER BY table_name;

-- 查找当前用户可访问的表及其所属架构。
SELECT owner, table_name
FROM all_tables
WHERE table_name LIKE UPPER('%ORDER%')
ORDER BY owner, table_name;

-- 按表名查找当前用户可访问的列。
SELECT owner, table_name, column_name, data_type
FROM all_tab_columns
WHERE table_name = UPPER('EMPLOYEES')
ORDER BY owner, column_id;
```

### 4.3 查找视图、索引和序列

```sql
-- 查找当前用户拥有的视图。
SELECT view_name FROM user_views ORDER BY view_name;

-- 查找当前用户拥有的索引及其表名。
SELECT index_name, table_name, uniqueness
FROM user_indexes
ORDER BY table_name, index_name;

-- 查找当前用户拥有的序列。
SELECT sequence_name, min_value, max_value, increment_by
FROM user_sequences
ORDER BY sequence_name;
```

### 4.4 查询表数据

```sql
-- 查询指定表的前 20 条记录；将表名替换为实际表名。
SELECT *
FROM employees
WHERE ROWNUM <= 20;

-- 按条件查询指定列，并按创建时间降序排列。
SELECT employee_id, first_name, last_name, hire_date
FROM employees
WHERE department_id = 10
ORDER BY hire_date DESC;
```

Oracle 12c 及更高版本也支持 `FETCH FIRST` 分页语法；Oracle 9i 应使用 `ROWNUM` 方式。

```sql
-- Oracle 12c 及更高版本：查询前 20 条记录。
SELECT employee_id, first_name, last_name
FROM employees
ORDER BY employee_id
FETCH FIRST 20 ROWS ONLY;
```

## 5. 用户、密码与权限管理

### 5.1 查找密码与重置密码

Oracle 不会提供可查询的用户明文密码。若忘记密码，应由有权限的管理员重置；不能通过 `SELECT` 找回原密码。

优先在 SQL*Plus 中使用交互式密码修改：

```sql
-- 修改当前用户密码；SQL*Plus 会交互提示输入旧密码和新密码。
PASSWORD

-- 管理员为指定用户重置密码；SQL*Plus 会提示输入新密码。
PASSWORD app_user
```

如果使用 `ALTER USER`，新密码会出现在 SQL 文本中，可能被终端历史、审计或脚本记录。仅在受控管理场景使用占位符，不要把真实密码写进文档或脚本：

```sql
-- 将 app_user 的密码重置为实际的新密码。
-- 请勿将真实密码提交到脚本、工单、聊天记录或版本库。
ALTER USER app_user IDENTIFIED BY "<请替换为符合密码策略的新密码>";
```

若账户被锁定，先确认锁定原因、登录失败来源和安全事件，再由管理员解锁：

```sql
-- 查询账户状态；需要访问 DBA_USERS 的权限。
SELECT username, account_status, lock_date, expiry_date
FROM dba_users
WHERE username = UPPER('APP_USER');

-- 解锁指定账户；仅在确认账户安全后执行。
ALTER USER app_user ACCOUNT UNLOCK;
```

密码策略由 Profile 等配置控制。不要为了方便而取消密码复杂度、有效期或失败锁定策略；依据组织安全基线调整。

### 5.2 创建用户与配额

```sql
-- 创建应用用户并指定默认表空间；替换占位符和表空间名。
-- 不要将真实密码写入脚本；可用交互式安全部署流程设置凭据。
CREATE USER app_user
IDENTIFIED BY "<请替换为符合密码策略的新密码>"
DEFAULT TABLESPACE users
QUOTA 100M ON users;
```

原始示例使用 `QUOTA UNLIMITED ON users`。无限配额会允许用户持续占用表空间，生产环境应优先设置合理限额并监控容量：

```sql
-- 为 app_user 设置 users 表空间无限配额；仅在明确批准时使用。
ALTER USER app_user QUOTA UNLIMITED ON users;
```

### 5.3 授予与回收权限

优先授予完成工作所需的最小权限，不要为应用账户授予 `DBA` 或不必要的系统权限。

```sql
-- 允许 app_user 登录数据库；CREATE SESSION 是常见的最小登录权限。
GRANT CREATE SESSION TO app_user;

-- 允许 app_user 在自己的架构中创建表；按实际职责增减权限。
GRANT CREATE TABLE TO app_user;

-- 允许 app_user 查询 HR 架构的 employees 表。
GRANT SELECT ON hr.employees TO app_user;

-- 撤销之前授予 app_user 的表查询权限。
REVOKE SELECT ON hr.employees FROM app_user;

-- 撤销 app_user 的建表系统权限。
REVOKE CREATE TABLE FROM app_user;
```

对象权限（例如某张表的 `SELECT`）和系统权限（例如 `CREATE TABLE`）语法与作用范围不同。授予用户角色时，应检查角色包含的实际权限及其传递范围。

## 6. DDL、DML 与事务

### 6.1 DDL：创建、修改和删除对象

```sql
-- 创建示例表；列名、类型和约束应按业务模型调整。
CREATE TABLE departments_demo (
    department_id NUMBER(10) PRIMARY KEY,
    department_name VARCHAR2(100) NOT NULL,
    created_at DATE DEFAULT SYSDATE
);

-- 在表中添加可空列。
ALTER TABLE departments_demo
ADD location VARCHAR2(100);

-- 修改列的数据类型或长度；修改前确认已有数据可转换且长度足够。
ALTER TABLE departments_demo
MODIFY department_name VARCHAR2(200);

-- 删除列；此操作会移除列及其数据，执行前先备份并检查依赖。
ALTER TABLE departments_demo
DROP COLUMN location;

-- 删除表及其依赖约束；这是破坏性操作，确认对象和备份后再执行。
DROP TABLE departments_demo CASCADE CONSTRAINTS;
```

Oracle DDL 通常会隐式提交事务；不能依赖 `ROLLBACK` 撤销已完成的 `CREATE`、`ALTER` 或 `DROP`。

### 6.2 DML：插入、查询、更新和删除

```sql
-- 向表中插入一条记录；列和值的顺序必须对应。
INSERT INTO departments (department_id, department_name)
VALUES (10, 'Engineering');

-- 查询指定列；WHERE 用于筛选，ORDER BY 用于排序。
SELECT department_id, department_name
FROM departments
WHERE department_id = 10
ORDER BY department_name ASC;

-- 更新匹配条件的记录；执行前先用相同 WHERE 条件运行 SELECT 确认范围。
UPDATE departments
SET department_name = 'Platform Engineering'
WHERE department_id = 10;

-- 删除匹配条件的记录；执行前先用相同 WHERE 条件运行 SELECT 确认范围。
DELETE FROM departments
WHERE department_id = 10;

-- 提交当前事务中的 DML 更改，使其持久化。
COMMIT;

-- 撤销当前事务中尚未提交的 DML 更改。
ROLLBACK;
```

更新或删除前先确认影响行数。遗漏 `WHERE` 子句会作用于整张表：

```sql
-- 先预览将被修改的行；确认结果范围后再执行 UPDATE。
SELECT department_id, department_name
FROM departments
WHERE department_id = 10;
```

### 6.3 SQL*Plus 常用客户端命令

```sql
-- 显示当前用户。
SHOW USER

-- 设置 SQL*Plus 每页显示的行数；常用于便于阅读查询输出。
SET PAGESIZE 100

-- 设置每行显示宽度；根据终端宽度调整。
SET LINESIZE 160

-- 退出 SQL*Plus。
EXIT
```

## 7. 表空间查询

`DBA_` 开头的数据字典视图通常需要 DBA 权限。普通用户可查询 `USER_` 和 `ALL_` 范围内的信息。

```sql
-- 查看表空间名称、状态和管理方式；需要 DBA_TABLESPACES 查询权限。
SELECT tablespace_name, status, contents, extent_management
FROM dba_tablespaces
ORDER BY tablespace_name;

-- 查看数据文件路径、所属表空间及文件大小；需要 DBA_DATA_FILES 查询权限。
SELECT tablespace_name, file_name, bytes / 1024 / 1024 AS size_mb, autoextensible
FROM dba_data_files
ORDER BY tablespace_name, file_name;

-- 汇总永久表空间的可用空间；需要 DBA_DATA_FILES 与 DBA_FREE_SPACE 查询权限。
SELECT df.tablespace_name,
       ROUND(df.total_mb, 2) AS total_mb,
       ROUND(NVL(fs.free_mb, 0), 2) AS free_mb,
       ROUND(df.total_mb - NVL(fs.free_mb, 0), 2) AS used_mb
FROM (
    SELECT tablespace_name, SUM(bytes) / 1024 / 1024 AS total_mb
    FROM dba_data_files
    GROUP BY tablespace_name
) df
LEFT JOIN (
    SELECT tablespace_name, SUM(bytes) / 1024 / 1024 AS free_mb
    FROM dba_free_space
    GROUP BY tablespace_name
) fs
ON fs.tablespace_name = df.tablespace_name
ORDER BY df.tablespace_name;
```

Oracle 9i 与较新版本的数据字典字段、视图和容量统计方式可能存在差异。执行查询前用目标版本的数据字典文档核对字段。

## 8. 启停数据库与监听器

### 8.1 关闭数据库

先确认数据库实例、连接身份和维护窗口。以下命令在 SQL*Plus 中以 SYSDBA 身份执行：

```sql
-- 立即关闭数据库；不接受新连接，断开当前会话并回滚未提交事务。
-- 通常用于计划维护，仍需等待回滚等清理工作完成。
SHUTDOWN IMMEDIATE;
```

`SHUTDOWN ABORT` 会立即终止实例，不执行正常关闭清理；下次启动通常需要实例恢复。只有在正常关闭无法完成且已评估风险时才使用：

```sql
-- 强制中止实例；可能要求下次启动进行崩溃恢复。
-- 不会提交未完成事务，但不能把它视为安全的常规关闭方式。
SHUTDOWN ABORT;
```

`SHUTDOWN TRANSACTIONAL` 会等待活动事务完成后关闭；`SHUTDOWN NORMAL` 会等待用户主动断开，可能长时间无法完成。按业务要求选择关闭模式。

### 8.2 启动数据库

```sql
-- 启动实例并打开数据库；仅适用于数据库已关闭且控制文件可用的场景。
STARTUP;
```

数据库常见启动阶段：

```sql
-- 仅启动实例，不挂载数据库；常用于恢复控制文件等操作。
STARTUP NOMOUNT;

-- 启动实例并挂载数据库，但不向普通用户开放。
STARTUP MOUNT;

-- 将已挂载的数据库打开；执行前应确认恢复状态和控制文件一致性。
ALTER DATABASE OPEN;
```

### 8.3 启停监听器

以下命令在操作系统命令行执行，而不是在 SQL*Plus 提示符中执行：

```cmd
REM 停止当前 ORACLE_HOME 对应的监听器；通常不会关闭数据库本身。
lsnrctl stop

REM 启动监听器。
lsnrctl start

REM 查看监听器状态、监听地址和已注册服务。
lsnrctl status
```

监听器名称不是默认值时，可在命令后指定监听器名称，例如 `lsnrctl status LISTENER_NAME`。停止监听器会阻止新的远程连接，但通常不会终止已经建立的数据库会话。

### 8.4 Windows 服务说明

Windows 上数据库实例常由 `OracleService<SID>` 服务承载，监听器也可能作为单独服务运行。用 Services 管理控制台或服务命令管理时，必须确认 Oracle Home、SID 和服务名：

```cmd
REM 查询名称中包含 Oracle 的服务；根据实际 SID 和安装目录确认目标。
sc query state= all | findstr /I "Oracle"
```

不要把停止监听器、停止数据库实例服务和正常执行 `SHUTDOWN IMMEDIATE` 混为一谈。生产库优先通过 SQL*Plus/RMAN 正常关闭，再按维护流程管理操作系统服务。

## 9. RMAN 备份

### 9.1 备份前检查

1. 确认 Oracle 版本、数据库名称、DBID、归档模式、RMAN 连接方式和备份目录。
2. 确认目标介质容量、权限、保留周期及异地副本策略。
3. 确认备份文件不写入空间不足的系统盘。
4. 首次配置后执行备份校验，并定期在独立环境中测试恢复。
5. 不要使用 `DELETE INPUT`，除非确认归档日志备份成功、保留策略满足恢复目标且日志删除符合运维制度。

在服务器操作系统命令行启动 RMAN，并通过本地 OS 认证连接目标数据库：

```cmd
REM 启动 RMAN 客户端并连接本地目标数据库；要求当前操作系统账户获准使用 SYSDBA。
rman target /
```

RMAN 连接后会显示 `RMAN>` 提示符。RMAN 命令应在该提示符中输入，不要在 CMD 或 SQL*Plus 提示符中执行。

### 9.2 查看和设置备份配置

```text
# 在 RMAN> 提示符中显示当前 RMAN 配置。
SHOW ALL;

# 开启控制文件自动备份；执行前确认目标版本支持并已规划存储空间。
CONFIGURE CONTROLFILE AUTOBACKUP ON;

# 将保留策略设置为保留最近两个冗余备份；按恢复目标和存储容量调整。
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
```

保留策略决定 RMAN 识别哪些备份为过时，不代表自动删除所有文件。删除过时备份前先检查清单和恢复需求。

### 9.3 数据库全备与归档日志

现代 Oracle 版本中可用 `BACKUP DATABASE PLUS ARCHIVELOG` 生成数据库及归档日志备份。使用显式目录格式时，先确认目录存在、数据库服务账户有写入权限且空间足够：

```text
# 在 RMAN> 提示符中备份数据库及归档日志。
# FORMAT 中 %d 为数据库名，%T 为日期，%U 为唯一备份片标识。
BACKUP AS COMPRESSED BACKUPSET
  DATABASE
  PLUS ARCHIVELOG
  FORMAT 'D:\OracleBackup\%d_%T_%U.bkp';
```

压缩备份会消耗额外 CPU；确认目标版本和许可支持后再使用。`FORMAT` 中的 `%d`、`%T` 和 `%U` 是 RMAN 格式替换变量，不是 Windows 环境变量。

也可以将归档日志备份作为单独任务：

```text
# 备份当前可用的所有归档日志；备份频率应满足数据恢复点目标。
BACKUP ARCHIVELOG ALL
  FORMAT 'D:\OracleBackup\arch_%d_%T_%U.bkp';
```

备份完成后检查 RMAN 输出、作业状态、备份片清单和文件实际存在性：

```text
# 列出 RMAN 记录的备份摘要，检查完成时间、类型和状态。
LIST BACKUP SUMMARY;

# 校验 RMAN 可读取备份集；校验会产生 I/O 负载。
RESTORE DATABASE VALIDATE;
```

### 9.4 增量备份

增量备份以 0 级备份为基线。只有 1 级备份而没有可用的 0 级基线，不构成完整备份方案。

```text
# 创建 0 级增量备份，作为后续 1 级增量的基线。
BACKUP INCREMENTAL LEVEL 0 DATABASE
  FORMAT 'D:\OracleBackup\%d_L0_%T_%U.bkp';

# 创建 1 级增量备份，只备份自适用基线以来发生变化的数据块。
BACKUP INCREMENTAL LEVEL 1 DATABASE
  FORMAT 'D:\OracleBackup\%d_L1_%T_%U.bkp';
```

是否采用差异增量或累积增量、运行频率和保留周期应结合恢复时间目标（RTO）、恢复点目标（RPO）和版本支持设计。

### 9.5 RMAN 脚本文件

可将 RMAN 命令保存为文本脚本，通过 RMAN 客户端运行。不要在脚本中写入明文数据库密码：

```text
# run_backup.rman：数据库全备、归档日志和控制文件备份示例。
RUN {
  BACKUP AS COMPRESSED BACKUPSET
    DATABASE
    PLUS ARCHIVELOG
    FORMAT 'D:\OracleBackup\%d_%T_%U.bkp';
}
```

```cmd
REM 启动 RMAN 并执行脚本文件；目标连接采用本机 OS 认证。
rman target / @D:\OracleScripts\run_backup.rman
```

## 10. RMAN 恢复

> **恢复操作具有破坏性。**以下步骤是通用流程示意，不是适用于所有故障的固定脚本。恢复前确认数据库版本、DBID、控制文件、数据文件位置、备份可用性、归档日志范围及恢复目标；优先在隔离环境演练。

### 10.1 常规恢复流程

只有在确认数据库需要整库恢复、具备可用备份且已停止业务连接后，才按恢复方案操作：

```text
# 在 RMAN> 提示符中正常关闭数据库；连接须具有 SYSDBA 管理权限。
SHUTDOWN IMMEDIATE;

# 启动实例并挂载数据库，为整库恢复准备控制文件和实例状态。
STARTUP MOUNT;

# 从 RMAN 可识别的备份恢复数据文件。
RESTORE DATABASE;

# 应用可用的归档日志和重做信息，使数据文件恢复到一致状态。
RECOVER DATABASE;
```

恢复后如何打开数据库取决于恢复是否完整、是否使用备份控制文件以及恢复到哪个时间点。完成完整介质恢复且控制文件/日志状态允许时，通常尝试：

```text
# 完整恢复后尝试正常打开数据库；若 RMAN/Oracle 要求 RESETLOGS，先确认恢复路径。
ALTER DATABASE OPEN;
```

`OPEN RESETLOGS` 会创建新的数据库 incarnation。只有 Oracle 在不完全恢复、使用备份控制文件等场景要求时才执行，并应先记录当前恢复信息、保留备份并确认后续备份策略：

```text
# 仅在恢复流程明确要求时执行；这会重置日志序列并开始新的 incarnation。
ALTER DATABASE OPEN RESETLOGS;
```

不要把 `ALTER DATABASE OPEN RESETLOGS` 作为每次恢复后的固定步骤。

### 10.2 控制文件丢失

控制文件丢失时，需使用 RMAN 自动备份或其他可用副本恢复控制文件。实际流程取决于是否使用恢复目录、是否知道 DBID、自动备份位置和数据库版本；应按 Oracle 对应版本的恢复文档制定操作步骤。恢复控制文件后通常需要挂载数据库，再恢复数据库和归档日志。

```text
# 仅作为流程提示：恢复控制文件的 RMAN 命令必须按现场自动备份位置和 DBID 确定。
# 不要直接执行未经环境核对的示例命令。
```

### 10.3 单个数据文件恢复

单文件恢复需要确定数据文件编号、文件路径、表空间状态及数据库模式。先查询和确认目标，再按 Oracle 版本文档执行：

```text
# RMAN 命令模板：将 <文件编号> 替换为已核实的数据文件编号。
RESTORE DATAFILE <文件编号>;

# RMAN 命令模板：对已恢复的数据文件应用所需归档日志。
RECOVER DATAFILE <文件编号>;
```

不要将恢复数据文件与删除文件、重建文件或表空间恢复混为一谈。若需要恢复到特定时间点，应设计数据库不完全恢复方案，并明确影响范围。

### 10.4 恢复验证

```text
# 校验备份是否可用于数据库恢复；会读取备份介质并产生 I/O。
RESTORE DATABASE VALIDATE;

# 查看备份摘要，确认 RMAN 仓库中存在预期备份。
LIST BACKUP SUMMARY;
```

真正的恢复能力只能通过隔离环境中的完整恢复演练验证。至少记录恢复耗时、恢复点、缺失的归档日志、应用验证结果和回退方式。

## 11. Oracle 9i 注意事项

Oracle 9i 已停止常规支持多年，存在严重安全、兼容性和工具可用性风险。若必须维护，应隔离网络、限制账户与来源、保留离线备份，并规划升级迁移。

- Oracle 9i 的 RMAN 功能和语法与现代版本不同。
- `BACKUP AS COMPRESSED BACKUPSET` 是后续版本能力，不应直接用于 Oracle 9i。
- RMAN 的增量备份、自动控制文件备份、格式替换变量和恢复命令应以具体 9i 版本及补丁集文档为准。
- 本文的现代 RMAN 示例不能直接视为 Oracle 9i 脚本。
- Oracle 9i 备份必须结合归档模式、控制文件备份、归档日志保留、操作系统/平台和数据文件布局设计。
- 执行恢复前，应使用同版本、同平台约束的隔离环境验证流程，不能仅凭“原理类似”推断命令兼容。

在已核对 Oracle 9i 具体版本和 RMAN 语法、并完成测试的前提下，旧环境常见的非压缩备份思路如下。这里不使用压缩备份集，也不删除已备份归档日志：

```text
# 在 RMAN> 提示符中执行；先确认目标数据库连接和备份目录配置。
# 备份数据库以及归档日志；按具体 9i 版本核对 PLUS ARCHIVELOG 与 FORMAT 支持。
BACKUP DATABASE PLUS ARCHIVELOG;

# 备份当前可用的归档日志；根据容量和恢复点目标安排频率。
BACKUP ARCHIVELOG ALL;

# 创建 0 级增量备份基线；确认目标 9i 补丁集支持相应增量语法。
BACKUP INCREMENTAL LEVEL 0 DATABASE;

# 创建 1 级增量备份；必须保留对应可用的 0 级基线与所需归档日志。
BACKUP INCREMENTAL LEVEL 1 DATABASE;
```

若具体 Oracle 9i 版本支持并已测试控制文件自动备份和保留策略，可按该版本文档配置。不能仅因现代版本命令可用就假设 9i 一定支持相同语法或行为。

## 12. 常见操作检查清单

### 登录前

- 确认主机、端口、SID、服务名和容器/PDB。
- 确认连接账户和所需权限；日常查询不使用 SYSDBA。
- 使用交互式密码输入，避免在命令行、脚本和文档中暴露密码。

### 修改用户或数据前

- 先查询目标账户、对象和影响范围。
- DML 先用相同条件执行 `SELECT` 预览，再执行 `UPDATE`/`DELETE`。
- DDL 会隐式提交，确认对象定义和依赖后再变更。
- 密码重置前确认账户所有者、服务依赖、应用连接池和密码轮换安排。

### 关闭、备份或恢复前

- 确认数据库版本、实例身份、当前状态和维护授权。
- 确认备份文件可读取、空间充足、归档日志链完整。
- 区分关闭监听器与关闭数据库实例。
- 仅在恢复路径要求时使用 `OPEN RESETLOGS`。
- 恢复后检查数据库打开模式、应用连接、关键对象和业务数据。

## 参考命令速查

以下命令均已在正文标明运行环境；这里仅列出名称，执行时请返回相应章节查看注释和前置条件。

| 目的                 | 命令                                   |
| -------------------- | -------------------------------------- |
| 本机管理员登录       | `sqlplus / as sysdba`                  |
| 启动实例并打开数据库 | `STARTUP;`                             |
| 计划关闭数据库       | `SHUTDOWN IMMEDIATE;`                  |
| 检查监听器           | `lsnrctl status`                       |
| 连接 RMAN            | `rman target /`                        |
| 查询当前用户         | `SELECT USER FROM dual;`               |
| 交互式修改密码       | `PASSWORD`                             |
| 查看表空间           | `SELECT ... FROM dba_tablespaces;`     |
| 备份数据库及归档日志 | `BACKUP DATABASE PLUS ARCHIVELOG ...;` |



---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/6f1c594/  

