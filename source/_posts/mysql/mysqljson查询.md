---
title: mysql 关于 json查询
date: 2021-03-04 12:24:50
tags: mysql
---
mysql 不单单支持单字符串的查询 还支持 json字堵的查询 最近在工作中用到 顺便记录下
 
### mysql根据json字段的内容检索查询数据
   + 1.使用 字段->'$.json属性'进行查询条件
   + 2.根据json数组查询，用JSON_CONTAINS(字段,JSON_OBJECT('json属性', "内容"))
   

  一般数据库存储json类型的数据会用json类型或者text类型
>   注意：用json类型的话
    1）JSON列存储的必须是JSON格式数据，否则会报错。
    2）JSON数据类型是没有默认值的。 
  
  ![数据类型](/../../static/mysql/json.png)  
  payload 里面存储的是json字符串 里面依然包含着json数据
  
  执行查询sql
  
  1 mapper注解的形式查询
  ```java
          @Select("""<script>
                  select zrhr.* from zn_record_his_reg zrhr                 
                  <if test="dept_code != null and dept_code != '' "> and zrhr.payload-> '$.customer.dept_code' = #{dept_code}</if>
                  <if test="item_code != null and item_code != '' "> and zrhr.payload-> '$.customer.item_code' = #{item_code}</if>
                  <if test="status != null and status != 0"> and zrhr.payload-> '$.customer.status' = #{status}</if>
                  <if test="doctor_name != null and doctor_name != ''"> and zrhr.payload-> '$.customer.doctor_name' = #{doctor_name}</if>
                  <if test="txt != null and txt != ''"> AND (zrhr.payload-> '$.customer.name' like concat('%', #{txt}, '%') or zrhr.payload-> '$.customer.phone' = #{txt})</if>
          </script>""")
```
   customer 是payload 里面的json对象 
   
   2 用JSON_CONTAINS(字段,JSON_OBJECT(‘json属性’, “内容”))
  ```java
    select * from zn_record_his_reg where is_del = 0 and 用JSON_CONTAINS(payload, JSON_OBJECT('customer.dept_code', '133'))

  ```     

