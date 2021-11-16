# Bidirectional Machine Reading Comprehension for Aspect Sentiment Triplet Extraction 

面向ASTE的双向阅读理解



## Task Definition

属性情感三元抽取(Aspect sentiment triplet extraction, ASTE)是细粒度意见挖掘中的一项新兴任务，旨在从评论句子中识别属性，以及相应的意见表达和情感。ASTE由多个子任务组成，包括意见实体抽取、关系检测和情感分类。



## Abstract

In this paper, we transform ASTE taskinto a multi-turn machine reading comprehension (MTMRC) task and propose a bidirectional MRC (BMRC) framework to address this challenge.  

本文将ASTE任务转化为多回合机器阅读理解(MTMRC)任务，并提出了一个双向MRC (BMRC)框架来解决这一挑战（挑战：捕获并利用三个子任务的关系）。

具体是如何解决？——设计了三种查询类型，包括非限制性抽取查询、限制性抽取查询和情感分类查询，以建立不同子任务之间的关联。【所谓非限制性就是即可以查询属性词也可以查询意见词，所谓限制性就是上一轮查了属性词则第当前只能查找意见词】

双向MRC？——一个方向是依次判别属性词、意见词以及情感，从而获得三元组；另一个方向是依次判别意见词、属性词、情感。



## Conclusion

- 将ASTE定义为一种多回合MRC任务，并提出带有精心设计的查询的BMRC框架。具体来说，非限制性和限制性抽取查询的设计是为了将意见实体抽取和关系检测自然融合，从而增强两者之间的依赖性。

- 通过设计双向的MRC结构，可以确保一个属性或一个意见表达可以触发属性-意见对，就像人类的阅读行为一样。

- 此外，采用情感分类查询和联合学习方式，结合属性和意见表达的关系，进一步促进情感分类。




## Contribution

- 我们将ASTE任务形式化为**多回合机器阅读理解(MTMRC)**的任务。基于这种形式，我们可以在一个统一的框架中轻易地识别属性情感三元组。
- 我们提出了一个**双向机器阅读理解(BMRC)框架**。通过设计三回合的查询，我们的模型可以有效地建立意见实体抽取、关系检测和情感分类之间的关联。
- 我们在四个基准数据集上进行了广泛的实验。实验结果表明，我们的模型达到了**SOTA的性能**。





## Introduction

挑战：

- 评论句中，属性词与意见词是一起出现的，并且有明确的对应关系。——如何充分了解ATE和OTE之间的联系，使两者互相辅助？
- 属性词与意见词的对应关系是多样的，有一对多、多对一，甚至是重叠或嵌在其中。——如何准确地检测到它们之间的关系？
- 一个句子可能有多种情感，对不同属性的情感是要考虑它与相应情感词的关系的。——如何恰当地在情感分类中处理这些关系？



为应对上述挑战，我们采取了应对措施并将其形式化为机器阅读理解(MRC)任务。给定一个查询和一个上下文，MRC任务的目标是捕获它们之间的交互，并从上下文中提取特定的信息作为答案。

与一般的MRC任务不同，由于ASTE的复杂性，我们进一步设计了多回合查询来识别属性情感三元组。我们将这种形式定义为多回合机器阅读理解任务。通过将前一回合的答案作为先验知识引入当前回合，可以有效地学习不同子任务之间的关联。

例如：我们可以在第一个回合中识别属性词food，并将其引入到第二个回合的查询中（What opinions given the aspect food?  ），联合识别意见表达delicious，以及delicious与food的关系。然后，我们可以使用属性词food和意见表达delicious作为第三轮查询的先验知识来预测food的情感是正向的。

根据这些回合，我们可以灵活地捕捉ATE和OTE之间的关联，检测意见实体之间的复杂关系，并利用这些关系来指导情感分类。

由于抽取属性和意见表达时没有内在的顺序，我们进一步提出了一个双向结构来识别属性-意见对。



## Related Work

### Fine-grained Opinion Mining

ATE、OTE、ASC、属性-情感联合抽取、属性分类与情感分类、属性-意见对抽取、多任务。

