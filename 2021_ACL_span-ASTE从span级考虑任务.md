# Learning Span-Level Interactions for Aspect Sentiment Triplet Extraction

面向属性情感三元抽取学习span级交互



## Task Definition

Aspect Sentiment Triplet Extraction (ASTE) is the most recent subtask of ABSA which outputs triplets of an aspect target, its associated sentiment, and the corresponding opinion term.   

ASTE是ABSA的子任务，它输出属性目标、相关情感和相应意见词的三元组



## Abstract

Recent models perform the triplet extraction in an end-to-end manner but heavily rely on the interactions between each target word and opinion word. Thereby, they cannot perform well on targets and opinions which contain multiple words.  

最近很多模型都是用端到端的方式执行ASTE，但它们都严重依赖于每个目标词和意见词之间的交互。因此，他们在包含多个词的目标和意见上不能很好有很表现。

Our proposed span-level approach explicitly considers the interaction between the whole spans of targets and opinions when predicting their sentiment relation.  Thus, it can make predictions with the semantics of whole spans, ensuring better sentiment consistency.  

我们提出的span级方法明确地考虑了目标和观点在预测其情感关系时的整个跨度之间的相互作用。因此，它**可以用整个跨度的语义进行预测**，更好地确保情感的一致性。

To ease the high computational cost caused by span enumeration, we propose a dual-channel span
pruning strategy by incorporating supervision from the Aspect Term Extraction (ATE) and Opinion Term Extraction (OTE) tasks.  

为了减轻跨度枚举带来的高计算代价，我们提出了一种**双通道跨度剪枝策略**，这种策略通过**结合来自属性抽取任务和意见抽取任务的监督**实现。（这种计算方式还可以更好地区分意见范围和目标范围）



## Introduction

An example of ASTE:

![image-20211030155332261](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030155332261.png)

情感分类（ASC）是在给定属性词的情况下，预测情感极性。但实际应用中，不会给每个句子特别标明属性词。属性抽取（ATE）就是去抽取属性目标，意见抽取（OTE）就是抽取意见词。ASTE通过一个属性目标词、相应意见词和情感的三元组形式建立了一种情感信息的更加完备的画面。例如上图所示，展示了两个三元组。



## Conclusions & Contribution

- 考虑span级交互并开展实验证明其有效性，性能得到很大提高
- 提出了一种双通道跨度剪枝策略，结合了ATE和OTE任务显式监督，以减轻跨度枚举带来的高计算成本，并最大限度地将有效目标和候选意见词配对在一起
- 这个模型不仅对ASTE有效，而且对ATE和OTE都有效

**虽然性能的提高主要表现在多个词构成的三元组上，但是单词和多词的三元组之间仍然存在显著的性能差距。**



## Related Work

- Two-Stage Pipeline  (it **breaks the interaction** within the triplet structure, and usually suffer from the **error propagation problem**)
- End-to-End (they heavily **rely on word-to-word interactions** to predict the sentiment relation for the target-opinion pair. 应当以一个跨度的词作为单位去考虑，而非单纯考虑单个词之间的交互)
  - 2020年提出一种改进：提出一种位置感知的标记方案，允许模型将目标跨度中的每个词与所有可能的观点跨度相结合（但它不能直接对整个目标跨度和整个意见跨度之间建立跨度到跨度的交互）

Our model explicitly generates span representations for all possible target and opinion spans, and their paired sentiment relation is independently predicted for all possible target and opinion pairs.  

我们的模型显式地生成所有可能目标和意见范围的跨度表示，并独立地预测所有可能目标和意见对的成对情感关系。



## Model

### Task Formulation

给定一个长度为n的X序列：$X = \lbrace x_1,\ x_2,\ \cdots,\ x_n \rbrace$，$S = \lbrace s_{1,1},\ s_{1,2},\ \cdots,\ s_{i,j},\ \cdots,\ s_{n,n}\rbrace$枚举了X中所有可能的跨度。i表示起始位置，j表示结束位置，另外限制了跨度长度最大为L（作者设置为8）。

### Architecture

![image-20211030184641616](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030184641616.png)

【看图解释，第一列是token级的表示，通过拼接token的Embedding，获得Span的Embedding，然后经过一个全连接层判断是属性词、情感词还是无效span，经过剪枝去掉无效span，将有效的target和opinion依次拼接在一起，经过一个全连接层判断是无效或者是什么情感极性】

【上述拼接有两种方案，一种是拼接首尾两个单词的Embedding以及一个可训练的表示了宽度的Embedding，第二种是pooling（最大或平均）跨度内所有token的表示，消融实验证明第一种方案更好】



如图所示，三个模块：sentence encoding、mention module、triplet module

数据流：

1. 输入到sentence encoding获得token级的表示

2. 从1中得到的表示，针对每个枚举的跨度获得span级的表示

3. 采用ATE和OTE任务监督双通道跨度剪枝策略（留下最有可能的spans，即候选意见词与候选目标）

4. 将候选意见词与候选目标组合到一起来判断情感关系

   

*个人理解：step1-3其实就是在做属性抽取和意见抽取两个任务：编码、拼接、分类。只是分类是基于一个更长维度的隐藏空间来进行。思考：是否可以在这里做优化？提高属性抽取与意见抽取的性能，从而提高整体的性能。*



#### Sentence Encoding

选择两种方法：BiLSTM、BERT

BiLSTM使用了dropout，比例为0.5

BERT使用了uncased版本，进行了微调

![image-20211030213236948](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030213236948.png)

$f_{width}(i,j)$是可学习的特征向量，表示了span的宽度：$j-i+1$



#### Mention Module

通过预测好的意见与目标span的分数，用ATE和OTE指导双通道跨度剪枝策略。通过span分数预测m，其中，$m ∈ \{Target,\ Opinion, \ Invalid\}$：

![image-20211030213852894](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030213852894.png)

Mention scores：$Φ_{target}(S_{i,j}) = P(m = target|S_{i,j}); Φ_{opinion}(S_{i,j}) = P(m = opinion|S_{i,j})$

我们使用mention分数选择候选跨度$S^t=\{…, s^t_{a,b},…\}；S^o=\{…, s^o_{c,d},…\}$

n是句子长度，z是一个比例（作者设置为0.5），认为每个句子中候选跨度的个数最多为nz，即$|S^t| \leq nz$。

#### Triplet Module

意见span和属性span的表示进行拼接，得到意见-属性对：

![image-20211030220246125](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030220246125.png)

$f_{distance}(a,b,c,d)$是可训练的特征向量，表示了意见span和目标span的距离：$min(|b-c|,|a-d|)$

将这个属性意见对的表示交给全连接层来分类，确定这是一个什么样的情感极性$r ∈R = \{Positive; Negative; Neutral; Invalid  \}$：

![image-20211030221147167](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030221147167.png)

### Training

![image-20211030221338225](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030221338225.png)

目的是让两个概率越大越好，作用在梯度下降法上，则对其取负数，即下降意味着概率提高。



## Experiment

### Dataset

![image-20211030221940316](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030221940316.png)

数据来源于SemEval比赛，经过后来人额外的标注。

### Experiment Result

![image-20211030223104339](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030223104339.png)

显然在两种编码方式上都达到了SOTA性能。

## Analysis

![image-20211030224652647](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030224652647.png)

虽然方法考虑了span提高了性能，但是显然对span的性能还是不太高。

![image-20211030224546992](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211030224546992.png)

当三元组包含多词的意见词时，性能较低。这种趋势可能是因为包含多词目标词或意见词的三元组的数据分布不均衡。



![image-20211103190550748](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103190550748.png)













