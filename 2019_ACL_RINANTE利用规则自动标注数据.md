# Neural Aspect and Opinion Term Extraction with Mined Rules as Weak Supervision  
用挖掘规则作为弱监督进行神经属性词与意见词的抽取



## 模型

![image-20211109153927809](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109153927809.png)

该论文的重点主要是提出了一种挖掘算法来自动标注数据，也就是通过这种方式来增加训练数据，但因为是利用规则来辅助这个挖掘算法，所以标注数据的质量很难得到保障，因此自动标记的数据只用来作弱监督的学习。

弱监督的学习，即把任务分成三种类型，一种是预测挖掘出来的属性词，一种是挖掘出来的预测意见词，最后一种是预测人工标注的数据。训练停止条件是看对人工标注数据的性能。

任务实际上是要同时抽取属性词与意见词，但是规则自动标注的数据不能保持一致性，一个词可能既被标注成属性词又被标注成意见词，所以用两个解码器来处理规则标注的数据。

## 训练

两种训练方式：

- 三个任务轮流训练，直到CRF-M的性能达到一定程度(Alt)
- 先轮流训练CRF-RA与CRF-RO到一定程度，再训练CRF-M到一定程度(Pre)

## 实验

![image-20211109154038208](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109154038208.png)