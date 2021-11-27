---
title: 线性回归
date: 2021-11-25 00:41:51 +0800
categories: [人工智能, 机器学习]
tags: [人工智能, 机器学习, 线性回归]
toc: true
comments: false
math: true
pin: true
render_with_liquid: false
---

## 场景

波士顿房价

## 数学问题

### 特征向量

使用特征向量表示数据集中的一个样本，一个样本的特征向量包含了该样本所有特征（feature）的值，但不包含需要预测的那一个属性的值：

$$
\boldsymbol{{x}^{(i)}} = \begin{pmatrix}
    x_{1}^{(i)} \\
    x_{2}^{(i)} \\
    \vdots \\
    x_{n}^{(i)}
 \end{pmatrix}
$$

1. $\boldsymbol{{x}^{(i)}}$是第$i$个样本（sample）的特征向量。
2. $x_{j}^{(i)}$是第$i$个样本的第$j$个特征（feature）的值。
3. $n$是特征的数量，一个特征向量包含了一个样本的$n$个特征的值，在波士顿房价数据集中，$n = 13$。

> 粗体的字母表示矩阵或向量，包含多个数值；未加粗的字母表示标量，表示单个数值。

### 特征矩阵

所有样本的特征向量组成了特征矩阵：

$$
\boldsymbol{X} = \begin{pmatrix}
    x_{1}^{(1)} & x_{2}^{(1)} & \ldots & x_{n}^{(1)} \\
    x_{1}^{(2)} & x_{2}^{(2)} & \ldots & x_{n}^{(2)} \\
    \vdots      & \vdots      & \vdots & \vdots      \\
    x_{1}^{(m)} & x_{2}^{(m)} & \ldots & x_{n}^{(m)}
\end{pmatrix} = \begin{pmatrix}
    {\boldsymbol{{x}^{(1)}}}^{T} \\
    {\boldsymbol{{x}^{(2)}}}^{T} \\
    \vdots                       \\
    {\boldsymbol{{x}^{(m)}}}^{T}
\end{pmatrix}
$$

1. $\boldsymbol{X}$是特征矩阵，每一行的值由一个样本的特征向量的值组成。
2. $m$是样本的数量。
3. $n$是特征的数量。
4. ${\boldsymbol{{x}^{(i)}}}^{T} = \begin{pmatrix} x_{1}^{(i)} & x_{2}^{(i)} & \ldots & x_{n}^{(i)} \end{pmatrix}$是第$i$个样本的特征向量的行向量形式。

> 向量通常是指列向量，加转置符号$^{T}$的向量表示行向量。

### 样本标记

样本标记是需要预测的样本的属性的值：

$$
y^{(i)}
$$

所有样本的标记组成了样本标记的向量：

$$
\boldsymbol{y} = \begin{pmatrix}
    y^{(1)} \\
    y^{(2)} \\
    \vdots  \\
    y^{(m)}
\end{pmatrix}
$$

## 线性回归

### 模型

对于一个样本：

$$
\begin{aligned}
    \hat{y}^{(i)} = & h_{\boldsymbol{w}, b}(\boldsymbol{x^{(i)}}) \\
                  = & \boldsymbol{w}^{T} \boldsymbol{x^{(i)}} + b \\
                  = & w_{1} x_{1}^{(i)} + w_{2} x_{2}^{(i)} + \ldots + w_{n} x_{n}^{(i)} + b
\end{aligned}
$$

对于所有样本：

$$
\boldsymbol{\hat{y}} = h_{\boldsymbol{w}, b}(\boldsymbol{X}) = \boldsymbol{X} \boldsymbol{w} + b
$$

1. $\hat{y}^{(i)}$是模型（model）根据第$i$个样本的特征向量预测出的值。
2. $\boldsymbol{\hat{y}}$是模型根据所有样本的特征矩阵预测出的值，包含每一个样本的预测值。
3. $h_{\boldsymbol{w}, b}$是模型的数学表示，在上面的公式中相当于一个函数名，$\boldsymbol{{w}}$和$b$相当于函数的常量，$\boldsymbol{x^{(i)}}$和$\boldsymbol{X}$相当于函数的自变量。
4. $\boldsymbol{w}= \begin{pmatrix} w_{1} & w_{2} & \ldots & w_{n} \end{pmatrix}^{T}$是权重（weight），$b$是偏置项（bias），$\boldsymbol{w}$和$b$是需要训练学习的模型参数（parameters）。
5. 从直观上理解，需要预测的值应该与样本的特征有关，给每个特征一个权重，表示每个特征对预测结果的影响程度，偏置项则是考虑特征以外的影响。

