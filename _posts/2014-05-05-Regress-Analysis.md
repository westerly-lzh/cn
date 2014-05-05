---
layout: post
title:  回归分析
categories:
- 统计
tags:
- 回归分析

---

## 线性模型
一般化的模型可以表示为$$Y=f(X_1,X_2,X_3)+\varepsilon$$ 但当$f$较为复杂时，通过数据是很难对其进行较为直接准确地预测的，于是通过增加更加严格的条件，把$f$限制为一次线性结构，可以得到$$Y=\beta_0 + \beta_1X_1+\beta_2X_2+\beta_3X_3+ \dots +\varepsilon$$,当给定对应模型的实际数据时，模型的样式为$$y_i = \beta_0 + \beta_1x_{1i} +\beta_2x_{2i} + \beta_3x_{3i}+\varepsilon_i$$ 采用矩阵形式可以表示为$$y=X\beta + \varepsilon$$其中 
$$
y=(y_1,y_2,\dots,y_n)^T,\\\\
\beta =(\beta_0,\beta_1,\dots,\beta_3)^T,\\\\
X= \begin{vmatrix}
1 & x\_{11} & \dots & x\_{13} \\\\
1 & x\_{21} & \dots & x\_{23} \\\\
\dots &\dots & \dots & \dots \\\\
1 & x\_{n1} & \dots & x\_{n3}
\end{vmatrix} 
$$

## 最小二乘估计
可以通过定义使得残差最小的$\beta$为最优拟合，则通过求取
 $$\min \sum \varepsilon_i^2 = \varepsilon^T\varepsilon = (y-X\beta)^T(y-X\beta)\\\\=y^Ty-2\beta X^Ty+\beta^TX^TX\beta$$,对$\beta$分别求偏导，并设偏导为0，可以得到$\beta$的估计，$$\hat \beta = X(X^TX)^{-1}X^Ty\\\\ 预测值: \hat y = X \hat \beta,\\\\ 残差： \hat \varepsilon = y - X \hat \beta = (I - X(X^TT)^{-1}X^T)y,\\\\残差平方和: \hat \varepsilon ^T \hat \varepsilon = y^T (I - X(X^TT)^{-1}X^T) (I - X(X^TT)^{-1}X^T)y$$


##  拟合效果

通过定义方差解释百分比，可以测量拟合效果$$R^2 = 1 - \frac{\sum(\hat y_i - y_i)^2}{\sum(y_i - \bar y)^2} = 1- \frac{RSS}{Total \ SS}$$ 当$R^2$越接近1时，拟合效果越好。


## 假设检验
$$H_0 : \beta_1 = \dots \beta_{p-1} =0$$

在线性拟合中，如果一个模型$\omega$使用的指标变量$(X)$数目小于小于模型$\Omega$使用的指标变量的数目，且$\omega$ 的 指标变量集合是$\Omega$ 指标变量集合的子集，如果两个模型拟合效果比较接近，那么我们倾向于使用小便量集合的模型，那么如何度量这两个模型的的拟合效果的差异程度呢？可以通过定义$$\frac{RSS_\omega- RSS_\Omega}{RSS_\Omega}$$这样一个度量来衡量.如果$\Omega$的维度为q，$\omega$的维度为q,则$$\frac{RSS_\omega - RSS_\Omega}{q-p} \sim \sigma^2\chi_{q-p}^2\\\\ \frac{RSS_\Omega}{n-q} \sim \sigma^2\chi_{n-q}$$那么可以定义$F$分布为$$F = \frac{(RSS_\omega - RSS_\Omega)/(q-p)}{RSS_\Omega/(n-p)} \sim F_{q-p,n-q}$$