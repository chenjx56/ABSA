# Position-Aware Tagging for Aspect Sentiment Triplet Extraction

面向属性情感三元抽取的位置感知标记



## 1Task Definition

Aspect Sentiment Triplet Extraction (ASTE) is the task of extracting the triplets of target entities, their associated sentiment, and opinion spans explaining the reason for the sentiment.

属性情感三元抽取的任务是抽取目标实体、相关情感以及意见span的三元组，意见span是解释了情感的原因。

Haiyun Peng, Lu Xu, Lidong Bing, Fei Huang, Wei Lu, and Luo Si. 2019. Knowing what, how and why: A near complete solution for aspect-based sentiment analysis. In Proc. of AAAI.【基于属性的情感分析的近似完整解决方案——首次提出ASTE任务，使用的是管道方法】

## 2摘要

现在主要是用管道方法（依次进行多个任务）来解决ASTE任务，也就是把它拆成多个阶段进行的任务。其实三元组的各个元素之间有很强的关系，所以作者要使用序列标记的方法，建立一个联合模型来抽取这样的三元组。并为此提出一种端到端的模型，使用了新颖的位置感知标记模式。



## 3引言

![image-20211104191308878](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211104191308878.png)

Peng等人用的管道方法，分两个阶段。第一阶段：采用一个统一的标记模式，将基于BIOES的属性词标记和情感标记融合在一起，它是基于LSTM、CRF、GCN执行序列标注，抽取带有情感的目标（属性词，标记示例：“B-POS”）和意见词。第二阶段：使用基于MLP的分类器，将每个目标与相应的意见词配对，并获得有效的三元组。

其实三元组的各个元素之间有很强的关系，所以联合捕捉三元素之间的丰富交互应该是非常有效的方法。现有的BIOES标记没有对任何位置信息进行编码，由于表达能力有限，不能明确目标与其意见span之间的联系，也不能明确三个元素之间丰富的交互作用。使得不得不在第二阶段用额外的步骤连接每个目标与意见span。

**贡献：**

- 提出一种新颖的位置感知标记模式
- 提出一种新颖的方法JET，基于上述种标记模式联合抽取三元组
- 开展实验，证明其有效性



## 4方法

### 4.1位置感知的标记模式

拓展了BIOES中B、S两个标记变成  $B^∈_{j,k}, S^∈_{j,k}$。

其中，∈有三种取值：{+， 0， -} 表示情感极性，j,k分别表示属性词与意见词两端的距离。如下图food位置为1， so so位置为3和4，故距离分别为2和3。

![image-20211104212109462](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211104212109462.png)

提出的位置感知标签方案能够处理一些特殊情况，这是标注BIOES方法无法处理的。



### 4.2模型

作者的模型基于位置感知标记，运用CRF和半马尔科夫CRF，能够解码并分解目标的词级特征和意见span的块级特征。给定长度为n的序列x，希望得到基于位置感知标记的标记序列y：

![image-20211104220157370](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211104220157370.png)

s(x, y)是一个分数方法：

![image-20211104220346359](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211104220346359.png)

前半部分是转移分数，后半部分是分解特征分数。

后半部分基于一个简单LSTM的神经网络实现。

分解特征分数能够像标注CRF一样编码词级特征，像半马尔科夫CRF一样编码块级特征，也能编码一个三元组中的目标、它的情感、意见span的交互。

![image-20211104210856245](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211104210856245.png)

#### 4.2.1神经模型

首先$X \rightarrow E$，然后利用BiLSTM使$E \rightarrow H$，计算$g_{a,b}$作为意见span的块表示。

![image-20211105092219435](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105092219435.png)

#### 4.2.2分解特征分数

![image-20211105092353099](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105092353099.png)

线性层$f_t$用来计算目标的局部上下文的分数，输入隐藏状态$h_i$得到一个长度为5的向量（分别表示BIOES）

若$y_i∈B^∈_{j,k}, S^∈_{j,k}$，则需要计算另外3个部分的分解特征分数来捕捉结构特征。

