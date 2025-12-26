# Oracle基本操作


<!--more-->

## Oracle基本操作

以具有足够权限的用户身份登录操作系统：通常需要具有管理员权限或oracle用户权限。

停止监听器（如果需要）:

打开命令提示符或终端，输入以下命令来停止监听器服务：

lsnrctl stop

通过SQL*Plus登录数据库:

执行如下命令以系统管理员身份登录SQL*Plus：

sqlplus / as sysdba

关闭数据库:

在SQL*Plus中，执行以下命令以正常方式关闭数据库（这可能需要等待所有活动会话结束）：

SHUTDOWN IMMEDIATE;

如果需要快速关闭而不等待事务完成，可以使用SHUTDOWN ABORT，但这可能会导致未提交的事务丢失。

启动数据库:

数据库关闭后，再次在SQL*Plus中执行以下命令以启动数据库：

STARTUP;

这个命令会启动数据库实例并打开数据库。

启动监听器（如果之前已停止）:

回到命令提示符或终端，输入以下命令启动监听器：

lsnrctl start

验证数据库和监听器状态:

oracle基本命令

DDL (Data Definition Language) 常用命令

创建表:

Sql

CREATE TABLE table_name (

 column1 datatype,

 column2 datatype,

 …

);

修改表结构:

Sql

ALTER TABLE table_name

ADD column_name datatype;

ALTER TABLE table_name

MODIFY column_name datatype;

ALTER TABLE table_name

DROP COLUMN column_name;

删除表:

Sql

DROP TABLE table_name CASCADE CONSTRAINTS;

创建用户:

Sql

CREATE USER username IDENTIFIED BY password DEFAULT TABLESPACE users QUOTA unlimited ON users;

授予权限:

Sql

GRANT privilege TO username;

DML (Data Manipulation Language) 常用命令

插入数据:

Sql

INSERT INTO table_name (column1, column2, …) VALUES (value1, value2, …);

更新数据:

Sql

UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;

删除数据:

Sql

DELETE FROM table_name WHERE condition;

查询数据:

Sql

SELECT column1, column2, … FROM table_name [WHERE condition] [ORDER BY column ASC|DESC];

DCL (Data Control Language) 常用命令

授予权限 (已在DDL部分提及)

回收权限:

Sql

REVOKE privilege FROM username;

其他常用命令

连接数据库:

Sql

sqlplus username/password@database

显示当前用户:

Sql

SELECT user FROM dual;

查看表空间信息:

Sql

SELECT * FROM dba_tablespaces;

启动/关闭数据库 (需要以SYSDBA身份):

启动:

Sql

STARTUP;

关闭:

Sql

SHUTDOWN IMMEDIATE;

启动/停止监听器:

启动监听器:

Cmd

lsnrctl start

停止监听器:

Cmd

lsnrctl stop

备份恢复命令

备份方案与操作步骤：

配置RMAN:

确保RMAN已正确配置并与数据库相连。这通常涉及到设置RMAN的配置参数，如备份保留策略、备份目的地等。

创建备份脚本（可选）:

为了自动化备份过程，可以创建RMAN脚本文件，包含备份命令。

执行全备:

打开命令行工具，如CMD或PowerShell，使用如下命令执行全库备份：

Cmd

rman target /

RMAN> RUN {

 BACKUP AS COMPRESSED BACKUPSET DATABASE INCLUDE CURRENT CONTROLFILE FORMAT ‘%d_DB_%U’;

 }

这里%d和%U是格式化字符串，代表日期和唯一标识符，帮助组织备份文件。

执行增量备份（可选）:

如果需要，可以执行增量备份来节省存储空间。例如，执行级别1的增量备份：

Cmd

RMAN> RUN {

 BACKUP INCREMENTAL LEVEL 1 DATABASE;

 }

归档日志备份:

为了完全恢复，也需要定期备份归档日志：

Cmd

RMAN> BACKUP ARCHIVELOG ALL;

恢复方案与操作步骤：

模拟恢复测试（推荐）:

在真正需要恢复前，先进行一次模拟恢复测试，确保恢复流程无误。

启动RMAN并连接到数据库:

在命令行中，连接到数据库：

Cmd

rman target /

关闭数据库:

如果数据库还开着，需要先关闭：

Sql

RMAN> SHUTDOWN IMMEDIATE;

启动数据库到挂载状态:

接下来，启动数据库到挂载状态，以便进行恢复操作：

Sql

RMAN> STARTUP MOUNT;

恢复数据库:

使用备份文件恢复数据库：

Sql

RMAN> RESTORE DATABASE;

应用归档日志:

应用自上次备份以来的所有归档日志，确保数据库的一致性：

Sql

RMAN> RECOVER DATABASE;

打开数据库:

恢复完成后，打开数据库：

Sql

RMAN> ALTER DATABASE OPEN RESETLOGS;

虽然Oracle 9i数据库版本较旧，但基本的备份与恢复原理与后续版本类似，主要也是依赖于RMAN（Recovery Manager）工具。下面是针对Windows环境下Oracle 9i数据库的备份与恢复方案以及操作步骤概述：

备份方案与操作步骤：

配置RMAN环境

确保RMAN已安装并配置：首先确认RMAN客户端已安装，并且环境变量如ORACLE_HOME和ORACLE_SID已正确设置。

创建RMAN脚本（可选）：为了自动化备份流程，你可以创建一个RMAN脚本来执行定期备份任务。

执行备份

启动RMAN：打开命令提示符（CMD），输入rman target /以以sysdba身份连接到数据库。

执行全备份：

Cmd

RMAN> BACKUP AS COMPRESSED BACKUPSET DATABASE PLUS ARCHIVELOG DELETE INPUT;

此命令将创建一个压缩的数据库备份，并包括所有归档日志，完成后删除已备份的归档日志文件。

配置备份策略（如增量备份）：

如果需要，可以配置增量备份策略以减少备份所需的存储空间。

Cmd

RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;

RMAN> CONFIGURE RETENTION POLICY TO REDUNDANCY 2; — 根据需要设置保留策略

RMAN> BACKUP INCREMENTAL LEVEL 1 DATABASE;

恢复方案与操作步骤：

准备恢复环境

停止所有与数据库相关的服务：包括数据库服务和监听器服务，确保没有活动连接。

启动RMAN：如同备份步骤一样，打开命令提示符并连接到数据库。

执行恢复

关闭数据库（如果尚未关闭）：

Sql

RMAN> SHUTDOWN IMMEDIATE;

启动到nomount状态：

Sql

RMAN> STARTUP NOMOUNT;

恢复控制文件（如有必要）：

如果控制文件丢失，需要先从备份恢复控制文件。

加载数据库：

Sql

RMAN> STARTUP MOUNT;

执行恢复操作：

如果是全库恢复：

Sql

RMAN> RESTORE DATABASE;

RMAN> RECOVER DATABASE;

如果是特定文件恢复，需要指定文件：

Sql

RMAN> RESTORE DATAFILE <file_number>;

RMAN> RECOVER DATAFILE <file_number>;

打开数据库：

Sql

RMAN> ALTER DATABASE OPEN RESETLOGS;

注意事项

在进行任何恢复操作之前，请确保有足够的备份，并了解所执行命令的影响。

如果数据库配置了归档模式，恢复过程中可能需要应用归档日志以保证数据的一致性

登录，搜索

查找密码修改密码


---

> 作者: <no value>  
> URL: http://localhost:1313/posts/6f1c594/  

