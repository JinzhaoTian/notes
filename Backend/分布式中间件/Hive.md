Hive 是建立在 [Hadoop](Hadoop.md) 上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载（ETL），这是一种可以存储、查询和分析存储在 Hadoop 中的大规模数据的机制。Hive 定义了简单的类 SQL 查询语言，称为 HQL（或者Hive SQL），它允许熟悉 SQL 的用户查询数据。同时，这个语言也允许熟悉 MapReduce 开发者的开发自定义的 mapper 和 reducer 来处理内建的 mapper 和 reducer 无法完成的复杂的分析工作。

## Hive特性

1. Hive本身不存储和计算数据，它完全依赖于HDFS和MapReduce，Hive中的表纯逻辑。
2. Hive是高延迟、结构化和面向分析的，Hive数据仓库在hadoop上是高延迟的。
3. Hive可以认为是map-reduce的一个包装，Hive的意义就是把好写的Hive的sql转换为复杂难写的map-reduce程序。由于 Hive 采用了 SQL 的查询语言 HQL，因此很容易将 Hive 理解为数据库。其实从结构上来看，Hive 和数据库除了拥有类似的查询语言，再无类似之处。



## Hive导出表数据


通过重定向方式,将查询结果写到指定的文件中:
```
hive -e "set hive.cli.print.header=false;desc dp_data_db.xxx;"  > /mnt/Data/xxx/header_xxx.csv

hive -e "set hive.cli.print.header=false;select * from dp_data_db.xxx;"  > /mnt/Data/xxx/data_xxx.csv
```

