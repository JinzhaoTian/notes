LINQ（语言集成查询）是一系列直接将查询功能集成到 C# 语言的技术统称。数据查询历来都表示为简单的字符串，没有编译时类型检查或 IntelliSense 支持，需要针对每种类型的数据源了解不同的查询语言：SQL 数据库、XML 文档、各种 Web 服务等。然而，LINQ 的出现改变了这一现状，它使查询成为了与类、方法和事件同等重要的高级语言构造。通过 LINQ，开发者能够以声明性的方式查询和操作数据，极大地提高了开发效率和代码的可维护性。


## 特性

1. 强类型：编译时验证查询逻辑，减少运行时错误。
2. 延迟执行：LINQ 查询通常是延迟执行的，即查询表达式本身不会立即执行，直到实际遍历结果时才触发查询。使用 `Tolist()`、 `ToArray()`、 `ToDictionary()`、 `FirstorDefault()` 等方法可立即执行。
3. 支持多种数据源：LINQ 可以用于查询多种数据源，如 `LINQ to Objects`、`LINQ to XML`、`LINQ to SQL`、 `LINQ to Entities`（Entity Framework）等。

## 方法

1. `Where()`：用于过滤集合中的元素，通过一个谓词(返回布尔值的条件)筛选集合中的元素，生成一个仅包含满足条件元素的新序列。
2. `Select()`：用于将集合中的每个元素投影(转换)为新序列。
3. `SelectMany()`：用于将多个集合(嵌套集合，如集合的集合)展平为一个集合。
4. `ToList()`：将实现了 `IEnumerable<T>` 接口的集合转换为一个 `List<T>` 类型的对象，属于将集合转换为特定类型列表的方法。
5. `ToArray()`：将一个实现了 `IEnumerable<T>` 接口的集合转换为一个数组，属于将集合转换为数组类型的方法。
6. `ToDictionary()`：将-个 `IEnumerable<T>` 集合转换为一个 `Dictionary<Tkey,TValue>` 键值对集合(字典)的方法，注意ToDictionary要求键唯一，否则抛出异常。
7. `ToLookup()`：将-个 `IEnumerable<T>` 集合转换为一个泛型 `Lookup<TKey,TElement>`， `Lookup<TKey,TElement>` 一个一对多字典，用于将键映射到值的集合。