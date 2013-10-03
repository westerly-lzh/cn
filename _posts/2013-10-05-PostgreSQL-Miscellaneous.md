---
layout: post
title: PostgreSQL杂记
categories:
- PostgreSQL
tags:
- SQL
- PostgreSQL

---



今天在实用yahoo answer的api做数据收集，遇到了一个实际的问题，这里先把该问题的场景描述下。我想把yahoo answer上针对某个特定关键字的问题内容收集存储到数据库中，但出现的一个场景是我有很多个关键字，如果一个问题中同时出现了多个关键字，那么在我的数据收集过程中该问题就会被收集多次，那么，为了去掉重复的内容，我需要在收集完成数据后，针对数据库中的数据进行去重操作。这里我实用的是PostgreSQL数据库。

    创建数据库
    create table questions(  q_id varchar(20),subject text);
    
插入测试数据

    insert into questions values (1,'a');     
    insert into questions values (1,'a');
    insert into questions values (1,'a');
    insert into questions values (1,'a');
    insert into questions values (2,'b');
    insert into questions values (2,'b');
    insert into questions values (3,'c');
    insert into questions values (4,'d');
    insert into questions values (5,'e');
    
测试查询，这里实用ctid来唯一的标识每一行，如果表中有主键时，可以使用主键来代替ctid。

    select ctid,* from questions;
    
ctid | id | name
-----|----|-----
(0,1)| 1  | a
(0,2)| 1  | a
(0,3)| 1  | a
(0,4)| 1  | a
(0,5)| 2  | b
(0,6)| 2  | b
(0,7)| 3  | c
(0,8)| 4  | d
(0,9)| 5  | e

查找重复数据

    select distinct id ,count(*) from questions group by id having count(*) > 1;
 
id  | count
----|------
1   | 4
2   | 2

查找要保留的数据

    select ctid,* from questions where ctid in (select min(ctid) from questions group by id);

ctid | id | name
-----|----|-----
(0,1)| 1  | a
(0,5)| 2  | b
(0,7)| 3  | c
(0,8)| 4  | d
(0,9)| 5  | e
    
删除重复数据

delete from questions where ctid not in (select min(ctid) from questions group by id);

当然还有一个在数据量比较大的情况下更加有效的方法：

    delete from questions as a where a.ctid <> (select min(b.ctid) from questions b where a.id = b.id);
