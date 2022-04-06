---
title: mysql锁表
date: 2022-04-06 15:03:20
tags: mysql
---
###  1.业务场景: 
某个页面的查询列表比较慢，有时候会报服务器异常 500。 在排除网络稳定，sql语句没问题的情况下，在看看是不是别的问题，猜测是不是锁表的问题。
锁表: 

### 2 排查原因:

查询是否锁表
show OPEN TABLES where In_use > 0;
![](/../../static/mysql/锁表问题.jpg)
查看了下 发现 base_house 这个表的In_use 这个值有40。
通过查询线上的sql语句日志发现sql存在in的部分
```xml

   SELECT longitude AS longitude, latitude AS latitude, house_code houseCode, create_date AS createDate, 
   update_date ASupdateDate, data_from AS dataFrom, org_id AS orgId, house_address AS houseAddress, 
   is_standard AS is_standard, housestandard_id AS housestandard_id, owner_name_zs AS ownerNameZs, 
   owner_id_zs AS ownerIdZs, id AS houseId, NULL AS houseRentalId, living_situation AS livingSituation, 
   application_zs AS applicationZs, area AS area, house_type_zs AS houseTypeZs, building_code AS buildingCode,
    org_internal_code AS orgInternalCode, claim_status AS claimStatus, claim_user AS claimUser, claim_date AS claimDate,
     check_date AS checkDate FROM base_house WHERE 1 = 1 AND org_id IN ('1012387494622150657', 
     '1012387528981889025', '1014624932065198081', '1016893087114346497', '1019174711180935169', 
     '1012484182925918209', '1014629398831185921', ..., '1016892777876701185', '1016893258913038337',
     '1016892812236439553', '1012414295218077697', '1014631254257057793', '1012483976767488001', 
     '1014629604989616129', '1014629570629877761') AND house_address LIKE CONCAT('%', ?, '%') AND is_deleted = 0 AND claim_status = 0 AND tenant_id = '331000'
```
查看了下对应的字段 发现org_id 没有添加索引，故在org_id添加索引 解决了这个锁表问题
### 总结
主要是为了记录 在网络稳定，sql语句没问题的情况下，检查其余可能存在的问题导致sql查询慢，锁表也是一种常见的问题。

