---
layout: post
title: R语言统计分析学习系列之多元统计分析
categories:
- statistic
- 统计
tags:
- 多元分析

---

这里仍然是介绍在R中怎样做多元分析，其中包括判别分析（Discriminat Analysis），聚类分析，主成分分析，因子分析，对应分析，典型相关分析。主要参考了多元统计分析及R语言建模(王斌会，第二版)，R语言与统计分析(汤银才)及R语言实践(Rotert I. Kabacoff)。本人是学管理学出身，对一些比较深奥的统计建模过程不是很了解，所以这里只是讲述如何使用R来做上述的分析任务，具体的推理过程就略过了。原本想再这上面采用mathjax的，但是jekyll渲染后的效果不是很好，就先作罢了。
####1 判别分析(Discriminat Analysis)
判别分析是用于判别样本具体所属类型的一种统计分析方法，这里的前提是总体的分类已经确定。这里主要介绍Fisher判别和Bayes判别。Fisher判别属于确定性判别，这里介绍其中的线性判别和距离判别；而Bayes判别属于概率判别。

#####1.1 线性判别
<!--

线性判别采用线性判别函数 $$Y = {a_1}{X_1} + {a_2}{X_2} + \cdots +{a_n}{X_n}$$,使得该函数能够根据$X_i$ 的值区分各个样品。在该线性判别函数中，重要的是需要根据已有样本计算出的系数向量$\overrightarrow{a}$能使得各个类别之间的变异尽量大，而类内的变异尽量小。这里按照上述的要求如何求得$\overrightarrow{a}$就不做表述，但在有$Y = \mathbf{aX}$后,我们可以把各个类别的样品指标均值带入$Y_i = \overrightarrow{a}\overline{X_i}$, 求得$${Y_1},{Y_2},{\cdots},{Y_n}$$，其中$Y_1 < Y_2  < \cdots < Y_n$,则新样本$X$指标带入指标函数得到$Y$后，只需要先判断$Y$在 $[Y_i , Y_{i+1}]$,然后比较$Y$与$\frac{Y_i + Y_{i+1}}{2}$的大小，当$Y > \frac{Y_i + Y_ {i+1}}{2}$时，$X$属于$Y_{i+1}$类别，反之属于${Y_i}$类别

-->
线性判别采用线性判别函数，这里建立的线性判别函数为Y = a_1 X_1 + a_2 X_2 + … + a_n X_n ,使得该函数可以根据样本的不同指标(X_i)来区分样本。在该线性判别函数中重要的是求得判别函数的系数，使得在同组内部的变异尽可能的小，而不同组之间的变异尽可能的大，这里的系数也就是Fisher判别方法中的投影方向。在R中我们可以使用MASS包中的lda函数来进行线性判别分析。取得判别结果后，我们可以使用样品的真是分类和预测分类进行列链表分析，在使用lda时，其结果中线性判别系数(Coefficients of linear discriminants)一项，可能出现多组，而(Proportion trace)一项则指明了每一组线性判别系数在对样本进行分组判别时的贡献大小。

	data(iris)   
	ld <- lda(Species ~ Sepal.Length + Sepal.Width + Petal.Length + 	Petal.Width,data=iris)
	ld ##查看判别效果
	z <- predict(ld)
	cbind(iris$Species,z$class) ##对比实际类型和判别分类
	tab <- table(iris$Species,z$class) ##列链表显示判别结果
	tab
	diag(prop.table.table(tab,1))


在做判别分析中，要始终牢记样本集中样本的指标是否等方差，及协方差阵是否相同；不同的方差和协方差，在使用lda时，如果没有指明，其判别结果会有不同。

#####1.2 距离判别

距离判别中重要的是选择适当的距离函数，在下面的聚类分析中会再次提到距离的定义，所采用的距离定义不同，会导致不同的判别和聚类结果。在判别分析中比较常用的距离是马氏距离，但具体采用何种距离定义方式要根据实际的问题来决定。这里根据距离定义，会产生距离函数W(x)，对于k个总体,某个样本到各个总体的距离最小为 W_i = min(W(x)),则所要判定的样本可归入第i类别。在R中可以使用MASS包中的discrim.dist函数进行判别，默认情况下，该函数假定样本的方差不等。在王斌会那本书上，有一个mvstats包，提供了有供距离判别的函数，但没有找到该包，所以这里就作吧了。

#####1.3 Bayes判别
Bayes判别相比较前面的线性判别和距离判别，考虑了各个总体出现的概率(即先验概率)和错判的损失函数，综合考虑这两个条件，则使得损失函数最小化，可以作为判别分类的标准.在R中，我们依然可以使用lda函数进行Bayes判别，与之前说的线性判别不同的是，我们只要指定先验概率参数(prior)，使用lda进行Bayes判别时，情况变得有些复杂，具体可以参见lda的使用帮助。

以下观点来自多元统计分析与R语言建模(王斌会,第二版):
>判别分析中，距离判别和Fisher判别(线性判别)对判别变量的分布类型没有要求，两者只要求各类总体的二阶矩存在，而Bayes判别则要求知道变量的分布类型；当仅有两个总体时，若他们的协方差矩阵相同，则距离判别和Fisher判别等价，当判别变量服从正态分布时，他们还和Bayes判别等价，而当两类的协方差矩阵不同时，Fisher判别使用的是他们的额合并协方差矩阵，距离判别也与Bayes判别不同

####2 聚类分析(Cluster Analysis)