> 利用向量化的方法，可以一次性对所有样本的特征矩阵进行处理。NumPy内部的并行机制能够很高效地执行多维数组的运算，比自己写循环要快很多。

```python
"""
inputs: (num_samples, num_features)
weights: (num_features, 1)
bias: float
outputs: (num_samples, 1)
"""
outputs = inputs @ weights + bias
```

### 损失函数和代价函数

损失函数（loss function）的值是模型在一个单独的样本上的误差。

线性回归常用的损失函数是平方误差函数（squared error function）：

$$
L(\hat{y}^{(i)}, y^{(i)}) = \frac{1}{2} (\hat{y}^{(i)} - y^{(i)})^{2}
$$

代价函数（cost function）的值是模型在所有样本上的误差的均值，代价函数以$\boldsymbol{w}$和$b$作为自变量。

线性回归的代价函数是所有样本的平方误差的均值：

$$
J(\boldsymbol{w}, b) = \frac{1}{2m} \sum_{i = 1}^{m} (\hat{y}^{(i)} - y^{(i)})^{2}
$$

> 很多人甚至是学术界中，通常不使用损失函数，而是将代价函数称为损失函数。两者的概念通常不必过分细究。

```python
"""
outputs: (num_samples, 1)
labels: (num_samples, 1)
num_samples: int
loss: float
diff: (num_samples, 1)
"""
# implementation 1
loss = 0.5 / num_samples * np.sum((outputs - labels) ** 2)

# implementation 2
diff = outputs - labels
loss = 0.5 / num_samples * diff.T @ diff
```

### 优化目标

代价函数的值越小，代表模型的预测越接近于实际情况。为了使模型表现更好，需要不断降低代价函数的值，优化目标（optimization object）即为求解模型的常量$\boldsymbol{w}$和$b$使得$J(\boldsymbol{w}, b)$最小化。

### 梯度下降

通常使用梯度下降（gradient descent）求解$\boldsymbol{w}$和$b$：

$$
\begin{aligned}
    \boldsymbol{w} & := \boldsymbol{w} - \alpha \frac{\partial}{\partial \boldsymbol{w}} J(\boldsymbol{w}, b) \\
    b & := b - \alpha \frac{\partial}{\partial b} J(\boldsymbol{w}, b)
\end{aligned}
$$

1. $\alpha$是学习率（learning rate），表示参数沿梯度下降的幅度大小。
2. $\frac{\partial}{\partial \boldsymbol{w}} J(\boldsymbol{w}, b)$是$J$在$\boldsymbol{w}$上的偏导：
    $$
    \frac{\partial}{\partial \boldsymbol{w}} J(\boldsymbol{w}, b) = \frac{1}{m} \boldsymbol{X}^{T} (\boldsymbol{\hat{y}} - \boldsymbol{y})
    $$
3. $\frac{\partial}{\partial b} J(\boldsymbol{w}, b)$是$J$在$b$上的偏导：
    $$
    \frac{\partial}{\partial b} J(\boldsymbol{w}, b) = \frac{1}{m} \sum_{i = 1}^{m} (\boldsymbol{\hat{y}} - \boldsymbol{y})
    $$
4. 线性回归中使用梯度下降的优点：在特征数n很大时效果较好。
5. 线性回归中使用梯度下降的缺点：需要选取学习率；需要多次迭代。

> $:=$表示将右边表达式的值赋值给左边的变量。

求偏导的过程：

$$
\begin{aligned}
    \frac{\partial}{\partial w_{1}} J (w_{1}, b) = & \frac{\partial}{\partial w_{1}} (\frac{1}{2m} \sum_{i = 1}^{m} (\hat{y}^{(i)} - y^{(i)})^{2}) \\
    = & \frac{\partial}{\partial w_{1}} (\frac{1}{2m} [(\hat{y}^{(1)}- y^{(1)})^{2} + \ldots + (\hat{y}^{(m)} - y^{(m)})^{2}]) \\
    = & \frac{1}{m} (\hat{y}^{(1)} - y^{(1)}) \frac{\partial}{\partial w_{1}} (\hat{y}^{(1)} - y^{(1)}) + \ldots \\
    & + \frac{1}{m} (\hat{y}^{(m)} - y^{(m)}) \frac{\partial}{\partial w_{1}} (\hat{y}^{(m)} - y^{(m)}) \\
    = & \frac{1}{m} (\hat{y}^{(1)} - y^{(1)}) \frac{\partial}{\partial w_{1}} (w_{1} x_{1}^{(1)} \ldots + w_{n} x_{n}^{(1)} + b - y^{(1)}) + \ldots \\
    & + \frac{1}{m} (\hat{y}^{(m)} - y^{(m)}) \frac{\partial}{\partial w_{1}} (w_{1} x_{1}^{(m)} \ldots + w_{n} x_{n}^{(m)} + b - y^{(m)}) \\
    = & \frac{1}{m} [(\hat{y}^{(1)} - y^{(1)}) x_{1}^{(1)} + \ldots + (\hat{y}^{(m)} - y^{(m)}) x_{1}^{(m)}]
