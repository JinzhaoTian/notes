# 风控指标

为了挑选并构造出对目标变量有较高预测力的自变量，需要对变量进行WOE编码，通过IV值的看变量的贡献。

##  数据指标

### WOE

weight of Evidence，证据权重，是对原始自变量的一种编码形式。要对一个变量进行WOE编码，需要首先把这个变量进行分组处理（也叫离散化、分箱）。公式如下：
$$
WOE_i = ln(\frac{B_i/B_T}{G_i/G_T})
$$
其中，$B_i$ 是第i箱中的坏客户的人数，$G_i$是第i箱中的好客户的个数，$B_T$是总共的坏客户人数，$G_T$是总共的好客户人数。实质上WOE表示的是当前分箱中好坏客户的各自占总的好坏客户比例的差异，也叫优势比。

如果WOE的**绝对值**越大，这种差异就越明显，绝对值越小就表明差异不明显。如果WOE为0，则说明该分箱中好坏客户比例等于随机坏客户和好客户比值，此时这个分箱就无预测能力。（感觉类似于熵的意思）

### IV

information value，信息值，是预测模型中选择重要变量的方式之一，它能根据预测变量的重要性对预测变量进行排序，IV值计算公式如下：
$$
IV = \sum^n_{i = 1} (B_i/B_T - Gi/G_T)ln(\frac{B_i/B_T}{Gi/G_T})
$$
假设变量x有n个分箱，每个分箱的WOE编码取值为$WOE_i$,该分箱的IV值就是用该分箱响应比例与未响应比例之差再乘上$WOE_i$。注意这里的响应比例和未响应比例都是用当前分箱中响应数量/整体样本响应数量和当前分箱未响应数量/整体样本未响应数量计算得到。如果IV值大于0.5，则考虑要对这个变量进行分群处理。即根据这个变量拆分成几个样本子集，分别在各个样本子集上建模。

### PSI

> 参考：[风控模型—群体稳定性指标(PSI)深入理解应用](https://zhuanlan.zhihu.com/p/79682292)

群体稳定性指标（Population Stability Index，PSI），公式是：
$$
psi = \sum^{n}_{i=1} (A_i - E_i)ln(A_i/E_i)
$$
其中，$A_i$ 是实际占比，$E_i$ 是预期占比。

计算步骤：

1. 将变量预期分布（excepted）进行分箱（binning）离散化，统计各个分箱里的样本占比。
2. 按相同分箱区间，对实际分布（actual）统计各分箱内的样本占比。
3. 计算各分箱内的$A_i - E_i$和$ln(A_i / E_i)$。
4. 将各分箱的index进行求和，即得到最终的PSI。

业务含义:

* PSI数值越小，两个分布之间的差异就越小，代表越稳定。
* 严格来说，当PSI大于0.2时，该特征很不稳定；当PSI在0.1~0.2时，较不稳定；当PSI在0.1以下时，稳定。



## 模型评估指标

### 基本指标

介绍这些指标要首先介绍混淆矩阵，混淆矩阵的横坐标是真实值的positive和negative，纵坐标是预测值的positive和negative：

<img src="D:/TechKnowledge/imgs/FraudDetection-image-3.png" style="zoom:50%;" />

根据这个混淆矩阵可以衍生出一系列评估标准：

* 准确率（Accuracy）：
  $$
  Accuracy = \frac{TP+TN}{TP+TN+FP+FN}
  $$

* 查准率（Precision）：预测器检索出的信息要尽可能的满足用户（即，预测感兴趣（正类）要更准，减少预测出用户不感兴趣的信息数（假正类的数量））
  $$
  Precision = \frac{TP}{TP + FP}
  $$

* 查全率（Recall）：用户感兴趣的信息要尽可能检索出来（即，用户感兴趣的信息要预测全，减少预测为不感兴趣的信息而实际确实感兴趣信息的数量（假负类））
  $$
  Recall = \frac{TP}{TP + FN}
  $$

* F1-Score：
  $$
  F_1 = 2 \cdot \frac{Precison \cdot Recall}{Precision + Recall}
  $$

### AUC
AUC的全称是(Area Under then Curve Of ROC)，是ROC曲线下面积，而ROC(Receiver operating characteristic curve)是接收者操作特征曲线。ROC曲线的横轴为False Positive Rate，也叫假阳率（FPR），纵轴为True Positive Rate，也叫真阳率（TPR）。ROC曲线实际上就是一个$[0,1] \times [0, 1]$的坐标空间，坐标上的每一个点都是特定**阈值**下的（**1-specificity，sensitivity**)。

* 特异度（Specificity）：
  $$
  Specificity = \frac{TN}{FP + TN}
  $$
  $ FPR = 1- Specificity $.

* 灵敏度（Sensitivity）：
  $$
  Sensitivity = \frac{TP}{FP + FN}
  $$
  $ Sensitivity = Recall $.

**当测试集中的正负样本的分布变化的时候，ROC曲线能够保持不变**。在实际的数据集中经常会出现类不平衡（class imbalance）现象，即负样本比正样本多很多（或者相反），而且测试数据中的正负样本的分布也可能随着时间变化。

**使用AUC值作为评价标准**是因为很多时候ROC曲线并不能清晰的说明哪个分类器的效果更好，而作为一个数值，对应AUC更大的分类器效果更好。

### KS

> 参考：[风控模型—区分度评估指标(KS)深入理解应用](https://zhuanlan.zhihu.com/p/79934510)

Kolmogorov-Smirnov两样本检验法，简称为**KS检验法**。基于经验分布函数的距离而构造，**检验两个累积分布的区分度**，传统上说KS值就是KS检验法的统计量。我们通过预测概率或者分数和真实的label就可以计算出KS值。

KS的另一种解释为真阳率(TPR)与假阳率(FPR)随阀值变化差的最大值。

在探索性数据分析（EDA）中，若想大致判断自变量x对于因变量y有没有区分度，我们常会分**正负样本群体**来观察该变量的**分布差异**。那么，如何判断自变量是有用的？直观理解，如果这两个分布的重叠部分越小，代表正负样本的差异性越大，自变量就能把正负样本更好地区分开。

KS统计量是基于经验累积分布函数（Empirical Cumulative Distribution Function，ECDF）建立的，公式定义：
$$
ks = max\{|cum(bad\_rate) - cum(good\_rate)|\}
$$
计算方式：

1. 对变量进行分箱（binning），可以选择等频、等距，或者自定义距离。
2. 计算每个分箱区间的好账户数(goods)和坏账户数(bads)。
3. 计算每个分箱区间的累计好账户数占总好账户数比率(cum_good_rate)和累计坏账户数占总坏账户数比率(cum_bad_rate)。
4. 计算每个分箱区间累计坏账户占比与累计好账户占比差的绝对值，得到**KS曲线**。
5. 在这些绝对值中取**最大值**，得到此变量最终的**KS值**。

指标解释：

* 区分度越大，说明模型的风险排序能力（ranking ability）越强。

### Log-loss

很多机器学习的算法通常会用logloss作为模型评价的指标，对数损失（Log loss）亦被称为逻辑回归损失（Logistic regression loss）或交叉熵损失（Cross-entropy loss），简单来说就是逻辑回归的损失函数。
