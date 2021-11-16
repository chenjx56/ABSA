# Aspect-based Sentiment Analysis with Type-aware Graph Convolutional Networks and Layer Ensemble  
用**类型感知**的**图卷积网络**以及**层集成**进行基于属性的情感分析



## 摘要

现在非常流行应用神经图模型在ABSA任务上，这样可以**基于依存分析**来利用词的关系，为任务提供更好的语义指导，以分析上下文和属性词。

然而，这些研究大多**只利用依赖关系**，而没有考虑它们的**依赖类型**，缺乏有效的机制来**区分重要的关系**，以及从基于图的模型的不同层次学习。

本文提出**类型感知的图卷积网络**（T-GCN），利用了依赖类型。其中，用注意力区分图中不同的边（关系），并集成注意层来彻底学习T-GCN的不同层。



## 引言

GCN模型可以从不同的词-词关系中学习，这对ABSA任务有很大帮助。但是在这些研究中使用的GCN模型，由于忽略了依赖类型所携带的信息，对图中所有的词-词关系都一视同仁，因此不重要的关系可能无法区分，从而导致对ABSA的误导。

而且他们只将最后一个GCN层的输出用于ABSA任务，忽略了中间层，可能丢失了很重要的信息。



这篇论文总的来说：类型感知的图卷积网络、多层集成。【融合词-词关系与依赖类型】

首先，我们通过现成的工具箱获得输入文本的依赖解析结果。

然后，在依赖树上构建图，每条边都用两个相连的单词之间对应的依赖类型标记。

然后，在图上应用注意机制，根据所有边对任务的贡献来加权所有边。

最后，使用注意层集成对从不同GCN层学习到的上下文信息进行加权和组合。

在此基础上，本文提出的T-GCN模型不仅可以对词汇关系及其依赖类型进行建模，而且还可以从这些关系中区分重要的上下文信息，从而增强ABSA。



## 方法

任务范式：给定句子与属性词，预测属性词的情感极性。

### 架构

![image-20211103145633777](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103145633777.png)

![image-20211103145935556](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103145935556.png)

从公式和图上看：过一层BERT、三层T-GCN、用ALE集成三层T-GCN的输出、通过全连接+softmax得出预测结果

### 类型感知的图结构

我们使用现成的工具包来获取依赖项结果，可以用依赖项元组列表来表示。

邻接矩阵A，通过记录所有元组中的单词关系来呈现图

关系类型矩阵R，代表边。

对于A，若$x_i,x_j$之间有边，则$a_{i,j}=1$，否则为0。

对于R，$r_{i,j}$记录了词与词之间的关系类型。

![image-20211103151645992](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103151645992.png)

<center><b>Figure 3</b></center>

为了利用关系类型，使用了转移矩阵映射关系类型到相应的embedding $e^r_{i,j}$。

### T-GCN

L层的T-GCN，对每一层中的图的每条边都应用注意力机制来根据他们的贡献，对他们进行加权操作。

![image-20211116153950833](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116153950833.png)

如上图，对于词$x_i$与词$x_j$之间的每条边，第l层GCN利用上一层的隐状态$h_i$与$h_j$，用相应的关系嵌入向量拼接它们：

![image-20211103152304524](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103152304524.png)

然后计算这条边的权重：

![image-20211116154410506](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116154410506.png)

调整关系嵌入向量的维度去适应上一层的隐状态：

![image-20211116154601057](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116154601057.png)

最后计算当前层的隐状态，这个操作类似于GCN：

![image-20211116154800146](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116154800146.png)

### 注意层集成

每层T-GCN都融合了所有与词$x_i$相关的信息，所以多T-GCN可以学到长距离的间接词关系。因此，假设不同层有独一无二的编码上下文信息的能力。为了利用这种能力，我们提出通过注意力层集成全面学习所有T-GCN层。

首先，从每个T-GCN层获得输出，平均所有属性词的隐向量：

![image-20211116160851005](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116160851005.png)

然后，对每层的输出求加权和：

![image-20211116160954991](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116160954991.png)

### 编码与解码

编码：

- 对句子编码
- 对句子-属性对编码

解码：

1. 编码结果喂给T-GCN
2. T-GCN的输出喂给ALE
3. ALE的输出喂给全连接层，映射到标记空间



## 实验

### 数据集

四个经典数据集（移除了分类为冲突情绪的句子，也移除了没有属性词的句子）、TWITTER、MAMS。

冲突情绪的句子：“这个寿司肯定不是全国最好吃的，但它足够新鲜。”

![image-20211116162016451](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116162016451.png)



### 实验结果

![image-20211116162925124](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116162925124.png)

显然只用GCN也有效，但T-GCN效果更加明显。

![image-20211116164044490](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116164044490.png)



## 分析

### 消融实验

![image-20211116165545786](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116165545786.png)

不用ALE即只使用最后一层的T-GCN的输出。

每层T-GCN的作用（所占权重）：

![image-20211116170716903](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116170716903.png)

Twitter是社交媒体的数据，句子通常很短且没有任何组织，对于这种句子，模型可能需要一些局部上下文以及整个句子的信息，所以第一层和第三层的贡献更大一些。而其他的数据集，第二层的权重更大，因为第一层只能关注到直接的关系，第三层可能会关注很多不合理的关系，而第二层相比之下，没有那么多问题。

![image-20211116172054113](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211116172054113.png)





















































