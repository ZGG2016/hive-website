# HBaseBulkLoad

[TOC]

> This page explains how to use Hive to bulk load data into a new (empty) HBase table per [HIVE-1295](https://issues.apache.org/jira/browse/HIVE-1295). (If you're not using a build which contains this functionality yet, you'll need to build from source and make sure this patch and HIVE-1321 are both applied.)

这个页面解释了如何使用 Hive 将数据批量加载到一个新的(空的)HBase表中。

(如果你还没有使用包含这个功能的版本，你需要从源代码开始构建，并确保这个补丁和 HIVE-1321 都被应用。)

## 1、Overview

> Ideally, bulk load from Hive into HBase would be part of [HBaseIntegration](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration), making it as simple as this:

理想情况下，从 Hive 批量加载到 HBase 将是 HBaseIntegration 的一部分，使其简单如下：

```sql
CREATE TABLE new_hbase_table(rowkey string, x int, y int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:x,cf:y");
 
SET hive.hbase.bulk=true;
 
INSERT OVERWRITE TABLE new_hbase_table
SELECT rowkey_expression, x, y FROM ...any_hive_query...;
```

> However, things aren't quite as straightforward as that yet. Instead, a procedure involving a series of SQL commands is required. It should still be a lot easier and more flexible than writing your own map/reduce program, and over time we hope to enhance Hive to move closer to the ideal.

然而，事情还没有那么简单。相反，要求包含一系列 SQL 命令的 procedure。

它应该仍然比编写自己的 map/reduce 程序更容易和灵活，随着时间的推移，我们希望增强 Hive，使其更接近理想。

> The procedure is based on [underlying HBase recommendations](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/package-summary.html#bulk), and involves the following steps:

procedure 基于 HBase 底层建议，主要包括以下步骤:

> Decide how you want the data to look once it has been loaded into HBase.

- 1.决定数据加载到 HBase 后的样子。

> Decide on the number of reducers you're planning to use for parallelizing the sorting and HFile creation. This depends on the size of your data as well as cluster resources available.

- 2.决定用于并行排序和 HFile 创建的 reducers 的数量。这取决于数据的大小以及可用的集群资源。

> Run Hive sampling commands which will create a file containing "splitter" keys which will be used for range-partitioning the data during sort.

- 3.执行 Hive sampling 命令将创建一个包含 “splitter” 键的文件，该键将在排序过程中用于数据的范围分区。

> Prepare a staging location in HDFS where the HFiles will be generated.

- 4.在 HDFS 中准备一个将要生成 HFiles 的 staging 位置。

> Run Hive commands which will execute the sort and generate the HFiles.

- 5.使用 Hive 命令将进行排序并生成 HFiles。

> (Optional: if HBase and Hive are running in different clusters, distcp the generated files from the Hive cluster to the HBase cluster.)

- 6.(可选：如果 HBase 和 Hive 运行在不同的集群中，需要将 Hive 集群生成的文件 distcp 到 HBase 集群中。)

> Run HBase script loadtable.rb to move the files into a new HBase table.

- 7.执行 HBase 脚本 loadtable.rb 将文件移动到一个新的 HBase 表中。

> (Optional: register the HBase table as an external table in Hive so you can access it from there.)

- 8.(可选：在 Hive 中将 HBase 表注册为外部表，这样就可以从外部表访问它。)

> The rest of this page explains each step in greater detail.

本页面的其余部分将更详细地解释每一步。

## 2、Decide on Target HBase Schema

## 3、Estimate Resources Needed

## 4、Add necessary JARs

## 5、Prepare Range Partitioning

## 6、Prepare Staging Location

## 7、Sort Data

## 8、Run HBase Script

## 9、Map New Table Back Into Hive

## 10、Followups Needed