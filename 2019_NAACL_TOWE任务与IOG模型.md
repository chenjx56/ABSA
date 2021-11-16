# Target-oriented Opinion Words Extraction with Target-fused Neural Sequence Labeling  
使用目标融合的神经序列标记进行目标导向的意见词抽取

## 摘要

本文提出的任务就是，给定意见目标，抽取对应的意见词。使用了一种目标融合的序列标注神经网络模型。使用了一种Inward-Outward的LSTM来编码属性词的信息，然后合并属性词的左右上下文以及全局上下文来寻找对应的意见词。



## 引言

挑战：学习特定目标的上下文表示。

解决思路：设计一个编码器融合目标信息，并生成目标融合的上下文。（提出一种Inward-Outward的LSTM来传递目标信息给它的左右上下文，合并左右上下文与全局上下文来编码句子，并进行序列标注）。



## 模型

### 任务定义

用BIO标记来进行序列标记，一个句子成为多个样本，因为每个样本只包含一对属性-意见，而一个句子可以有多个属性-意见对。



### 框架

![image-20211109131508285](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109131508285.png)

### Traget-Fused Encoder

首先使用一个Embedding将句子S映射成E，作为词表示。

直接使用BiLSTM编码整个句子是完全与属性独立的，一个句子有多个属性词时，它们返回的都是相同的表示。

解决方案：

将句子分成三个部分：左边、属性词、右边

用左边的LSTM构建左边与属性词的上下文表示，右边的LSTM构建右边与属性词的上下文表示。



#### Inward-LSTM

实际上是两个LSTM，一个从左侧开始，一个从右侧开始，都是到包含属性词后结束。实际上就是把上下文信息传递给属性词。

![image-20211109133209105](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109133209105.png)

取平均值，得到属性词的最终表示：

![image-20211109133329850](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109133329850.png)

即两个LSTM最后变成了一个隐状态，分别是左边的、平均后的属性词的、右边的三个隐状态拼接形成。



#### Outward-LSTM

要把属性词的信息传递给上下文。且与Inward-LSTM一样求平均得到属性词的最终表示：

![image-20211109133838926](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109133838926.png)



#### IO-LSTM

合并之前两个LSTM的输出。



#### IOG

使用双向LSTM对整个句子进行建模，得到全局隐状态表示。

然后拼接IO-LSTM与这个双向LSTM的输出。

最后交给解码器进行序列标注。



### 解码器

用了两种解码方式

#### 贪婪解码

使用sotfmax计算概率

每次选择概率值最高的标记，不考虑标记之间的依赖关系，运行得很快

![image-20211109134642533](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109134642533.png)

#### CRF

![image-20211109134743969](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109134743969.png)

![image-20211109134839718](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109134839718.png)

![image-20211109134857719](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109134857719.png)

## 实验

### 数据集

作者基于SemEval的四个经典数据集，自己构建了适应于所提出的任务的数据集。

![image-20211109135424595](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109135424595.png)

### 设置

在无标注的840百万词上预训练了300维的Glove词向量空间，在下游任务的训练期间，没有进行微调。



### 评估指标

以span为评估单位



### 比较方法

距离规则（使用距离与词性标记决定意见词）

依赖规则（使用基于依赖树的模板去识别意见对）

LSTM/BiLSTM（只使用一个LSTM或BiLSTM）

Pipeline（包含BiLSTM与距离规则的方法，先用BiLSTM抽取出所有意见词，再根据距离选择最近的意见词）

TC-LSTM（通过拼接把属性词信息融入句子：属性词向量通过平均池化目标词嵌入向量得到，每个位置的表示是属性词向量与词嵌入向量拼接得到，然后交给BiLSTM进行序列标注）

### 结果

![image-20211109140615429](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109140615429.png)

基于规则的方法缺乏鲁棒性。

### 分析

![image-20211109142251978](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109142251978.png)

### 样例分析

![image-20211109142602660](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109142602660.png)