\end{aligned}
$$

1. 在求导时可将求和符号$\sum$展开便于理解。

$$
\begin{aligned}
    \frac{\partial}{\partial \boldsymbol{w}} J(\boldsymbol{w}, b) = & \begin{pmatrix}
        \frac{\partial}{\partial w_{1}} J (w_{1}, b) \\
        \frac{\partial}{\partial w_{2}} J (w_{2}, b) \\
        \vdots \\
        \frac{\partial}{\partial w_{n}} J (w_{n}, b)
    \end{pmatrix} \\
    = &\frac{1}{m} \begin{pmatrix}
        (\hat{y}^{(1)} - y^{(1)}) x_{1}^{(1)} + \ldots + (\hat{y}^{(m)} - y^{(m)}) x_{1}^{(m)} \\
        (\hat{y}^{(1)} - y^{(1)}) x_{2}^{(1)} + \ldots + (\hat{y}^{(m)} - y^{(m)}) x_{2}^{(m)} \\
        \vdots \\
        (\hat{y}^{(1)} - y^{(1)}) x_{n}^{(1)} + \ldots + (\hat{y}^{(m)} - y^{(m)}) x_{n}^{(m)}
    \end{pmatrix} \\
    = &  \frac{1}{m} \boldsymbol{X}^{T} (\boldsymbol{\hat{y}} - \boldsymbol{y})
\end{aligned}
$$

1. 对向量求导，可转化为分别对向量的每一个元素求导，再将所有元素的导数组合为向量。
2. 求和符号中如果是两个数的乘积，通常可以转化为矩阵乘法。

> 上式最后一步可以反推回去验证。

$$
\begin{aligned}
    \frac{\partial}{\partial b} J (\boldsymbol{w}, b) = & \frac{\partial}{\partial b} (\frac{1}{2m} \sum_{i = 1}^{m} (\hat{y}^{(i)} - y^{(i)})^{2}) \\
    = & \frac{\partial}{\partial b} (\frac{1}{2m} [(\hat{y}^{(1)}- y^{(1)})^{2} + \ldots + (\hat{y}^{(m)} - y^{(m)})^{2}]) \\
    = & \frac{1}{m} (\hat{y}^{(1)} - y^{(1)}) \frac{\partial}{\partial b} (\hat{y}^{(1)} - y^{(1)}) + \ldots + \frac{1}{m} (\hat{y}^{(m)} - y^{(m)}) \frac{\partial}{\partial b} (\hat{y}^{(m)} - y^{(m)}) \\
    = & \frac{1}{m} (\hat{y}^{(1)} - y^{(1)}) \frac{\partial}{\partial b} (w_{1} x_{1}^{(1)} \ldots + w_{n} x_{n}^{(1)} + b - y^{(1)}) + \ldots \\
    & + \frac{1}{m} (\hat{y}^{(m)} - y^{(m)}) \frac{\partial}{\partial b} (w_{1} x_{1}^{(m)} \ldots + w_{n} x_{n}^{(m)} + b - y^{(m)}) \\
    = & \frac{1}{m} [(\hat{y}^{(1)} - y^{(1)}) + \ldots + (\hat{y}^{(m)} - y^{(m)})] \\
    = & \frac{1}{m} \sum_{i = 1}^{m} (\boldsymbol{\hat{y}} - \boldsymbol{y})
\end{aligned}
$$

```python
"""
outputs: (num_samples, 1)
labels: (num_samples, 1)
inputs: (num_samples, num_features)
num_samples: int
learning_rate: type=float
weights: (num_features, 1)
bias: float
"""
weights -= learning_rate / num_samples * inputs.T @ (outputs - labels)
bias -= learning_rate / num_samples * np.sum(outputs - labels)
```

## 完整实现

### NumPy

### scikit-learn

### PyTorch

## 正规方程

线性回归模型的模型参数还可以使用正规方程（regulation equation）求闭式解（closed-form solution）。

对代价函数求模型参数的导数，令导数为零，解出的模型参数为最优解：

$$
\boldsymbol{\hat{w}} = (\boldsymbol{X}^{T} \boldsymbol{X})^{-1} \boldsymbol{X}^{T} \boldsymbol{y}
$$

