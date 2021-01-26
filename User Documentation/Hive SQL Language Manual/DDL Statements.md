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

支持的 Row Format 的表格见原文：[https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RowFormats&SerDe](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-RowFormats&SerDe)

##### 1.4.1.4、Partitioned Tables

> Partitioned tables can be created using the PARTITIONED BY clause. A table can have one or more partition columns and a separate data directory is created for each distinct value combination in the partition columns. Further, tables or partitions can be bucketed using CLUSTERED BY columns, and data can be sorted within that bucket via SORT BY columns. This can improve performance on certain kinds of queries.

使用 PARTITIONED BY 子句创建分区表。一个表可以有一个或多个分区列，并且为分区列中的每个不同的值组合创建一个单独的数据目录【一个分区一个数据目录】。

此外，可以使用 CLUSTERED BY columns 对表或分区进行分桶，并且可以通过 SORT BY columns 在桶内对数据进行排序。

这可以提高某些查询的性能。

> If, when creating a partitioned table, you get this error: "FAILED: Error in semantic analysis: Column repeated in partitioning columns," it means you are trying to include the partitioned column in the data of the table itself. You probably really do have the column defined. However, the partition you create makes a pseudocolumn on which you can query, so you must rename your table column to something else (that users should not query on!).

如果在创建分区表时，出现这样的错误：`FAILED: error in semantic analysis: Column repeated in partitioning columns`，这意味着你试图将在表本身的数据中的列当做分区列。

你可能确实定义了列，但是，你创建的分区会生成一个可以查询的伪列，因此必须将表列重命名为其他东西(用户不应该查询的东西!)。

> For example, suppose your original unpartitioned table had three columns: id, date, and name.

例如，假设原始的未分区表有三列:id、date 和 name。

	id     int,
	date   date,
	name   varchar

> Now you want to partition on date. Your Hive definition could use "dtDontQuery" as a column name so that "date" can be used for partitioning (and querying).

现在你想在 date 列上分区。你的 Hive 定义可以使用 dtDontQuery 作为列名，这样 date 就可以用来分区了（查询）。

```sql
create table table_name (
  id                int,
  dtDontQuery       string,
  name              string
)
partitioned by (date string)
```

> Now your users will still query on "where date = '...'" but the second column dtDontQuery will hold the original values.

现在用户仍在 `where date = '...'` 上查询，但是第二列 dtDontQuery 将保持原有的值。

> Here's an example statement to create a partitioned table:

下面是创建分区表的示例：

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 STORED AS SEQUENCEFILE;
```

> The statement above creates the page_view table with viewTime, userid, page_url, referrer_url, and ip columns (including comments). The table is also partitioned and data is stored in sequence files. The data format in the files is assumed to be field-delimited by ctrl-A and row-delimited by newline.

上面的语句创建表 page_view，它包含了 viewTime、userid、page_url、referrer_url、ip（包含注释）列。表被分区，数据存入 sequence files 中。文件中的数据格式假设是 ctrl-A 分隔字段，换行符分隔行。

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '\001'
STORED AS SEQUENCEFILE;
```

> The above statement lets you create the same table as the previous table.

上面的语句让你创建和上面相同的表。

> In the previous examples the data is stored in <hive.metastore.warehouse.dir>/page_view. Specify a value for the key [hive.metastore.warehouse.dir](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.metastore.warehouse.dir) in the Hive config file hive-site.xml.

在上面的示例中，数据存储在 `<hive.metastore.warehouse.dir>/page_view` 中。在 `hive-site.xml` 中为 `hive.metastore.warehouse.dir` 指定一个值。

##### 1.4.1.5、External Tables

> The EXTERNAL keyword lets you create a table and provide a LOCATION so that Hive does not use a default location for this table. This comes in handy if you already have data generated. When dropping an EXTERNAL table, data in the table is NOT deleted from the file system. Starting Hive 4.0.0 ( HIVE-19981 - Managed tables converted to external tables by the HiveStrictManagedMigration utility should be set to delete data when the table is dropped RESOLVED ) setting table property external.table.purge=true, will also delete the data.

EXTERNAL 关键字允许创建一个表，并提供一个 LOCATION，这样 Hive 就不会为该表使用默认位置。

如果已经生成了数据，这就很方便了。删除 EXTERNAL 表时，表中的数据不会从文件系统中删除。Hive 4.0.0 设置表属性 `external.table.purge=true` 也将删除数据。

> An EXTERNAL table points to any HDFS location for its storage, rather than being stored in a folder specified by the configuration property hive.metastore.warehouse.dir.

一个 EXTERNAL 表指向它的存储的任意 HDFS 位置，而不是 `hive.metastore.warehouse.dir` 指定的文件夹下。

```sql
CREATE EXTERNAL TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User',
     country STRING COMMENT 'country of origination')
 COMMENT 'This is the staging page view table'
 ROW FORMAT DELIMITED FIELDS TERMINATED BY '\054'
 STORED AS TEXTFILE
 LOCATION '<hdfs_location>';
```

> You can use the above statement to create a page_view table which points to any HDFS location for its storage. But you still have to make sure that the data is delimited as specified in the CREATE statement above.

你可以使用上述语句创建 page_view 表，它指向任意的 HDFS 位置。但是，你仍然要确保数据按照上面的 `CREATE statement` 语句指定的方式划分。

> For another example of creating an external table, see [Loading Data](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-LoadingData) in the Tutorial.

另一个创建外部表的示例见 Tutorial 中的 Loading Data。

##### 1.4.1.6、Create Table As Select (CTAS)

> Tables can also be created and populated by the results of a query in one create-table-as-select (CTAS) statement. The table created by CTAS is atomic, meaning that the table is not seen by other users until all the query results are populated. So other users will either see the table with the complete results of the query or will not see the table at all.

可以在 `create-table-as-select` (CTAS)语句中，创建表，并使用一个查询结果填充。

CTAS 创建的表是原子的，这意味着在填充所有的查询结果之前，其他用户不会看到该表。因此，其他用户要么看到包含完整查询结果的表，要么根本看不到该表。

> There are two parts in CTAS, the SELECT part can be any [SELECT statement](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select) supported by HiveQL. The CREATE part of the CTAS takes the resulting schema from the SELECT part and creates the target table with other table properties such as the SerDe and storage format.

在 CTAS 中有两个部分，SELECT 部分可以是 HiveQL 支持的任何 SELECT 语句。CREATE 部分获取从 SELECT 部分产生的结果模式，并使用其他表属性(如SerDe和存储格式)创建目标表。

