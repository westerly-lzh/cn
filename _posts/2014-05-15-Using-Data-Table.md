---
layout: post
title:  R语言中的数据操作
categories:
- R
tags:
- 数据操作

---
## 起因

R在中data.frame是使用非常频繁地数据结构之一，而data.frame的结构非常类似关系数据库中的表，但如何像使用SQL操作数据库中表那样方便的操作data.frame呢？在R中又sqldf这个包可共使用，支持使用标准的SQL语句来操作data.frame。但是当我发现R中有data.table这个包后，我果断放弃使用sqldf这个包转而使用data.table来实现操作对数据框的了。data.table继承自data.frame，提供了比data.frame更加丰富和快速的操作方法，但data.table提供的操作语法比较复杂。下面将按照SQL中一些语法的形式来介绍data.table.在开始使用data.table之前，我们需要准备好样例数据：

| CustomId 		| CustomName 	|  City			|
|: ------------|:------------- |: ------------ |
| 1 			| Jeff 			|  Berlin 		|
| 3				| Jim			|  NewYork		|
| 7				| Tom			|  ChengDu		|

| OrderId 		| CustomId 		|  ToCity		|
|: ------------|:------------- |: ------------ |
| 2 			| 1 			|  Berlin 		|
| 3				| 2				|  BeiJing		|
| 5				| 5				|  ChengDu		|


## Syntax

	## S3 method for class 'data.table'
	x[i, j, by, keyby, with = TRUE,
	nomatch = getOption("datatable.nomatch"),                   # default: 	NA_integer_
	mult = "all",
	roll = FALSE,
	rollends = if (roll=="nearest") c(TRUE,TRUE)
             else if (roll>=0) c(FALSE,TRUE)
             else c(TRUE,FALSE),
	which = FALSE,
	.SDcols,
	verbose = getOption("datatable.verbose"),                   # default: FALSE
	allow.cartesian = getOption("datatable.allow.cartesian"),   # default: FALSE
	drop = NULL,
	rolltolast = FALSE   # deprecated
	]

## CREATE 
	
	CREATE TABLE table_name
	( column_name1 data_type(size),
	  column_name2 data_type(size),
	  column_name3 data_type(size),
	  ....);

创建一个data.table 可以使用如下的形式：

	customers <- data.table(CustomId = c(1,3,7),
							 CustomName = c("Jeff","Jim","Tom"),
							 City = c("Berlin","NewYork","ChengDu"))
							 
当然，由于data.table继承自data.frame，则可以通过使用一个data.frame来创建data.table，如下：
	
	orders <- data.table(data.frame(OrderId = c(2,3,5),
									 CustomId = c(1,2,5),
									 ToCity = c("Berlin","BeiJing","ChengDu")))

## SELECT

	SELECT column_name,column_name FROM table_name;

	
在data.table中选择某列数据，需要使用data.table的特殊语法，关于data.frame的语法结构参见[Data Table Syntax](#Syntax),但简单的数据列选择可以使用如下：

	customers[,list(CustomId,CustomName)]

正如data.table的文档中所说，参数j指定的是一个可以被计算的表达式。所以如果希望选择一列数据，那么可以使用如下：

	customers[,CustomName]
	[1] "Jeff" "Jim"  "Tom" 

	customers[,"CustomName"]
	[1] "CustomName"

	## 而当需要进行条件查询时，可以借助于，参数i
	customers[CustomName=="Jeff"]
   		CustomId 	CustomName   	City
	1:        1       Jeff 			Berlin


SELECT JOIN用于两个或者多个表之间的关系，从这些表中查询数据
	
	SELECT CUSTOMERS.CUSTOMID AS 	CUSTOMID,CUSTOMERS.CUSTOMNAME AS CUSTOMNAME, 	ORDERS.ORDERID AS ORDERID, CUSTOMERS.CITY AS FROMCITY,ORDERS.TOCITY AS TOCITY 	FROM CUSTOMERS, ORDERS WHERE CUSTOMERS.CUSTOMEID = ORDERS.CUSTOMID

该SQL JOIN 用于从两张表中查找那些CUSTOMID相同的记录，相同的操作在 data.table中可以如下实现：
	
	setkey(customers,"CustomId")
	setkey(orders,"CustomId")
	customers[orders,nomatch=0,mult="all"]
	##输出：
	   		CustomId CustomName   City OrderId ToCity
	1:        1       Jeff Berlin       2 Berlin

SQL INNER JOIN,同JOIN。

SQL LEFT JOIN,从左表中返回行，即使右表没有那些行

	CUSTOMERS.CUSTOMID AS 	CUSTOMID,CUSTOMERS.CUSTOMNAME AS CUSTOMNAME, 	ORDERS.ORDERID AS ORDERID, CUSTOMERS.CITY AS FROMCITY,ORDERS.TOCITY AS TOCITY 	FROM CUSTOMERS LEFT JOIN ORDERS WHERE CUSTOMERS.CUSTOMEID = ORDERS.CUSTOMID
	
该实现，在data.table中如下：
	
	setkey(customers,"CustomId")
	setkey(orders,"CustomId")
	orders[customers,nomatch=NA,mult="all"]
	
	##输出：
	   	CustomId OrderId ToCity CustomName    City
	1:        1       2 Berlin       Jeff  Berlin
	2:        3      NA     NA        Jim NewYork
	3:        7      NA     NA        Tom ChengDu
	
SQL RIGHT JOIN，如LEFT JOIN.

##参数说明

+ i：如果i是这个table列名的表达式，那么将会根据这个表达式计算一个类似于SELF JOIN的查询。



