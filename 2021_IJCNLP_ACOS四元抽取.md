# Aspect-Category-Opinion-Sentiment Quadruple Extraction with Implicit Aspects and Opinions  
属性-类别-意见-情感四元抽取

（大概意思是这个四元抽取还会抽取隐性的属性词与意见词）



## 摘要

当下的很多ABSA研究都忽略了隐性的属性词与意见词。

提出四元组抽取任务，抽取评论句中所有四元组，包括隐性的属性词与意见词。

为了开展实验，进一步构建了新的数据集：Restaurant-ACOS、Laptop-ACOS

Restaurant-ACOS主要是从SemEval Restaurant中拓展而来，Laptop-ACOS是比SemEval大的多的全新的数据集。这个数据子不仅包含四元组，也标记了隐性属性词、隐性意见词（隐性词的下标被标记为-1）。

代码：https://github.com/NUSTM/ACOS.  



## 引言

![image-20211111145806797](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111145806797.png)

该表说明隐性属性词与隐性意见词占了很大一部分。

当前缺少统一的框架充分讨论并解决隐性的属性词和隐性意见词的问题。

![image-20211111150105392](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111150105392.png)

该图说明，对于句子中没从字面上提到的属性词，可以对它所属的类别进行情感分类。



## 数据集

### Restaurant-ACOS

来自SemEval 2016 以及它的一些变体（TOWE、ASTE），合并了这些数据集，另外标注了隐性的意见词。

### Laptop-ACOS

从亚马逊平台提取了2017-2018年的商品评论，进行了人工标注。（属性类别采用了SemEval中的）

![image-20211111152831169](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111152831169.png)

![image-20211111152948329](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111152948329.png)



## 方法

用四个基线模型对ACOS四元抽取任务进行了基准测试：

- Double-Propagation-ACOS  ：先用原方法抽取属性词、意见词、情感极性，然后根据三元组判断属性类别。
- JET-ACOS  ：同上。第二步：用BERT编码句子，对属性词、意见词的隐状态取平均值。拼接交给全连接层（有N个，N表示类别的数量）来进行二分类：![image-20211111154909564](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111154909564.png)。若$y^c=1$则表示是当前类别。
- TASBERT-ACOS（属性类别情感三元抽取的SOTA模型）  ：为了适应ACOS任务，提出采用输入转换策略，执行以类别-情感为条件的属性-意见联合抽取，然后通过过滤掉无效的属性-意见对构建最终的四元组。![image-20211111160146079](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111160146079.png)，用BERT编码它得到H（$h_r,h_{cs}$)，针对H进行属性-意见联合抽取，处理成一个序列标注任务{BA,IA,BO,IO,O}。![image-20211111160508641](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111160508641.png),（CRF的输入就是$h_r$)。此时得到了所有的属性词、意见词和原有的类别、情感，枚举所有可能的四元组，然后平均属性词与意见词的向量，拼接起来绕过线性层判断是否为正确的四元组：![image-20211111161004331](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111161004331.png)。
- Extract-Classify-ACOS  ：首先是原模型进行属性、意见联合抽取。然后给定抽取出来的属性-意见对来预测类别-情感。

### Extract-Classify-ACOS

![image-20211111165116000](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111165116000.png)

中间这个显示属性意见联合抽取就是基于CRF用{BA,IA,BO,IO,O}进行序列标注。

用[CLS]的隐状态判断是否存在隐性属性词与隐性意见词。

通过这三个任务，得到了两个候选池，一个池子里装着属性词，一个池子里装着意见词。

然后枚举所有的情况，假设有N个属性类别，则有N个全连接层对其进行分类，每个全连接层代表一个属性，也就是每个属性-意见对都会被作为N次输入，进行一个四分类，判断它是否有效，若有效是代表了什么情感极性。



## 实验

只有四个元素与真实的四元素组合完全相同时，才认为是正确的。

![image-20211111171403135](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111171403135.png)

DP很差：在评论句中仅使用规则很难识别多个隐式元素及其复杂组合

### 隐性属性与意见建模的效果

![image-20211111171723536](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111171723536.png)

### 样本研究

![image-20211111172154295](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211111172154295.png)



















