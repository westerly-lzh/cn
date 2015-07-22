---
layout: post
title:  PostgreSQL-- 技巧1
categories:
- database
tags:
- postgresql

---

本系列为在使用PostgreSQL数据库期间学习到的一些小技巧。

## 如何取得插入行的id
有时，我们的应用场景需要我们在插入数据时，可以获取插入的数据在表中的某一列的值（尤其是自增主键），这是可以采用如下的方式：

	CREATE TABLE T_A
	(
	ID SERIAL NOT NULL,
	NAME CHARACTER VARYING(50),
	CONSTRAINT T_A_PKEY PRIMARY KEY (ID)
	)
	
	CREATE OR REPLACE FUNCTION FC_RT(NAME VARCHAR) RETURNS INTEGER AS $$
    DECLARE RE INTEGER;
    BEGIN
		INSERT INTO T_A (NAME) VALUES(NAME) RETURNING ID INTO RE;
		RETURN RE;
	END
	$$ LANGUAGE PLPGSQL;
	
	SELECT FC_RT('123'); ## 返回插入行的ID
	
	INSERT INTO T_A (NAME) VALUES('456') RETURNING ID ## 也可以直接返回插入行的ID
	

## BETWEEN的区间
不同的数据库对 BETWEEN...AND 操作符的处理方式是有差异的。在PostgreSQL中，BETWEEN ... AND ...的两端都是闭区间。

	##下面两行是等价的
	A BETWEEN X AND Y 
	A >= X AND A <= Y
	## 下面两行是等价的
	A NOT BETWEEN X AND Y
	A < X OR A > Y

## 多条件
我们经常在应用中遇到这样的场景，需要对某一列的值进行归类，比如客户关系管理中，把年消费低于1000的归为普通用户，年消费介于1000和2000之间的归为高级用户，而消费高于2000的为VIP客户，这时可以使用如下的多条件（创建一个新列来表示类别）：
	
	CASE WHEN CONSUME < 1000 THEN '普通用户'
    	 WHEN CONSUME BETWEEN 1000 AND 2000 THEN '高级用户'
    	 ELSE 'VIP客户'
	END AS 客户等级

 当然，CASE...WHEN 也可以用于WHERE 子句中:

	SELECT * FROM T_A WHERE CASE WHEN NAME IS NOT NULL THEN NAME LIKE %ABC% ELSE FALSE END;
	

 



