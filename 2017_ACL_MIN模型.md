# Deep Multi-Task Learning for Aspect Term Extraction with Memory Interaction  
使用记忆交互的深度多任务学习进行属性抽取



## 模型 MIN

![image-20211109151258176](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109151258176.png)

![image-20211109152245139](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109152245139.png)

![image-20211109153401521](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109153401521.png)

定义了三种操作：

- READ：从历史隐藏状态中挑选$n_m$个建立$H^A_t与H^O_t$
- DIGEST：$m^O_t$是蒸馏出来的，$m^A_t$是交互中产生的
- INTERACT：交互过程看如下公式

### DIGEST

这是意见词的蒸馏过程

![image-20211109152355558](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109152355558.png)

### INTERACT

这是属性词与意见摘要交互的过程，意见词与属性摘要交互的过程与其类似。

![image-20211109152406188](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109152406188.png)

其中$h^A_t$就是$m^A_t$，即属性摘要。



S-LSTM通过拼接属性与意见摘要，判断一个句子是否为情感句（作者认为情感句才能进行属性词与意见词的判断。）

## 训练

![image-20211109152928430](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109152928430.png)

用这个损失函数，反向传播属性词（意见词）预测，每词都计算。

![image-20211109153053704](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109153053704.png)

用这个损失函数，反向传播判断句子是否为意见句的预测，每个句子算一次。



## 实验

![image-20211109153138516](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109153138516.png)

![image-20211109153147079](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20211109153147079.png)