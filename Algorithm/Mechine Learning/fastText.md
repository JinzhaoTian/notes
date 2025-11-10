# fastText

fastText是Facebook于2016年开源的一个词向量计算和文本分类工具，在学术上并没有太大创新。但是它的优点也非常明显，在文本分类任务中，fastText（浅层网络）往往能取得和深度网络相媲美的精度，却在训练时间上比深度网络快许多数量级。在标准的多核CPU上， 能够训练10亿词级别语料库的词向量在10分钟之内，能够分类有着30万多类别的50多万句子在1分钟之内。



## 原理

一般情况下，使用fastText进行文本分类的同时也会产生word embedding，即word embedding是fastText分类的产物。除非你决定使用预训练的word embedding来训练fastText分类模型。

Word2Vec把语料库中的每个单词当成原子的，它会为每个单词生成一个向量，这忽略了单词内部的形态特征。fastText使用了字符级别的n-grams来表示一个单词。对于单词“apple”，假设n的取值为3，则它的trigram有“<ap”, “app”, “ppl”, “ple”, “le>”，其中，<表示前缀，>表示后缀。于是，我们可以用这些trigram来表示“apple”这个单词，进一步，我们可以用这5个trigram的向量叠加来表示“apple”的词向量。

这带来两点**好处**：

1. 对于低频词生成的词向量效果会更好。因为它们的n-gram可以和其它词共享。
2. 对于训练词库之外的单词，仍然可以构建它们的词向量。我们可以叠加它们的字符级n-gram向量。



fastText模型架构和Word2Vec的CBoW模型架构非常相似:

<img src="../imgs/Word2Vec-image-5.png" style="zoom:50%;" />

此架构图没有展示词向量的训练过程。可以看到，和CBoW一样，fastText模型也只有三层：输入层、隐含层、输出层（Hierarchical Softmax），输入都是多个经向量表示的单词，输出都是一个特定的target，隐含层都是对多个词向量的叠加平均。不同的是，CBoW的输入是目标单词的上下文，fastText的输入是多个单词及其n-gram特征，这些特征用来表示单个文档；CBoW的输入单词被onehot编码过，fastText的输入特征是被embedding过；CBoW的输出是目标词汇，fastText的输出是文档对应的类标。



## 代码

