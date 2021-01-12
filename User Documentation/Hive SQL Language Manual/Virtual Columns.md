# LanguageManual VirtualColumns

[TOC]

## 1、Virtual Columns

> Hive 0.8.0 provides support for two virtual columns:

Hive 0.8.0 提供了对两个虚拟列的支持：

- INPUT__FILE__NAME：一个 mapper 任务的输入文件的名字。

- BLOCK__OFFSET__INSIDE__FILE：当前全局文件的位置。对于块压缩文件，它是当前块的文件偏移量，即当前块的第一个字节的文件偏移量。

> One is INPUT__FILE__NAME, which is the input file's name for a mapper task.

> the other is BLOCK__OFFSET__INSIDE__FILE, which is the current global file position.

> For block compressed file, it is the current block's file offset, which is the current block's first byte's file offset.

> Since Hive 0.8.0 the following virtual columns have been added:

从 Hive 0.8.0 开始，添加了如下的虚拟列：

- ROW__OFFSET__INSIDE__BLOCK

- RAW__DATA__SIZE

- ROW__ID

- GROUPING__ID

> It is important to note, that all of the virtual columns listed here cannot be used for any other purpose (i.e. table creation with columns having a virtual column will fail with "SemanticException Error 10328: Invalid column name..")

重要的是，要注意：这里列出的所有虚拟列不能用于任何其他目的

(例如，创建表具有虚拟列的列将失败，产生"SemanticException Error 10328: Invalid column name..")

### 1.1、Simple Examples

```sql
select INPUT__FILE__NAME, key, BLOCK__OFFSET__INSIDE__FILE from src;

select key, count(INPUT__FILE__NAME) from src group by key order by key;

select * from src where BLOCK__OFFSET__INSIDE__FILE > 12000 order by key;
```

----------------------------------------------------

```sh
[root@zgg ~]# hadoop fs -ls /user/hive/warehouse/employees
Found 5 items
-rw-r--r--   1 root supergroup          0 2020-11-27 15:55 /user/hive/warehouse/employees/_SUCCESS
-rw-r--r--   1 root supergroup         27 2020-11-27 15:54 /user/hive/warehouse/employees/part-m-00000_copy_1
-rw-r--r--   1 root supergroup          0 2020-11-27 15:54 /user/hive/warehouse/employees/part-m-00001_copy_1
-rw-r--r--   1 root supergroup         27 2020-11-27 15:54 /user/hive/warehouse/employees/part-m-00002_copy_1
-rw-r--r--   1 root supergroup         13 2020-11-27 15:55 /user/hive/warehouse/employees/part-m-00003_copy_1

[root@zgg conf]# hadoop fs -cat /user/hive/warehouse/employees/part-m-00000_copy_1
10001,Georgi
10002,Bezalel

[root@zgg conf]# hadoop fs -cat /user/hive/warehouse/employees/part-m-00001_copy_1

[root@zgg conf]# hadoop fs -cat /user/hive/warehouse/employees/part-m-00002_copy_1
10005,Kyoichi
10006,Anneke

[root@zgg conf]# hadoop fs -cat /user/hive/warehouse/employees/part-m-00003_copy_1
10009,Georgi

hive> select * from employees;
OK
10001   Georgi
10002   Bezalel
10005   Kyoichi
10006   Anneke
10009   Georgi
Time taken: 2.261 seconds, Fetched: 5 row(s)

hive> desc  employees;
OK
emp_no                  int                                         
first_name              string                                      
Time taken: 0.16 seconds, Fetched: 2 row(s)

# 这行数据在哪个文件中，这个文件的名称
hive> select emp_no,INPUT__FILE__NAME from employees;
OK
10001   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00000_copy_1
10002   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00000_copy_1
10005   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00002_copy_1
10006   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00002_copy_1
10009   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00003_copy_1
Time taken: 0.14 seconds, Fetched: 5 row(s)


[root@zgg ~]# hadoop fs -cat /user/hive/warehouse/employees/part-m-00000_copy_1
10001,Georgi
10002,Bezalel

# 块内偏移量
# 如果是RCFile或者是SequenceFile块压缩格式文件，则显示Block file Offset，也就是当前快在文件的第一个字偏移量
# 如果是TextFile，显示当前行的第一个字节在文件中的偏移量
hive> select emp_no,BLOCK__OFFSET__INSIDE__FILE from employees;
OK
10001   0
10002   13
10005   0
10006   14
10009   0
Time taken: 0.227 seconds, Fetched: 5 row(s)

# 行偏移量
# 需要设置 hive.exec.rowoffset=true; 启用 
# RCFile和SequenceFile显示row number, textfile显示为0
hive> select emp_no,ROW__OFFSET__INSIDE__BLOCK from employees;
OK
10001   0
10002   0
10005   0
10006   0
10009   0
Time taken: 0.192 seconds, Fetched: 5 row(s)

hive> select emp_no,INPUT__FILE__NAME,BLOCK__OFFSET__INSIDE__FILE,ROW__OFFSET__INSIDE__BLOCK from employees;
OK
10001   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00000_copy_1       0       0
10002   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00000_copy_1       13      0
10005   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00002_copy_1       0       0
10006   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00002_copy_1       14      0
10009   hdfs://zgg:9000/user/hive/warehouse/employees/part-m-00003_copy_1       0       0
Time taken: 0.164 seconds, Fetched: 5 row(s)
```

