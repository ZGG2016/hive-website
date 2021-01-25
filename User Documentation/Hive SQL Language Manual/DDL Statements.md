# LanguageManual DDL

[TOC]

## 1、Hive Data Definition Language

### 1.1、Overview

> HiveQL DDL statements are documented here, including:

DDL 语句包括了：

- CREATE DATABASE/SCHEMA, TABLE, VIEW, FUNCTION, INDEX
- DROP DATABASE/SCHEMA, TABLE, VIEW, INDEX
- TRUNCATE TABLE
- ALTER DATABASE/SCHEMA, TABLE, VIEW
- MSCK REPAIR TABLE (or ALTER TABLE RECOVER PARTITIONS)
- SHOW DATABASES/SCHEMAS, TABLES, TBLPROPERTIES, VIEWS, PARTITIONS, FUNCTIONS, INDEX[ES], COLUMNS, CREATE TABLE
- DESCRIBE DATABASE/SCHEMA, table_name, view_name, materialized_view_name

> PARTITION statements are usually options of TABLE statements, except for SHOW PARTITIONS.

PARTITION 语句通常是 TABLE 语句的选项，除了 SHOW PARTITIONS。

### 1.2、Keywords, Non-reserved Keywords and Reserved Keywords

表格见原文：[https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-Keywords,Non-reservedKeywordsandReservedKeywords](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-Keywords,Non-reservedKeywordsandReservedKeywords)

> Version information. REGEXP and RLIKE are non-reserved keywords prior to Hive 2.0.0 and reserved keywords starting in Hive 2.0.0 (HIVE-11703).

版本信息：REGEXP 和 RLIKE 是 Hive 2.0.0 之前的非保留关键字，保留关键字是从 Hive 2.0.0 开始的。