1. $\boldsymbol{\hat{w}} = (\boldsymbol{w}; b)$是$\boldsymbol{w}$和$b$的拼接。
2. $\boldsymbol{X}$不同于之前公式中的$\boldsymbol{X}$。为了方便运算，在最右边多加一列$1$，此时：
    $$
    \boldsymbol{X} = \begin{pmatrix}
        x_{1}^{(1)} & x_{2}^{(1)} & \ldots & x_{n}^{(1)} & 1 \\
        x_{1}^{(2)} & x_{2}^{(2)} & \ldots & x_{n}^{(2)} & 1 \\
        \vdots      & \vdots      & \vdots & \vdots      & \vdots \\
        x_{1}^{(m)} & x_{2}^{(m)} & \ldots & x_{n}^{(m)} & 1
    \end{pmatrix}
    $$
3. 线性回归的代价函数，即平方误差函数，总是一个凸函数，只有一个全局最优解。
4. 使用正规方程不需要使用特征缩放。
5. 线性回归中使用正规方程的优点：不需要选取学习率；不需要多次迭代的训练。
6. 线性回归中使用正规方程的缺点： 求矩阵的逆时间复杂度为$O(n^{3})$，当特征数n很大时非常耗时；有些矩阵不可逆。
7. 如果矩阵$\boldsymbol{X}$中有两个特征的值可以通过一个线性方程联系起来，那么$\boldsymbol{X}^{T} \boldsymbol{X}$不可逆。解决方法是删除冗余的特征。
8. 如果样本数$m$小于特征数$n$，有时候会使$\boldsymbol{X}^{T} \boldsymbol{X}$不可逆。解决方法是删除一些特征，或是使用正则化。

求闭式解的过程：

损失函数：

$$
\begin{aligned}
    J(\boldsymbol{\hat{w}}) = & \frac{1}{2m} \sum_{i = 1}^{m} (\hat{y}^{(i)} - y^{(i)})^{2} \\
    = & \frac{1}{2m} (\boldsymbol{\hat{y}} - \boldsymbol{y})^{T} (\boldsymbol{\hat{y}} - \boldsymbol{y}) \\
    = & \frac{1}{2m} (\boldsymbol{X} \boldsymbol{\hat{w}} - \boldsymbol{y})^{T} (\boldsymbol{X} \boldsymbol{\hat{w}} - \boldsymbol{y})
\end{aligned}
$$

$J$对$\boldsymbol{\hat{w}}$求偏导：

$$
\begin{aligned}
    \frac{\partial}{\partial \boldsymbol{\hat{w}}} J(\boldsymbol{\hat{w}}) = \frac{1}{m} \boldsymbol{X}^{T} (\boldsymbol{X} \boldsymbol{\hat{w}} - \boldsymbol{y})
\end{aligned}
$$

令$\frac{\partial}{\partial \boldsymbol{\hat{w}}} J(\boldsymbol{\hat{w}}) = 0$解得$\boldsymbol{\hat{w}}$：

$$
\begin{aligned}
    \boldsymbol{X}^{T} (\boldsymbol{X} \boldsymbol{\hat{w}} - \boldsymbol{y}) = & 0 \\
    \boldsymbol{X}^{T} \boldsymbol{X} \boldsymbol{\hat{w}} = & \boldsymbol{X}^{T} \boldsymbol{y} \\
    \boldsymbol{\hat{w}} = & (\boldsymbol{X}^{T} \boldsymbol{X})^{-1} \boldsymbol{X}^{T} \boldsymbol{y}
\end{aligned}
$$

```python
"""
labels: (num_samples, 1)
inputs: (num_samples, num_features)
inputs(new): (num_samples, num_features + 1)
inputs_t: (num_features + 1, num_samples)
params: (num_features + 1, 1)
"""
inputs = np.concatenate((inputs, np.ones((inputs.shape[0], 1))), axis=1)
inputs_t = inputs.T
params = np.linalg.inv(inputs_t @ inputs) @ inputs_t @ labels
```

## 总结

```text
    _________    _   __________  ____  _____    _   __   _  __ __________
   / ____/   |  / | / / ____/\ \/ / / / /   |  / | / /  | |/ //  _/ ____/
  / /_  / /| | /  |/ / / __   \  / / / / /| | /  |/ /   |   / / // __/
 / __/ / ___ |/ /|  / /_/ /   / / /_/ / ___ |/ /|  /   /   |_/ // /___
/_/   /_/  |_/_/ |_/\____/   /_/\____/_/  |_/_/ |_/   /_/|_/___/_____/
```
