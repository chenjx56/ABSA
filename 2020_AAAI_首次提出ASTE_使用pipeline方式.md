# Knowing What, How and Why: A Near Complete Solution for Aspect-based Sentiment Analysis  
理解是什么，怎么样，为什么：面向ABSA的一个近乎完整的结局方案



## 任务定义

Target-based sentiment analysis or aspect-based sentiment analysis (ABSA) refers to addressing various sentiment analysis tasks at a fine-grained level, which includes but is not limited to aspect extraction, aspect sentiment classification, and opinion extraction.

基于目标的情感分析或基于属性的情感分析(ABSA)是指在细粒度的水平上处理各种情感分析任务，包括但不限于ATE、ASC和OTE。



## 摘要

提出一个新的任务——ASTE——抽取三元组（What，How，Why）——即（属性词，情感，意见词）

提出了一种两阶段的框架来处理这个任务，第一阶段用一个同意模型预测三种元素，第二阶段将属性词（与情感极性已经是一个整体）与意见词配对来输出三元组。



## 引言

![image-20211105133954612](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105133954612.png)

如图，中间层的圆形代表了一个实现目标的直接子任务。

例如"Sentiment"连接属性词，代表了ATC（属性情感分类）；"Extract"连接属性词，代表了ATE。

底层则表示了一些联合任务，例如属性抽取与情感分类、属性与意见联合抽取、属性类别与情感分类。

之前没有任何工作研究三元组，没有利用三者之间的关系，所以作者提出ASTE，也就是图片底部被蓝色加深的圆圈。

提出了一个两阶段的模型，第一阶段用{B-POS, B-NEG, B-NEU，E-POS，S-POS，I-POS，……}的标记获得带情感的属性标记序列，用{B，I，O， E， S}的标记获得意见词的标记序列。

![image-20211105135934154](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105135934154.png)

对于统一标记模式，是建立在两个双向LSTM上的，上面那个会生成属性词与情感标记的结果，下面那个LSTM是对属性边界做一个辅助预测，目的是指导上面那个双向LSTM。另外会设置门机制用来维持每个multi-word属性的情感的一致性。

对于意见词标记模式，它建立在一个双向LSTM层和一个图卷积网络GCN之上，来利用语义与句话信息。“一个词/短语被认为是一个属性词的话，它应该与相同的意见词一起出现，该意见词揭露了它的情感极性”。因此，属性信息能够帮助意见词抽取，所以作者特别设计了一个目标指导模块来传递属性信息给意见词抽取。



第一阶段之后，该模型得到了一堆带有情感极性的属性词，和一堆意见表达。第二阶段就是把它们配对。而一个句子如果有多个属性词和多个意见词，单词距离对于正确配对是非常有帮助的，正如上表的第二行所示。所以设计了距离嵌入向量来捕捉属性与意见表达之间的距离。



## 模型

### 概况

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105162539451.png" alt="image-20211105162539451" style="zoom:120%;" />

阶段一：

**左半部**分采用统一标记模式抽取属性词与情感极性，使用了一个源组件TG与右半部分进行共享，帮助预测意见词。左半部分两个BLSTM，下面那个是用BIO标记来检测属性词的边界，它的输出可以辅助上面那个BLSTM以及TG（目标指导）、BG（边界指导）工作。上面那个LSTM直接辅助SC（情感一致性）工作。SC的输出作为主要信号交给BG来预测统一标记。BG也将纯目标边界序列转化为指导统一标记序列的预测。

**右半部分**是用来预测意见词的。句子交给GCN，通过依赖关系学习目标与意见词的交互。将学得的信息交给TG与一个双向BLSTM。TG会拼接BIO标记序列（属性词）与GCN的输出，即利用目标信息进行意见抽取。这个TG受到了意见抽取的强烈监督，因此$BLSTM^T$和GCN都会收到它的后向传播的帮助。$BLSTM^{OPT}$的输出会将句子上下文放到GCN输出的顶部。它将被用于意见词提取，以及指导统一的标签预测。

阶段二：

首先使用属性和意见表达（第一阶段预测得到的）来生成所有可能的属性意见对。

基于每个对中意见与属性的距离，给每个意见词和属性词应用一个位置向量。非属性词与非意见词拥有相同的位置向量（这里为0）。经过一个BLSTM编码器得到，拼接来自属性词与意见词的隐藏状态表示进行二分类。