> Starting with Hive 3.2.0, CTAS statements can define a partitioning specification for the target table ([HIVE-20241](https://issues.apache.org/jira/browse/HIVE-20241)).

从 Hive 3.2.0 开始，CTAS 语句可以为目标表定义分区规范。

> CTAS has these restrictions:
> The target table cannot be an external table.
> The target table cannot be a list bucketing table.

CTAS 有以下限制:

- 目标表不能是外部表
- 目标表不能是一列分桶表

```sql
CREATE TABLE new_key_value_store
   ROW FORMAT SERDE "org.apache.hadoop.hive.serde2.columnar.ColumnarSerDe"
   STORED AS RCFile
   AS
SELECT (key % 1024) new_key, concat(key, value) key_value_pair
FROM key_value_store
SORT BY new_key, key_value_pair;
```

> The above CTAS statement creates the target table new_key_value_store with the schema (new_key DOUBLE, key_value_pair STRING) derived from the results of the SELECT statement. If the SELECT statement does not specify column aliases, the column names will be automatically assigned to `_col0`, `_col1`, and `_col2` etc. In addition, the new target table is created using a specific SerDe and a storage format independent of the source tables in the SELECT statement.

上面的 CTAS 语句使用 SELECT 语句产生的结果的模式(new_key DOUBLE, key_value_pair STRING)创建目标表 new_key_value_store。

如果 SELECT 语句没有指定列别名，那么列名将自动分配 `_col0`、`_col1`和 `_col2` 等。此外，在 SELECT 语句中使用特定的 SerDe 和独立于源表的存储格式创建新的目标表。

> Starting with [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-1180), the SELECT statement can include one or more common table expressions (CTEs), as shown in the [SELECT syntax](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select#LanguageManualSelect-SelectSyntax). For an example, see [Common Table Expression](https://cwiki.apache.org/confluence/display/Hive/Common+Table+Expression#CommonTableExpression-CTEinViews,CTAS,andInsertStatements).

从 Hive 0.13.0 开始，SELECT 语句可以包含一个或多个 CTEs，如 SELECT syntax 所示。例如，请参见 CTEs。

> Being able to select data from one table to another is one of the most powerful features of Hive. Hive handles the conversion of the data from the source format to the destination format as the query is being executed.

能够从一个表选择数据到另一个表是 Hive 最强大的功能之一。在执行查询时，Hive 处理从源格式到目标格式的数据转换。

##### 1.4.1.7、Create Table Like

> The LIKE form of CREATE TABLE allows you to copy an existing table definition exactly (without copying its data). In contrast to CTAS, the statement below creates a new empty_key_value_store table whose definition exactly matches the existing key_value_store in all particulars other than table name. The new table contains no rows.

CREATE TABLE 的 LIKE 形式允许精确地复制现有的表定义(而不复制其数据)。

与 CTAS 不同，下面的语句创建了一个新的 empty_key_value_store 表，它的定义与现有的 key_value_store 在除表名以外的所有细节中完全匹配。

新表不包含任何行。

```sql
CREATE TABLE empty_key_value_store
LIKE key_value_store [TBLPROPERTIES (property_name=property_value, ...)];
```

> Before Hive 0.8.0, CREATE TABLE LIKE view_name would make a copy of the view. In Hive 0.8.0 and later releases, CREATE TABLE LIKE view_name creates a table by adopting the schema of view_name (fields and partition columns) using defaults for SerDe and file formats.

在 Hive 0.8.0 之前，`CREATE TABLE LIKE view_name` 会复制视图。在 Hive 0.8.0 及以后的版本中，`CREATE TABLE LIKE view_name` 通过使用 view_name 模式(字段和分区列)来创建表，在 SerDe 和文件格式中使用默认值。

##### 1.4.1.8、Bucketed Sorted Tables

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
 ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '\001'
   COLLECTION ITEMS TERMINATED BY '\002'
   MAP KEYS TERMINATED BY '\003'
 STORED AS SEQUENCEFILE;
```

> In the example above, the page_view table is bucketed (clustered by) userid and within each bucket the data is sorted in increasing order of viewTime. Such an organization allows the user to do efficient sampling on the clustered column - in this case userid. The sorting property allows internal operators to take advantage of the better-known data structure while evaluating queries, also increasing efficiency. MAP KEYS and COLLECTION ITEMS keywords can be used if any of the columns are lists or maps.

在上面的示例中，page_view 表使用 `(clustered by) userid` 分桶，在每个桶中，数据按 viewTime 的递增顺序排序。

这样的组织允许用户对聚集列(在本例中为userid)进行有效的抽样。排序属性允许内部操作符在计算查询时利用已知的数据结构，这也提高了效率。

如果任何列是列表或映射，则可以使用 MAP KEYS 和 COLLECTION ITEMS 关键字。

> The CLUSTERED BY and SORTED BY creation commands do not affect how data is inserted into a table – only how it is read. This means that users must be careful to insert data correctly by specifying the number of reducers to be equal to the number of buckets, and using CLUSTER BY and SORT BY commands in their query.

CLUSTERED BY 和 SORTED BY 创建命令并不影响数据插入表的方式，只影响数据的读取方式。

这意味着用户必须小心地正确插入数据，具体做法是将 reducer 的数量指定为桶的数量，并在查询中使用 CLUSTER by 和 SORT by 命令。

> There is also an example of [creating and populating bucketed tables](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL+BucketedTables).

这里还有一个创建和填充分桶表的示例。

##### 1.4.1.9、Skewed Tables

> Version information. As of Hive 0.10.0 ([HIVE-3072](https://issues.apache.org/jira/browse/HIVE-3072) and [HIVE-3649](https://issues.apache.org/jira/browse/HIVE-3649)). See [HIVE-3026](https://issues.apache.org/jira/browse/HIVE-3026) for additional JIRA tickets that implemented list bucketing in Hive 0.10.0 and 0.11.0.

> Design documents. Read the [Skewed Join Optimization](https://cwiki.apache.org/confluence/display/Hive/Skewed+Join+Optimization) and [List Bucketing](https://cwiki.apache.org/confluence/display/Hive/ListBucketing) design documents for more information.

> This feature can be used to improve performance for tables where one or more columns have [skewed](https://cwiki.apache.org/confluence/display/Hive/Skewed+Join+Optimization) values. By specifying the values that appear very often (heavy skew) Hive will split those out into separate files (or directories in case of [list bucketing](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)) automatically and take this fact into account during queries so that it can skip or include the whole file (or directory in case of [list bucketing](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)) if possible.

对于一个或多个列有倾斜值的表，可以使用此特性提高性能。

通过指定的值经常出现(重斜)，Hive 自动将这些分割成单独的文件(或list bucketing中的目录)，在查询期间，使用这一事实，以便它可以跳过或者包含整个文件(或目录的列表用桶装)，如果可能的话。

> This can be specified on a per-table level during table creation.

这可以在创建表时按表级别指定。

> The following example shows one column with three skewed values, optionally with the STORED AS DIRECTORIES clause which specifies list bucketing.

下面的示例显示了一个具有三个倾斜值的列，可选地使用 `STORED AS DIRECTORIES` 子句指定 list bucketing。

```sql
CREATE TABLE list_bucket_single (key STRING, value STRING)
  SKEWED BY (key) ON (1,5,6) [STORED AS DIRECTORIES];
```

> And here is an example of a table with two skewed columns.

这是一个有两列倾列的表的例子。

```sql
CREATE TABLE list_bucket_multiple (col1 STRING, col2 int, col3 STRING)
  SKEWED BY (col1, col2) ON (('s1',1), ('s3',3), ('s13',13), ('s78',78)) [STORED AS DIRECTORIES];
```

> For corresponding ALTER TABLE statements, see [LanguageManual DDL#Alter Table Skewed or Stored as Directories](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTableSkewedorStoredasDirectories) below.

##### 1.4.1.10、Temporary Tables

> Version information. As of Hive 0.14.0 ([HIVE-7090](https://issues.apache.org/jira/browse/HIVE-7090)).

> A table that has been created as a temporary table will only be visible to the current session. Data will be stored in the user's scratch directory, and deleted at the end of the session.

已创建为临时表的表只对当前会话可见。数据将存储在用户的 scratch 目录中，并在会话结束时删除。

> If a temporary table is created with a database/table name of a permanent table which already exists in the database, then within that session any references to that table will resolve to the temporary table, rather than to the permanent table. The user will not be able to access the original table within that session without either dropping the temporary table, or renaming it to a non-conflicting name.

如果使用数据库中已经存在的永久表的数据库/表名创建临时表，那么在该会话中，对该表的任何引用都将解析到临时表，而不是永久表。

如果不删除临时表或将其重命名为不冲突的名称，用户将无法在该会话中访问原始表。

> Temporary tables have the following limitations:
> Partition columns are not supported.
> No support for creation of indexes.

临时表有以下限制:

- 不支持分区列。
- 不支持创建索引。

> Starting in [Hive 1.1.0](https://issues.apache.org/jira/browse/HIVE-7313) the storage policy for temporary tables can be set to memory, ssd, or default with the [hive.exec.temporary.table.storage](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.temporary.table.storage) configuration parameter (see [HDFS Storage Types and Storage Policies](http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-hdfs/ArchivalStorage.html#Storage_Types_and_Storage_Policies)).

从 Hive 1.1.0 开始，临时表的存储策略可以设置为 memory、ssd 或者 `hive.exec.temporary.table.storage` 设置的默认值。

```sql
CREATE TEMPORARY TABLE list_bucket_multiple (col1 STRING, col2 int, col3 STRING);
```

##### 1.4.1.11、Transactional Tables

> Version information. As of Hive 4.0 ([HIVE-18453](https://issues.apache.org/jira/browse/HIVE-18453)).

> A table that supports operations with ACID semantics. See [this](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions) for more details about transactional tables.

支持 ACID 语义的表。

```sql
CREATE TRANSACTIONAL TABLE transactional_table_test(key string, value string) PARTITIONED BY(ds string) STORED AS ORC;
```

##### 1.4.1.12、Constraints

> Version information. As of Hive 2.1.0 ([HIVE-13290](https://issues.apache.org/jira/browse/HIVE-13290)).

> Hive includes support for non-validated primary and foreign key constraints. Some SQL tools generate more efficient queries when constraints are present. Since these constraints are not validated, an upstream system needs to ensure data integrity before it is loaded into Hive.

Hive 支持未经验证的主键和外键约束。当存在约束时，一些 SQL 工具会生成更有效率的查询。

由于这些约束没有被验证，上游系统在加载到 Hive 之前需要确保数据的完整性。

```sql
create table pk(id1 integer, id2 integer,
  primary key(id1, id2) disable novalidate);
 
create table fk(id1 integer, id2 integer,
  constraint c1 foreign key(id1, id2) references pk(id2, id1) disable novalidate);
```

> Version information. As of Hive 3.0.0 ([HIVE-16575](https://issues.apache.org/jira/browse/HIVE-16575), [HIVE-18726](https://issues.apache.org/jira/browse/HIVE-18726), [HIVE-18953](https://issues.apache.org/jira/browse/HIVE-18953)).

> Hive includes support for UNIQUE, NOT NULL, DEFAULT and CHECK constraints. Beside UNIQUE all three type of constraints are enforced.

Hive 支持 UNIQUE、NOT NULL、DEFAULT 和 CHECK 约束。除了 UNIQUE 之外，还执行了所有三种类型的约束。

```sql
create table constraints1(id1 integer UNIQUE disable novalidate, id2 integer NOT NULL,
  usr string DEFAULT current_user(), price double CHECK (price > 0 AND price <= 1000));
 
create table constraints2(id1 integer, id2 integer,
  constraint c1_unique UNIQUE(id1) disable novalidate);
 
create table constraints3(id1 integer, id2 integer,
  constraint c1_check CHECK(id1 + id2 > 0));
```

> DEFAULT on complex data types such as map, struct, array is not supported.

在复杂数据类型上，不支持 DEFAULT。

#### 1.4.2、Drop Table

	DROP TABLE [IF EXISTS] table_name [PURGE];     -- (Note: PURGE available in Hive 0.14.0 and later)

> DROP TABLE removes metadata and data for this table. The data is actually moved to the `.Trash/Current` directory if Trash is configured (and PURGE is not specified). The metadata is completely lost.

DROP TABLE 删除该表的元数据和数据。如果配置了 Trash(并且没有指定PURGE)，数据实际上会移动到 `.Trash/Current` 目录。元数据完全丢失。

> When dropping an EXTERNAL table, data in the table will NOT be deleted from the file system. Starting Hive 4.0.0 ( HIVE-19981 - Managed tables converted to external tables by the HiveStrictManagedMigration utility should be set to delete data when the table is dropped RESOLVED   ) setting table property external.table.purge=true, will also delete the data.

删除 EXTERNAL 表时，表中的数据不会从文件系统中删除。从 Hive 4.0.0开始，设置 `external.table.purge=true` 也将删除数据。

> When dropping a table referenced by views, no warning is given (the views are left dangling as invalid and must be dropped or recreated by the user).

当删除视图引用的表时，不会给出任何警告(视图被当作无效而悬空，必须由用户删除或重新创建)。

> Otherwise, the table information is removed from the metastore and the raw data is removed as if by 'hadoop dfs -rm'. In many cases, this results in the table data being moved into the user's .Trash folder in their home directory; users who mistakenly DROP TABLEs may thus be able to recover their lost data by recreating a table with the same schema, recreating any necessary partitions, and then moving the data back into place manually using Hadoop. This solution is subject to change over time or across installations as it relies on the underlying implementation; users are strongly encouraged not to drop tables capriciously.

否则，表信息将从 metastore 中删除，原始数据将被删除，就像使用 `hadoop dfs -rm` 一样。

在很多情况下，这会导致表数据被移动到用户的主目录下的 `.Trash` 文件夹中；

因此，错误执行 `DROP TABLE` 的用户可以通过使用相同的模式重新创建表、重新创建任何必要的分区，然后使用 Hadoop 手动将数据移回原来的位置，来恢复丢失的数据。

由于依赖于底层实现，此解决方案可能随时间或跨安装而发生更改；强烈建议用户不要随意删除表。

> Version information: PURGE. The PURGE option is added in version 0.14.0 by [HIVE-7100](https://issues.apache.org/jira/browse/HIVE-7100).

版本信息：PURGE 选项在版本 0.14.0 中添加。

> If PURGE is specified, the table data does not go to the .Trash/Current directory and so cannot be retrieved in the event of a mistaken DROP. The purge option can also be specified with the table property auto.purge (see [TBLPROPERTIES](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties) above).

如果指定了 PURGE，则表数据不会转到 `.Trash/Current` 目录，因此在错误删除时无法检索。还可以使用表属性 `auto.purge` 指定清除选项。

> In Hive 0.7.0 or later, DROP returns an error if the table doesn't exist, unless IF EXISTS is specified or the configuration variable [hive.exec.drop.ignorenonexistent](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent) is set to true.

在 Hive 0.7.0 或更高版本中，如果表不存在，DROP 返回错误，除非指定了 if EXISTS 或者配置变量 `hive.exec.drop.ignorenonexistent` 设置为true。

> See the Alter Partition section below for how to [drop partitions](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropPartitions).

#### 1.4.3、Truncate Table

> Version information. As of Hive 0.11.0 ([HIVE-446](https://issues.apache.org/jira/browse/HIVE-446)).

	TRUNCATE [TABLE] table_name [PARTITION partition_spec];
	 
	partition_spec:
	  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)

> Removes all rows from a table or partition(s). The rows will be trashed if the filesystem Trash is enabled, otherwise they are deleted (as of Hive 2.2.0 with [HIVE-14626](https://issues.apache.org/jira/browse/HIVE-14626)). Currently the target table should be native/managed table or an exception will be thrown. User can specify partial partition_spec for truncating multiple partitions at once and omitting partition_spec will truncate all partitions in the table.

从表或分区中删除所有行。

如果文件系统垃圾被启用，这些行将被放到垃圾箱，否则它们将被删除。

当前目标表应该是原生/受管表，否则将抛出异常。用户可以指定 partial partition_spec 来一次清除多个分区，省略 partition_spec 将清除表中的所有分区。

> Starting with HIVE 2.3.0 ([HIVE-15880](https://issues.apache.org/jira/browse/HIVE-15880)) if the table property "auto.purge" (see [TBLPROPERTIES](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties) above) is set to "true" the data of the table is not moved to Trash when a TRUNCATE TABLE command is issued against it and cannot be retrieved in the event of a mistaken TRUNCATE. This is applicable only for managed tables (see [managed tables](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ManagedandExternalTables)). This behavior can be turned off if the "auto.purge" property is unset or set to false for a managed table.

从 HIVE 2.3.0 开始，如果表属性 auto.purge 为 true，当对表执行 TRUNCATE TABLE 命令时，表的数据不会被移到 Trash 中，并且在 TRUNCATE 错误的情况下不能被检索。这只适用于受管表。如果未设置 auto.purge 属性，或将其设置为 false，这个行为可以被关闭。

> Starting with Hive 4.0 ([HIVE-23183](https://issues.apache.org/jira/browse/HIVE-23183)) the TABLE token is optional, previous versions required it.

从 Hive 4.0 开始，TABLE 是可选的，以前的版本需要它。

### 1.5、Alter Table/Partition/Column

> Alter table statements enable you to change the structure of an existing table. You can add columns/partitions, change SerDe, add table and SerDe properties, or rename the table itself. Similarly, alter table partition statements allow you change the properties of a specific partition in the named table.

Alter table 语句允许更改现有表的结构。

可以添加列/分区、更改 SerDe、添加表和 SerDe 属性，或者重命名表本身。

类似地，alter table partition 语句允许更改指定表中特定分区的属性。

#### 1.5.1、Alter Table

##### 1.5.1.1、Rename Table

	ALTER TABLE table_name RENAME TO new_table_name;

> This statement lets you change the name of a table to a different name.

将表的名称更改为不同的名称。

> As of version 0.6, a rename on a [managed table](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-managedTable) moves its HDFS location. Rename has been changed as of version 2.2.0 ([HIVE-14909](https://issues.apache.org/jira/browse/HIVE-14909)) so that a managed table's HDFS location is moved only if the table is created without a [LOCATION](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-CreateTable) clause and under its database directory. Hive versions prior to 0.6 just renamed the table in the metastore without moving the HDFS location.

从 0.6 版本开始，受管表上的重命名将移动它的 HDFS 位置。

Rename 在版本 2.2.0 中发生了改变，所以只有在创建表时没有指定 LOCATION 子句，并且是在数据库目录下时，受管表的 HDFS 位置才会被移动。

Hive 0.6 之前的版本只是在 metastore 中重命名表，而没有移动 HDFS 的位置。

##### 1.5.1.2、Alter Table Properties

	ALTER TABLE table_name SET TBLPROPERTIES table_properties;
 
	table_properties:
	  : (property_name = property_value, property_name = property_value, ... )

> You can use this statement to add your own metadata to the tables. Currently last_modified_user, last_modified_time properties are automatically added and managed by Hive. Users can add their own properties to this list. You can do DESCRIBE EXTENDED TABLE to get this information.

使用此语句将自己的元数据添加到表中。

目前 last_modified_user 和 last_modified_time 属性由 Hive 自动添加和管理。

用户可以将自己的属性添加到这个列表中。可以使用 `DESCRIBE EXTENDED TABLE ` 来获取此信息。

For more information, see the [TBLPROPERTIES](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties) clause in Create Table above.

有关更多信息，请参阅上面 Create Table 中的 TBLPROPERTIES 子句。

###### 1.5.1.2.1、Alter Table Comment

> To change the comment of a table you have to change the comment property of the TBLPROPERTIES:

为修改表的注释，你必须改变 TBLPROPERTIES 的注释属性：

	ALTER TABLE table_name SET TBLPROPERTIES ('comment' = new_comment);

##### 1.5.1.3、Add SerDe Properties

	ALTER TABLE table_name [PARTITION partition_spec] SET SERDE serde_class_name [WITH SERDEPROPERTIES serde_properties];
 
	ALTER TABLE table_name [PARTITION partition_spec] SET SERDEPROPERTIES serde_properties;
 
	serde_properties:
		: (property_name = property_value, property_name = property_value, ... )

> These statements enable you to change a table's SerDe or add user-defined metadata to the table's SerDe object.

更改表的 SerDe 或向表的 SerDe 对象添加用户定义的元数据。

> The SerDe properties are passed to the table's SerDe when it is being initialized by Hive to serialize and deserialize data. So users can store any information required for their custom SerDe here. Refer to the [SerDe documentation](https://cwiki.apache.org/confluence/display/Hive/SerDe) and [Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe) in the Developer Guide for more information, and see [LanguageManual DDL#Row Format, Storage Format, and SerDe](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-RowFormat,StorageFormat,andSerDe) above for details about setting a table's SerDe and SERDEPROPERTIES in a CREATE TABLE statement.

当表被 Hive 初始化以序列化和反序列化数据时，SerDe 属性被传递给表的 SerDe。

因此，用户可以在这里存储自定义 SerDe 所需的任何信息。更多信息请参考 SerDe 文档和 Developer Guide 中的 Hive SerDe。关于在 CREATE TABLE 语句中设置表的 SerDe 和 SERDEPROPERTIES 的详细信息参见 LanguageManual DDL#Row Format, Storage Format, and SerDe。

> Note that both property_name and property_value must be quoted.

注意 property_name 和 property_value 都必须用引号括起来。

例如：

```sql
ALTER TABLE table_name SET SERDEPROPERTIES ('field.delim' = ',');
```

##### 1.5.1.4、Remove SerDe Properties

> Version information. Remove SerDe Properties is supported as of Hive 4.0.0 ([HIVE-21952](https://issues.apache.org/jira/browse/HIVE-18842)).

	ALTER TABLE table_name [PARTITION partition_spec] UNSET SERDEPROPERTIES (property_name, ... );

> These statements enable you to remove user-defined metadata to the table's SerDe object.

将删除表 SerDe 对象的用户定义的元数据。

> Note that property_name must be quoted.

注意，property_name 必须用引号括起来。

例如：

```sql
ALTER TABLE table_name UNSET SERDEPROPERTIES ('field.delim');
Alter Table Storage Properties
ALTER TABLE table_name CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name, ...)]
  INTO num_buckets BUCKETS;
```

> These statements change the table's physical storage properties.

这些语句修改表的物理存储属性。

> NOTE: These commands will only modify Hive's metadata, and will NOT reorganize or reformat existing data. Users should make sure the actual data layout conforms with the metadata definition.

注意：这些命令只会修改 Hive 的元数据，不会重新组织或重新格式化现有的数据。用户应该确保实际的数据布局与元数据定义一致。

##### 1.5.1.5、Alter Table Storage Properties

##### 1.5.1.6、Alter Table Skewed or Stored as Directories

> Version information. As of Hive 0.10.0 ([HIVE-3072](https://issues.apache.org/jira/browse/HIVE-3072) and [HIVE-3649](https://issues.apache.org/jira/browse/HIVE-3649)). See [HIVE-3026](https://issues.apache.org/jira/browse/HIVE-3026) for additional JIRA tickets that implemented list bucketing in Hive 0.10.0 and 0.11.0.

> A table's SKEWED and STORED AS DIRECTORIES options can be changed with ALTER TABLE statements. See LanguageManual DDL#Skewed Tables above for the corresponding CREATE TABLE syntax.

可以用 ALTER TABLE 语句更改表的 SKEWED 和 STORED AS DIRECTORIE 选项。

###### 1.5.1.6.1、Alter Table Skewed

	ALTER TABLE table_name SKEWED BY (col_name1, col_name2, ...)
	  ON ([(col_name1_value, col_name2_value, ...) [, (col_name1_value, col_name2_value), ...]
	  [STORED AS DIRECTORIES];

> The STORED AS DIRECTORIES option determines whether a [skewed](https://cwiki.apache.org/confluence/display/Hive/Skewed+Join+Optimization) table uses the [list bucketing](https://cwiki.apache.org/confluence/display/Hive/ListBucketing) feature, which creates subdirectories for skewed values.

STORED AS DIRECTORIES 选项决定倾斜表是否使用 list bucketing 特性，该特性会为倾斜值创建子目录。

###### 1.5.1.6.2、Alter Table Not Skewed

	ALTER TABLE table_name NOT SKEWED;

> The NOT SKEWED option makes the table non-skewed and turns off the list bucketing feature (since a list-bucketing table is always skewed). This affects partitions created after the ALTER statement, but has no effect on partitions created before the ALTER statement.

NOT SKEWED 选项使表是不倾斜的，并关闭 list bucketing 功能(因为 list bucketing 表总是倾斜的)。这将影响在 ALTER 语句之后创建的分区，但对在 ALTER 语句之前创建的分区没有影响。

###### 1.5.1.6.3、Alter Table Not Stored as Directories

	ALTER TABLE table_name NOT STORED AS DIRECTORIES;

> This turns off the list bucketing feature, although the table remains skewed.

关闭 list bucketing 功能，尽管表是倾斜的。

###### 1.5.1.6.4、Alter Table Set Skewed Location

	ALTER TABLE table_name SET SKEWED LOCATION (col_name1="location1" [, col_name2="location2", ...] );

> This changes the location map for list bucketing.

改变 list bucketing 位置映射。

##### 1.5.1.7、Alter Table Constraints

> Version information. As of Hive release [2.1.0](https://issues.apache.org/jira/browse/HIVE-13290).

> Table constraints can be added or removed via ALTER TABLE statements.

可以通过 ALTER TABLE 语句添加、删除表限制。

	ALTER TABLE table_name ADD CONSTRAINT constraint_name PRIMARY KEY (column, ...) DISABLE NOVALIDATE;
	
	ALTER TABLE table_name ADD CONSTRAINT constraint_name FOREIGN KEY (column, ...) REFERENCES table_name(column, ...) DISABLE NOVALIDATE RELY;
	
	ALTER TABLE table_name ADD CONSTRAINT constraint_name UNIQUE (column, ...) DISABLE NOVALIDATE;
	
	ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name NOT NULL ENABLE;
	
	ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name DEFAULT default_value ENABLE;
	
	ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name CHECK check_expression ENABLE;
 
	ALTER TABLE table_name DROP CONSTRAINT constraint_name;

##### 1.5.1.8、Additional Alter Table Statements

> See [LanguageManual DDL#Alter Either Table or Partition](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterEitherTableorPartition) below for more DDL statements that alter tables.

#### 1.5.2、Alter Partition

> Partitions can be added, renamed, exchanged (moved), dropped, or (un)archived by using the PARTITION clause in an ALTER TABLE statement, as described below. To make the metastore aware of partitions that were added directly to HDFS, you can use the metastore check command ([MSCK](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-RecoverPartitions(MSCKREPAIRTABLE))) or on Amazon EMR you can use the RECOVER PARTITIONS option of ALTER TABLE. See [LanguageManual DDL#Alter Either Table or Partition](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterEitherTableorPartition) below for more ways to alter partitions.

通过在 ALTER TABLE 语句中使用 PARTITION 子句，可以添加、重命名、交换(移动)、删除或归档(解当)分区，如下所述。

为了让 metastore 能够识别直接添加到 HDFS 的分区，可以使用 metastore check 命令(MSCK)，或者在 Amazon EMR 上使用 ALTER TABLE 的 RECOVER PARTITIONS 选项。请参阅下面的 LanguageManual DDL#Alter Either Table or Partition 以了解更多更改分区的方法。

> Version 1.2+: As of Hive 1.2 ([HIVE-10307](https://issues.apache.org/jira/browse/HIVE-10307)), the partition values specified in partition specification are type checked, converted, and normalized to conform to their column types if the property [hive.typecheck.on.insert](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert) is set to true (default). The values can be number literals.

1.2 以上的版本：从 Hive 1.2 开始，如果属性 `hive.typecheck.on.insert` 设置为 true(默认)，分区规范中指定的分区值会被类型检查、转换和规范化，以符合它们的列类型。这些值可以是数字字面量。

##### 1.5.2.1、Add Partitions

	ALTER TABLE table_name ADD [IF NOT EXISTS] PARTITION partition_spec [LOCATION 'location'][, PARTITION partition_spec [LOCATION 'location'], ...];
	 
	partition_spec:
	  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)

> You can use ALTER TABLE ADD PARTITION to add partitions to a table. Partition values should be quoted only if they are strings. The location must be a directory inside of which data files reside. (ADD PARTITION changes the table metadata, but does not load data. If the data does not exist in the partition's location, queries will not return any results.) An error is thrown if the partition_spec for the table already exists. You can use IF NOT EXISTS to skip the error.

可以使用 ALTER TABLE ADD PARTITION 向表添加分区。

只有当分区值是字符串时，才应该使用引号。位置必须是数据文件驻留在其中的目录。（ADD PARTITION 修改表元数据，但不加载数据。如果数据在分区的位置不存在，查询将不会返回任何结果。）

如果该表的 partition_spec 已经存在，则抛出错误。可以使用 IF NOT EXISTS 来跳过错误。

> Version 0.7: Although it is proper syntax to have multiple partition_spec in a single ALTER TABLE, if you do this in version 0.7 your partitioning scheme will fail. That is, every query specifying a partition will always use only the first partition.

版本0.7：虽然在一个 ALTER TABLE 中包含多个 partition_spec 是正确的语法，但如果在 0.7 版本中这样做，分区方案将会失败。也就是说，指定分区的每个查询将始终只使用第一个分区。

> Specifically, the following example will FAIL silently and without error in Hive 0.7, and all queries will go only to dt='2008-08-08' partition, no matter which partition you specify.

具体来说，下面的例子将会在 Hive 0.7 中悄无声息地失败，没有错误。并且所有的查询都只会进入 dt='2008-08-08' 分区，不管你指定的是哪个分区。

例如：

```sql
ALTER TABLE page_view ADD PARTITION (dt='2008-08-08', country='us') location '/path/to/us/part080808'
                          PARTITION (dt='2008-08-09', country='us') location '/path/to/us/part080809';
```

> In Hive 0.8 and later, you can add multiple partitions in a single ALTER TABLE statement as shown in the previous example.

在 Hive 0.8 及以后的版本中，可以在一个 ALTER TABLE 语句中添加多个分区，如上面的例子所示。

> In Hive 0.7, if you want to add many partitions you should use the following form:

在 Hive 0.7 中，如果你想要添加多个分区，你应该使用以下形式:

```sql
ALTER TABLE table_name ADD PARTITION (partCol = 'value1') location 'loc1';
ALTER TABLE table_name ADD PARTITION (partCol = 'value2') location 'loc2';
...
ALTER TABLE table_name ADD PARTITION (partCol = 'valueN') location 'locN';
```

###### 1.5.2.1.1、Dynamic Partitions

> Partitions can be added to a table dynamically, using a Hive INSERT statement (or a Pig STORE statement). See these documents for details and examples:

- [Design Document for Dynamic Partitions](https://cwiki.apache.org/confluence/display/Hive/DynamicPartitions)
- [Tutorial: Dynamic-Partition Insert](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-Dynamic-PartitionInsert)
- [Hive DML: Dynamic Partition Inserts](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-DynamicPartitionInserts)
- [HCatalog Dynamic Partitioning](https://cwiki.apache.org/confluence/display/Hive/HCatalog+DynamicPartitions)
	- [Usage with Pig](https://cwiki.apache.org/confluence/display/Hive/HCatalog+DynamicPartitions#HCatalogDynamicPartitions-UsagewithPig)
	- [Usage from MapReduce](https://cwiki.apache.org/confluence/display/Hive/HCatalog+DynamicPartitions#HCatalogDynamicPartitions-UsagefromMapReduce)

##### 1.5.2.2、Rename Partition

> Version information. As of Hive 0.9.

	ALTER TABLE table_name PARTITION partition_spec RENAME TO PARTITION partition_spec;

> This statement lets you change the value of a partition column. One of use cases is that you can use this statement to normalize your legacy partition column value to conform to its type. In this case, the type conversion and normalization are not enabled for the column values in old partition_spec even with property [hive.typecheck.on.insert](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert) set to true (default) which allows you to specify any legacy data in form of string in the old partition_spec.

该语句允许更改分区列的值。

其中一个用例是，你可以使用此语句来规范化你的遗留分区列值，使其符合其类型。

在这种情况下，即使 `hive.typecheck.on.insert` 属性设置为 true(默认)，也不会对旧 partition_spec 中的列值启用类型转换和规范化。这个属性允许你在旧的 partition_spec 中以字符串形式指定任何遗留数据。

##### 1.5.2.3、Exchange Partition

> Partitions can be exchanged (moved) between tables.

分区可以在表之间交换(移动)。

> Version information. As of Hive 0.12 ([HIVE-4095](https://issues.apache.org/jira/browse/HIVE-4095)). Multiple partitions supported in Hive versions [1.2.2, 1.3.0, and 2.0.0+](https://issues.apache.org/jira/browse/HIVE-11745).

```sql
-- Move partition from table_name_1 to table_name_2
ALTER TABLE table_name_2 EXCHANGE PARTITION (partition_spec) WITH TABLE table_name_1;
-- multiple partitions
ALTER TABLE table_name_2 EXCHANGE PARTITION (partition_spec, partition_spec2, ...) WITH TABLE table_name_1;
```

> This statement lets you move the data in a partition from a table to another table that has the same schema and does not already have that partition. For further details on this feature, see [Exchange Partition](https://cwiki.apache.org/confluence/display/Hive/Exchange+Partition) and [HIVE-4095](https://issues.apache.org/jira/browse/HIVE-4095).

该语句允许将一个分区中的数据从一个表移动到另一个具有相同模式但尚未拥有该分区的表。

有关此特性的详细信息，请参阅 Exchange Partition 和 HIVE-4095。

##### 1.5.2.4、Discover Partitions

> Automatically discovers and synchronizes the metadata of the partition in Hive Metastore. 

自动发现，并同步 Hive Metastore 中的分区的元数据。

> When External Partitioned Tables are created, "discover.partitions"="true" table property gets automatically added. For managed partitioned tables, "discover.partitions" table property can be manually added. When Hive Metastore Service (HMS) is started in remote service mode, a background thread (PartitionManagementTask) gets scheduled periodically every 300s (configurable via metastore.partition.management.task.frequency config) that looks for tables with "discover.partitions" table property set to true and performs msck repair in sync mode. If the table is a transactional table, then Exclusive Lock is obtained for that table before performing msck repair. With this table property, "MSCK REPAIR TABLE table_name SYNC PARTITIONS" is no longer required to be run manually. 

当创建外部分区表时，会自动添加 "discover.partitions"="true" 表属性。对于受管分区表，可以手动添加 "discover.partitions" 属性。

当 Hive Metastore Service (HMS) 在远程服务模式下启动时，一个后台线程(PartitionManagementTask)每 300 秒定时(通过`metastore.partition.management.task.frequency`配置)查找 "discover.partitions" 设为 true 的表，并在同步模式下执行 msck repair。

如果表是事务表，则在执行 msck repair 之前为该表获得 Exclusive Lock。有了这个表属性，`MSCK REPAIR table table_name SYNC PARTITIONS` 不再需要手动运行。

> Version information. As of Hive 4.0.0 ([HIVE-20707](https://issues.apache.org/jira/browse/HIVE-20707)). 

##### 1.5.2.5、Partition Retention

> Table property "partition.retention.period" can now be specified for partitioned tables with a retention interval. When a retention interval is specified, the background thread running in HMS (refer Discover Partitions section), will check the age (creation time) of the partition and if the partition's age is older than the retention period, it will be dropped. Dropping partitions after retention period will also delete the data in that partition. For example, if an external partitioned table with 'date' partition is created with table properties "discover.partitions"="true" and "partition.retention.period"="7d" then only the partitions created in last 7 days are retained.

现在可以为具有保留间隔的分区表指定 `partition.retention.period` 表属性。

当指定了保留间隔时，HMS 中运行的后台线程将检查分区的年龄(创建时间)，如果分区的年龄大于保留时间，则将删除该分区。

在保留期之后，删除分区也会删除该分区中的数据。例如，如果一个带有 date 分区的外部分区表使用表属性 "discover.partitions"="true" 和 "partition.retention.period"="7d" 创建，则只保留最近 7 天创建的分区。

> Version information. As of Hive 4.0.0 ([HIVE-20707](https://issues.apache.org/jira/browse/HIVE-20707)). 

##### 1.5.2.6、Recover Partitions (MSCK REPAIR TABLE)

> Hive stores a list of partitions for each table in its metastore. If, however, new partitions are directly added to HDFS (say by using hadoop fs -put command) or removed from HDFS, the metastore (and hence Hive) will not be aware of these changes to partition information unless the user runs ALTER TABLE table_name ADD/DROP PARTITION commands on each of the newly added or removed partitions, respectively.

Hive 在其 metastore 中为每个表存储一个分区的列表。

然而，如果新的分区直接添加到 HDFS(比如通过使用 `hadoop fs -put`命令)或从 HDFS 删除，metastore 不会意识到这些分区信息的变化，除非用户分别在每个新添加或删除的分区上，运行 ALTER TABLE table_name ADD/DROP PARTITION 命令。

> However, users can run a metastore check command with the repair table option:

但是，用户可以使用 repair 表选项运行 metastore 检查命令:

	MSCK [REPAIR] TABLE table_name [ADD/DROP/SYNC PARTITIONS];

> which will update metadata about partitions to the Hive metastore for partitions for which such metadata doesn't already exist. The default option for MSC command is ADD PARTITIONS. With this option, it will add any partitions that exist on HDFS but not in metastore to the metastore. The DROP PARTITIONS option will remove the partition information from metastore, that is already removed from HDFS. The SYNC PARTITIONS option is equivalent to calling both ADD and DROP PARTITIONS. See [HIVE-874](https://issues.apache.org/jira/browse/HIVE-874) and [HIVE-17824](https://issues.apache.org/jira/browse/HIVE-17824) for more details. When there is a large number of untracked partitions, there is a provision to run MSCK REPAIR TABLE batch wise to avoid OOME (Out of Memory Error). By giving the configured batch size for the property hive.msck.repair.batch.size it can run in the batches internally. The default value of the property is zero, it means it will execute all the partitions at once. MSCK command without the REPAIR option can be used to find details about metadata mismatch metastore.

对于元数据已经不存在的分区，这将更新关于分区的元数据到 Hive metastore 中。

MSC 命令的默认选项是 ADD PARTITIONS 。使用这个选项，它将把所有存在于 HDFS 上，但不在 metastore 中的分区添加到 metastore中。

DROP PARTITIONS 选项将从 metastore 中删除已经从 HDFS 删除的分区的信息。

SYNC PARTITIONS 选项相当于同时调用 ADD 和 DROP PARTITIONS。详情请参阅 HIVE-874 和 HIVE-17824。

当存在大量未跟踪的分区时，可以批量运行 MSCK REPAIR TABLE，以避免内存溢出错误(OOME)。

通过为属性 hive.msck.repair.batch 配置批次的大小，它可以在内部批量运行。该属性的默认值是零，这意味着它将一次执行所有的分区。

不带 REPAIR 选项的 MSCK 命令可用于查找有关元数据不匹配 metastore 的详细信息。

> The equivalent command on Amazon Elastic MapReduce (EMR)'s version of Hive is:

Amazon Elastic MapReduce (EMR) 的 Hive 版本的等价命令是:

	ALTER TABLE table_name RECOVER PARTITIONS;

> Starting with Hive 1.3, MSCK will throw exceptions if directories with disallowed characters in partition values are found on HDFS. Use hive.msck.path.validation setting on the client to alter this behavior; "skip" will simply skip the directories. "ignore" will try to create partitions anyway (old behavior). This may or may not work.

从 Hive 1.3 开始，如果在 HDFS 上发现分区值中有不允许字符的目录，MSCK 会抛出异常。在客户端上使用 `hive.msck.path.validation` 设置以更改此行为；"skip" 将简单地跳过目录。"ignore" 将尝试创建分区(旧的行为)。这可能行得通，也可能行不通。

##### 1.5.2.7、Drop Partitions

	ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec[, PARTITION partition_spec, ...]
	  [IGNORE PROTECTION] [PURGE];            -- (Note: PURGE available in Hive 1.2.0 and later, IGNORE PROTECTION not available 2.0.0 and later)

> You can use ALTER TABLE DROP PARTITION to drop a partition for a table. This removes the data and metadata for this partition. The data is actually moved to the .Trash/Current directory if Trash is configured, unless PURGE is specified, but the metadata is completely lost (see [LanguageManual DDL#Drop Table above](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropTable)).

可以使用 ALTER TABLE DROP PARTITION 删除表的分区。

这将删除该分区的数据和元数据。如果配置了 Trash，数据实际上会移动到 `.Trash/Current` 目录，除非指定了 PURGE，但是元数据会完全丢失。

> Version Information: PROTECTION. IGNORE PROTECTION is no longer available in versions 2.0.0 and later. This functionality is replaced by using one of the several security options available with Hive (see [SQL Standard Based Hive Authorization](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization)). See [HIVE-11145](https://issues.apache.org/jira/browse/HIVE-11145) for details.

版本信息:PROTECTION。 IGNORE PROTECTION 在 2.0.0 及更高版本中不再可用。此功能被 Hive 提供的几种安全选项之一所替代(参见SSQL Standard Based Hive Authorization)。详细信息请参见 HIVE-11145。

> For tables that are protected by [NO_DROP CASCADE](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTable/PartitionProtections), you can use the predicate IGNORE PROTECTION to drop a specified partition or set of partitions (for example, when splitting a table between two Hadoop clusters):

对于受 NO_DROP CASCADE 保护的表，可以使用谓词 IGNORE PROTECTION 删除指定的分区或分区集(例如，在两个 Hadoop 集群之间拆分一个表时):

	ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec IGNORE PROTECTION;

> The above command will drop that partition regardless of protection stats.

上面的命令将删除该分区，而不考虑保护状态。

> Version information: PURGE. The PURGE option is added to ALTER TABLE in version 1.2.1 by [HIVE-10934](https://issues.apache.org/jira/browse/HIVE-10934).

版本信息:PURGE。 在版本 1.2.1 中，PURGE 选项添加到 ALTER TABLE 中。

> If PURGE is specified, the partition data does not go to the .Trash/Current directory and so cannot be retrieved in the event of a mistaken DROP:

如果指定了 PURGE，分区数据不会进入 `.Trash/Current` 目录，因此在错误删除时无法检索:

	ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec PURGE;     -- (Note: Hive 1.2.0 and later)

> The purge option can also be specified with the table property auto.purge (see [TBLPROPERTIES](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties) above).

还可以使用`auto.purge` 表属性指定 purge 选项。

> In Hive 0.7.0 or later, DROP returns an error if the partition doesn't exist, unless IF EXISTS is specified or the configuration variable [hive.exec.drop.ignorenonexistent](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent) is set to true.

在 Hive 0.7.0 或更高版本中，如果分区不存在，DROP 会返回错误，除非指定了 IF EXISTS 或者配置变量 `hive.exec.drop.ignorenonexistent` 设置为true。

	ALTER TABLE page_view DROP PARTITION (dt='2008-08-08', country='us');

##### 1.5.2.8、(Un)Archive Partition

	ALTER TABLE table_name ARCHIVE PARTITION partition_spec;
	ALTER TABLE table_name UNARCHIVE PARTITION partition_spec;

> Archiving is a feature to moves a partition's files into a Hadoop Archive (HAR). Note that only the file count will be reduced; HAR does not provide any compression. See [LanguageManual Archiving](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Archiving) for more information.

归档是一种将分区文件移动到 Hadoop Archive(HAR)的功能。

注意，只有文件数会减少；HAR不提供任何压缩。有关更多信息，请参阅 LanguageManual Archiving。

#### 1.5.3、Alter Either Table or Partition

##### 1.5.3.1、Alter Table/Partition File Format

	Alter Table/Partition File Format
	ALTER TABLE table_name [PARTITION partition_spec] SET FILEFORMAT file_format;

> This statement changes the table's (or partition's) file format. For available file_format options, see the section above on [CREATE TABLE](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-CreateTable). The operation only changes the table metadata. Any conversion of existing data must be done outside of Hive.

这条语句改变了表(或分区)的文件格式。有关可用的 file_format 选项，请参阅上面关于 CREATE TABLE 的部分。该操作仅更改表元数据。任何现有数据的转换都必须在 Hive 之外完成。

##### 1.5.3.2、Alter Table/Partition Location

	ALTER TABLE table_name [PARTITION partition_spec] SET LOCATION "new location";

##### 1.5.3.3、Alter Table/Partition Touch

##### 1.5.3.4、Alter Table/Partition Protections

##### 1.5.3.5、Alter Table/Partition Compact

##### 1.5.3.6、Alter Table/Partition Concatenate

##### 1.5.3.7、Alter Table/Partition Update columns

#### 1.5.4、Alter Column

##### 1.5.4.1、Rules for Column Names

##### 1.5.4.2、Change Column Name/Type/Position/Comment

##### 1.5.4.3、Add/Replace Columns

##### 1.5.4.4、Partial Partition Specification

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