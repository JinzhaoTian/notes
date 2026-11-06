# Decision Tree

决策树是一种十分常用的分类方法，需要监管学习（Supervised Learning），监管学习就是给出一堆样本，每个样本都有一组属性和一个分类结果，也就是分类结果已知，那么通过学习这些样本得到一个决策树，这个决策树能够对新的数据给出正确的分类。

决策树是一种树形结构，其中每个内部节点表示一个属性上的判断，每个分支代表一个判断结果的输出，最后每个叶节点代表一种分类结果。

决策树的生成主要分以下两步：

1. 节点的分裂：一般当一个节点所代表的属性无法给出判断时，则选择将这一节点分成2个

   子节点。

2. 阈值的确定：选择适当的阈值使得分类错误率 （Training Error）最小。



## 决策树的生成算法

决策树的生成算法有ID3, C4.5和C5.0等。

### ID3

由增熵（Entropy）原理来决定哪个做父节点，哪个节点需要分裂。对于一组数据，熵越小说明分类结果越好。当Entropy最大为1的时候，是分类效果最差的状态（因为各种状态发生的概率是完全相等的），当它最小为0的时候，是完全分类的状态。因为熵等于零是理想状态，一般实际情况下，熵介于0和1之间。
$$
Entropy(t) = - \sum^{c-1}_{i=0} p(i|t)log_2p(i|t)
$$
其中，$p(i|t)$​ 代表节点 t 为分类 i 的概率。



### C4.5

ID3存在一个问题，那就是越细小的分割分类错误率越小，所以ID3会越分越细，会有过拟合的问题（Overfitting）。C4.5对ID3进行了改进，C4.5中，优化项要除以分割太细的代价，这个比值叫做信息增益率，显然分割太细分母增加，信息增益率会降低。除此之外，其他的原理和ID3相同。
$$
Gain(D, a) = Entropy(D)-\sum^k_{i=1} \frac{|D_i|}{|D|} Entropy(D_i)
$$
即，划分之前的信息熵与给定一个特征a划分样本之后的信息熵之差。



### CART

#### 原理

**ID3 和 C4.5 算法可以生成二叉树或多叉树，而 CART 只支持二叉树**。CART是一个二叉树，也是回归树，同时也是分类树，CART的构成简单明了。CART只能将一个父节点分为2个子节点。CART用**GINI指数**来决定如何分裂。
$$
Gini(p) = \sum^{K}_{k = 1} p_k(1 - p_k) = 1 - \sum^{K}_{k = 1}p_k^2
$$
其中，$p_k$​ 是样本点属于第k类的概率。

>  GINI指数：总体内包含的类别越杂乱，GINI指数就越大（跟熵的概念很相似）。

CART还是一个回归树，回归解析用来决定分布是否终止。理想地说每一个叶节点里都只有一个类别时分类应该停止，但是很多数据并不容易完全划分，或者完全划分需要很多次分裂，必然造成很长的运行时间，所以CART可以对每个叶节点里的数据分析其均值方差，当方差小于一定值可以终止分裂，以换取计算成本的降低。

CART和ID3一样，存在过拟合问题，为了解决这一问题，对特别长的树进行**剪枝**处理。



#### sklearn代码表示

1. CART分类树：

   ```python
   # encoding=utf-8
   from sklearn.model_selection import train_test_split
   from sklearn.metrics import accuracy_score
   from sklearn.tree import DecisionTreeClassifier
   from sklearn.datasets import load_iris
   
   # 准备数据集
   iris=load_iris()
   
   # 获取特征集和分类标识
   features = iris.data
   labels = iris.target
   
   # 随机抽取33%的数据作为测试集，其余为训练集
   train_features, test_features, train_labels, test_labels = train_test_split(features, labels, test_size=0.33, random_state=0)
   
   # 创建CART分类树
   clf = DecisionTreeClassifier(criterion='gini')
   
   # 拟合构造CART分类树
   clf = clf.fit(train_features, train_labels)
   
   # 用CART分类树做预测
   test_predict = clf.predict(test_features)
   
   # 预测结果与测试集结果作比对
   score = accuracy_score(test_labels, test_predict)
   print("CART分类树准确率 %.4lf" % score)
   ```

   

