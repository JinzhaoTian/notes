
> 参考：[API Reference](https://spark.apache.org/docs/latest/api/python/reference/index.html)

PySpark 是 [Spark](../../Backend/分布式中间件/Spark.md) 的 Python API，使用 PySpark，您也可以使用 Python 编程语言处理 RDD。

## 操作模式

1. 交互式 PySpark shell
2. 使用 Python 运行

## 基本模块

> 参考：[API Reference](https://spark.apache.org/docs/latest/api/python/reference/index.html)

PySpark 根据 Spark 的不同模块，也设计了如下的 7 个 API 接口模块：
- **Spark Core**
- **Spark Streaming**
- **Spark SQL**
- **Structured Streaming**
- **MLlib（RDD-based）**
- **MLlib（DataFrame-based）**
- **Resource Management**

### Spark Core

> 参考：[RDD Programming Guide](https://spark.apache.org/docs/latest/rdd-programming-guide.html)

用以下代码创建存储一组单词的RDD（spark使用parallelize方法创建RDD），我们现在将对单词进行一些操作。

#### RDD创建

主要使用两种方式进行创建：

1. `sc.paralize()`：从上下文的list，或者array中创建
2. `sc.textFile()`：从文件进行创建，参数1是路径，参数2是代表数据集被划分的分区数。如果是本地文件的话，路径前面需要加上`file://`。

#### RDD常用内置方法

指的是对整个数据进行操作，因为常常要把很大的数据，处理成我们想要的格式。为了方便处理，所以设计了如下的接口：

1. `map(f, preservesPartitioning=False)`
   * 参数：
     * `f: func`：自定义行处理函数，这个函数的输入参数是RDD中的一行，返回值是处理后结果。
     * `preservesPartitioning: bool`：表示是否保留父RDD的partitioner分区信息。
   * 功能：将函数 f 应用在每一个RDD元素上，也就是对每一行做转换。
2. `reduce(f)`
   * 参数：
     * `f: func`：自定义处理函数，参数固定有两个。
   * 功能：将RDD中元素两两传递给输入函数，同时产生一个新的值，新产生的值与RDD中下一个元素再被传递给输入函数直到最后只有一个值为止。
3. `reduceByKey(f, numPartitions, partitionFunc)`
   * 参数：
     * `f: func`：自定义处理函数
     * `numPartitions`：
     * `partitionFunc`：
   * 功能：对元素为KV对的RDD中Key相同的元素的Value进行reduce，因此，Key相同的多个元素的值被reduce为一个值，然后与原RDD中的Key组成一个新的KV对。
4. `groupBy(f[, numPartitions, partitionFunc])`
   * 参数：
     * `f: func`：自定义的处理函数
     * `numPartitions`：
     * `partitionFunc`：
   * 功能：接收一个函数，这个函数返回的值作为key，然后通过这个key来对里面的元素进行分组，返回一个聚合的RDD
   * 例子：`rdd.groupBy(lambda x: x % 2).collect()`
5. `groupByKey([numPartitions, partitionFunc])`
   * 参数：
     * `numPartitions`：
     * `partitionFunc`：
   * 功能：直接将键值对类型的数据的key作为group的key 值，返回一个聚合的RDD
   * 例子：`rdd.groupByKey().mapValues(list).collect()`
6. `filter(f)`
   * 参数：
     * `f: func`：自定义行处理函数，返回值要求是布尔值。
   * 功能：过滤，在RDD中筛选符合特定条件的数据元素。
7. `flatMap(f, preservesPartitioning=False)`：
   * 参数：同map
   * 功能：与map类似，但返回的是一个扁平的结果，不是一个列表。
8. `foreach(f)`
   * 参数：
     * `f: func`：
   * 功能：对这个RDD的每个元素都使用函数f
9. `join(other, numPartitions=None)`
   * 参数：
     * `other`：另一个RDD
   * 功能：将这个RDD与另一个RDD通过key值连接，返回一个新的RDD
   * 例子：`x.join(y).collect()`
10. `collect()`
    * 功能：返回一个包含这个RDD所有元素的list
11. `count()`
    * 功能：返回这个RDD中的元素的数量
12. `distict(numPartitions=None)`：
    * 参数：
      * `numPartitions`：
    * 功能：去重，返回一个新的RDD，这个RDD只包含父RDD不同的元素。
13. `sample(withReplacement, fraction, seed=None)`：
    * 参数：
      * `withReplacement: bool`：元素是否能够重复采样
      * `fraction: float`：采样比率，必须在[0, 1]之间，采样出来的RDD大小是原来RDD大小的多少
      * `seed: int optional`：随机种子
    * 功能：生成一个此RDD的采样子集

### Spark SQL

> 参考：[Spark SQL, DataFrames and Datasets Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)



### 相关聚合函数

库是`pyspark.sql.functions`，

1. max
2. min
3. count
4. sum
5. avg
6. mean
7. sumDistinct：去重后求和
8. collect_list
9. collect_set
10. stddev
11. variance

用法：

```
from pyspark.sql.functions import collect_list

after_data = data_df.groupBy(data_df.mobile).agg(collect_list(data_df.app_code))
```