聚类分析和上面讲过的判别分析虽然都是要把样本分类，但是其区别在于判别分析是在已知总体的分类情况下，把新样本归到已知的某一类中；而聚类分析是在未知总体的分类情况下，把样本集按照某种分类方法区分成若干类。在聚类分析中常区分为Q型聚类和R型聚类，Q型聚类分析时对样品进行分类处理，而R型聚类是对变量进行分类处理，在经济管理中多用Q型聚类(王斌会，多元统计分析及R语言建模)。下表列出了聚类分析中一些常用的统计量。

聚类统计量				| 类型(R参数名)		
-------					| ----				
绝对值距离(Manhattan)		| 距离(manhattan)	<!-- | sum(|x_ik - x_jk |,k=1, k=p)-->
欧氏距离(Euclidean)		| 距离(euclidean)	<!-- | [sum((x_ik - x_jk )^2 ,k=1, k=p)]^0.5-->
闵可夫斯基距离(Minkowski)	| 距离(minkowski)	<!-- | [sum((x_ik - x_jk )^q ,k=1, k=p)]^(1/q)-->
切比雪夫距离(Chebyshev)	| 距离(q = inf)		<!-- | max(|x_ik - x_jk |, k=1, k=p)-->
马氏距离(Mahalanobis)	| 距离				<!-- | (X_i - X_j )^' S^-1 (X_i - X_j)-->
兰氏距离(canberra)		| 距离(canberra)		<!-- |-->
夹角余弦					| 相似系数			<!-- |-->
相关系数					| 相似系数			<!-- |-->

在R中，有dist函数可以用来计算各种距离，其中method参数指明了采用那种距离统计量，而p参数则指明在采用Minkowski距离统计量时的指数项(Minkowski距离是绝对距离，欧氏距离和切比雪夫距离的推广)

#####2.1 系统聚类法
上面是样本间距离的定义，而在聚类分析中，还有一类距离需要定义，这就是类与类之间的距离定义。在R中，hclust方法的method参数提供了，single(最短距离法),ward(Ward法，利差平方和法),complete(最长距离法),average(类平均距离),mcquitty(),median(中间距离法),centroid(重心法)。下面列出系统聚类法的一般步骤(王斌会，多元统计分析及R语言)：

1. 计算n个样品两两间的距离，得距离矩阵D。
2. 构造n个类别，每个类别只包含一个样品。
3. 合并距离最近的两个类为一个新类。
4. 计算新类与当前各个类之间的距离，若类个数为1，转到步骤（5），否则回到步骤（3）。
5. 画出聚类图
6. 决定聚类的个数和类别

下面以R中iris数据集为例

	d <- dist(cbind(iris$Sepal.Length,iris$Sepal.Width,iris$Petal.Length,iris$Petal.Width),method='minkowski',p=2)
	hc <- hclust(d,method="centroid")		plot(hc)
	

#####2.2 快速聚类法（kmeans聚类法）
快速聚类法首先要指定总体中又多少个类别，然后按照距离进行聚类；kmeans聚类和系统聚类的区别就在于系统聚类会产生虽有的聚类层次，而kmeans聚类只会产生指定的聚类层次，相比下有明显的优势。但这种优势是在指定聚类个数的前提下实现的，但系统中具体有多少个类别，这还需要使用其他方法判定。在实现中，我们也可以不指定类别，而是通过指定每个类的初始中心来代替，因为在kmeans聚类中，初始的聚类中心位置并不会对最终的聚类结果产生影响。
我们可以通过R中的kmeans函数来进行快速聚类分析

kmeans聚类，对样本集中的噪音和孤立异常点比较敏感，少量的该点会对聚类结果产生比较大的影响，

#####2.3 聚类分析中的注意点
1. 最好做数据的标准化或者其他的变换，以适应相应的聚类模型，并且可以防止量纲对聚类分析的影响。

####3 主成分分析(Principle Component Analysis)
主成分分析是通过降维技术，把多维空间中的相关变量指标化简为少量且独立的新的综合指标。这里需要注意的是，首先，样本的指标变量要有所相关，如果不相关，那么降维目的就很难实现了；新产生的综合性指标变量的方差(变异)要尽量大，这样就越能反映原来数据的差异。主成分分析用到了线性代数中的特征值和特征向量。主成分分析的过程：

1. 将原始数据标准化，消除变量间量纲和数量级的影响
2. 求标准化数据的相关矩阵
3. 求相关矩阵的特征向量和特征值
4. 计算方差贡献率和累计贡献率，每个主成分的贡献率代表了原数据的总信息量的百分比
5. 确定主成分，
6. 用原指标的线性组合来计算各个主成分的得分
7. 计算综合得分
8. 得分排名
在R中坐主成分分析，我们可以使用princomp函数，主要由两种调用方式

* princomp(formula,data =NULL ,…)
* princomp(x,cor=FALSE, scores = TRUE,covmat=NULL,)

cor参数表明是使用样本的相关矩阵做主成分分析(cor=TRUE)，还是使用样本的协方差阵做主成分分析(cor=FALSE)，

	x <- cbind(iris$Sepal.Length,iris$Sepal.Width,iris$Petal.Length,iris$Petal.Width)
	pc <- princomp(x,cor=TRUE,scores=TRUE,data=iris)
	summary(pc)
	screeplot(c,type='lines')
	
	
R帮助文档中强调：

>princomp only handles so-called R-mode PCA, that is feature extraction of variables. If a data matrix is supplied (possibly via a formula) it is required that there are at least as many units as variables. For Q-mode PCA use prcomp.

####4 因子分析






	

