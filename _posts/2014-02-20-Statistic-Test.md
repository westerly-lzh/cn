---
layout: post
title: 统计假设检验实验设计类别及其对应检验方法
categories:
- 统计
tags:
- 实验设计
- 检验方法

---

## 设计类型及其对应检验方法

![alt text](/media/img/test.png "统计检验设计")



## 正态性检验 

* Kolmogorov-Smirnov Tests，检验*单一*样本是不是服从预先假设的特定分布方法.检验方法是以样本数据的累计频数分布与特定理论分布比较，若两者间的差距很小(p-value足够大)，则推论该样本取自某特定分布族。
	* H_0：样本所来自的总体分布服从某特定分布
	* H_1：样本所来自的总体分布不服从某特定分布
	* 令F_0(x)表示预先假设的理论分布，F_n(x)表示随机样本的累计概率(频率)函数，设 D=max|F_0(x) - F_n(x)|
	* 结论：当D > D(n, a), 则拒绝H0, 反之则接受H_0假设。其中D(n,a) 是显著水平为a且样本容量为n时的拒绝临界值（查表可得）
	* 单样本 K-S 检验可以将一个变量的实际频数分布与高斯分布(Gaussian)、均匀分布(Uniform)、泊松分布(Poisson)、指数分布(Exponential)等进行比较
	* R 语言中对应的方法为ks.test(x, y, ...,alternative = c("two.sided", "less", "greater"),exact = NULL),其中x为观测值向量，y要么为观测值向量，要么为代表累积分布函数的字符变量要么为一个真正的累计分布函数。做正态性检验时y可以为pnorm
* Shapiro-Wilk Normality Test，Shapiro-Wilk检验用于验证一个随机样本数据是否来自正态分布。在实际使用中，除了Shapiro-Wilk检验的结果，还应配上normal probability plot，提供样本分布形状方面的非量化信息。设 $Y_1< Y_2 < … < Y_n $是数量是n的一个排序的样本，需要验证其是否符合正态分布。
	* H_0: 样本数据与正态分布没有显著区别。
	* H_1: 样本数据与正态分布存在显著区别。
	* w = $(Σa_i\*y_i )^2/Σ(y_i-mean(y))^2$ ,如果样本数据的确来自一个正态分布，统计量W的分子和分母均会趋向一个常数：(n -1)\*σ2的估计值。对于非正态分布的数据而言，分子和分母通常不会趋向同一个常数。它的值越高，越表示样本与正态分布匹配
	* 适用于样本含量《= 50的情况

* 其他方法
	* 在nortest包中
		* lillie.test()可以实行更精确的Kolmogorov-Smirnov检验。
    	* ad.test()进行Anderson-Darling正态性检验。
    	* cvm.test()进行Cramer-von Mises正态性检验。
    	* pearson.test()进行Pearson卡方正态性检验。
    	* sf.test()进行Shapiro-Francia正态性检验。适用于50<n<100的情况
    * 在fBasics包中
    	* normalTest()进行Kolmogorov-Smirnov正态性检验。
    	* ksnormTest()进行Kolmogorov-Smirnov正态性检验。
    	* shapiroTest()进行Shapiro-Wilk's正态检验。
    	* jarqueberaTest()进行jarque-Bera正态性检验。
    	* dagoTest进行D'Agostino正态性检验。
    	* gofnorm采用13种方法进行检验，并输出结果。
    	
## 方差齐次性检验
* Bartlett's Test，在样本为正态性的假设条件下，使用该检验方法。基于此原因，统计学家建议使用下面介绍的 *Levene's Test* 和*Brown-Forsythe Test*
	* R中作Bartlett检验的方法为bartlett.test(x,...)

* Levene's Test，Levene检验主要用于检验两个或两个以上样本间的方差是否齐性。要求样本为随机样本且相互独立。常见的Bartlett多样本方差齐性检验主要用于正态分布的资料,对于非正态分布的数据,检验效果不理想。Levene检验既可以用于正态分布的资料,也可以用于非正态分布的资料或分布不明的资料,其检验效果比较理想。	* R语言中car包有leveneTest(y, …)方法进行该检验,lawstat包中有levene.test()方法* Brown-Forsythe Test

		