这些研究都不能在一个统一的架构内识别属性、意见表达和情感。

Peng等人(2020)提出了两阶段框架来解决属性情感三元组抽取任务，目的是提取属性、意见表达和情感的三元组。但由于模型采用两阶段框架，存在**错误传播**问题。另外，**意见实体的抽取和配对的分离，意味着没有充分考虑不同任务之间的关联。**



### Machine Reading Comprehension

机器阅读理解(MRC)旨在基于给定的上下文回答特定的问题。最近的研究提出了各种有效的体系结构，它充分学习了查询和上下文之间的交互。例如：BiDAF、QANet

最近，有一种趋势——将MRC应用于许多NLP任务，包括命名实体识别、实体关系提取、摘要等。



## Problem Formulation

给定一个长度为n的句子$X=\{x_1,x_2,…,x_N\}$，ASTE目的是识别出三元组集合$T=\{(a_i,o_i,s_i\}^{|T|}_{i=1}$，a表示属性词，o表示意见词，s表示情感，|T|表示三元组的数量。

非限制性queries $Q^N = \{q^N_i\}^{|Q^N|}_{i=1}$ ，限制性queries $Q^R=\{q^R_i\}^{|Q^R|}_{i=1}$ ，以及情感分类queries $Q^S=\{q^S_i\}^{|Q^S|}_{i=1}$ 。



## Model

### Framework

![image-20211031170448575](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031170448575.png)

采用了BERT作为编码层，对输入进行编码，以获得更加丰富的语义。



### Query Construction

$A\rightarrow O$

非限制性抽取：$A = \{a_i\}^{|A|}_{i=1}$

限制性抽取：$O_{a_i} = \{o_{a_i,j}\}^{|O_{a_i}|}_{j=1}$

$O\rightarrow A$

非限制性抽取：$O = \{o_i\}^{|O|}_{i=1}$

限制性抽取：$A_{o_i} = \{a_{o_i,j}\}^{|A_{o_i}|}_{j=1}$

**情感分类query $q^S$: What sentiment given the aspect $a_i$ and the opinion $o_{a_i,j}?$  **



### Encoding Layer

采用BERT作为编码器

首先，拼接$q_i$和$X$得到输入$I=\{[CLS],q_{i,1},q_{i,2},...,q_{i,|q_i|},[SEP],x_1,x_2,...,x_N\}$

然后，融入位置编码，得到表示序列$E=\{e_1,e_2,...,e_{|q_i|+N+2}\}$

最后，经过BERT得到隐藏表示序列$H=\{h_1,h_2,...,h_{|q_i|+N+2}\}$

【需理解，此处q泛指三种query，它们的拼接情况都是如此】



### Answer Prediction

![image-20211031201005391](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031201005391.png)

看上一节输入$I$的元素内容，|q|是query的维度，2是CLS和SEP的2个维度，i表示第i个x，所以这里是|q|+2+i，即第i个词的隐藏表示乘以权重，通过softmax得到一个向量，判断当前是否为属性词（或情感词）的起始位置（或结束位置），这里的q泛指$q^N和q^R$

![image-20211031204624280](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031204624280.png)

这里的q是指$q^S$，$h_1$是$H^S$中表示[CLS]的那个隐藏向量。



### Joint Learning

融合了不同query的损失函数。

**The loss of non-restrictive extraction queries   **
$$
L_N = -\sum^{|Q^N|}_{i=1}\sum^{N}_{j=1}[p(y^{start}_j|x_j,q^N_i)log\widehat{p}(y^{start}_j|x_j,q^N_i)+p(y^{end}_j|x_j,q^N_i)log\widehat{p}(y^{end}_j|x_j,q^N_i)]
$$
**The loss of restrictive extraction queries  **
$$
L_N = -\sum^{|Q^R|}_{i=1}\sum^{N}_{j=1}[p(y^{start}_j|x_j,q^R_i)log\widehat{p}(y^{start}_j|x_j,q^R_i)+p(y^{end}_j|x_j,q^R_i)log\widehat{p}(y^{end}_j|x_j,q^R_i)]
$$
**The loss of sentiment classification queries    **
$$
L_S = -\sum^{|Q^S|}_{i=1}p(y^{S}|X,q^S_i)log\widehat{p}(y^{S}|X,q^S_i)
$$
**Loss objective of the entire model**
$$
L(θ)=L_N+L_R+L_S
$$

### Inference

融合不同query的答案，最终获得三元组。

具体来说，通过双向模型，得到以下两个集合：

$V_{A\rightarrow O} = [(a_k,o_k)]^K_{k=1}$

$V_{O\rightarrow A} = [(a_l,o_l)]^L_{l=1}$

然后要合并两个集合：
$$
V = V’\ \bigcup\ \{(a,o)|(a,o)∈V'',p(a,o)\ ＞\ δ\}
$$

$$
p(a,o)=
\begin{cases}
p(a)p(o|a)\ \ if(a,o)∈V_{A\rightarrow O} \\
p(o)p(a|o)\ \ if(a,o)∈V_{O\rightarrow A}
\end{cases}
$$

V‘ 和 V''代表了两个集合的交集与并集。【也就是说，交集全都要，交集以外的，只要概率大与δ的，作者设置δ为0.8】

每个意见实体的概率是通过其开始和结束位置的概率相乘计算出来的。



## Experiments

### Datasets

![image-20211031214616943](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031214616943.png)

SemEval的四个经典数据集。【该数据集移除了一个意见表达对应多个属性词的样本】

### Baselines

TSF (Peng et al. 2020)  、RINANTE (Dai and Song 2019)  、Li-unified (Li et al. 2019a)  、RACL (Chen and Qian 2020)   

### Results

![image-20211031220624726](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031220624726.png)

结果表明，抽取管道中的意见实体和关系会导致严重的错误积累。通过利用BMRC框架，我们的模型有效地融合和简化了ATE、OTE和关系检测的任务，避免了上述问题。该数据集移除了一个意见表达对应多个属性词的样本。

AFOE数据集保留了那些被删除的样本，作者在这上面也进行了实验。

![image-20211031222631647](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031222631647.png)

IMN+IOG，首先用IMN（交互式多任务学习网络）识别属性词和它们的情感，然后使用Inward-Outward LSTM抽取属性导向的情感表达。【存在错误传播的问题】



## Ablation Study

四个问题：

- 限制性抽取查询是否建立了意见实体抽取和关系检测之间的关联?
- 双向结构是否提升了属性-意见对抽取的性能?
- 属性与意见表达之间的关系是否增强了情感分类?
- BERT能带来多少改进?

### Effect of the Restrictive Extraction Query  

![image-20211031224925488](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031224925488.png)

结论：随着限制性抽取查询的移除，意见实体抽取和关系检测被分离，并且‘Ours w/o REQ’将不会捕获任何依赖，所以(c)和(d)性能下降很大。【REQ表示移除了限制性抽取query】



### Effect of the Bidirectional MRC Structure  

为了探讨双向MRC结构的影响，将该模型与两个单向模型进行了比较，如上图三所示，即每4条中的中间两条。

结论：属性和意见表达都可以触发属性-意见对，当关系被迫只由属性或意见检测时，模型将有偏差。



### Effect of Relation-Aware Sentiment Classification  

为了检验属性和观点表达之间的关系对情感分类的好处，我们比较了我们的模型和“我们的w/o REQ”模型的性能。

![image-20211031225743495](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031225743495.png)

结论：虽然从联合学习中去除关系检测不会对属性项抽取性能造成严重影响，但属性项和情感联合抽取的性能均明显减弱。这清楚地表明，属性与意见表达之间的关系可以有效地提高情感分类的性能。



### Effect of BERT  

我们从两个角度分析BERT的影响和我们的贡献。首先，我们基于BiDAF构建模型，并将其称为“Ours w/o BERT”（BiDAF是一个典型的不带BERT的阅读理解模型）。

![image-20211031230137214](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211031230137214.png)









