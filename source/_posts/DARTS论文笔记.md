---
title: DARTS论文笔记
date: 2020-01-19 12:13:59
tags: [NAS]
categories: [Deep learning]
mathjax: true
---
论文链接：[DARTS: Differentiable Architecture Search](https://arxiv.org/abs/1806.09055)

### 1. 概述

DARTS 是用于自动构建神经网络结构并优化网络参数，基于梯度的算法，通过连续化神经网络的*architecture representation* 以便使用梯度来优化网络结构，时间开销较传统的RL法、evolution法相比降低了很多。

### 2. 算法

#### 2.1 算法内容

![image-20191128234835766](C:\Users\85126\AppData\Roaming\Typora\typora-user-images\image-20191128234835766.png)



0. 一个完整的网络由多个cell组成，cell用图来表示，每个cell有两个输入结点和一个输出结点，图的每一个结点 $x^{(i)}$ 作为*latent representation*（神经网络中间层的输出，如feature map），由之前结点与对应的operation相乘再求和得到，图的边表示一种运算$o^{(i,j)}$ ，用于转变$x^i$ 到 $x^j$：

$$
x^j = \sum_{i<j}o^{(i,j)}(x^i)
$$

1. 对图中每一条边，将操作 $ o^{(i,j)}$ 变为 $\bar{o}^{(i,j)}$ 以连续化 *search space* ，然后将两结之间运算 $o$ 的输出与其网络参数混合在一起构建成 $\alpha^{(i,j)}$ ，其中
   $$
   \bar{o}^{(i,j)}(x)=\sum_{o \in \mathcal O}\frac{exp(\alpha_o^{(i,j)})}{\Sigma_{o'\in \mathcal O} exp(\alpha_{o'}^{(i,j)})}o(x)
   $$
   
2. 计算梯度 $\bigtriangledown_\alpha \mathcal{L}_{val}(w-\xi\bigtriangledown_w\mathcal{L}_{train}(w,\alpha),\alpha)$ ，通过梯度下降优化网络结构 $\alpha$，具体地：

   记  $w'=w-\xi \bigtriangledown_\alpha \mathcal L_{train}(w,\alpha)$ ，可知：

$$
\begin{equation}
\begin{aligned}
&\bigtriangledown_\alpha \mathcal{L}_{val}(w^*(\alpha),\alpha)\\
&\approx \bigtriangledown_\alpha \mathcal{L}_{val}(w-\xi \bigtriangledown_w
\mathcal{L}_{train}(w,\alpha),\alpha)\\
&=\bigtriangledown_\alpha \mathcal{L}_{val}(w',\alpha)-\xi 
\bigtriangledown_{\alpha,w}^2 \mathcal{L}_{train}(w,\alpha)
\bigtriangledown_{w'}\mathcal{L}_{val}(w',\alpha)\\
\end{aligned}
\end{equation}
$$

​		其中，二阶导数可近似为：
$$
\bigtriangledown_{\alpha,w}^2 \mathcal{L}_{train}(w,\alpha)
\bigtriangledown_{w'}\mathcal{L}_{val}(w',\alpha)
\approx \frac{\bigtriangledown_\alpha \mathcal L_{train}(w^{+},\alpha)-
\bigtriangledown_\alpha \mathcal L_{train}(w^-,\alpha)}{2 \epsilon}
\\where \quad w^\pm = w \pm \epsilon \bigtriangledown_{w'} \mathcal L_{val}(w', \alpha)
$$


1. 计算梯度 $\bigtriangledown_w \mathcal{L}_{train}(w,\alpha)$ ，并通过梯度下降优化 $w$
2. 重复2，3步骤直到收敛
3. 离散化以得到网络结构

