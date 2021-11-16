# Conditional Augmentation for Aspect Term Extraction via Masked Sequence-to-Sequence Generation  
通过掩码seq2seq生成方式对属性抽取任务进行数据增强



## 模型

![image-20211109154415512](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109154415512.png)

利用该模型生成数据

首先随机采样样本，通过MaskFrag把样本句随机掩码一个片段，利用编码器-解码器架构生成一个新的片段，同时因为利用了标注序列的约束，一般不会生成标记有误的噪声样本。



## 实验

增强效果：

![image-20211109154712223](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109154712223.png)

若不使用标注序列约束：

![image-20211109154737636](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109154737636.png)