### 阶段一

**统一的属性边界与情感的标记**

![image-20211105135934154](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105135934154.png)

一个属性可能包含多个词，像上图这样的句子，有可能把把fugu标记为正向，sashimi标记为负向，所以引入一个门控机制来保持情感的一致性。

![image-20211105185013126](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105185013126.png)

通过这个机制，当前时间步的判断也会继承上个时间步的判断的特征，减少了情感标记发生改变的风险。

BG模块使用一个约束转移矩阵$W^{tr}∈R^{|y^T|X|y^S|}$，这是一个概率转移矩阵，表示BIEOS标记转移到统一标记模式下的标记的概率。

![image-20211105190705583](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105190705583.png)

最后合并$\widetilde{h}_t$与$h^{OPT}$得到增强表示$h^u$来进行统一标记预测：

![image-20211105192002738](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105192002738.png)

合并上述两个概率$z^{S'}_t,z^S_t$，通过目标边界标记器得到的分数，计算一个融合权重矩阵：

![image-20211105192642887](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105192642887.png)

$α_t$表示了这个目标边界标记器的置信度，然后利用它合并两个概率：

![image-20211105193338732](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105193338732.png)

**意见词抽取**

GCN的邻接矩阵基于句子的依存分析结果而构建，$W^{GCN}$（取值只有0或1）。这能够捕捉到属性词与意见词的关系，因为它们是被构建成了句法修饰对。

用TG模块合并目标边界信息与GCN的输出。针对TG的实现方式，做了多种实验，发现简单拼接更加有效，以此进行意见词预测：

![image-20211105195155687](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105195155687.png)

$BLSTM^{OPT}$将编码GCN的输出，得到$h^{POT}$，将其交给BG协助进行统一标记（上一部分已描述），同时也交给一个softmax分类器进行意见词预测：

![image-20211105195631063](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105195631063.png)

**阶段一的训练**

随机梯度下降法，四个损失函数都使用了交叉熵计算：

![image-20211105195848036](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105195848036.png)

![image-20211105195930297](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105195930297.png)

![image-20211111191049466](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111191049466.png)

训练目标就是最小化总损失：

![image-20211105200023576](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105200023576.png)



### 阶段二

阶段一产生两个集合：意见词、属性词

枚举所有的配对可能，并判断是否有效。

**位置嵌入向量**

为了利用属性与意见表示之间的位置关系，通过计算中间出现的单词数量来计算属性中心与意见表示中心之间的距离。例子如下表第二行：

![image-20211105135934154](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105135934154.png)

**对编码器与分类**

如图2所示，拼接预训练的GloVe词向量与位置嵌入向量构建词表示。

位置嵌入向量随机初始化，在每个训练步进行训练。然后把句子交给BLSTM层来编码句子上下文信息到属性与意见表达之中。基于句子项索引，平均隐藏状态的输出，作为特征。拼接属性词与意见词的特征交给softmax层进行二分类。



## 实验

### 数据集

在Fan等人2019年标注了意见词的基础上，合并了相同句子中的不同目标和意见注释样本。每个样本包括源句子、一个用统一属性标记的序列和一个有意见标记的序列。因为每个句子可能有多个属性词与意见词，所以要将每个属性词与它相应的意见词配对。例子：

![image-20211105203052951](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105203052951.png)

我们还对少数目标和意见有重叠的样本进行了修正。验证集是从训练集中随机选20%出来。

![image-20211105203456349](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105203456349.png)

### 结果

![image-20211105205526185](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105205526185.png)



![image-20211105205541397](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105205541397.png)



Li-Unified-R比Li-Unified好，证明给定意见词的真实标记，对意见抽取进行显式建模有助于提高意见抽取的性能。

### 消融实验

![image-20211105214745143](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105214745143.png)

移除TG模块在前两个数据集上表现反而更好，由于TG模块研究的是属性词和意见词之间的相互影响，所以我们怀疑它们之间的相互关系不是很强。也就是说，TG模块也可能会带来噪声。

### 样例研究

当一个词对应多个情感词时，可能出现问题，或许可以设置一个词只能构成一个三元组，但实验证明不是很有效。

![image-20211105220019560](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211105220019560.png)











