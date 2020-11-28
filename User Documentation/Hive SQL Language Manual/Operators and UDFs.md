# LanguageManual UDF

[TOC]

[Operators and UDFs](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)

## 1、Built-in Operators

### 1.1、Operators Precedences

### 1.2、Relational Operators

### 1.3、Arithmetic Operators

### 1.4、Logical Operators

### 1.5、String Operators

### 1.6、Complex Type Constructors

### 1.7、Operators on Complex Types

## 2、Built-in Functions

### 2.1、Mathematical Functions

#### 2.1.1、Mathematical Functions and Operators for Decimal Datatypes

### 2.2、Collection Functions

### 2.3、Type Conversion Functions

### 2.4、Date Functions

### 2.5、Conditional Functions

Return Type | Name(Signature) | Description
---|:---|:---
boolean | isnull( a ) | Returns true if a is NULL and false otherwise.
boolean | isnotnull ( a ) | Returns true if a is not NULL and false otherwise.
T | if(boolean testCondition, T valueTrue, T valueFalseOrNull) | Returns valueTrue when testCondition is true, returns valueFalseOrNull otherwise.
T | nvl(T value, T default_value) | Returns default value if value is null else returns value (as of HIve 0.11).
T | COALESCE(T v1, T v2, ...) | Returns the first v that is not NULL, or NULL if all v's are NULL.
T | CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END  | When a = b, returns c; when a = d, returns e; else returns f.
T | CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END | When a = true, returns b; when c = true, returns d; else returns e.
T | nullif( a, b )	| Returns NULL if a=b; otherwise returns a **(as of Hive 2.2.0)**.Shorthand for: CASE WHEN a = b then NULL else a
void | assert_true(boolean condition) | Throw an exception if 'condition' is not true, otherwise return null (as of Hive 0.8.0). For example, select assert_true (2<1).

```sh
hive> select * from emp;
OK
ID      Name    Salary  Designation     Dept
1201    Gopal   45000   Technical manager       TP
1202    Manisha 45000   Proofreader     PR
1203    Masthanvali     40000   Technical writer        TP
1204    Krian   40000   Hr Admin        HR
1205    Kranthi 30000   Op Admin        Admin
1206    Mike    20000   Worker  NULL
Time taken: 0.036 seconds, Fetched: 7 row(s)

hive> select name,if(dept=='TP',1,0) from emp;
OK
Name    0
Gopal   1
Manisha 0
Masthanvali     1
Krian   0
Kranthi 0
Mike    0
Time taken: 0.06 seconds, Fetched: 7 row(s)

hive> select Name,nvl(Dept,'AA')  from emp;
OK
Name    Dept
Gopal   TP
Manisha PR
Masthanvali     TP
Krian   HR
Kranthi Admin
Mike    AA
Time taken: 0.056 seconds, Fetched: 7 row(s)

hive> select assert_true(Dept) from emp;
OK
Failed with exception java.io.IOException:org.apache.hadoop.hive.ql.metadata.HiveException: ASSERT_TRUE(): assertion failed.
Time taken: 0.072 seconds
hive> select assert_true(id) from emp;
OK
NULL
NULL
NULL
NULL
NULL
NULL
NULL
Time taken: 0.046 seconds, Fetched: 7 row(s)

hive> select coalesce(null,2,3); 
OK
2
Time taken: 0.038 seconds, Fetched: 1 row(s)
```

### 2.6、String Functions

### 2.7、Data Masking Functions

### 2.8、Misc. Functions

#### 2.8.1、xpath

#### 2.8.2、get_json_object

## 3、Built-in Aggregate Functions (UDAF)

## 4、Built-in Table-Generating Functions (UDTF)

### 4.1、explode

### 4.2、posexplode

### 4.3、json_tuple

### 4.4、parse_url_tuple

## 5、GROUPing and SORTing on f(column)

## 6、UDF internals

> The context of a UDF's evaluate method is one row at a time. A simple invocation of a UDF like

**UDF 的 evaluate 方法是一次处理一行数据**。对 UDF 的简单调用：

	SELECT length(string_col) FROM table_name;

> would evaluate the length of each of the string_col's values in the map portion of the job. The side effect of the UDF being evaluated on the map-side is that you can't control the order of rows which get sent to the mapper. It is the same order in which the file split sent to the mapper gets deserialized. Any reduce side operation (such as SORT BY, ORDER BY, regular JOIN, etc.) would apply to the UDFs output as if it is just another column of the table. This is fine since the context of the UDF's evaluate method is meant to be one row at a time.

在 job 的 map 部分，计算每个 string_col 值的长度。

**在 map 端计算 UDF 的缺点是，无法控制发送到 mapper 的行的顺序。它与发送到 mapper 的文件分片被反序列化的顺序相同。**

**任何 reduce 端操作(如SORT BY、ORDER BY、正常 JOIN等)都将应用在 UDFs 输出**，就好像它只是表的另一列一样。这很好，因为 UDF 的 evaluate 方法是一次处理一行数据。

> If you would like to control which rows get sent to the same UDF (and possibly in what order), you will have the urge to make the UDF evaluate during the reduce phase. This is achievable by making use of [DISTRIBUTE BY, DISTRIBUTE BY + SORT BY, CLUSTER BY](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SortBy). An example query would be:

如果你想**控制哪些行被发送到相同的 UDF 中**(并且可能是以什么顺序)，那么你需要在 reduce 阶段执行 UDF evaluate。这可以通过使用 DISTRIBUTE BY、DISTRIBUTE by + SORT BY、CLUSTER BY 来实现。查询的一个例子是:

	SELECT reducer_udf(my_col, distribute_col, sort_col) FROM
	(SELECT my_col, distribute_col, sort_col FROM table_name DISTRIBUTE BY distribute_col SORT BY distribute_col, sort_col) t

> However, one could argue that the very premise of your requirement to control the set of rows sent to the same UDF is to do aggregation in that UDF. In such a case, using a User Defined Aggregate Function (UDAF) is a better choice. You can read more about writing a UDAF [here](https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy). Alternatively, you can user a custom reduce script to accomplish the same using [Hive's Transform functionality](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Transform). Both of these options would do aggregations on the reduce side.

但是，有人可能会说，控制发送到同一个 UDF 的行集的前提是在那个 UDF 中进行聚合操作。

在这种情况下，使用 UDAF 是更好的选择。你可以在这里阅读更多关于编写 UDAF 的内容。

或者，你也可以使用个性化的 reduce 脚本来完成和 Hive 的转换功能相同的操作。

这两种选项都会在 reduce 端做聚合。

## 7、Creating Custom UDFs

> For information about how to create a custom UDF, see [Hive Plugins](https://cwiki.apache.org/confluence/display/Hive/HivePlugins) and [Create Function](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateFunction).

	select explode(array('A','B','C'));
	select explode(array('A','B','C')) as col;
	select tf.* from (select 0) t lateral view explode(array('A','B','C')) tf;
	select tf.* from (select 0) t lateral view explode(array('A','B','C')) tf as col;