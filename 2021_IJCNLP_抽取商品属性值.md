# AdaTag: Multi-Attribute Value Extraction from Product Profiles with Adaptive Decoding  

基于自适应编码从商品概要中进行多属性值抽取



## 摘要

以前的工作是通过独立的解码器或完全独立的模型构建属性特定的模型。这种方法限制了不同属性之间的知识共享。

另一种方法是使用单一的多属性模型，使用不同的技术融入属性信息。但是共享所有属性的整个网络参数会限制模型捕捉特定属性特征的能力。

我们提出AdaTag：使用自适应的解码处理抽取。通过一个超网络和一个混合专家模块，用预训练的属性嵌入向量参数化解码器。它允许为不同的属性**动态生成独立但语义相关的解码器**。这种方法有助于知识共享，同时保持每个属性的特殊性。



##  引言

属性值自动提取的目的是从商品简介中获取结构化的产品特征。

输入商品简介中的文本序列，以及从潜在的大量属性中提取的所需属性。

输出就是相应的提取出来的属性值。 如下所示：

![image-20211109162432805](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109162432805.png)

解决过去问题的方法：基于每个属性的嵌入向量，动态生成每个属性的编码器。

具体做法：使用CRF作为解码器，通过一个超网络和一个混合专家模块使用属性嵌入向量，将编码层参数化。针对属性名以及它的潜在值，使用了与上下文相关的词嵌入向量与静态的词嵌入向量，来捕获有意义的语义表示。

## 背景

### 问题定义

从$X = [x_1,…,x_n]$中抽取所有描述属性$r$的span。如果没有则返回空集。

假设：不同属性所对应的属性值在文本中是没有重叠的。

用的是BIOE标记，做的是序列标注任务。



### BiLSTM-CRF结构

输入序列已经经过词特征表示

![image-20211109165303339](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109165303339.png)

CRF解码器由一个线性层和一个转移矩阵构成，线性层计算发射概率，转移矩阵计算转移分数。对于输入文本的分数：

![image-20211109165631411](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109165631411.png)

P代表发射分数，T代表转移矩阵，$z_i$代表了标记



## 模型

### 概况

多属性值提取任务可以认为是一组对应不同的属性的抽取子任务，该模型的核心思想是根据给定属性对应的特定子任务动态地调整提取模型的参数。

共享相同的文本编码器来获得融了上下文信息的每个词的隐藏表示。然后使用CRF解码得到标记序列，基于属性嵌入空间动态生成解码器的参数。该方法联合训练不同的子任务，并基于属性嵌入向量对不同解码器进行关联。

![image-20211109172137319](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109172137319.png)

### 自适应的基于CRF的解码器

为了让模型感知搜索的属性词，我们需要合并属性信息到BiLSTM-CRF架构的一些位置。

这个基于BiLSTM的文本编码器负责编码文本序列（理解句子）。基于CRF的编码器则根据编码序列预测每个词的标记。

CRF有发射分数与转移矩阵，发射分数的计算过程就是利用了编码后的上下文编码序列，转移矩阵则学习了标记间的依赖关系，避免预测不可能的标记序列。

### 超网络

使用一个网络去生成另一个网络的参数。使用两个不同的线性转换，映射属性嵌入向量到解码器的线性层的参数。

![image-20211109182758168](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109182758168.png)

### 混合专家

这个概念是以前人提出来的，让一组网络通过动态决定的权重，联合做出决定。不像先前的那些合并每个专家的预测的方法，我们合并参数来生成转移矩阵。k是用来参数化转移矩阵的专家数量。为k个专家，引入k个可学习的矩阵。每个专家矩阵为一类原型，采用一个线性门控网络来计算将给定属性分配给每个专家的概率：（作者最终设为3个专家）

![image-20211109190041704](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109190041704.png)

![image-20211109190140462](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109190140462.png)

其实就是按概率求和k个矩阵，形成转移矩阵。

### 预训练属性嵌入向量

在获取属性感知的编码层中，属性嵌入向量r扮演了关键角色。

用属性名与可能取值作为代理，去捕获给定属性的属性值抽取任务的特征。

因此，属性嵌入可以**直接从训练数据中得到**，并作为初始化加载到属性嵌入层中。

对于每个属性r，首先收集训练数据中所有与r相关的句子。定义收集好的句子的值为![image-20211109191428317](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109191428317.png)，针对其r和v都可以计算一个值嵌入向量，可以用补充上下文信息的方法，也可以不用（后面详细介绍)。汇聚$D_r$中的所有实例，获得最终的属性名嵌入空间以及属性值嵌入空间，然后合并获得属性嵌入空间。

![image-20211109192003461](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109192003461.png)



### 上下文Embedding

把$v_i$换成$[BOA]\widetilde{r}[EOA]$，然后用BERT编码成$r^{name}_i$，编码$X_i$得到$r_i^{value}$。



### 非上下文Embedding

50维的Golve



### 模型训练

训练三类参数：encoder、hyper、moe

![image-20211109192837762](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109192837762.png)

## 实验

### 数据集

从亚马逊搜集产品简介。选择了32种不同频率的属性。

![image-20211109194527458](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109194527458.png)



### 比较方法

**BiLSTM、BiLSTM-CRF、BERT、BERT-CRF**，这四种方法没有考虑搜索属性。所以作者设置了两种改变方法：1）用类似B-SkinType这种标记，来预测结果（也就是变成了32*3+1=97分类问题）。2）用N个代表不同属性的模型。

第二种方法不利于知识共享。所以提出使用相同的编码层，不同的解码层。

**SUOpenTag**用BERT编码了文本序列以及搜索属性，采用了跨注意力机制来为每个词获得一种属性感知的表达。

**AdaTag**使用与作者相同的架构，但使用的是随机初始化可学习的属性嵌入向量。



### 结果

![image-20211109201042044](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109201042044.png)

“N tag sets”性能最差，可能是因为训练集中严重的数据不平衡问题。

所有属性相同的解码器，可能会导致学习偏向于高资源的属性。

第三组性能更高证明了知识共享的作用。

BERT表现比BiLSTM差，很可能是因为预训练的语料库不同，差别太大。



### 比较低资源与高资源的属性

![image-20211109202401511](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109202401511.png)

低资源属性从共享知识中获益更大。



### 消融实验

**属性嵌入向量**

![image-20211109210146811](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109210146811.png)

表明了属性名比属性值更有用。



**解码器参数化**

![image-20211109210624780](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109210624780.png)



### 属性数量的影响

![image-20211109212613327](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109212613327.png)

















