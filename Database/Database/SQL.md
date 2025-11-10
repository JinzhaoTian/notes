
> 参考：[SQL基本语法](https://www.cnblogs.com/zj420255586/p/11574614.html)

要进行一定的 Sql 训练，包括一般的sql，以及Hive Sql，Spark Sql，等等。

### 窗口函数

窗口函数，基本语法：

```SQL
<窗口函数> over (partition by <用于分组的列名>
                order by <用于排序的列名>)
```

常见的窗口函数有：

1. 专用窗口函数：包括后面要讲到的 rank，dense_rank，row_number 等专用窗口函数。
2. 聚合函数：如 sum，avg，count，max，min 等
