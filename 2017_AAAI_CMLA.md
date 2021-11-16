# Coupled Multi-layer Attentions for Co-extraction of Aspect and Opinion Terms  

面向属性词与意见词的联合抽取的双重多层注意力

南洋理工大学（新加坡）



## Task Definition

Aspect and opinion terms co-extraction, which aims at **identifying aspect terms** and **opinion terms** from texts, is an important task in fine-grained sentiment analysis.

从文本中抽取属性词与意见词，是细粒度情感分析（例如：Aspect-Based Sentiment Analysis）的一个重要子任务。属性词描述一个实体或实体的特征，意见词表达了情感。



## Abstract

To achieve this task, one effective approach is to exploit relations between aspect terms and opinion terms by parsing syntactic structure for each sentence.   

过去的方法：通过对每个句子的句法结构进行解析来挖掘属性词和意见词之间的关系。

但这种方法：贵！质量不好把控！所以作者提出：耦合多层注意力。

它提供了一个端到端解决方案，不需要任何解析器或其他语言资源进行预处理。

The proposed model is a multilayer attention network, where each layer consists of a couple of attentions with tensor operators. One attention is for extracting aspect terms, while the other is for extracting opinion terms.    

它是一个多层注意力网络，其中每一层由有若干张量算子的一对注意力组成。一个注意力面向属性词抽取，另一个面向意见词抽取。（它们交互地学习，以在属性词与意见词之间双向地传播信息。）

Through multiple layers, the model can further exploit indirect relations between terms for more precise information extraction.  

该模型通过多层结构，进一步挖掘词汇之间的间接关系，实现更精确的信息抽取。



## Conclusion

提出了面向属性-意见联合抽取的、带有一对多层注意力的、端到端的网络（CMLA）。不需要任何的分析与语言资源。它们的两个注意力通过张量运算，利用输入token之间的相关性，特别是属性和意见词之间的相关性。另外，多层结构有助于抽取间接关系的不明显的目标。作者在三个数据集上证明了它的有效性。



## Introduction

In the literature, there exist many lines of work for aspect and/or opinion terms extraction which can be categorized as rule-based, feature-engineering-based, or deep learning-based approaches.  

现在有多种类别的方法：基于规则的、基于特征工程的、基于深度学习的方法。（之前的这些方法都很依赖句法分析的质量，而且用户生成的文本很可能不一定遵循标准的语法，所以会对它们的性能有很大影响。而且解析长句子可能会非常耗时。）

因此，作者使用带张量算子的注意力机制代替句法分析/依存分析的角色，来捕获句子中各个token的关系。

动机：利用属性与意见词的关系，但不需要任何分析或是语言的资源。



## 问题定义

一个句子：$s_i = {w_{i1},...,w_{in_i}}$（每个句子长度为$n_i$）

抽取显示的属性词$A_i = {a_{i1},...,a_{ij}}$与意见词$P_i = {p_{i1},...,p_{im}}$。（属性词个数为j，意见词个数为m）

标记方案：{BA, IA, BP, IP, O}



## 耦合多层注意力（CMLA）

- 一对注意力
  - 学习属性词或意见词的**原型向量**，学习每个词的**高级特征向量、注意力分数**
  - 高级特征向量、注意力分数评估了每个token间相关联的范围
  - 原型向量使用了一种张量算子，当评估token与原型的相关性时，会捕获给定token的不同上下文
- 捕获属性词与意见词之间的直接关系
- 捕获属性词与意见词之间的间接关系

### 带张量算子的注意力

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113110701561.png" alt="image-20211113110701561" style="zoom:150%;" />

在属性注意力中，我们首先为属性生成一个原型向量$u_a$，它可以被看作是属性词的一般特征表示。这个属性原型将指导模型关注最相关的标记。给定$u_a$和H，模型扫描输入序列，计算第i个token的注意力向量$r^a_i$和注意力得分$e^a_i$。($u_a$是随机初始化的，然后是可学习的)

![image-20211113111356977](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113111356977.png)

其中，![image-20211113111426638](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113111426638.png)。

张量算子可以被看作是多个双线性项，可以建模两个单元之间更复杂的组成。如上图所示，$G^a$可以分为K个部分，每个部分$G^a_k∈R^{d * d}$就是一个双线性项，让两个向量交互，捕捉一类组合。所以tanh中三个元素相乘就继承了不同的K种组成。从而描述了每个token与属性原型的复杂关系。

另外，引入门控机制：

![image-20211113113056986](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113113056986.png)![image-20211113113107101](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113113107101.png)![image-20211113113119067](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113113119067.png)![image-20211113113222974](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113113222974.png)

g是重置门，z是更新门，控制了来自之前时间步的信息流

应用了门控以后，注意力向量$r^a_i∈R^K$变得上下文相关了，即能够继承以前的信息了。

可以简写成如下形式：

![image-20211113120951172](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113120951172.png)![image-20211113121002536](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113121002536.png)

注意力分数的计算：![image-20211113114103732](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113114103732.png)。

${v^{a}}^T∈R^K$可以被视为一个权重向量，对每个特征进行相应的加权。

预测每个词：![image-20211113114800299](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113114800299.png)，![image-20211113114809550](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113114809550.png)。



### 双重传播的耦合注意力

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113133027459.png" alt="image-20211113133027459" style="zoom:150%;" />

不同于单个注意力，这里有两个原型。

![image-20211113115808203](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113115808203.png)

用m表示是因为有两种组合。$D^m$用来建模与意见词的关系（在属性注意力这边时)，从而实现属性注意力与意见注意力之间的双向传播。

![image-20211113120922121](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113120922121.png)



### 多层耦合注意力

一层耦合注意力，只能捕获属性词与意见词之间的直接的关系，没法捕获间接的关系。

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113115425038.png" alt="image-20211113115425038" style="zoom:150%;" />

![image-20211113133226704](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113133226704.png)

如上图(c)所示，每层耦合注意力的原型向量都是根据前一层的原型向量得到的：

![image-20211113133448041](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113133448041.png)

![image-20211113133631180](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113133631180.png)

公式反应了：根据注意力分数所占权重去提取每个词的隐状态。

例如第一层，属性词Fish burger很容易被属性注意力关注到，意见词best也很容易被意见注意力关注到。通过属性注意力与意见注意力的交互生成了注意力分数，然后因为dish和burger与best有很强的关联性，所以第二层就会关注到dish，又因为dish与tastes有很强的关系，从而又会帮助去关注tastes。

这说明多层注意网络能够逐步注意到那些不明显或是有间接关系的属性词与意见词。

通过累加每层的注意力向量，得到最终的注意力向量，从而进行预测：

![image-20211113140350751](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113140350751.png)



## 实验

### 数据集与设置

![image-20211113140547609](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113140547609.png)

源数据只有属性词的标记，对于S1、S2是基于Wang等人在2016年标注的情感序列。

S3是自己纯手工标记的情感词。

预训练词向量：word2vec。对于餐厅则用Yelp挑战数据集训练（200维），对于笔记本电脑则用亚马逊商店上电子产品领域的评论训练（150维）。

### 实验结果

![image-20211113142444965](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113142444965.png)

![image-20211113143125187](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113143125187.png)

ASL：只有属性注意力，训练数据也只用属性标注序列。

ASL+OPL：独立训练属性注意力与意见注意力。

![image-20211113144205106](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113144205106.png)

句子的语法不是很正式，所以RNCRF不能找全，但CMLA可以，因为CMLA不依赖于句法分析。

![image-20211113144431847](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211113144431847.png)



































