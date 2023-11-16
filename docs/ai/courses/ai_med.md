---
comment: true
counter: true
---

# 人工智能与医学图像分析

!!! abstract
    A HARDCORE(FAKE) course of Dr.Zhao.

## Lesson 1

### 临床问题

主要任务分类：

- 诊断
- 定位
- 分割
- 生成
- 预测

### 人工智能

> *如何评判一张医学图像的好坏*

**金标准**：医生主观判断

传统特征工程（量化指标）：

- 边缘清晰
- 对比度高
- 信噪比高
- 没有运动伪影
- ...

机器学习本质是归纳**样本分布**

可解释性：传统特征工程与人工智能的区别
（近几年有大量的研究改善）

### 从低维到高维特征

> 为什么要高维特征

将非线性问题转换成线性问题（简单问题）

??? answer "AI History (可略过)"
    ### AI History

    #### 贝叶斯估计

    先验知识对结果影响很大（一个例子）
    $$
    P(A|B) = \frac{P(B|A)P(A)}{P(B)}
    $$

    #### 马尔可夫链
    略
    #### Neuron/Perception
    神经元模型，略
    $$
    Y=A(Wx+b)
    $$
    #### Alan Turing
    - 信息论
    - initial seed的重要性
    #### Nearest Neighbor search
    > 聚类和分类问题

    - K-Means
    - SVM
  
    某些任务里，传统的机器学习也会优于深度学习
    #### Multiple Layer Perception

    - Forward
    - Backward
  
    #### AI Winter
    略
    #### RNN
    略
    #### LeNeT
    1998 Convolution Neural Network
    #### LSTM
    略
    #### AlexNet
    ImgaeNet by LeeFeifei，略
    #### GAN
    2014 lan Goodfellow
    #### Transformer
    2017 Google Brain Attention Is All You Need
    #### Diffusion
    略
