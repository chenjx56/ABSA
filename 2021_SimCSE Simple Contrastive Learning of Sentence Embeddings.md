# SimCSE: Simple Contrastive Learning of Sentence Embeddings  

句子嵌入向量的简单对比学习



## Abstract

我们首先描述一种只用标准dropout作为噪声的无监督的方法，该方法用一个输入句在一个对比目标中预测其本身。这个简单方法的效果出奇的好，与之前的有监督的方法相当。我们发现，**dropout作为一种极小的数据增强方式，删除它会导致表示崩溃**。然后，我们提出了一种监督方法，使用“隐含”对作为正样本，“矛盾”对作为硬负样本。该方法将自然语言推理数据集中的标记对引入到我们的对比学习框架中。对比学习目标使得预训练的嵌入空间更加均匀，当监督信号是可获取的时候，它能更好地使正例对对齐。



## Introduction

我们的无监督SimCSE只是简单地预测了输入句子本身，只是用了dropout作为噪声。

让相同地句子进入预训练编码器两次，通过应用两次标准的dropout，可以获得两种不同的嵌入向量作为”正例对“。然后使用相同batch下的其它句子作为”负例“，并且模型会预测负例中的正例。





## Background

Alignment：计算配对实例的嵌入向量之间的期望距离

![image-20211102214508414](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211102214508414.png)

Uniformity：评估嵌入向量是否均匀分布

![image-20211102214614350](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211102214614350.png)

**两个指标描述了：正例应该保持接近，而随机实例的嵌入向量应该分散在超球上。**



## Unsupervised SimCSE

一系列句子$\{x_i\}^m_{i=1}$和$x_i^+=x_i$，让完全相同的正例对起作用的关键因素是通过独立采样的dropout掩码$x_i$和$x^+_i$。

![image-20211103121706882](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103121706882.png)

$h^z_i = f_θ(x_i,z)$，z是一个dropout的随机掩码，一个输入句两次进入编码器，用了不同的两个dropout掩码$z，z'$，训练目标：

![image-20211103121948790](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103121948790.png)

做了一些实验证明dropout比其它数据增强的方法好，而且用一个编码器比用两个编码器好。



**原因：**

alignment分数很稳定，uniformity得到很大降低



## Supervised SimCSE



![image-20211103130922535](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211103130922535.png)









































