![image-20211105092732113](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105092732113.png)

$f_s$计算情感分数【g表示意见span，h表示target的局部上下文，情感主要受这两者影响】，只用后向h是因为，目标词的相对位置只使用第一个标记B来表示，当目标词长度超过1时，剩余部分必然在位置i的右侧，利用后向隐状态可以把这些信息利用起来。最后输出是长度为3的向量，表示了情感极性。

$f_o$计算意见span的分数

$f_r$计算位置分数，$w_r$是随机初始化的，用来编码不同的位置

![image-20211105102657880](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105102657880.png)

### 4.3一个属性对应多个意见span

上述模型可以实现多个属性词，对应同一个意见span。但不能处理一个属性词关联多个意见span的情况。

解决方案：交换意见span与目标的角色，再训练一个模型。即BIOES标记用来标记意见span。

第一个模型称为$JET^t$，第二个模型称为$JET^o$，第二个模型的标记结果如下图所示：

![image-20211105103531195](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105103531195.png)

### 4.4训练与推断

![image-20211105103822317](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105103822317.png)

训练基于上述损失函数，推断使用了类似于维特比算法的MAP推断，时间复杂度为$O(nM^2)$，远低于Peng等人的$O(n^2)$，因为M取值为2到6，n最大可以到80，可以知道，M远小于n。

## 5实验

### 5.1数据集

提炼了Peng等人创建的数据集，称为ASTE-Data-V2（原版称为V1）。V1没有包含那些一个意见span对应多个属性目标的例子。

例如：“Best service and atmosphere”，一个意见词对应两个属性词，可以产生两个三元组。V1并没有全部标记这些三元组。作者拓展了这些被忽略的三元组到V2版本。【移除了一些三元组，这些三元组被SemEval标记为“conflict”】

![image-20211105105050842](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105105050842.png)

5.2Baselines

- RINANTE+（peng等人，2019）：利用挖掘规则捕捉依存关系，第二阶段生成所有可能的三元组，用MLP区分是否为有效三元组。
- CMLA+（peng等人，2019）：利用注意力机制捕捉词与词之间的依赖，从而联合抽取目标、情感、意见span。然后第二阶段与上述一致。
- Li-unified-R（peng等人，2019）：基于一个多层LSTM神经架构联合抽取三个元素，第二阶段同上。
- 基于Li-unified-R启发，同时抽取三元素，用到了GCN来捕捉依存关系，第二阶段依旧如上。



### 5.3 设置

词嵌入向量：300维的GloVe；$w_r$：100维；LSTM：300维；BERT：bert-base-uncased

丢弃了训练集中偏移量大于M的三元组

### 5.4 结果

![image-20211105112236962](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105112236962.png)

M表示偏移量，图中M表示偏移量最大值（即丢弃了偏移量大于M的三元组）

## 分析

### 鲁棒性分析

![image-20211105122819586](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105122819586.png)

$JET^o(M=6)$在目标词长度小于等于3的时候，抽取性能比$JET^t(M=6)$好

意见span上，长度1或4的时候，O的表现更好。

位移量大于等于4的时候，也是O的表现更好。

当长度都增加时，性能整体都在下降，证明了：长度都增加时，对边界建模越困难。



第二行的图用了另两种评估方法，p表示边界有重叠的时候，认为是部分正确。

这样的比较表明，当子标签BIOES被用来明确地建模意见或目标时，意见范围或目标范围的界限可能会被更好地捕获。



### 质量分析

![image-20211105125305349](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105125305349.png)

### 消融实验

![image-20211105125602291](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105125602291.png)

### 集成分析

两个模型互相集成

集成方法，例如把o的三元组结果（t没有的）添加到t中。

三元组重叠定义：意见span和目标词都有重叠

如果o产生了一个三元组，和t产生的三元组没有任何重叠，则把它加入到t中。

![image-20211105130736572](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105130736572.png)

F1分数提升很小，但是recall得到极大提高（更多的正例被找到）。







