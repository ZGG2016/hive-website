# Views

[TOC]

## 1、Use Cases

> Views ([http://issues.apache.org/jira/browse/HIVE-972](http://issues.apache.org/jira/browse/HIVE-972)) are a standard DBMS feature and their uses are well understood. A typical use case might be to create an interface layer with a consistent entity/attribute naming scheme on top of an existing set of inconsistently named tables, without having to cause disruption due to direct modification of the tables. More advanced use cases would involve predefined filters, joins, aggregations, etc for simplifying query construction by end users, as well as sharing common definitions within ETL pipelines.

视图是一种标准的 DBMS 特性，它们的用途是很容易理解的。

一个典型的用例可能是在现有的一组命名不一致的表上创建一个具有一致实体/属性命名方案的接口层，而不必由于直接修改表而造成中断。

更高级的用例包括预定义的过滤器、连接、聚合等，以简化最终用户的查询构造，以及在 ETL 管道中共享公共定义。

## 2、Scope

> At a minimum, we want to

至少，我们想这样做

> add queryable view support at the SQL language level (specifics of the scoping are under discussion in the Issues section below)

- 在 SQL 语言级别添加可查询视图支持

	- 不支持可更新视图

> updatable views will not be supported (see the [Updatable Views](https://cwiki.apache.org/confluence/display/Hive/UpdatableViews) proposal)
> make sure views and their definitions show up anywhere tables can currently be enumerated/searched/described

- 确保视图及其定义显示在当前可以枚举/搜索/描述表的任何位置

> where relevant, provide additional metadata to allow views to be distinguished from tables

- 在必要的情况下，提供额外的元数据，以便将视图与表区分开来

> Beyond this, we may want to

除此之外，我们可能还想

> expose metadata about view definitions and dependencies (at table-level or column-level) in a way that makes them consumable by metadata-driven tools

- 公开关于视图定义和依赖项的元数据(表级或列级)，使元数据驱动的工具能够使用这些元数据。任何位置

## 3、Syntax

	CREATE VIEW [IF NOT EXISTS] view_name [(column_name [COMMENT column_comment], ...) ]
	[COMMENT table_comment]
	AS SELECT ...
	 
	DROP VIEW view_name

## 4、Implementation Sketch

> The basics of view implementation are very easy due to the fact that Hive already supports subselects in the FROM clause.

由于 Hive 已经在 FROM 子句中支持了子选择，所以基本的视图实现非常简单。

> For CREATE VIEW v AS view-def-select, we extend SemanticAnalyzer to behave similarly to CREATE TABLE t AS select, except that we don't actually execute the query (we stop after plan generation). It's necessary to perform all of plan generation (even though we're not actually going to execute the plan) since currently some validations such as type compatibility-checking are only performed during plan generation. After successful validation, the text of the view is saved in the metastore (the simplest approach snips out the text from the parser's token stream, but this approach introduces problems described in the issues section below).

对于 `CREATE VIEW v AS view-def-select`，我们扩展了 SemanticAnalyzer，使其行为类似于 `CREATE TABLE t AS select`，只是我们实际上不执行查询(在计划生成之后停止)。

有必要执行所有的计划生成(即使我们实际上不打算执行计划)，因为目前一些验证，例如类型兼容性检查只在计划生成期间执行。

在成功验证之后，视图的文本被保存在 metastore 中(最简单的方法从解析器的令牌流中剪掉文本，但是这种方法会引入下面的问题一节中描述的问题)。

> For select ... from view-reference, we detect the view reference in SemanticAnalyzer.getMetaData, load the text of its definition from the metastore, parse it back into an AST, prepare a QBExpr to hold it, and then plug this into the referencing query's QB, resulting in a tree equivalent to select ... from (view-def-select); plan generation can then be carried out on the combined tree.

对 select ... from view-reference，我们在 `SemanticAnalyzer.getMetaData` 中检测视图引用，从 metastore 加载其定义的文本，将其解析回 AST，准备一个 QBExpr 来保存它，然后将其插入到引用查询的 QB 中，生成一个与 `select ... from (view-def-select);` 等价的树，然后可以在合并后的树上执行计划生成。

## 5、Issues

> Some of these are related to functionality/scope; others are related to implementation approaches. Opinions are welcome on all of them.

其中一些与功能/范围有关；其他则与实施方法有关。欢迎对所有这些问题发表意见。

### 5.1、Stored View Definition

### 5.2、Metastore Modeling

### 5.3、Dependency Tracking

### 5.4、Dependency Invalidation

### 5.5、View Modification

### 5.6、Fast Path Execution

### 5.7、ORDER BY and LIMIT in view definition

### 5.8、Underlying Partition Dependencies

## 6、Metastore Upgrades

### 6.1、Automatic ALTER TABLE

### 6.2、Explicit ALTER TABLE

### 6.3、Existing Row UPDATE