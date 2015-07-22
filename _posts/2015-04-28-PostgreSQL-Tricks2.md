---
layout: post
title:  PostgreSQL-- 技巧2
categories:
- database
tags:
- postgresql

---

本系列为在使用PostgreSQL数据库期间学习到的一些小技巧。

## 窗口函数
>A window function performs a calculation across a set of table rows that are somehow related to the current row. This is comparable to the type of calculation that can be done with an aggregate function. But unlike regular aggregate functions, use of a window function does not cause rows to become grouped into a single output row — the rows retain their separate identities. Behind the scenes, the window function is able to access more than just the current row of the query result.
>
窗口函数的语法结构如下：

	function_name ([expression [, expression ... ]]) [ FILTER ( WHERE filter_clause ) ] OVER window_name
	function_name ([expression [, expression ... ]]) [ FILTER ( WHERE filter_clause ) ] OVER ( window_definition )
	function_name ( * ) [ FILTER ( WHERE filter_clause ) ] OVER window_name
	function_name ( * ) [ FILTER ( WHERE filter_clause ) ] OVER ( window_definition )

where **window_definition** has the syntax

	[ existing_window_name ]
	[ PARTITION BY expression [, ...] ]
	[ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
	[ frame_clause ]

and the optional **frame_clause** can be one of

	{ RANGE | ROWS } frame_start
	{ RANGE | ROWS } BETWEEN frame_start AND frame_end

where **frame_start** and **frame_end** can be one of

	UNBOUNDED PRECEDING
	value PRECEDING
	CURRENT ROW
	value FOLLOWING
	UNBOUNDED FOLLOWING

Here, **expression** represents any value expression that does not itself contain window function calls.

一个例子如下：

	--计算每时每刻brt上的保留乘客数量(累计求和)
	INSERT INTO  BRT_ACCU 
	SELECT 
	B.线路编号,
	B.交易日期,
	B.交易时间,
	B.进出站计数,
	SUM(B.进出站计数) OVER(PARTITION BY B.线路编号,B.交易日期 ORDER BY B.交易时间)AS 当前保留乘客数量
	FROM BRT B;

上面这段SQL做的事情是计算每条线路每天到某时某刻（交易时间）为止，当前公交系统中保留的乘客数量。这里数据的“窗口” 是线路和日期,即按照线路和日期进行分组，然后每一组内按照交易时间进行排序，对这样的数据进行累计求和。

所以，对于ORDER BY子句部分，COUNT这样的聚合函数并不合适的。


## 递归查询
业务系统经常在数据库表中存储树状结构的数据（具有ID，PARENT_ID这样类似的字段），可以使用SQL把具有这样结构的数据查询出来，可以很快速方面的解决很多业务需求。PostgreSQL提供的WITH RECURSIVE子句可以很方便的解决这类问题（Using RECURSIVE, a WITH query can refer to its own output.）The general form of a recursive WITH query is always a non-recursive term, then UNION (or UNION ALL), then a recursive term, where only the recursive term can contain a reference to the query's own output. 

	WITH RECURSIVE TMP AS (
		SELECT * FROM TREE_TB WHERE ID = '12345'
		UNION ALL
		SELECT B.* FROM TREE_TB B INNER JOIN TMP C ON  B.PARENT_ID = C.ID
	)
	SELECT * FROM TMP;
	
	## 等价于：
	WITH RECURSIVE TMP AS (
		SELECT * FROM TREE_TB WHERE ID = '12345'
		UNION ALL
		SELECT B.* FROM TREE_TB B, TMP C WHERE B.PARENT_ID = C.ID
	)
	SELECT * FROM TMP;

	
