---
layout: post
title: R数据重塑：reshape2 学习[2]
categories:
- IT
tags:
- R

---
在数据分析中，经常需要对原始数据进行重塑，在R中对数据进行重塑，可以使用reshape包，也可以使用reshape2，reshape2是对reshape的重构，这里介绍reshape2中方法的使用及其功能。

##colsplit(string, pattern, names)
* string，将会被分解的因子或者字符向量
* pattern，分解规则的正则表达式
* names，分解后产生的向量的列名。

该函数对给定的向量或者因子变量按照pattern给定的规则做分解，分解后的产生的向量以names指定的名称命名。

    x <- c("A_a","B_b","C_c")
    colsplit(x,"_",names=c("Up","Low"))
    ##输出
      Up Low
    1  A   a
    2  B   b
    3  C   c
   
##acast(data,formula,fun.aggregate,margins,subset,fill,drop,value.var,...)

* data, 输入需要被转换的数据，该数据是melt函数的返回结果
* formula, 指定数据转换的模式，
* fun.aggregate,
* margins,
* subset,
* fill,
* drop,
* value.var