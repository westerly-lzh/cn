---
layout: post
title:  PostgreSQL-- 技巧1
categories:
- database
tags:
- postgresql

---

本系列为在使用PostgreSQL数据库期间学习到的一些小技巧。本文讨论PostgreSQL中的连接。
## 交叉连接

	T1 CROSS JOIN T2

交叉连接对来自T1，T2的行进行组合，得到T1，T2的笛卡尔积。如果T1里面有M行，T2里面有N行，那么交叉连接的结果有M*N行。

	FROM T1 CROSS JOIN T2 	<==>
	FROM T1,T2				<==>
	FROM T1 INNER JOIN T2 ON TRUE

## 条件连接

	T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 ON boolean_expression    	##ON
	T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 USING ( join column list )  	## USING
	T1 NATURAL { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2   ## NATURAL
	
	
### INNER JOIN
内连接，对于 T1 中的每一行 R1 ，如果能在 T2 中找到一个或多个满足连接条件的行， 那么这些满足条件的每一行都在连接表中生成一行。

### LEFT OUTER JOIN
先执行一次INNER JOIN，然后对对T1中在INNER JOIN中无法匹配到的行生成一行，该行中对应的T2的列用NULL补齐。即对T1里的每一行，如果T2中有匹配的行则匹配，没有则用NULL匹配。

### RIGHT OUTER JOIN
先执行一次INNER JOIN，然后对T2重在INNER JOIN中无法匹配的行生成一行，改行中对应的T1的列用NULL不齐。即对T2中的每一行，如果T1中有匹配的行则匹配，否则用NULL匹配。

所以LEFT OUTER JOIN 和RIGHT OUTER JOIN 等价于：

	FROM T1 LEFT OUTER JOIN T2 ON (...)		<==>
	FROM T2 RIGHT OUTER JOIN T1 ON (...)
	

### FULL OUTER JOIN
先执行一次INNER JOIN，然后对T1中在INNER JOIN 中无法匹配的行生成一行，该行中对应的T2列部分用NULL填充，然后再对T2重在INNER JOIN 中无法匹配的行生生一行，该行中对应的T1列的部分用NULL填充

假设:
	
	T1(ACOL,BCOL,CCOL),T2(ACOL,BCOL,CCOL,DCOL)

#### ON条件
ON条件中给出的是BOOL表达式，形式和WHERE子句中的形式一致。

	SELECT * FROM T1 INNER JOIN T2 ON T1.ACOL = T2.ACOL;
	
	##ACOL BCOL CCOL ACOL BCOL CCOL DCOL

上述SQL语句返回的列中将会同时包含 T1 的 ACOL 列和T2 中的 ACOL列。

### USING 条件
USING 条件中给出的是需要比较的在T1和T2中**都**有出现的列。
	
	SELECT * FROM T1 INNER JOIN T2 USING(ACOL,BCOL);
	
	##ACOL BCOL CCOL CCOL DCOL
	
对于上面SQL语句，表示T1和T2中都包含ACOL和BCOL列，返回的结果中只有ACOL和BCOL只出现一次。


### NATURAL 条件
NATURAL条件对T1和T2中出现的**共同**列进行计算。

	SELECT * FROM T1 NATURAL INNER JOIN T2 ;

	##ACOL BCOL CCOL DCOL
	
上面的SQL语句对T1和T2中出现的共同列进行条件判断，结果中对于同名的字段，只会出现一次。


	