2. CART回归树

   ```python
   # encoding=utf-8
   from sklearn.metrics import mean_squared_error
   from sklearn.model_selection import train_test_split
   from sklearn.datasets import load_boston
   from sklearn.metrics import r2_score,mean_absolute_error,mean_squared_error
   from sklearn.tree import DecisionTreeRegressor
   
   # 准备数据集
   boston=load_boston()
   
   # 探索数据
   print(boston.feature_names)
   
   # 获取特征集和房价
   features = boston.data
   prices = boston.target
   
   # 随机抽取33%的数据作为测试集，其余为训练集
   train_features, test_features, train_price, test_price = train_test_split(features, prices, test_size=0.33)
   
   # 创建CART回归树
   dtr=DecisionTreeRegressor()
   
   # 拟合构造CART回归树
   dtr.fit(train_features, train_price)
   
   # 预测测试集中的房价
   predict_price = dtr.predict(test_features)
   
   # 测试集的结果评价
   print('回归树二乘偏差均值:', mean_squared_error(test_price, predict_price))
   print('回归树绝对值偏差均值:', mean_absolute_error(test_price, predict_price))
   ```

#### Python源码

1. 计算数据集的熵：

   ```python
   def calcShannonEnt(dataset):
       numEntries = len(dataset)     # 样本数量
       labelCounts = {}
       for featVec in dataset:
           currentLabel = featVec[-1]         # 遍历每个样本，获取标签
           if currentLabel notin labelCounts.keys():
               labelCounts[currentLabel] = 0
           labelCounts[currentLabel] += 1
       
       shannonEnt = 0.0
       for key in labelCounts:
           prob = float(labelCounts[key]) / numEntries
           shannonEnt -= prob * math.log(prob, 2)
       
       return shannonEnt
   ```

2. 按照给定的特征划分数据

   ```python
   def splitDataSet(dataset, axis, value):
       retDataSet = []
       for featVec in dataset:
           if featVec[axis] == value:
               reducedFeatVec = featVec[:axis]
               reducedFeatVec.extend(featVec[axis+1:])
               retDataSet.append(reducedFeatVec)
       return retDataSet
   ```

   

3. 选择最好的划分数据集的数据

   ```python
   def chooseBestFeatureToSplit(dataSet):
       numFeatures = len(dataSet[0]) - 1# 获取总的特征数
       baseEntropy = calcShannonEnt(dataSet)
       bestInfoGain = 0.0
       bestFeature = -1
       
       # 下面开始变量所有特征， 对于每个特征，要遍历所有样本， 根据遍历的样本划分开数据集，然后计算新的香农熵
       for i in range(numFeatures):
           featList = [example[i] for example in  dataSet]   #  获取遍历特征的这一列数据，接下来进行划分
           uniqueVals = set(featList)              # 从列表中创建集合是Python语言得到唯一元素值的最快方法
           newEntropy = 0.0
           for value in uniqueVals:
               subDataSet = splitDataSet(dataSet, i, value)
               prob = len(subDataSet) / float(len(dataSet))
               newEntropy += prob * calcShannonEnt(subDataSet)
           infoGain = baseEntropy - newEntropy
           if (infoGain > bestInfoGain):
               bastInfoGain = infoGain
               bestFeature = i
       
       return bestFeature
   ```

4. 递归的创建决策树

   ```python
   # 返回最多的那个标签
   def majorityCnt(classList):
       classCount = {}
       for vote in classList:
           if vote notin classCount.keys():
               classCount[vote] = 0
               classCount[vote] += 1
       sortedClassCount = sorted(classCount.values(), reverse=True)
       
       return sortedClassCount[0]
   
   # 递归构建决策树
   def createTree(dataSet, labels):
       classList = [example[-1] for example in dataSet]
       if classList.count(classList[0]) == len(classList):   # 类别完全相同则停止划分 这种 (1,2,'yes') (3,4,'yes')
           return classList[0]
       if len(dataSet[0]) == 1:          # 遍历所有特征时，(1, 'yes') (2, 'no')这种形式 返回出现次数最多的类别
           return majorityCnt(classList)
       
       bestFeat = chooseBestFeatureToSplit(dataSet)    # 选择最好的数据集划分方式,返回的是最好特征的下标
       bestFeatLabel = labels[bestFeat]         # 获取到那个最好的特征
       myTree = {bestFeatLabel:{}}        # 创建一个myTree,保存创建的树的信息
       del(labels[bestFeat])          # 从标签中药删除这个选出的最好的特征，下一轮就不用这个特征了
       featValues = [example[bestFeat] for example in dataSet]    # 获取到选择的最好的特征的所有取值
       uniqueVals = set(featValues)
       for value in uniqueVals:
           subLabels = labels[:]
           myTree[bestFeatLabel][value] = createTree(splitDataSet(dataSet, bestFeat, value), subLabels)  # 这是个字典嵌套字典的形式
       
       return myTree
   ```

   

