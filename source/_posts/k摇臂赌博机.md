---
title: k摇臂赌博机
date: 2020-03-04 21:41:58
tags: k-armed bandit
categories: 强化学习
Mathjax: true
---

​	在强化学习中，为最大化收益，我们可以贪心地选取已知收益最大的决策，但是在未探索的决策中，可能存在比已知的最好决策更好的决策，因此我们需要在一些时候选择去探索，平衡*exploration*和*exploitation*的一个方法是$\epsilon$-*greedy*，即以$1-\epsilon$的概率去贪心，以$\epsilon$(较小)的概率去探索。那么如何确定一个决策的收益呢？假定一个k摇臂赌博机的例子——有k中决策方式，每一种决策的收益不是常数，且不同决策会带来不同的收益（*reward*）。以下为几种经典的方法来确定*reward*并进行决策。

### 1. Action-value Methods

​	核心思想：决策a的期望收益定义为a已出现的收益的平均值，通过$\epsilon$-*greedy*算法，我们可以得出一个较好的决策方案。
$$
Q_t(a)=\frac{\sum_{i=1}^{t-1}R_i \cdot \mathbb{1}_{A_i=a}}{\sum
_{i=1}^{t-1} \mathbb{1}_{A_i=a}}\\
A_t=\mathop{\arg\max}_aQ_t(a)
$$
当$t\to\infty$时，$Q_t(a) \to q_*(a)$, 

其中 $q_*(a)=\mathbb{E}[R_t|A_t=a]$

* 为节约随时间改变引起的计算开销增大，可将Q表示为差分方程的形式：

$$
Q_{n+1}=Q_n+\frac{1}{n}[R_n-Q_n]
$$

该方程的普遍形式为：
$$
NewEstimate \leftarrow OldEstimate + StepSize*error
$$

* 对于收益会随着时间变化而变化的情况，使用求平均的方式不能很好的确定真实收益，因为以前的收益并不能很好地反映变化后的真实收益，因此可以通过改变stepsize，达到让近期收益占的权重大一些的效果：

$$
Q_{n+1}=Q_n+\alpha[R_n-Q_n]\\
...\\
Q_{n+1}=(1-\alpha)^nQ_1+\sum\limits_{i=1}^n\alpha(1-\alpha)^{n-i}R_i
$$

* 通过将初始的$Q_1(a)$设置得较大，可以让Agent偏向于去努力尝试更好的决策，一个很大的初始Q值甚至可能比最好的决策的Q值大。



### 2. Upper-Confidence-Bound Action Selection (UCB)

​	$\epsilon$-*greedy*的一个不好的地方是在强制Agent去以某一概率进行随机试探，这样的试探对于接近最好或者完全未知的决策没有偏好，可以在Q函数增加一个附加项来表示这种偏好，agent选择方式定义为：
$$
A_t=\mathop{\arg\max}_a\left [Q_t(a)+c\sqrt\frac{\ln t}{N_t(a)}\right]
$$
随着试探a的次数增大$\ln t$ 和$N_t(a)$都增大，但是$\ln{}$增长速率比$N_t$小，因此，随着选择某一个决策a的次数增多，它的preference就会下降。当$N_t=0$时，该函数就会退化成一个单纯的argmax.



### 3. Gradient Bandit Algorithms

通过概率来选择决策，用softmax定义选择决策a的概率为：
$$
\Pr(A_t=a)=\frac{e^{H_t(a)}}{\sum_{b=1}^{k}e^{H_t(b)}}=\pi_t(a)
$$
其中$H_t$表示偏好函数，可以通过以下迭代公式来更新：
$$
H_{t+1}(A_t)=H_t(A_t)+\alpha(R_t-\bar{R}_t)(1-\pi_t(A_t)),\\
H_{t+1}(a)=H_t(a)-\alpha(R_t-\bar R_t)\pi_t(a), \quad a\neq A_t
$$
其中$\alpha>0$代表一个step-size参数（学习率），$\bar{R}_t$表示t之前**所有**的reward的平均

​	通过公式可看出，每次迭代更新时，对于选择到的决策，若当前reward比之前的平均reward大时，该决策的偏好函数值增大，否则减小；对于没选择到的决策，其变化与选择到的决策相反。

* 以上迭代公式与SGD形式是等价的：
  $$
  H_{t+1}(a)=H_t(a)+\alpha \frac{\partial \mathbb E[R_t]}{\partial H_t(a)}\\
  \mathbb E[R_t]=\sum\limits_x \pi_t(x)q_*(x)
  $$
  
* 要特别注意，$\sum\limits_x\pi_t(x)=1$，因此有：$\sum_x \frac{\partial \pi_t(x)}{\partial H_t(a)}=0$

  