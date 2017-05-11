title: 决策树笔记
date: 2016-07-08 19:53
categories: machine_learning
tags: [机器学习,决策树]
description: 记录了决策树的基本内容，包括熵、信息增益、信息增益比、基尼指数的概念；ID3决策树、C4.5决策树、CART决策树的构建，最后简要记录了剪枝。
---

一般的，一棵决策树包含一个**根结点**、若干个**内部结点**和若干个**叶结点**；**叶结点**对应于决策结果，其他每个结点则对应于一个属性测试；每个结点包含的样本集合根据属性测试的结果被划分到子结点中；根结点包含样本全集。从根结点到每个叶结点的**路径**对应了一个判定测试序列。
例如，周志华老师的《机器学习》中的西瓜例子，叶子结点对应了决策结果（好瓜、坏瓜）；根结点（纹理）和内部结点（根蒂、触感、色泽等）对应了属性测试。

![Alt text | center | 480*0](http://7qn7rt.com1.z0.glb.clouddn.com/ml/decision_tree_example.jpg)








决策树学习的**目的**是为了产生一棵泛化能力强，即处理未见示例能力强的决策树。

决策树学习基本算法如下：

![Alt text | center | 600](http://7qn7rt.com1.z0.glb.clouddn.com/ml/decision_tree_algrithom.jpg)








决策树学习的关键是上述算法的第8行，即**如何选择最优划分属性**。一般而言，随着划分过程的不断进行，我们希望决策树的分支结点所包含的样本尽可能属于同一类别，即结点的“纯度”（purity）越来越高。

根据属性选择方法的不同，建立决策树主要有ID3、C4.5、CART三种算法。


## ID3

信息论与概率统计中，**熵**（entropy）是随机变量不确定性的度量。
设X是一个取值有限的离散随机变量，其概率分布为
$$P(X=x_i) = p_i,\quad i=1,2,\cdots,n$$
则随机变量X的**熵**定义为
$$H(X) = -\sum_{i=1}^{n}p_i \log p_i$$

设随机变量 $(X,Y)$ 的概率分布为
$$P(X=x_i,Y=y_j) = p_{ij},\quad i=1,2,\cdots,n;\ j=1,2\cdots,m$$
**条件熵**（conditional entropy）$H(Y\ |\ X)$ 表示已知随机变量X的条件下随机变量Y的不确定性，其定义为
$$H(Y\ |\ X) = \sum_{i=1}^{n}p_iH(Y\ |\ X=x_i)$$
这里，$p_i = P(X=x_i),\quad x=1,2,\cdots,n$。

**信息增益**：特征 $a$ 对训练数据集 $D$ 的信息增益 $g(D, a)$，定义为集合 $D$ 的经验熵 $H(D)$ 与特征 $a$ 给定条件下 $D$ 的经验条件熵 $H(D\ |\ a)$ 之差，即
$$g(D, a) = H(D) - H(D\ |\ a)$$
一般地，熵与条件熵之差称为互信息（mutual entropy），决策树学习中的信息增益等价于训练数据集中类与特征的互信息。

显然，信息增益 $g(D, a)$ 表示由于特征 $a$ 而使得对数据集 $D$ 的分类的不确定性减少的程度，因此可以选择信息增益最大的属性作为划分属性。

设训练数据集为 $D$，$\mid D \mid$ 表示样本个数。有 $K$ 个类 $C\_k$，$k=1,2,\cdots,K$，$\mid C\_k \mid$ 为属于类 $C\_k$ 的样本个数，$ \sum\_{i=1}^{K} \mid C\_k \mid = \mid D \mid $。设特征 $a$ 有 $n$ 个不同的取值$ \{a^1, a^2, \cdots , a^n\}$，根据特征 $a$ 的取值将 $ D $ 划分为 $ n$ 个子集 $\ D\_1, D\_2, \cdots, D\_n$，$ \mid D\_i \mid$ 为 $D\_i$ 的样本个数，$\sum\_{i=1}^{n} \mid D\_i \mid = \mid D \mid $。记子集 $ D\_i $ 中属于类 $C\_k$ 的样本的集合为 $ D\_{ik} $，即 $ D\_{ik} = D\_i \cap C\_k$，$ \mid D\_{ik} \mid$ 为 $D\_{ik}$ 样本的个数，信息增益的具体计算如下：

-----

**输入**：训练数据集 $D$ 和特征 $a$；
**输出**：特征 $a$ 对训练数据集 $D$ 的信息增益 $g(D, a)$。

1. 计算数据集 $D$ 的经验熵
$$H(D) = -\sum_{k=1}^{K} \frac{|C_k|}{|D|}\log_2 \frac{|C_k|}{|D|}$$
2. 计算特征 $a$ 对数据集 $D$ 的经验条件熵
$$H(D\ |\ a) = \sum_{i=1}^{n} \frac{|D_i|}{|D|}H(D_i) = -\sum_{i=1}^{n}\frac{|D_i|}{|D|} \sum_{k=1}^{K} \frac{|D_{ik}|}{|D_i|} \log_2 \frac{|D_{ik}|}{|D_i|}$$
3. 计算信息增益
$$g(D, a) = H(D) - H(D\ |\ a)$$

------

**ID3决策树学习算法就是以信息增益为准则来选择划分属性**，即在算法第8行选择属性 $a\_* = \mathop{\arg \max}\limits\_{a \in A} g(D,a)$。

## C4.5

如果有一个属性可能的取值很多，甚至多到等于样本的数量，每个样本又正好对应一个取值，那么经过这个属性的决策后样本纯度提升会最大。但是这样的一个决策树是不具备泛化能力的，无法有效的预测新样本的分类。这种情况体现了信息增益准则的一个缺点：**对取值数目较多的属性有所偏好**。

C4.5使用**信息增益比**（information gain ratio）来选择最优划分属性，从而消除上述偏好。

特征 $a$ 对训练数据集 $D$ 的信息增益比 $g_R(D, a)$ 定义为其信息增益 $g(D, a)$ 与训练数据集 $D$ 的经验熵之比：
$$g_R (D, A) = \frac{g(D, A)}{H(D)}$$

信息增益比准则对取值数目较少的属性有所偏好。C4.5并不直接选择信息增益比最大的属性进行划分，而是使用了一个启发式：先从候选划分属性中找出信息增益高于平均水平的属性，再从中选择信息增益比最高的。

## CART

CART（Classification And Regression Tree，分类与回归树）决策树使用**基尼指数**（Gini index）来选择划分属性。

分类问题中，假设有 $K$ 个类，样本点属于第 $k$ 类的概率为 $p_k$，则概率分布的基尼指数定义为
$$\begin{align*} \text{Gini}(p) = \sum_{k=1}^{K} p_k (1- p_{k}) = 1 - \sum_{k=1}^{K}p_k^2 \end{align*}$$
对于给定的样本集合 $D$，其基尼指数为
$$\text{Gini} (D) = 1 - \sum_{k=1}^{K} \left( \frac{\mid C_k \mid}{\mid D \mid} \right) ^2$$
这里，$C_k$ 是 $D$ 中属于第 $k$ 类的样本子集，$K$ 是类的个数。

样本集合 $D$ 根据属性 $a$ 的取值可以分为 $n$ 部分：$D_1, D_2, \ldots, D_n$。则，在属性 $a$ 的条件下，集合 $D$ 的基尼指数定义为
$$\text{Gini} (D, a) = \sum_{i=1}^{n} \frac{\mid D_i \mid }{\mid D \mid} \text{Gini} (D_i) $$

基尼指数 $\text{Gini} (D)$ 表示集合 $D$ 的不确定性，基尼指数 $\text{Gini} (D, a)$  表示经属性 $a$ 分割后集合 $D$ 的不确定性。基尼指数越大，样本的不确定性也就越大。



CART决策树在候选属性集合 $A$ 中，选择那个使得划分后**基尼指数最小**的属性作为最优划分属性，即 $a\_* = \mathop{\arg \max} \limits\_{a \in A} \text{Gini}(D, a)$。


> **基尼指数与熵**
> 首先，对 $-\ln x$在 $x=1$ 处泰勒展开，忽略高阶无穷小，可以得到$-\ln x \approx 1 - x$
> 若熵中的对数不是以2为底，而是以e为底：
> $$\begin{align*} H(D) &= -\sum_{k=1}^{K}p_k \ln p_k \\ &\approx \sum_{k=1}^{K} p_k (1-p_k) \\ &= \text{Gini}(D) \end{align*} $$

## 剪枝

剪枝（pruning）是决策树学习算法对付“过拟合”的主要手段。主动去掉一些分支，防止决策树把训练集自身的特点当作所有数据都具有的一般性质。

决策树剪枝的基本策略有“预剪枝”（prepruning）和“后剪枝”（postpruning）。预剪枝是指在决策树生成过程中，对每个结点在划分前先进行估计，若当前结点的划分不能带来决策树泛化性能提升，则停止划分并将当前结点标记为叶结点；后剪枝是先从训练集生成一颗完整的决策树，然后自底向上地对非叶结点进行考察，若将改结点对应的子树替换为叶结点能带来决策树泛化性能提升，则将该子树替换为叶结点。

预剪枝训练时间短，但有欠拟合的风险。
后剪枝泛化性能往往优于预剪枝，但训练时间开销比未剪枝决策树和预剪枝决策树都要大得多。

##  参考资料

- 七月算法机器学习课程
- 机器学习 周志华
- 统计学习方法 李航