> Reserved keywords are permitted as identifiers if you quote them as described in [Supporting Quoted Identifiers in Column Names](https://issues.apache.org/jira/secure/attachment/12618321/QuotedIdentifier.html) (version 0.13.0 and later, see [HIVE-6013](https://issues.apache.org/jira/browse/HIVE-6013)). Most of the keywords are reserved through [HIVE-6617](https://issues.apache.org/jira/browse/HIVE-6617) in order to reduce the ambiguity in grammar (version 1.2.0 and later). There are two ways if the user still would like to use those reserved keywords as identifiers: (1) use quoted identifiers, (2) set [hive.support.sql11.reserved.keywords](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.support.sql11.reserved.keywords)=false. (version 2.1.0 and earlier) 

如果像在 Supporting Quoted Identifiers in Column Names 中描述的那样引用保留关键字作为标识符，则是允许的。

大多数关键字是保留的，以减少语法中的歧义。

如果用户仍然希望使用那些保留的关键字作为标识符，有两种方法:

(1)使用带引号的标识符

(2)设置 `hive.support.sql11.reserved.keywords=false`(版本2.1.0及更早)

### 1.3、Create/Drop/Alter/Use Database

#### 1.3.1、Create Database

	CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
	  [COMMENT database_comment]
	  [LOCATION hdfs_path]
	  [MANAGEDLOCATION hdfs_path]
	  [WITH DBPROPERTIES (property_name=property_value, ...)];

> The uses of SCHEMA and DATABASE are interchangeable – they mean the same thing. CREATE DATABASE was added in Hive 0.6 ([HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)).  The WITH DBPROPERTIES clause was added in Hive 0.7 ([HIVE-1836](https://issues.apache.org/jira/browse/HIVE-1836)).

**SCHEMA 和 DATABASE 的使用是可互换的：它们的意思是一样的**。

在 Hive 0.6 中添加了 CREATE DATABASE。

WITH DBPROPERTIES 子句在 Hive 0.7 中添加。

> MANAGEDLOCATION was added to database in Hive 4.0.0 ([HIVE-22995](https://issues.apache.org/jira/browse/HIVE-22995)). LOCATION now refers to the default directory for external tables and MANAGEDLOCATION refers to the default directory for managed tables. Its recommended that MANAGEDLOCATION be within metastore.warehouse.dir so all managed tables have a common root where common governance policies. It can be used with metastore.warehouse.tenant.colocation to have it point to a directory outside the warehouse root directory to have a tenant based common root where quotas and other policies can be set. 

MANAGEDLOCATION 在 Hive 4.0.0 中添加到数据库。

**LOCATION 现在指的是外部表的默认目录，MANAGEDLOCATION 指的是受管表的默认目录**。

建议 MANAGEDLOCATION 位于 `metastore.warehouse.dir` 内。

因此，所有受管表都有一个共同的根，其中有共同的治理策略。可与 `metastore.warehouse.tenant.colocation` 配套使用，让它指向仓库根目录之外的目录，从而拥有一个基于公共根的租户，可以在此设置配额和其他策略。

----------------------------------------------------------

```sql
-- 创建
create database if not exists testdb
    comment 'this is a test db!'
    with dbproperties('name'='zgg','create_date'='2020-11-11');

-- 查看
hive> desc database testdb;
OK
testdb  this is a test db!      hdfs://zgg:9000/user/hive/warehouse/testdb.db   root    USER

hive> desc database extended testdb;
OK
testdb  this is a test db!      hdfs://zgg:9000/user/hive/warehouse/testdb.db   root    USER    {name=zgg, create_date=2020-11-11}
```

----------------------------------------------------------

#### 1.3.2、Drop Database

	DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];

> The uses of SCHEMA and DATABASE are interchangeable – they mean the same thing. DROP DATABASE was added in Hive 0.6 (HIVE-675). The default behavior is RESTRICT, where DROP DATABASE will fail if the database is not empty. To drop the tables in the database as well, use DROP DATABASE ... CASCADE. Support for RESTRICT and CASCADE was added in Hive 0.8 ([HIVE-2090](https://issues.apache.org/jira/browse/HIVE-2090)).

SCHEMA 和 DATABASE 的使用是可互换的：它们的意思是一样的。`DROP DATABASE` 在 Hive 0.6 添加。

默认行为是 RESTRICT，如果数据库不是空的，则 DROP DATABASE 将失败。

要同时删除数据库中的表，请使用 `DROP DATABASE ... CASCADE` 。

Hive 0.8 中增加了对 RESTRICT 和 CASCADE 的支持。

----------------------------------------------------------

```sql
hive> use testdb;
OK
-- 在其中建个表
hive> create table test(id int);
OK

hive> show tables;
OK
test

-- 删除数据库
ive> drop database testdb;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database testdb is not empty. One or more tables exist.)

-- 重新尝试删除，带有cascade
hive> drop database testdb cascade;
OK

hive> show databases;
OK
default
```

----------------------------------------------------------

#### 1.3.3、Alter Database

	ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);   -- (Note: SCHEMA added in Hive 0.14.0)
	 
	ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;   -- (Note: Hive 0.13.0 and later; SCHEMA added in Hive 0.14.0)
	  
	ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path; -- (Note: Hive 2.2.1, 2.4.0 and later)
	 
	ALTER (DATABASE|SCHEMA) database_name SET MANAGEDLOCATION hdfs_path; -- (Note: Hive 4.0.0 and later)

> The uses of SCHEMA and DATABASE are interchangeable – they mean the same thing. ALTER SCHEMA was added in Hive 0.14 ([HIVE-6601](https://issues.apache.org/jira/browse/HIVE-6601)).

SCHEMA 和 DATABASE 的使用是可互换的：它们的意思是一样的。`ALTER SCHEMA` 在 Hive 0.14 添加。

> The ALTER DATABASE ... SET LOCATION statement does not move the contents of the database's current directory to the newly specified location. It does not change the locations associated with any tables/partitions under the specified database. It only changes the default parent-directory where new tables will be added for this database. This behaviour is analogous to how changing a table-directory does not move existing partitions to a different location.

**`ALTER DATABASE ... SET LOCATION` 语句不会将数据库当前目录的内容移动到新指定的位置**。

它不会改变与指定数据库下的任何表/分区相关联的位置。

它**只更改默认的父目录，这个父目录是添加到这个数据库的新表的添加位置**。这种行为类似于更改表目录不会将现有分区移动到不同的位置。

> The ALTER DATABASE ... SET MANAGEDLOCATION statement does not move the contents of the database's managed tables directories to the newly specified location. It does not change the locations associated with any tables/partitions under the specified database. It only changes the default parent-directory where new tables will be added for this database. This behaviour is analogous to how changing a table-directory does not move existing partitions to a different location.

`ALTER DATABASE ... SET MANAGEDLOCATION` 语句不会将数据库的受管表目录的内容移动到新指定的位置。

它不会改变与指定数据库下的任何表/分区相关联的位置。

它只更改默认的父目录，这个父目录是添加到这个数据库的新表的添加位置。这种行为类似于更改表目录不会将现有分区移动到不同的位置。

> No other metadata about a database can be changed. 

关于数据库的其他元数据不能更改。

----------------------------------------------------------

```sql
-- 创建
create database if not exists testdb
    comment 'this is a test db!'
    location '/test/hiveloc'
    with dbproperties('name'='zgg','create_date'='2020-11-11');

-- 查看
hive> desc database extended testdb;
OK
testdb  this is a test db!      hdfs://zgg:9000/test/hiveloc    root    USER    {name=zgg, create_date=2020-11-11}


-- 建个表，插入两条数据
hive> create table test(id int);
hive> insert into table testdb.test values(1),(2);


-- ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path;
alter database testdb set location 'hdfs://zgg:9000/test/hivelocccc';

hive> desc database extended testdb;
OK
testdb  this is a test db!      hdfs://zgg:9000/test/hivelocccc root    USER    {name=zgg, create_date=2020-11-11}

-- 查看 hdfs 的 hiveloc/ 目录，修改后，原来建的表的文件没有移动
[root@zgg ~]# hadoop fs -text /test/hiveloc/test/000000_0
1
2
-- 是空的
[root@zgg ~]# hadoop fs -ls /test/hivelocccc

-- 再建个表，插入两条数据
hive> create table test02(id int);
hive> insert into table testdb.test02 values(1),(2);

-- 查看 hdfs 的 hivelocccc/ 目录，新表数据在新目录下，旧的目录下没有。
[root@zgg ~]# hadoop fs -text /test/hivelocccc/test02/000000_0
1
2

[root@zgg ~]# hadoop fs -ls /test/hiveloc
drwxr-xr-x   - root supergroup          0 2021-01-25 15:20 /test/hiveloc/test
```

----------------------------------------------------------

#### 1.3.4、Use Database

	USE database_name;
	USE DEFAULT;

> USE sets the current database for all subsequent HiveQL statements. To revert to the default database, use the keyword "default" instead of a database name. To check which database is currently being used: SELECT [current_database()](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-Misc.Functions) (as of [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-4144)).

USE 为所有后续 HiveQL 语句设置当前数据库。

要恢复到默认数据库，请使用关键字 default ，而不是数据库名称。

要检查哪个数据库正在被使用：`SELECT current_database()`。

> USE database_name was added in Hive 0.6 ([HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)).

`USE database_name` 在Hive 0.6 中添加。

### 1.4、Create/Drop/Truncate Table

#### 1.4.1、Create Table

	CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
	  [(col_name data_type [column_constraint_specification] [COMMENT col_comment], ... [constraint_specification])]
	  [COMMENT table_comment]
	  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
	  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
	  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
	     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
	     [STORED AS DIRECTORIES]
	  [
	   [ROW FORMAT row_format] 
	   [STORED AS file_format]
	     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
	  ]
	  [LOCATION hdfs_path]
	  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
	  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
	 
	CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
	  LIKE existing_table_or_view_name
	  [LOCATION hdfs_path];
	 
	data_type
	  : primitive_type
	  | array_type
	  | map_type
	  | struct_type
	  | union_type  -- (Note: Available in Hive 0.7.0 and later)
	 
	primitive_type
	  : TINYINT
	  | SMALLINT
	  | INT
	  | BIGINT
	  | BOOLEAN
	  | FLOAT
	  | DOUBLE
	  | DOUBLE PRECISION -- (Note: Available in Hive 2.2.0 and later)
	  | STRING
	  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
	  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
	  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
	  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
	  | DATE        -- (Note: Available in Hive 0.12.0 and later)
	  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
	  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
	 
	array_type
	  : ARRAY < data_type >
	 
	map_type
	  : MAP < primitive_type, data_type >
	 
	struct_type
	  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
	 
	union_type
	   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
	 
	row_format
	  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
	        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
	        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
	  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
	 
	file_format:
	  : SEQUENCEFILE
	  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
	  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
	  | ORC         -- (Note: Available in Hive 0.11.0 and later)
	  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
	  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
	  | JSONFILE    -- (Note: Available in Hive 4.0.0 and later)
	  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
	 
	column_constraint_specification:
	  : [ PRIMARY KEY|UNIQUE|NOT NULL|DEFAULT [default_value]|CHECK  [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]
	 
	default_value:
	  : [ LITERAL|CURRENT_USER()|CURRENT_DATE()|CURRENT_TIMESTAMP()|NULL ] 
	 
	constraint_specification:
	  : [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
	    [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
	    [, CONSTRAINT constraint_name FOREIGN KEY (col_name, ...) REFERENCES table_name(col_name, ...) DISABLE NOVALIDATE 
	    [, CONSTRAINT constraint_name UNIQUE (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
	    [, CONSTRAINT constraint_name CHECK [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]

> CREATE TABLE creates a table with the given name. An error is thrown if a table or view with the same name already exists. You can use IF NOT EXISTS to skip the error.

`CREATE TABLE` 使用给定名字创建一个表。如果已存在相同名字的表或视图，就会抛出错误。可以使用 `IF NOT EXISTS` 跳过错误。

> Table names and column names are case insensitive but SerDe and property names are case sensitive.

- 表名和列名是大小写不敏感的，但是 SerDe 和属性名字是大小写敏感的。

	- 在 Hive 0.12 及更早版本，表名和列名仅允许字母数字和下划线字符。

	- 在 Hive 0.13 及后面版本，列名可以包含任意的 Unicode 字符，但是点号和冒号会在查询时产生错误，所以在 Hive 1.2.0 中，它们被禁止了。任意在反引号中指定的列名作为字面含义对待。在反引号字符串中，使用双反引号来表示一个反引号字符。反引号还允许对表和列标识符使用保留关键字。

	- 要恢复到 0.13.0 之前的行为，并将列名限制为字母数字和下划线字符，设置配置属性`hive.support.quoted.identifiers`为 none。在此配置中，反引号包围的名字被解释为正则表达式。有关详细信息，请参见 Supporting Quoted Identifiers in Column Names。

> In Hive 0.12 and earlier, only alphanumeric and underscore characters are allowed in table and column names.

> In Hive 0.13 and later, column names can contain any [Unicode](http://en.wikipedia.org/wiki/List_of_Unicode_characters) character (see [HIVE-6013](https://issues.apache.org/jira/browse/HIVE-6013)), however, dot (.) and colon (:) yield errors on querying, so they are disallowed in Hive 1.2.0 (see[HIVE-10120](https://issues.apache.org/jira/browse/HIVE-10120)). Any column name that is specified within backticks is treated literally. Within a backtick string, use double backticks to represent a backtick character. Backtick quotation also enables the use of reserved keywords for table and column identifiers.

> To revert to pre-0.13.0 behavior and restrict column names to alphanumeric and underscore characters, set the configuration property [hive.support.quoted.identifiers](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.support.quoted.identifiers) to none. In this configuration, backticked names are interpreted as regular expressions. For details, see [Supporting Quoted Identifiers in Column Names](https://issues.apache.org/jira/secure/attachment/12618321/QuotedIdentifier.html).

> Table and column comments are string literals (single-quoted).

- 表和列的注释是字符串字面量（单引号）。

> A table created without the [EXTERNAL](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ExternalTables) clause is called a [managed table](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ManagedandExternalTables) because Hive manages its data. To find out if a table is managed or external, look for tableType in the output of [DESCRIBE EXTENDED table_name](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DescribeTable/View/Column).

- 不使用 EXTERNAL 子句创建的表称为一个受管表，因为 Hive 管理它的数据。要确定一个表是受管表还是外部表，查看 `DESCRIBE EXTENDED table_name` 的输出中的 tableType。

> The TBLPROPERTIES clause allows you to tag the table definition with your own metadata key/value pairs. Some predefined table properties also exist, such as last_modified_user and last_modified_time which are automatically added and managed by Hive. Other predefined table properties include:

- TBLPROPERTIES 子句允许你使用自己的元数据键/值对标记表定义。此外，还存在一些预定义的表属性，如 last_modified_user 和 last_modified_time，这些属性是由 Hive 自动添加和管理的。其他预定义的表属性包括:

	- TBLPROPERTIES ("comment"="table_comment")

	- TBLPROPERTIES ("hbase.table.name"="table_name") – see [HBase Integration](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration#HBaseIntegration-Usage).

	- TBLPROPERTIES ("immutable"="true") or ("immutable"="false") in release 0.13.0+ ([HIVE-6406](https://issues.apache.org/jira/browse/HIVE-6406)) – see [Inserting Data into Hive Tables from Queries](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingdataintoHiveTablesfromqueries).

	- TBLPROPERTIES ("orc.compress"="ZLIB") or ("orc.compress"="SNAPPY") or ("orc.compress"="NONE") and other ORC properties – see [ORC Files](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax).

	- TBLPROPERTIES ("transactional"="true") or ("transactional"="false") in release 0.14.0+, the default is "false" – see [Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties).

	- TBLPROPERTIES ("NO_AUTO_COMPACTION"="true") or ("NO_AUTO_COMPACTION"="false"), the default is "false" – see [Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties).

	- TBLPROPERTIES ("compactor.mapreduce.map.memory.mb"="mapper_memory") – see [Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties).

	- TBLPROPERTIES ("compactorthreshold.hive.compactor.delta.num.threshold"="threshold_num") – see [Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties).

	- TBLPROPERTIES ("compactorthreshold.hive.compactor.delta.pct.threshold"="threshold_pct") – see [Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties).

	- TBLPROPERTIES ("auto.purge"="true") or ("auto.purge"="false") in release 1.2.0+ ([HIVE-9118](https://issues.apache.org/jira/browse/HIVE-9118)) – see [LanguageManual DDL#Drop Table](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropTable), [LanguageManual DDL#Drop Partitions](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropPartitions), [LanguageManual DDL#Truncate Table](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-TruncateTable), and [Insert Overwrite](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertOverwrite).

	- TBLPROPERTIES ("EXTERNAL"="TRUE") in release 0.6.0+ ([HIVE-1329](https://issues.apache.org/jira/browse/HIVE-1329)) – Change a managed table to an external table and vice versa for "FALSE".【将受管表改为外部表，反之亦然】

		- Hive 2.4.0 版本，EXTERNAL 属性的值被解析为一个布尔值(不区分大小写的true或false)，而不是区分大小写的字符串比较。

	- TBLPROPERTIES ("external.table.purge"="true") in release 4.0.0+ ([HIVE-19981](https://issues.apache.org/jira/browse/HIVE-19981)) when set on external table would delete the data as well.

> As of Hive 2.4.0 ([HIVE-16324](https://issues.apache.org/jira/browse/HIVE-16324)) the value of the property 'EXTERNAL' is parsed as a boolean (case insensitive true or false) instead of a case sensitive string comparison.

> To specify a database for the table, either issue the [USE database_name](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-UseDatabase) statement prior to the CREATE TABLE statement (in [Hive 0.6](https://issues.apache.org/jira/browse/HIVE-675) and later) or qualify the table name with a database name ("database_name.table.name" in [Hive 0.7](https://issues.apache.org/jira/browse/HIVE-1517) and later). The keyword "default" can be used for the default database.

- 要为表指定一个数据库，要么在建表表，执行 `USE database_name`，要么使用数据库名限定表名("database_name.table.name")。关键字 default 表示 default 数据库。

> See [Alter Table](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable) below for more information about table comments, table properties, and SerDe properties.

> See [Type System](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-TypeSystem) and [Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types) for details about the primitive and complex data types.

##### 1.4.1.1、Managed and External Tables

> By default Hive creates managed tables, where files, metadata and statistics are managed by internal Hive processes. For details on the differences between managed and external table see [Managed vs. External Tables](https://cwiki.apache.org/confluence/display/Hive/Managed+vs.+External+Tables).

默认情况下，Hive 创建受管表，它的文件、元数据和统计信息都由内部 Hive 进程管理。受管表和外部表的区别见 Managed vs. External Tables。

##### 1.4.1.2、Storage Formats

> Hive supports built-in and custom-developed file formats. See [CompressedStorage](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage) for details on compressed table storage. The following are some of the formats built-in to Hive:

Hive 支持内建的和自定义开发的文件格式。关于压缩表存储见 CompressedStorage。

Hive 支持的内建格式：

Storage Format       |  Description 
---|:---
STORED AS TEXTFILE   |  Stored as plain text files. TEXTFILE is the default file format, unless the configuration parameter [hive.default.fileformat](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat) has a different setting. Use the DELIMITED clause to read delimited files. Enable escaping for the delimiter characters by using the 'ESCAPED BY' clause (such as ESCAPED BY '\') . Escaping is needed if you want to work with data that can contain these delimiter characters. A custom NULL format can also be specified using the 'NULL DEFINED AS' clause (default is '\N'). (Hive 4.0) All BINARY columns in the table are assumed to be base64 encoded.  To read the data as raw bytes: TBLPROPERTIES ("hive.serialization.decode.binary.as.base64"="false")【存储为纯文本文件。`TEXTFILE` 是默认的文件格式，除非`hive.default.fileformat`参数设置了不同的配置。使用`DELIMITED`子句读取分隔的文件。通过使用`ESCAPED BY`子句(例如`ESCAPED BY '\'`)来启用分隔符字符转义。如果要处理可能包含这些分隔符的数据，则需要进行转义。可以使用`NULL DEFINED AS`子句来指定自定义NULL格式（默认是'\N'）。(Hive 4.0) 表中的所有的二进制列都假设是base64编码的。以原始字节的形式读取数据：`TBLPROPERTIES ("hive.serialization.decode.binary.as.base64"="false")`】
STORED AS SEQUENCEFILE|	Stored as compressed Sequence File.【读取为压缩的Sequence文件】
STORED AS ORC	      | Stored as [ORC file format](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax). Supports ACID Transactions & Cost-based Optimizer (CBO). Stores column-level metadata.【存储为ORC文件格式。支持ACID事务和CBO.存储列级别的元数据。】
STORED AS PARQUET	  | Stored as [Parquet](https://cwiki.apache.org/confluence/display/Hive/Parquet) format for the Parquet columnar storage format in [Hive 0.13.0](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.13andlater) and later; Use `ROW FORMAT SERDE ... STORED AS INPUTFORMAT ... OUTPUTFORMAT` syntax ... in [Hive 0.10, 0.11, or 0.12](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.10-0.12).【存储为Parquet格式，在Hive 0.13.0及后面的版本。在之前的版本中使用`ROW FORMAT SERDE ... STORED AS INPUTFORMAT ... OUTPUTFORMAT`】
STORED AS AVRO	      | Stored as Avro format in [Hive 0.14.0](https://issues.apache.org/jira/browse/HIVE-6806) and later (see [Avro SerDe](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)).【存储为AVRO格式，在Hive 0.14.0及后面的版本。】
STORED AS RCFILE	  | Stored as [Record Columnar File](https://en.wikipedia.org/wiki/RCFile) format.【存储为记录列文件格式】
STORED AS JSONFILE	  | Stored as Json file format in Hive 4.0.0 and later.【存储为Json文件格式，在4.0.0及后面的版本，】
STORED BY	          | Stored by a non-native table format. To create or link to a non-native table, for example a table backed by [HBase](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration) or [Druid](https://cwiki.apache.org/confluence/display/Hive/Druid+Integration) or [Accumulo](https://cwiki.apache.org/confluence/display/Hive/AccumuloIntegration). See [StorageHandlers](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers) for more information on this option.【以非原生的表格式存储。为创建或链接到非原生表，例如一个由HBase、Druid或Accumulo支持的表。有关此选项的更多信息，请参阅StorageHandlers。】
INPUTFORMAT and OUTPUTFORMAT  |	in the file_format to specify the name of a corresponding InputFormat and OutputFormat class as a string literal. For example, 'org.apache.hadoop.hive.contrib.fileformat.base64.Base64TextInputFormat'. For LZO compression, the values to use are 'INPUTFORMAT "com.hadoop.mapred.DeprecatedLzoTextInputFormat" OUTPUTFORMAT "[org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat](http://org.apache.hadoop.hive.ql.io/)"' (see [LZO Compression](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO)).【在file_format中指定对应的InputFormat和OutputFormat类的名称作为字符串文本。例如`org.apache.hadoop.hive.contrib.fileformat.base64.Base64TextInputFormat`。对于LZO压缩，使用的值是`INPUTFORMAT "com.hadoop.mapred.DeprecatedLzoTextInputFormat" OUTPUTFORMAT "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat"`】


##### 1.4.1.3、Row Formats & SerDe

> You can create tables with a custom SerDe or using a native SerDe. A native SerDe is used if ROW FORMAT is not specified or ROW FORMAT DELIMITED is specified. 
Use the SERDE clause to create a table with a custom SerDe. For more information on SerDes see:

可以使用自定义 SerDe 或使用原生 SerDe 创建表。如果未指定 ROW FORMAT 或指定了 ROW FORMAT DELIMITED，则使用原生 SerDe。

SERDE 子句使用自定义 SERDE 创建表。有关 SerDes 的更多信息，请参见:

- [Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)
- [SerDe](https://cwiki.apache.org/confluence/display/Hive/SerDe)
- [HCatalog Storage Formats](https://cwiki.apache.org/confluence/display/Hive/HCatalog+StorageFormats)

> You must specify a list of columns for tables that use a native SerDe. Refer to the [Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types) part of the User Guide for the allowable column types.

必须为使用原生 SerDe 的表指定一个列的列表。有关允许的列类型，请参阅 User Guide 的 Types 部分。

> A list of columns for tables that use a custom SerDe may be specified but Hive will query the SerDe to determine the actual list of columns for this table.

使用自定义 SerDe 的表的列列表可以被指定，但是 Hive 会查询 SerDe 来确定这个表的实际列列表。

> For general information about SerDes, see [Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe) in the Developer Guide. Also see [SerDe](https://cwiki.apache.org/confluence/display/Hive/SerDe) for details about input and output processing.

有关 SerDes 的一般信息，请参见 Developer Guide 中的 Hive SerDe。有关输入和输出处理的详细信息，请参见 SerDe。

> To change a table's SerDe or SERDEPROPERTIES, use the ALTER TABLE statement as described below in [LanguageManual DDL#Add SerDe Properties](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AddSerDeProperties).

要更改表的 SerDe 或 SERDEPROPERTIES，请使用下面在 LanguageManual DDL#Add SerDe Properties 中描述的 ALTER TABLE 语句。

##### 1.4.1.4、Partitioned Tables

##### 1.4.1.5、External Tables

##### 1.4.1.6、Create Table As Select (CTAS)

##### 1.4.1.7、Create Table Like

##### 1.4.1.8、Bucketed Sorted Tables

##### 1.4.1.9、Skewed Tables

##### 1.4.1.10、Temporary Tables

##### 1.4.1.11、Transactional Tables

##### 1.4.1.12、Constraints

#### 1.4.2、Drop Table

#### 1.4.3、Truncate Table



### 1.5、Alter Table/Partition/Column


### 1.6、Create/Drop/Alter View

### 1.7、Create/Drop/Alter Materialized View

### 1.8、Create/Drop/Alter Index

### 1.9、Create/Drop Macro

### 1.10、Create/Drop/Reload Function

### 1.11、Create/Drop/Grant/Revoke Roles and Privileges

### 1.12、Show

### 1.13、Describe

### 1.14、Abort

## 2、Scheduled queries

## 3、HCatalog and WebHCat DDL