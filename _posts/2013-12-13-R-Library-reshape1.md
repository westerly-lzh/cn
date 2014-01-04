---
layout: post
title: R数据重塑：reshape2 学习[1]
categories:
- R
tags:
- R
- reshape2

---
在数据分析中，经常需要对原始数据进行重塑，在R中对数据进行重塑，可以使用reshape包，也可以使用reshape2，reshape2是对reshape的重构，这里介绍reshape2中方法的使用及其功能

##add_margins(df,vars,margins=TRUE)
* df，将要被重塑的数据框

* vars，df中的列名list,

* margins，指定计算那些列的重塑。

该函数的作用是对给定的数据框进行重塑，重塑的内容是对给定的数据框，以给定的列名得到所有可能的组合.比如给定的vars=c("A","B","C"),那么所有可能的组合为c("A");c("B");c("C");c("A","B");c("B","C");c("A","C");c("A","B","C").然后按照把这些组合所表示的列从原始数据框中赋值为为"(all)",得到新的数据框。最后把这些数据框再按照行的形式拼接起来。而margins参数指定了那些列的组合将会被计算比如margins=c("A","B"),则由vars指定的那些列的组合中只有c("A"),c("B"),c("A","B")将会被处理。而margins=TRUE时则vars中指定的列都会被计算。


	d <- matrix(1:9,3,3)
	d <- as.data.frame(d)
	colnames(d)<-c("A","B","C")
	add_margins(as.data.frame(d),vars=c("A","B","C"),margins=c("A","B"))
	输出如下：
	       A     B C
	1      1     4 7
	2      2     5 8
	3      3     6 9
	4  (all)     4 7
	5  (all)     5 8
	6  (all)     6 9
	7      1 (all) 7
	8      2 (all) 8
	9      3 (all) 9
	10 (all) (all) 7
	11 (all) (all) 8
	12 (all) (all) 9

##melt
函数melt做的事情就是把输入的数据表示成相应的二维矩阵。
###melt.data.frame(data, id.vars,measure.vars,variable.name = "variable", ..., na.rm = FALSE,value.name = "value")

* data，将要被重塑的数据框
* id.vars,转变后的数据将以id.vars指定的列为
* measure.vars，重塑后的数据，将以measure.vars指定的列为控制列，如果没有指定measure.vars，那么将以data中除掉id.vars中的列为控制列，
* variable.name，指定measure.vars中列在重塑后数据中的变量名所在列的列名
* value.name,指定measure.vars中列在重塑后数据的变量值所在列的列名

下面是一个使用melt的例子：
	
	d <- matrix(1:16,4,4)
	d<- as.data.frame(d)
	colnames(d) <- c("A","B","C","D")
	melt(d,id.vars=c("A","B"))
	
	 输出如下：
	  A B variable value
	1 1 5        C     9
	2 2 6        C    10
	3 3 7        C    11
	4 4 8        C    12
	5 1 5        D    13
	6 2 6        D    14
	7 3 7        D    15
	8 4 8        D    16

###melt.array(data,varnames = names(dimnames(data)), ..., na.rm = FALSE,value.name = "value"))

* data,将要被重塑的数据，类型可以是数组，矩阵及table
* varnames,每个变量的名字

melt.array比较容易理解，我们可以这样来解释，对于多维数组来说，melt得到的变量实际上是多维数组中每个维度的坐标，得到的value实际上是处于那个坐标位置的值，这样，melt实际上是把一个多维数据用一个二维矩阵来表示了。现在来看一个例子

	a <- array(c(1:7, NA), c(2,2,2))
	a
	####输出：
	, , 1

     [,1] [,2]
	[1,]    1    3
	[2,]    2    4

	, , 2

     [,1] [,2]
	[1,]    5    7
	[2,]    6   NA
	
	melt(a)
		Var1 Var2 Var3 value
	1    1    1    1     1
	2    2    1    1     2
	3    1    2    1     3
	4    2    2    1     4
	5    1    1    2     5
	6    2    1    2     6
	7    1    2    2     7
	8    2    2    2    NA
	
上面melt得到的数据前三列Var1，Var2，Var3实际上是多维数组a上的维度坐标，而得到的value列就是多维数组在该坐标下得值。

###melt.list(data, ..., level = 1)
* data, 输入的list类型数据

针对list类型的melt比较简单，就是对list中的每一个元素递归的调用相应的melt方法得到一个二维矩阵，然后再把list中元素的坐标cbind到这个二维矩阵上。



