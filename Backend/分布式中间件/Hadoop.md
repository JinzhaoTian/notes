Hadoop 起源于 2003 年，Google发表了一篇技术学术论文，公开介绍了自己的谷歌文件系统**GFS（Google File System）**，这是 Google 公司为了存储海量搜索数据而设计的专用文件系统。2004年，Doug Cutting 基于 Google 的 GFS 论文，实现了**分布式文件存储系统**，并将它命名为**NDFS（Nutch Distributed File System）**。

同年 2004 年，Google 又发表了一篇技术学术论文，介绍自己的 [MapReduce](MapReduce.md) 编程模型。这个编程模型，用于大规模数据集（大于 1 TB）的并行分析运算。第二年（2005年），Doug Cutting 又基于 MapReduce ，在 Nutch 搜索引擎实现了该功能。

2006 年，加盟 Yahoo 之后，Doug Cutting 将 NDFS 和 MapReduce 进行了升级改造，并重新命名为**Hadoop**（NDFS 也改名为 HDFS ，Hadoop Distributed File System）。同年，Google 又发论文介绍了自己的**BigTable**，这是一种分布式数据存储系统，一种用来处理海量数据的非关系型数据库。Doug Cutting 当然没有放过，在自己的 Hadoop 系统里面，引入了 BigTable ，并命名为HBase。

Hadoop的**核心架构**，就是 HDFS 和 MapReduce ，HDFS 为海量数据提供了存储，而 MapReduce 为海量数据提供了计算框架。

## Hadoop能干什么？

我一直对 Hadoop 能干什么存在一些模糊的认知，下面我应该理清一下思路。首先，**Hadoop就是存储海量数据和分析海量数据的工具**。既然是数据存储，那就需要一个文件系统进行存储，对应的是 HDFS 。如果想对数据进行查询和处理就要有对应的接口，这里对应了 MapReduce 。由于 MapReduce 是用一种函数接口形式实现的，如果也想像单机的关系数据库一样用 sql 的话，可以使用 Hive 。此外 MapReduce 最主要的功能是数据处理，而且是海量数据的处理，比如把所有数据的某个字段更新。实际应用场景有以下几个部分：

1. 大数据存储：分布式存储
2. 日志处理：擅长日志分析
3. ETL：数据抽取到 oracle、mysql、DB2、mongdb 及主流数据库
4. 机器学习: 比如 Apache Mahout 项目
5. 搜索引擎：Hadoop + lucene 实现
6. 数据挖掘：目前比较流行的广告推荐，个性化广告推荐

## 安装

### Docker



### 手动安装

Hadoop 一般是在服务器集群安装，然后客户端远程连接，所以服务器端需要的预先准备的是**ssh服务端程序**。同时 Hadoop 是用 Java 编写的，所以必须要先**安装JDK**。这些准备完毕之后，就可以安装 Hadoop 了，首先可以去[官网](https://hadoop.apache.org/)下载安装包，把安装包上传到服务器端之后再解压到一个文件夹下，这个文件夹究竟是哪个我还不清楚，有说是`/opt/software/`下，也有说是`/usr/local`下。

```bash
$ sudo tar -zxvf hadoop-3.2.2.tar.gz -C /usr/local
```

然后给文件夹改个名：

```bash
$ sudo mv ./hadoop-3.2.2/ ./hadoop
```

#### 单机模式

单机模式直接解压就可以使用，但是分布式模式是需要再修改一下配置信息的。

可以查看一下hadoop版本信息：

```bash
$ cd /usr/local/hadoop/bin
$ ./hadoop version
```

查看自带的例子：

```bash
$ ./hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.2.jar
```



