---
title: es操作
date: 2021-09-12 09:29:50
tags: es
---
## es操作练习
```xml
   获取集群的状态: http://ip:端口/_cluster/health
   获取所有索引: http://localhost:9200/_cat/indices?v
   
   创建索引同时创建mapping
   PUT     /index_str
   {
       "mappings": {
           "properties": {
               "realname": {
               	"type": "text",
               	"index": true
               },
               "username": {
               	"type": "keyword",
               	"index": false
               }
           }
       }
   }
                                                               
   注：某个属性一旦被建立，就不能修改了，但是可以新增额外属性
   主要数据类型
   text, keyword, string
   long, integer, short, byte
   double, float
   boolean
   date
   object
   数组不能混，类型一致
   字符串
   text：文字类需要被分词被倒排索引的内容，比如商品名称，商品详情，商品介绍，使用text。
   keyword：不会被分词，不会被倒排索引，直接匹配搜索，比如订单状态，用户qq，微信号，手机号等，这些精确匹配，无需分词。 
   
   添加文档数据:
   POST /my_doc/_doc/1 -> {索引名}/_doc/{索引ID}（是指索引在es中的id，而不是这条记录的id，比如记录的id从数据库来是1001，并不是这个。如果不写，则自动生成一个字符串。建议和数据id保持一致> ）
   {
       "id": 1001,
       "name": "imooc-1",
       "desc": "imooc is very good, 慕课网非常牛！",
       "create_date": "2019-12-24"
   }

                                                              
   注：如果索引没有手动建立mappings，那么当插入文档数据的时候，会根据文档类型自动设置属性类型。这个就是es的动态映射，帮我们在index索引库中去建立数据结构的相关配置信息。
   “fields”: {“type”: “keyword”}
   对一个字段设置多种索引模式，使用text类型做全文检索，也可使用keyword类型做聚合和排序
   “ignore_above” : 256
   设置字段索引和存储的长度最大值，超过则被忽略
   
   
     

2-15 附：分词与内置分词器
什么是分词？
把文本转换为一个个的单词，分词称之为analysis。es默认只对英文语句做分词，中文不支持，每个中文字都会被拆分为独立的个体。

英文分词：I study in imooc.com
中文分词：我在慕课网学习
POST /_analyze
{
    "analyzer": "standard",
    "text": "text文本"
}

                                                                                          
POST /my_doc/_analyze
{
    "analyzer": "standard",
    "field": "name",
    "text": "text文本"
}                                                            
es内置分词器
standard：默认分词，单词会被拆分，大小会转换为小写。

simple：按照非字母分词。大写转为小写。

whitespace：按照空格分词。忽略大小写。

stop：去除无意义单词，比如the/a/an/is…

keyword：不做分词。把整个文本作为一个单独的关键词。

{
    "analyzer": "standard",
    "text": "My name is Peter Parker,I am a Super Hero. I don't like the Criminals."
}
```

### es 搜索
```angular2

```
