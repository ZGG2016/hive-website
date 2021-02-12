# Hive Tutorial

[TOC]

## 1、Concepts

### 1.1、What Is Hive

> Hive is a data warehousing infrastructure based on [Apache Hadoop](http://hadoop.apache.org/). Hadoop provides massive scale out and fault tolerance capabilities for data storage and processing on commodity hardware.

Hive是一个基于 Apache Hadoop 的数据仓库基础设施。Hadoop 为在商业硬件上存储和处理数据提供了大规模的向外扩展和容错能力。

> Hive is designed to enable easy data summarization, ad-hoc querying and analysis of large volumes of data. It provides SQL which enables users to do ad-hoc querying, summarization and data analysis easily. At the same time, Hive's SQL gives users multiple places to integrate their own functionality to do custom analysis, such as User Defined Functions (UDFs).  

Hive 可以方便地对大量数据进行汇总、ad-hoc 查询和分析。

它提供了 SQL，使用户能够轻松地进行 ad-hoc 查询、汇总和数据分析。同时，Hive 的 SQL 给用户提供了多个地方来集成他们自己的功能来做自定义分析，比如用户定义函数(UDFs)。

### 1.2、What Hive Is NOT

> Hive is not designed for online transaction processing.  It is best used for traditional data warehousing tasks.

Hive 不是为在线事务处理而设计的。它最适合用于传统的数据仓库任务。

### 1.3、Getting Started

> For details on setting up Hive, HiveServer2, and Beeline, please refer to the GettingStarted guide.

关于 Hive、HiveServer2 和 Beeline 的详细设置，请参考 GettingStarted guide。

> Books about Hive lists some books that may also be helpful for getting started with Hive.

关于 Hive 的书中列出了一些可能对开始使用 Hive 有帮助的书。

> In the following sections we provide a tutorial on the capabilities of the system. We start by describing the concepts of data types, tables, and partitions (which are very similar to what you would find in a traditional relational DBMS) and then illustrate the capabilities of Hive with the help of some examples.

在下面的章节中，我们将提供关于该系统功能的教程。我们首先描述数据类型、表和分区的概念(与传统关系 DBMS 非常相似)，然后通过一些例子说明 Hive 的功能。

### 1.4、Data Units

> In the order of granularity - Hive data is organized into:

hive 数据有如下组织形式：

> Databases: Namespaces function to avoid naming conflicts for tables, views, partitions, columns, and so on.  Databases can also be used to enforce security for a user or group of users.

- 数据库：名称空间函数，以避免表、视图、分区、列等的命名冲突。数据库还可以用于为一个用户或一组用户实施安全性。

> Tables: Homogeneous units of data which have the same schema. An example of a table could be page_views table, where each row could comprise of the following columns (schema):

> timestamp—which is of INT type that corresponds to a UNIX timestamp of when the page was viewed.
userid —which is of BIGINT type that identifies the user who viewed the page.
page_url—which is of STRING type that captures the location of the page.
referer_url—which is of STRING that captures the location of the page from where the user arrived at the current page.
IP—which is of STRING type that captures the IP address from where the page request was made.

- 表：具有相同schema的同种数据单元。一个表的示例就是 page_views 表，表中每行都由下面的列组成：

	- timestamp：INT类型，页面浏览时间
	- userid：BIGINT类型，
	- page_url：STRING类型，
	- referer_url：STRING类型，
	- IP：STRING类型，

> Partitions: Each Table can have one or more partition Keys which determines how the data is stored. Partitions—apart from being storage units—also allow the user to efficiently identify the rows that satisfy a specified criteria; for example, a date_partition of type STRING and country_partition of type STRING. Each unique value of the partition keys defines a partition of the Table. For example, all "US" data from "2009-12-23" is a partition of the page_views table. Therefore, if you run analysis on only the "US" data for 2009-12-23, you can run that query only on the relevant partition of the table, thereby speeding up the analysis significantly. Note however, that just because a partition is named 2009-12-23 does not mean that it contains all or only data from that date; partitions are named after dates for convenience; it is the user's job to guarantee the relationship between partition name and data content! Partition columns are virtual columns, they are not part of the data itself but are derived on load.

- 分区：

每个表有一个或多个分区 key ，这些 key 决定了数据如何被存储。

**除了作为存储单元之外，分区还允许用户有效地标识满足指定条件的行**。例如，STRING 类型的 date_partition 和 STRING 类型的 country_partition。

**每个唯一的分区 key 对应表的一个分区**。例如，从"2009-12-23"开始的所有"US"下的数据都是 page_views 表的一个分区中的数据。

因此，如果基于"US"下2009-12-23的数据分析，你可以只在相关分区下执行查询。从而，可以**提高分析效率**。

然而，**仅仅因为一个分区命名为2009-12-23，并不意味着它包含或仅包含该日期的所有数据，分区以日期命名是为了方便。**

保证分区名和数据内容之间的关系是用户的工作!**分区列是虚拟列，它们不是数据本身的一部分**，而是在加载时派生的。

> Buckets (or Clusters): Data in each partition may in turn be divided into Buckets based on the value of a hash function of some column of the Table. For example the page_views table may be bucketed by userid, which is one of the columns, other than the partitions columns, of the page_view table. These can be used to efficiently sample the data.

- 分桶:

**通过计算表的某些列的 hash 值，分区中的数据再被划分到桶中。这可以被用来高效地抽样数据。**

例如，page_views 表根据 userid 分桶，**userid 是 page_view 表的列之一，而不是分区列**。

> Note that it is not necessary for tables to be partitioned or bucketed, but these abstractions allow the system to prune large quantities of data during query processing, resulting in faster query execution.

分区或分桶并不是必要的。但是这些抽象允许系统在查询处理期间删除大量数据，从而加快查询的执行。

### 1.5、Type System

> Hive supports primitive and complex data types, as described below. See [Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types) for additional information.

Hive 支持基本和复杂数据类型，如下所述。有关更多信息，请参阅 Hive Data Types。

#### 1.5.1、Primitive Types

- Types are associated with the columns in the tables. The following Primitive types are supported:【类型和表中的列相关，下面是支持的基本数据类型：】

- Integers

	- TINYINT—1 byte integer
	- SMALLINT—2 byte integer
	- INT—4 byte integer
	- BIGINT—8 byte integer

- Boolean type
	
	- BOOLEAN—TRUE/FALSE

- Floating point numbers

	- FLOAT—single precision
	- DOUBLE—Double precision

- Fixed point numbers

	- DECIMAL—a fixed point value of user defined scale and precision

- String types

	- STRING—sequence of characters in a specified character set
	- VARCHAR—sequence of characters in a specified character set with a maximum length
	- CHAR—sequence of characters in a specified character set with a defined length

- Date and time types

	- TIMESTAMP — A date and time without a timezone ("LocalDateTime" semantics)
	- TIMESTAMP WITH LOCAL TIME ZONE — A point in time measured down to nanoseconds ("Instant" semantics)
	- DATE—a date

- Binary types

	- BINARY—a sequence of bytes

> The Types are organized in the following hierarchy (where the parent is a super type of all the children instances):

这些类型按以下层次结构组织(父实例是所有子实例的超类型)：

- Type
	- Primitive Type
		- Number
			- DOUBLE
				- FLOAT
					- BIGINT
						- INT
							- SMALLINT
								- TINYINT

				- STRING

			- BOOLEAN

> This type hierarchy defines how the types are implicitly converted in the query language. Implicit conversion is allowed for types from child to an ancestor. So when a query expression expects type1 and the data is of type2, type2 is implicitly converted to type1 if type1 is an ancestor of type2 in the type hierarchy. Note that the type hierarchy allows the implicit conversion of STRING to DOUBLE.

这种类型层次结构定义了如何在查询语言中隐式地转换类型。

允许从子类型到祖先类型的隐式转换。

因此，当查询表达式期望类型1且数据为类型2时，如果在类型层次结构中，类型1是类型2的祖先，则类型2将隐式转换为类型1。

请注意，类型层次结构允许隐式地将 STRING 转换为 DOUBLE。

> Explicit type conversion can be done using the cast operator as shown in the [#Built In Functions](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-BuiltInFunctions) section below.

显式类型转换可以使用强制转换操作符完成，如下面的 #Built in Functions 一节所示。

#### 1.5.2、Complex Types

> Complex Types can be built up from primitive types and other composite types using:

复杂类型可以从基本类型和其他组合类型构建：

> Structs: the elements within the type can be accessed using the DOT (.) notation. For example, for a column c of type STRUCT {a INT; b INT}, the a field is accessed by the expression c.a

- Structs：类型中的元素可以使用点号访问。例如，c 列的类型是 `STRUCT {a INT; b INT}`，通过 `c.a` 访问字段。

> Maps (key-value tuples): The elements are accessed using ['element name'] notation. For example in a map M comprising of a mapping from 'group' -> gid the gid value can be accessed using M['group']

- Maps（键值元组）：使用 `['element name']` 访问元素。例如，映射 M 由 `'group' -> gid` 组成，可以通过 `M['group']` 访问 gid 值。

> Arrays (indexable lists): The elements in the array have to be in the same type. Elements can be accessed using the [n] notation where n is an index (zero-based) into the array. For example, for an array A having the elements ['a', 'b', 'c'], A[1] retruns 'b'.

- Arrays（可索引的列表）：数组中的元素必须具有相同的类型。可以使用 [n] 来访问元素，n 是索引（从0开始）。例如，数组 A 有元素 `['a', 'b', 'c']`，那么 A[1] 将返回 'b'。

> Using the primitive types and the constructs for creating complex types, types with arbitrary levels of nesting can be created. For example, a type User may comprise of the following fields:

使用基本类型和用于创建复杂类型的构造，可以创建具有任意嵌套级别的类型。例如，一个类型用户可能包含以下字段:

- gender—which is a STRING.
- active—which is a BOOLEAN.

#### 1.5.3、Timestamp

> Timestamps have been the source of much confusion, so we try to document the intended semantics of Hive.

Timestamps 一直是很多困惑的根源，所以我们试图记录 Hive 的语义。

**Timestamp ("LocalDateTime" semantics)**

> Java's "LocalDateTime" timestamps record a date and time as year, month, date, hour, minute, and seconds without a timezone. These timestamps always have those same values regardless of the local time zone.

Java 的 “LocalDateTime” 时间戳将日期和时间记录为年、月、日、时、分和秒，而没有时区。

无论本地时区是什么，这些时间戳总是具有相同的值。

> For example, the timestamp value of "2014-12-12 12:34:56" is decomposed into year, month, day, hour, minute and seconds fields, but with no time zone information available. It does not correspond to any specific instant. It will always be the same value regardless of the local time zone. Unless your application uses UTC consistently, timestamp with local time zone is strongly preferred over timestamp for most applications. When users say an event is at 10:00, it is always in reference to a certain timezone and means a point in time, rather than 10:00 in an arbitrary time zone.

例如，“2014-12-12 12:34:56” 的时间戳值被分解为年、月、日、小时、分钟和秒字段，但是没有时区信息可用。

它不对应于任何特定的时刻。它将始终是相同的值，无论当地时区是什么。除非你的应用程序一致使用 UTC，否则具有当地时区的时间戳对于大多数应用程序来说都比时间戳更受欢迎。当用户说一个事件在 10:00 时，它总是与某个时区有关，意思是一个时间点，而不是任意时区中的 10:00。

**Timestamp with local time zone ("Instant" semantics)**

> Java's "Instant" timestamps define a point in time that remains constant regardless of where the data is read. Thus, the timestamp will be adjusted by the local time zone to match the original point in time.

Java 的 "Instant" 时间戳定义了一个无论从何处读取数据都保持不变的时间点。因此，时间戳将根据当地时区调整，以匹配原始的时间点。

Type | Value in America/Los_Angeles | Value in America/New York
---|:---|:---
timestamp | 2014-12-12 12:34:56 | 2014-12-12 12:34:56
timestamp with local time zone | 2014-12-12 12:34:56 | 2014-12-12 15:34:56

**Comparisons with other tools**

见原文：[https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-BuiltInOperatorsandFunctions](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-BuiltInOperatorsandFunctions)

### 1.6、Built In Operators and Functions

### 1.7、Language Capabilities

## 2、Usage and Examples

> NOTE: Many of the following examples are out of date.  More up to date information can be found in the [LanguageManual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual).

### 2.1、Creating, Showing, Altering, and Dropping Tables

### 2.2、Loading Data

### 2.3、Querying and Inserting Data