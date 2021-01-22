# Data Types

[TOC]

## 1、Overview

> This lists all supported data types in Hive. See [Type System](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-TypeSystem) in the [Tutorial](https://cwiki.apache.org/confluence/display/Hive/Tutorial) for additional information.

这里列出了 Hive 中支持的所有的数据列类型。见 Tutorial.Type System 更多信息。

> For data types supported by HCatalog, see:

HCatalog 支持的数据类型，见：

- [HCatLoader Data Types](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore#HCatalogLoadStore-HCatLoaderDataTypes)
- [HCatStorer Data Types](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore#HCatalogLoadStore-HCatStorerDataTypes)
- [HCatRecord Data Types](https://cwiki.apache.org/confluence/display/Hive/HCatalog+InputOutput#HCatalogInputOutput-HCatRecord)

### 1.1、Numeric Types

- [TINYINT](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-tinyint) (1-byte signed integer, from -128 to 127)

- [SMALLINT](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-smallint) (2-byte signed integer, from -32,768 to 32,767)

- [INT](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-int)/INTEGER (4-byte signed integer, from -2,147,483,648 to 2,147,483,647)

- [BIGINT](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-bigint) (8-byte signed integer, from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807)

- FLOAT (4-byte single precision floating point number)

- DOUBLE (8-byte double precision floating point number)

- DOUBLE PRECISION (alias for DOUBLE, only available starting with Hive [2.2.0](https://issues.apache.org/jira/browse/HIVE-13556))

- [DECIMAL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-decimal)

	- Introduced in Hive [0.11.0](https://issues.apache.org/jira/browse/HIVE-2693) with a precision of 38 digits

	- Hive [0.13.0](https://issues.apache.org/jira/browse/HIVE-3976) introduced user-definable precision and scale

- NUMERIC (same as DECIMAL, starting with [Hive 3.0.0](https://issues.apache.org/jira/browse/HIVE-16764))

### 1.2、Date/Time Types

- [TIMESTAMP](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-timestamp) (Note: Only available starting with Hive [0.8.0](https://issues.apache.org/jira/browse/HIVE-2272))

- [DATE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-date) (Note: Only available starting with Hive [0.12.0](https://issues.apache.org/jira/browse/HIVE-4055))

- [INTERVAL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-Intervals) (Note: Only available starting with Hive [1.2.0](https://issues.apache.org/jira/browse/HIVE-9792))

### 1.3、String Types

- [STRING](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-string)

- [VARCHAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-varchar) (Note: Only available starting with Hive [0.12.0](https://issues.apache.org/jira/browse/HIVE-4844))

- [CHAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-char) (Note: Only available starting with Hive [0.13.0](https://issues.apache.org/jira/browse/HIVE-5191))

### 1.4、Misc Types

- BOOLEAN

- BINARY (Note: Only available starting with Hive [0.8.0](https://issues.apache.org/jira/browse/HIVE-2380))

### 1.5、Complex Types

- arrays: `ARRAY<data_type>` (Note: negative values and non-constant expressions are allowed as of [Hive 0.14](https://issues.apache.org/jira/browse/HIVE-7325).)

- maps: `MAP<primitive_type, data_type>` (Note: negative values and non-constant expressions are allowed as of Hive [0.14](https://issues.apache.org/jira/browse/HIVE-7325).)

- structs: `STRUCT<col_name : data_type [COMMENT col_comment], ...>`

- union: `UNIONTYPE<data_type, data_type, ...>` (Note: Only available starting with Hive [0.7.0](https://issues.apache.org/jira/browse/HIVE-537).)

## 2、Column Types

### 2.1、Integral Types (TINYINT, SMALLINT, INT/INTEGER, BIGINT)

> Integral literals are assumed to be INT by default, unless the number exceeds the range of INT in which case it is interpreted as a BIGINT, or if one of the following postfixes is present on the number.

整数字面量默认是 INT，**除非如果数字超过了 INT 的范围，就解释成 BIGINT**，或者如果下面的前缀出现在数字后面。

Type     |  Postfix   | Example
---      |:---        |:---
TINYINT  |    Y       | 100Y
SMALLINT |    S       | 100S
BIGINT   |    L       | 100L 

> Version:INTEGER is introduced as a synonym for INT in Hive 2.2.0 ([HIVE-14950](https://issues.apache.org/jira/browse/HIVE-14950)).

INTEGER 作为 INT 的同义词被引入是在 Hive 2.2.0。

### 2.2、Strings

> String literals can be expressed with either single quotes (') or double quotes ("). Hive uses C-style escaping within the strings.

字符串字面量可以使用单引号，也可以使用双引号。Hive 在字符串中使用 C 风格的转义。

### 2.3、Varchar

> Varchar types are created with a length specifier (between 1 and 65535), which defines the maximum number of characters allowed in the character string. If a string value being converted/assigned to a varchar value exceeds the length specifier, the string is silently truncated. Character length is determined by the number of code points contained by the character string.

创建 **Varchar 类型可以指定了一个长度**（1 到 65535 之间)），这个长度定义了字符串中允许的最大字符数。

如果正在转换/分配给一个 varchar 类型值的字符串值超出了长度，则该字符串将**被静默截断**。字符长度由字符串包含的代码点数量决定。

> Like string, trailing whitespace is significant in varchar and will affect comparison results.

与字符串一样，末尾的空格在 varchar 中也很重要，并且会影响比较结果。

> Limitations:Non-generic UDFs cannot directly use varchar type as input arguments or return values. String UDFs can be created instead, and the varchar values will be converted to strings and passed to the UDF. To use varchar arguments directly or to return varchar values, create a GenericUDF.There may be other contexts which do not support varchar, if they rely on reflection-based methods for retrieving type information. This includes some SerDe implementations.

限制：非通用 UDFs 不能直接使用 varchar 类型作为输入参数或返回值。

可以创建 String UDFs，varchar 值将被转换为字符串，并传递给 UDF。要直接使用 varchar 参数或返回 varchar 值，请创建一个 GenericUDF。

如果其他上下文依赖于基于反射的方法来检索类型信息，那么它们可能不支持 varchar。这包括一些 SerDe 实现。

> Version:Varchar datatype was introduced in Hive 0.12.0 ([HIVE-4844](https://issues.apache.org/jira/browse/HIVE-4844)).

版本：Varchar 数据类型在 Hive 0.12.0 中被引入。

### 2.4、Char

> Char types are similar to Varchar but they are fixed-length meaning that values shorter than the specified length value are padded with spaces but trailing spaces are not important during comparisons. The maximum length is fixed at 255.

Char 类型类似 Varchar，但它是**固定长度的**，这就意味着小于指定长度的值会用空格填充，但比较时末尾空格并不重要。最大长度固定为 255。

	CREATE TABLE foo (bar CHAR(10))

> Version:Char datatype was introduced in Hive 0.13.0 ([HIVE-5191](https://issues.apache.org/jira/browse/HIVE-5191)).

版本：Char 数据类型在 Hive 0.13.0 中被引入。

### 2.5、Timestamps

> Supports traditional UNIX timestamp with optional nanosecond precision.

支持**传统的 UNIX 时间戳**，可选的纳秒精度。

> Supported conversions:

支持的转换:

- 整型数字类型:以秒为单位解释为 UNIX 时间戳

- 浮点数字类型:解释为 UNIX 时间戳，以秒为单位，具有十进制精度

- 字符串:符合 JDBC 的 `java.sql.Timestamp` 格式 "YYYY-MM-DD HH:MM:SS.fffffffff" (小数点h后9位精度)

> Integer numeric types: Interpreted as UNIX timestamp in seconds

> Floating point numeric types: Interpreted as UNIX timestamp in seconds with decimal precision

> Strings: JDBC compliant java.sql.Timestamp format "YYYY-MM-DD HH:MM:SS.fffffffff" (9 decimal place precision)

> Timestamps are interpreted to be timezoneless and stored as an offset from the UNIX epoch. Convenience UDFs for conversion to and from timezones are provided (to_utc_timestamp, from_utc_timestamp).

Timestamps 被解释为无时区的，并存储为从 UNIX 纪元的偏移量。为与时区之间转换，提供了方便的 UDFs(to_utc_timestamp、from_utc_timestamp)。

> All existing datetime [UDFs](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-DateFunctions) (month, day, year, hour, etc.) work with the TIMESTAMP data type.

**所有现有的 datetime UDFs(月、日、年、小时等)都使用 TIMESTAMP 数据类型**。

> Timestamps in text files have to use the format yyyy-mm-dd hh:mm:ss[.f...]. If they are in another format, declare them as the appropriate type (INT, FLOAT, STRING, etc.) and use a UDF to convert them to timestamps.

**文本文件中的 Timestamps 必须使用 `yyyy-mm-dd hh:mm:ss[.f...]` 格式**。

如果它们是另一种格式，则将它们声明为适当的类型(INT、FLOAT、STRING等)，并使用 UD F将它们转换为 timestamps。

> On the table level, alternative timestamp formats can be supported by providing the format to the [SerDe property](https://cwiki.apache.org/confluence/display/Hive/SerDe#SerDe-HiveQLforSerDes) "timestamp.formats" (as of release 1.2.0 with [HIVE-9298]https://issues.apache.org/jira/browse/HIVE-9298)). For example, yyyy-MM-dd'T'HH:mm:ss.SSS,yyyy-MM-dd'T'HH:mm:ss.

在表的层面，可以通过向 SerDe 属性 “timestamp” 提供格式来支持其他的 timestamp 格式。例如，`yyyy-MM-dd'T'HH:mm:ss.SSS`、`yyyy-MM-dd'T'HH:mm:ss`。

> Version:Timestamps were introduced in Hive 0.8.0 ([HIVE-2272](https://issues.apache.org/jira/browse/HIVE-2272)).

版本：Timestamps 数据类型在 Hive 0.8.0 中被引入。

### 2.6、Date

> DATE values describe a particular year/month/day, in the form YYYY-­MM-­DD. For example, DATE '2013-­01-­01'. Date types do not have a time of day component. The range of values supported for the Date type is 0000-­01-­01 to 9999-­12-­31, dependent on support by the primitive Java Date type.

DATE 值描述了一个特殊的年/月/日，**以 YYYY-­MM-­DD 格式**。例如，日期 '2013-­01-­01'。

Date 类型没有“一天的时间”组件【时分秒】。Date 类型支持的值范围是 0000—01—01 到 9999—12—31，这取决于基本的 Java 日期类型的支持。

> Version:Dates were introduced in Hive 0.12.0 ([HIVE-4055](https://issues.apache.org/jira/browse/HIVE-4055)).

版本：Dates 数据类型在 Hive 0.12.0 中被引入。

#### 2.6.1、Casting Dates

> Date types can only be converted to/from Date, Timestamp, or String types. Casting with user-specified formats is documented here.

Date 类型仅能在 Date、Timestamp 或 String 类型间转换。使用用户指定格式进行强制转换的文档在这里。

Valid cast to/from Data Type  |  Result
---|:---
cast(date as date)            |  Same date value 【相同的date值】
cast(timestamp as date)       |  The year/month/day of the timestamp is determined, based on the local timezone, and returned as a date value.【timestamp的年/月/日根据当地时区确定，并作为date值返回。】
cast(string as date)          |   If the string is in the form 'YYYY-MM-DD', then a date value corresponding to that year/month/day is returned. If the string value does not match this formate, then NULL is returned.【如果string是'YYYY-MM-DD'形式，那么就返回对应的年/月/日。如果string值和这个格式不匹配，返回NULL】
cast(date as timestamp)       |   A timestamp value is generated corresponding to midnight of the year/month/day of the date value, based on the local timezone.【根据当地时区，生成一个timestamp值，对应于date值的年/月/日的午夜。】
cast(date as string)          |   The year/month/day represented by the Date is formatted as a string in the form 'YYYY-MM-DD'.【Date表示的年/月/日格式化为一个'YYYY-MM-DD'格式的string】

-------------------------------------------------------------------------

```sql
hive> desc datetest;
OK
id                      int                                         
send_tim                timestamp                                   
send_dat                date  

hive> select * from datetest;
OK
1       2021-01-22 17:03:18.809 2020-11-11

hive> select cast(send_dat as date) from datetest where id=1;
OK
2020-11-11

hive> select cast(send_tim as date) from datetest where id=1;
OK
2021-01-22

hive> select cast(send_dat as timestamp) from datetest where id=1;
OK
2020-11-11 00:00:00

hive> select cast(send_dat as string) from datetest where id=1;
OK
2020-11-11

hive> select cast("2020-11-11" as date);
OK
2020-11-11

hive> select cast("2020-11-11 13:13:12" as date);
OK
2020-11-11

hive> select cast("2020:11:11" as date);
OK
NULL
```

-------------------------------------------------------------------------

### 2.6、Intervals

见原文：[https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-Intervals](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-Intervals)

-------------------------------------------------------

```sql
-- !一定对比着原文看

-- 当前时间减一天
hive> select cast(current_timestamp() as date),cast(current_timestamp()-interval '1' day as date);
OK
2021-01-22      2021-01-21

-- 当前时间减一年
hive> select cast(current_timestamp() as date),cast(current_timestamp()-interval '1' year as date);
OK
2021-01-22      2020-01-22

-- 当前时间减一年零两个月
hive> select cast(current_timestamp() as date),cast(current_timestamp()-interval '1-2' year to month as date);
OK
2021-01-22      2019-11-22

-- 当前时间加一年零两个月
hive> select cast(current_timestamp() as date),cast(current_timestamp()-interval '-1-2' year to month as date);
OK
2021-01-22      2022-03-22

-- Support for intervals with constant numbers
-- 当前时间减一天
hive> select cast(current_timestamp() as date),cast(current_timestamp()-interval 1 day as date);
OK
2021-01-22      2021-01-21

-- 当前时间减一天两小时三分钟四秒钟。注意后面的 `day to second`
hive> select current_timestamp(),current_timestamp()-interval '1 2:3:4' day to second;
OK
2021-01-22 17:34:01.822 2021-01-21 15:30:57.822

hive> select cast(current_timestamp() as date),cast(current_timestamp()- '1' day as date);
OK
2021-01-22      2021-01-21

-- ？？？
-- INTERVAL (1+dt) DAY
```

-------------------------------------------------------

### 2.7、Decimals 

> Version:Decimal datatype was introduced in Hive 0.11.0 ([HIVE-2693](https://issues.apache.org/jira/browse/HIVE-2693)) and revised in Hive 0.13.0 ([HIVE-3976](https://issues.apache.org/jira/browse/HIVE-3976)).NUMERIC is the same as DECIMAL as of Hive 3.0.0 ([HIVE-16764](https://issues.apache.org/jira/browse/HIVE-16764)).

版本：Decimal 数据类型在 Hive 0.11.0 中被引入，并在 Hive 0.13.0 中进行了修订。
NUMERIC 与Hive 3.0.0 的 DECIMAL 相同。

> The DECIMAL type in Hive is based on Java's [BigDecimal](http://docs.oracle.com/javase/6/docs/api/java/math/BigDecimal.html) which is used for representing immutable arbitrary precision decimal numbers in Java. All regular number operations (e.g. `+`, `-`, `*`, `/`) and relevant UDFs (e.g. Floor, Ceil, Round, and many more) handle decimal types. You can cast to/from decimal types like you would do with other numeric types. The persistence format of the decimal type supports both scientific and non-scientific notation. Therefore, regardless of whether your dataset contains data like 4.004E+3 (scientific notation) or 4004 (non-scientific notation) or a combination of both, DECIMAL can be used for it.

Hive 中的 DECIMAL 类型基于 Java 的 BigDecimal，BigDecimal 在 Java 中用于表示不可变的任意精度小数。

**所有常规的数字操作(例如`+`、`-`、`*`、`/`)和相关的 UDFs(例如Floor、Ceil、Round等)处理 decimal 类型**。

你可以像处理其他数字类型一样转换为或从转换 decimal 类型。

decimal 类型的持久性格式支持科学和非科学记数法。因此，**无论你的数据集是否包含诸如 4.004E+3(科学记数法)或 4004(非科学记数法)或两者的组合，DECIMAL 都可以用于它**。

> Hive 0.11 and 0.12 have the precision of the DECIMAL type fixed and limited to 38 digits.

- Hive 0.11 和 0.12 的 DECIMAL 类型的精度是固定的，并且限制为 38 位。

> As of Hive [0.13](https://issues.apache.org/jira/browse/HIVE-3976) users can specify scale and precision when creating tables with the DECIMAL datatype using a DECIMAL(precision, scale) syntax.  If scale is not specified, it defaults to 0 (no fractional digits). If no precision is specified, it defaults to 10.

- 在 Hive 0.13 中，当使用 `DECIMAL(precision, scale)`语法创建 DECIMAL 数据类型的表时，用户可以指定数值范围和精度。如果未指定数值范围，则默认为0(无小数)。如果没有指定精度，则默认为10。

```sql
CREATE TABLE foo (
  a DECIMAL, -- Defaults to decimal(10,0)
  b DECIMAL(9, 7)
)
```

> For usage, see [Floating Point Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-FloatingPointTypes) in the Literals section below.

有关用法，请参阅下面的文字部分中的 Floating Point Types。

#### 2.7.1、Decimal Literals

> Integral literals larger than BIGINT must be handled with Decimal(38,0). The Postfix BD is required. Example:

**大于 BIGINT 的整型字面量必须使用 `Decimal(38,0)` 处理。要求后缀 `BD`**

	select CAST(18446744073709001000BD AS DECIMAL(38,0)) from my_table limit 1;

#### 2.7.2、Decimal Type Incompatibilities between Hive 0.12.0 and 0.13.0

> With the changes in the Decimal data type in Hive 0.13.0, the pre-Hive 0.13.0 columns (of type "decimal") will be treated as being of type decimal(10,0).  What this means is that existing data being read from these tables will be treated as 10-digit integer values, and data being written to these tables will be converted to 10-digit integer values before being written. To avoid these issues, Hive users on 0.12 or earlier with tables containing Decimal columns will be required to migrate their tables, after upgrading to Hive 0.13.0 or later.

随着 Hive 0.13.0 中 Decimal 数据类型的变化，Hive 0.13.0 之前的列(类型为“Decimal”)将被视为 Decimal(10,0) 类型。这意味着从这些表中读取的现有数据将被视为 10 位整数值，而写入这些表的数据在写入之前将被转换为 10 位整数值。为了避免这些问题，Hive 0.12 或更早版本的用户在升级到 Hive 0.13.0 或更高版本后，如果表中有十进制列，则需要迁移他们的表。

##### 2.7.2.1、Upgrading Pre-Hive 0.13.0 Decimal Columns

> If the user was on Hive 0.12.0 or earlier and created tables with decimal columns, they should perform the following steps on these tables after upgrading to Hive 0.13.0 or later.

**如果用户在 Hive 0.12.0 或更早版本上创建了带有 decimal 列的表，那么需要对这些表执行以下操作，升级到 Hive 0.13.0 或更高版本**。

> Determine what precision/scale you would like to set for the decimal column in the table.

1.确定你想要设置的表中的 decimal 列的精度/数值范围。

> For each decimal column in the table, update the column definition to the desired precision/scale using the [ALTER TABLE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ChangeColumnName/Type/Position/Comment) command:

2.对于表中的每一个 decimal 列，使用 ALTER table 命令将列定义更新为所期望的精度/数值范围:

```sql
ALTER TABLE foo CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
```

> If the table is not a partitioned table, then you are done.  If the table has partitions, then go on to step 3.

如果表不是一个分区表，那么就完成了升级。如果是分区表，执行步骤3。

> If the table is a partitioned table, then find the list of partitions for the table:

3.如果该表是一个分区表，那么找到该表的分区列表:

```sql
SHOW PARTITIONS foo;
  
ds=2008-04-08/hr=11
ds=2008-04-08/hr=12
...
```

> Each existing partition in the table must also have its DECIMAL column changed to add the desired precision/scale.

4.表中的每个现有分区还必须更改其 DECIMAL 列，以添加所需的精度/数值范围。

> This can be done with a single [ALTER TABLE CHANGE COLUMN](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ChangeColumnName/Type/Position/Comment) by using dynamic partitioning (available for ALTER TABLE CHANGE COLUMN in Hive 0.14 or later, with [HIVE-8411](https://issues.apache.org/jira/browse/HIVE-8411)):

这可以通过使用动态分区，使用 ALTER TABLE CHANGE COLUMN 命令来实现(这可以在Hive 0.14或更高版本中可用):

```sql
SET hive.exec.dynamic.partition = true;
  
-- hive.exec.dynamic.partition needs to be set to true to enable dynamic partitioning with ALTER PARTITION
-- This will alter all existing partitions of the table - be sure you know what you are doing!
ALTER TABLE foo PARTITION (ds, hr) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
```

> Alternatively, this can be done one partition at a time using ALTER TABLE CHANGE COLUMN, by specifying one partition per statement (This is available in Hive 0.14 or later, with [HIVE-7971](https://issues.apache.org/jira/browse/HIVE-7971).):

另外，也**可以使用 ALTER TABLE CHANGE COLUMN 在每个语句中指定一个分区**(这可以在Hive 0.14或更高版本中可用):

```sql
ALTER TABLE foo PARTITION (ds='2008-04-08', hr=11) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
ALTER TABLE foo PARTITION (ds='2008-04-08', hr=12) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
...
```

> The Decimal datatype is discussed further in [Floating Point Types](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27838462#LanguageManualTypes-FloatingPointTypes) below.

### 2.8、Union Types

> UNIONTYPE support is incomplete.The UNIONTYPE datatype was introduced in Hive 0.7.0 ([HIVE-537](https://issues.apache.org/jira/browse/HIVE-537)), but full support for this type in Hive remains incomplete. Queries that reference UNIONTYPE fields in JOIN ([HIVE-2508](https://issues.apache.org/jira/browse/HIVE-2508)), WHERE, and GROUP BY clauses will fail, and Hive does not define syntax to extract the tag or value fields of a UNIONTYPE. This means that UNIONTYPEs are effectively pass-through-only.

UNIONTYPE 的支持是不完整的。

UNIONTYPE 数据类型在 Hive 0.7.0 中引入，但是 Hive 中对该类型的完全支持仍然不完整。如果 JOIN、WHERE 和 GROUP BY 子句中引用的 UNIONTYPE 字段的查询将会失败，Hive 没有定义语法来提取 UNIONTYPE 的 tag 或 value 字段。这意味着 UNIONTYPEs 是 pass-through-only。

> Union types can at any one point hold exactly one of their specified data types. You can create an instance of this type using the create_union UDF:

Union types 在任何时候都可以精确地保存它们指定的数据类型之一。你可以使用 `create_union UDF` 创建这种类型的实例:

```sql
CREATE TABLE union_test(foo UNIONTYPE<int, double, array<string>, struct<a:int,b:string>>);
SELECT foo FROM union_test;

{0:1}
{1:2.0}
{2:["three","four"]}
{3:{"a":5,"b":"five"}}
{2:["six","seven"]}
{3:{"a":8,"b":"eight"}}
{0:9}
{1:10.0}
```

> The first part in the deserialized union is the tag which lets us know which part of the union is being used. In this example 0 means the first data_type from the definition which is an int and so on.

反序列化 union 中的第一部分是 tag，它让我们知道 union 的哪一部分正在被使用。在本例中，0 表示定义中的第一个数据类型是一个 int，依此类推。

> To create a union you have to provide this tag to the create_union UDF:

要创建一个 union，你必须为 `create_union UDF` 提供这个 tag:

```sql
SELECT create_union(0, key), create_union(if(key<100, 0, 1), 2.0, value), create_union(1, "a", struct(2, "b")) FROM src LIMIT 2;

{0:"238"}	{1:"val_238"}	{1:{"col1":2,"col2":"b"}}
{0:"86"}	{0:2.0}	{1:{"col1":2,"col2":"b"}}
```

## 3、Literals

> Floating point literals are assumed to be DOUBLE. Scientific notation is not yet supported.

假设浮点字面量为 DOUBLE。科学计数法还不支持。

### 3.1、Floating Point Types

#### 3.1.1、Decimal Types

##### 3.1.1.1、Using Decimal Types

##### 3.1.1.2、Mathematical UDFs

##### 3.1.1.3、Casting Decimal Values

##### 3.1.1.4、Testing Decimal Types

## 4、Handling of NULL Values

## 5、Change Types

## 6、Allowed Implicit Conversions