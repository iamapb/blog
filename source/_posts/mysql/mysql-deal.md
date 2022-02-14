---
title: mysql-deal
date: 2021-11-04 15:52:58
tags: mysql
---
 ## mysql 行级锁问题
 1  mysql有三种锁的级别: 页级，表级，行级
 ```
 表级锁: 开销小，加锁快; 不会出现死锁，锁定力度大，发送锁冲突的概率高，并发度最低。
 行级锁: 开销大，加锁慢; 会出现死锁; 锁定粒度最小,发生锁冲突的概率最低，并发度也最高。
 页面锁: 开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一
 ```

 ## mysql select和 update同时使用
>也就是说将select出的结果再通过中间表select一遍，这样就规避了错误。注意，这个问题只出现于mysql，mssql和Oracle不会出现此问题

 ```
update base_ppl_330900 INNER JOIN 
(select base_ppl_id, base_ppl_type from (SELECT base_ppl_type, base_ppl_id from base_ppl_expand_relation_330900 where base_ppl_id in (SELECT id from base_ppl_330900 where people_type is null )) 
as a) b
on b.base_ppl_id = base_ppl_330900.id 

set base_ppl_330900.people_type = b.base_ppl_type;
 ```
