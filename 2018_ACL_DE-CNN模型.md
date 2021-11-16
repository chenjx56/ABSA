# Double Embeddings and CNN-based Sequence Labeling for Aspect Extraction  
面向属性抽取的基于双嵌入词向量与卷积网络的序列标注

## 模型 DE-CNN

![image-20211109150853627](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109150853627.png)

领域嵌入向量必须是完全吻合的领域，不可以是包含关系（例如笔记本电脑领域不可以用电子设备领域的数据去训练）



两个嵌入向量拼接在一起，交给卷积层去提取高级特征，从而辅助softmax进行分类。



## 实验结果

![image-20211109151033809](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109151033809.png)

