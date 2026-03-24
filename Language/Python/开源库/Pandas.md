
> 参考：[官方API](https://pandas.pydata.org/docs/reference/index.html#api)

Pandas是python的一个数据分析包，是基于NumPy的一种工具，该工具是为了解决数据分析任务而创建的。Pandas纳入了大量库和一些标准的数据模型，提供了高效地操作大型数据集所需的工具。Pandas提供了大量能使我们快速便捷地处理数据的函数和方法。

## 数据结构

Pandas 主要有 Series（一维数组），DataFrame（二维数组），Panel（三维数组），Panel4D（四维数组），PanelND（更多维数组）等数据结构。其中 Series 和 DataFrame 应用的最为广泛。

### Series

Series 是一维带标签的数组，它可以包含任何数据类型。包括整数，字符串，浮点数，Python 对象等。Series 可以通过标签来定位。

#### 创建数据类型

1. 从列表创建 Series

   ```python
   arr = [0, 1, 2, 3, 4]
   s1 = pd.Series(arr)        # 如果不指定索引，则默认从 0 开始
   ```

   

2. 从加入索引创建 Series

   ```python
   import numpy as np
   n = np.random.randn(5)     # 创建一个随机 Ndarray 数组
   index = [‘a’, ‘b’, ‘c’, ‘d’, ‘e’]
   s2 = pd.Series(n, index=index)
   ```

   

3. 从字典创建 Series

   ```python
   d = {‘a’: 1, ‘b’: 2, ‘c’: 3, ‘d’: 4, ‘e’: 5}
   s3 = pd.Series(d)
   ```

#### 基本操作

```python
s1.index = ['A', 'B', 'C', 'D', 'E']   # 修改索引操作

s4 = s3.append(s1)                     # 纵向拼接

s4 = s4.drop('e')                      # 按指定索引删除元素

s4['A'] = 6                            # 按索引修改数值

s4['B']                                # 按指定索引查找元素

s4[:3]                                 # 切片操作
```

#### 运算操作

```python
s1.add(s2)    # 加法(Series 的加法运算是按照索引计算，如果索引不同则填充为 `NaN`（空值））
s1.sub(s2)    # 减法
s1.mul(s2)    # 乘法
s1.div(s2)    # 除法
s1.median()   # 求中位数
s1.max()      # 求最大值
s1.sum()      # 求和
```



### DataFrame

DataFrame 是二维的带标签的数据结构。我们可以通过标签来定位数据。这是 NumPy 所没有的。

#### 创建数据类型

1. 通过 NumPy 数组创建 DataFrame(三个list，分别为值，索引，列名)

   ```python
   dates = pd.date_range('today', periods=6)      # 定义时间序列作为 index
   num_arr = np.random.randn(6, 4)                # 传入 numpy 随机数组
   columns = ['A', 'B', 'C', 'D']                 # 将列表作为列名
   df1 = pd.DataFrame(num_arr, index=dates, columns=columns)
   ```

2. 通过字典来创建DataFrame

   ```python
   data = {'animal': ['cat', 'cat', 'snake', 'dog', 'dog', 'cat', 'snake', 'cat', 'dog', 'dog'],
           'age': [2.5, 3, 0.5, np.nan, 5, 2, 4.5, np.nan, 7, 3],
           'visits': [1, 3, 2, 3, 2, 3, 1, 1, 2, 1],
           'priority': ['yes', 'yes', 'no', 'yes', 'no', 'no', 'no', 'yes', 'no', 'no']}
   
   labels = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j']
   df2 = pd.DataFrame(data, index=labels)
   ```

#### 基本操作

```python
df2.head()
df2.tail(3)											# 查看前5个值和后三个值

df2.columns
df2.values
df2.index       								# 查看列名，值，以及索引

df2.describe()                  # 查看统计数据

df2.T                           # 转置操作

df2['age'，'animal']            # 通过标签查询

df2.iloc[1:3]                   # 按行查询

df3.iat[1,0]                    # 按照坐标查询

df3.loc['f','age']              # 按照标签和索引进行查询

num = pd.Series([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], index=df3.index)
df3['No.'] = num                # 添加列数据（创建一个series，然后添加）

data.drop('m1',axis=1) 			    # 丢弃数据, 相当于delete table a where yid='m1'
data.drop(['a','c'])  			    # 相当于delete table a where xid='a' or xid='c'
df5.dropna(how = 'any')  	      # 删除缺失值所在的行

df4.fillna(value=3)             # 缺失值处理

df[df['age'] > 3]
df[(df['animal'] == 'cat') & (df['age'] < 3)] 
df3[df3['animal'].isin(['cat', 'dog'])]                # 条件查找 df['条件']

df.iloc[2:4, 1:3]                                      # 行列索引切片

df.sort_values(by=['age', 'visits'], ascending=[False, True])    # 排序操作

df['priority'].map({'yes': True, 'no': False})         # yes替换为True，no为False, DataFrame 多值替换

df4.groupby('animal').sum()      # 分组操作
```



#### 文件操作

```python
df3.to_csv('animal.csv')   							# CSV 文件写入

df_animal = pd.read_csv('animal.csv')   # CSV 文件读取

df3.to_excel('animal.xlsx', sheet_name='Sheet1')  											 # Excel 写入操作

pd.read_excel('animal.xlsx', 'Sheet1', index_col=None, na_values=['NA']) # Excel 读取操作

s1.median() 		# 求中位数

s1.max() 				# 求最大值

s1.sum() 				# 求和
```