```sh
[root@zgg conf]# hadoop fs -ls /user/hive/warehouse/apps
Found 1 items
-rw-r--r--   1 root supergroup        254 2020-12-20 16:40 /user/hive/warehouse/apps/apps.txt

hive> select * from apps;
OK
1       'QQ APP'        'http://im.qq.com/'     'CN'
2       '微博 APP'      'http://weibo.com/'     'CN'
3       '淘宝 APP'      'https://www.taobao.com/'       'CN'
4       'FACEBOOK APP'  'https://www.facebook.com/'     'USA'
5       'GOOGLE'        'https://www.google.com/'       'USA'
6       'LINE'  'https://www.line.com/' 'JP'
Time taken: 0.249 seconds, Fetched: 6 row(s)

hive> desc apps;
OK
id                      int                                         
app_name                string                                      
url                     string                                      
country                 string                                      
Time taken: 0.131 seconds, Fetched: 4 row(s)

hive> select id,INPUT__FILE__NAME,BLOCK__OFFSET__INSIDE__FILE,ROW__OFFSET__INSIDE__BLOCK from apps;
OK
1       hdfs://zgg:9000/user/hive/warehouse/apps/apps.txt       0       0
2       hdfs://zgg:9000/user/hive/warehouse/apps/apps.txt       36      0
3       hdfs://zgg:9000/user/hive/warehouse/apps/apps.txt       76      0
4       hdfs://zgg:9000/user/hive/warehouse/apps/apps.txt       122     0
5       hdfs://zgg:9000/user/hive/warehouse/apps/apps.txt       173     0
6       hdfs://zgg:9000/user/hive/warehouse/apps/apps.txt       216     0
Time taken: 0.215 seconds, Fetched: 6 row(s)
hive> 

hive> select country,count(*),GROUPING__ID from apps group by country grouping sets(country);
....
OK
'CN'    3       0
'JP'    1       0
'USA'   2       0

# 指定号码 
hive> select country,count(*),1 as GROUPING__ID from apps group by country;
....
OK
'CN'    3       1
'JP'    1       1
'USA'   2       1
Time taken: 75.912 seconds, Fetched: 3 row(s)

# 在一个GROUP BY查询中，根据不同的维度组合进行聚合，等价于将不同维度的GROUP BY结果集进行UNION ALL。
# GROUPING__ID，表示结果属于哪一个分组集合。
# 等价于：
# select id,NULL,count(*),1 as GROUPING__ID from apps group by id
# union all
# select NULL,country,count(*), 2 as GROUPING__ID from apps group by country;
hive> select id,country,count(*),GROUPING__ID from apps group by id,country grouping sets(id,country,(id,country));
....
NULL    'CN'    3       2
NULL    'JP'    1       2
NULL    'USA'   2       2
1       NULL    1       1
1       'CN'    1       0
2       NULL    1       1
2       'CN'    1       0
3       NULL    1       1
3       'CN'    1       0
4       NULL    1       1
4       'USA'   1       0
5       NULL    1       1
5       'USA'   1       0
6       NULL    1       1
6       'JP'    1       0
Time taken: 84.166 seconds, Fetched: 15 row(s)
```