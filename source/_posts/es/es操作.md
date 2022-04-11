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

英文分词：I study in baidu.com
中文分词：我在百度学习
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
```xml
请求参数的查询(QueryString)
查询[字段]包含[内容]的文档

GET     /shop/_doc/_search?q=desc:百度
GET     /shop/_doc/_search?q=nickname:百&q=age:25
1
2
                                代码块                                                            
text与keyword搜索对比测试(keyword不会被倒排索引，不会被分词)

GET     /shop/_doc/_search?q=nickname:super
GET     /shop/_doc/_search?q=username:super
GET     /shop/_doc/_search?q=username:super hero
1
2
3
                                代码块                                                            
这种方式称之为QueryString查询方式，参数都是放在url中作为请求参数的。

DSL基本语法
QueryString用的很少，一旦参数复杂就难以构建，所以大多查询都会使用dsl来进行查询更好。

Domain Specific Language
特定领域语言
基于JSON格式的数据查询
查询更灵活，有利于复杂查询
DSL格式语法：

# 查询
POST     /shop/_doc/_search
{
    "query": {
        "match": {
            "desc": "慕课网"
        }
    }
}
# 判断某个字段是否存在
{
    "query": {
        "exists": {
	        "field": "desc"
	    }
    }
}
 
# 查询
POST     /shop/_doc/_search
{
    "query": {
        "match": {
            "desc": "慕课网"
        }
    }
}
# 判断某个字段是否存在
{
    "query": {
        "exists": {
	        "field": "desc"
	    }
    }
}

                            
语法格式为一个json object，内容都是key-value键值对，json可以嵌套。
key可以是一些es的关键字，也可以是某个field字段，后面会遇到
搜索不合法问题定位
DSL查询的时候经常会出现一些错误查询，出现这样的问题大多都是json无法被es解析，他会像java那样报一个异常信息，根据异常信息去推断问题所在，比如json格式不对，关键词不存在未注册等等，甚至有时候不能定位问题直接复制错误信息到百度一搜就能定位问题了。






3-6 附：DSL搜索 - 查询所有与分页
match_all
在索引中查询所有的文档

GET     /shop/_doc/_search
1
                                代码块                                                            
或

POST     /shop/_doc/_search
{
    "query": {
        "match_all": {}
    },
    "_source": ["id", "nickname", "age"]
}


分页查询
默认查询是只有10条记录，可以通过分页来展示

POST     /shop/_doc/_search
{
    "query": {
        "match_all": {}
    },
    "from": 0,
    "size": 10
}

{
	"query": {
		"match_all": {}
	},
	"_source": [
		"id",
		"nickname",
		"age"
	],
	"from": 5,
	"size": 5
}


  

3-8 附： DSL搜索 - term/match
term精确搜索与match分词搜索
搜索的时候会把用户搜索内容，比如"百度强大”作为一整个关键词去搜索，而不会对其进行分词后再搜索

POST     /shop/_doc/_search
{
    "query": {
        "term": {
            "desc": "百度"
        }
    }
}
对比
{
    "query": {
        "match": {
            "desc": "百度"
        }
    }
}

terms 多个词语匹配检索
相当于是tag标签查询，比如百度的标签，可以完全匹配做类似标签的查询

POST     /shop/_doc/_search
{
    "query": {
        "terms": {
            "desc": ["百度", "学习", "骚年"]
        }
    }
}


3-10 附:DSL搜索 - match_phrase
match_phrase 短语匹配
match：分词后只要有匹配就返回，match_phrase：分词结果必须在text字段分词中都包含，而且顺序必须相同，而且必须都是连续的。（搜索比较严格）

slop：允许词语间跳过的数量

POST     /shop/_doc/_search
{
    "query": {
        "match_phrase": {
            "desc": {
            	"query": "大学 毕业 研究生",
            	"slop": 2
            }
        }
    }
}


  

3-12 附：DSL搜索 - match（operator）/ids
match 扩展
operator

or：搜索内容分词后，只要存在一个词语匹配就展示结果
and：搜索内容分词后，都要满足词语匹配
POST     /shop/_doc/_search
{
    "query": {
        "match": {
            "desc": "百度"
        }
    }
}
# 等同于
{
    "query": {
        "match": {
            "desc": {
                "query": "xbox游戏机",
                "operator": "or"
            }
        }
    }
}
# 相当于 select * from shop where desc='xbox' or|and desc='游戏机'
                            
minimum_should_match: 最低匹配精度，至少有[分词后的词语个数]x百分百，得出一个数据值取整。举个例子：当前属性设置为70，若一个用户查询检索内容分词后有10个词语，那么匹配度按照 10x70%=7，则desc中至少需要有7个词语匹配，就展示；若分词后有8个，则 8x70%=5.6，则desc中至少需要有5个词语匹配，就展示。

minimum_should_match 也能设置具体的数字，表示个数

POST     /shop/_doc/_search
{
    "query": {
        "match": {
            "desc": {
                "query": "女友生日送我好玩的xbox游戏机",
                "minimum_should_match": "60%"
            }
        }
    }
}
                                                            
根据文档主键ids搜索
GET /shop/_doc/1001
                                                            
查询多个

POST     /shop/_doc/_search

{
    "query": {
        "ids": {
            "type": "_doc",
            "values": ["1001", "1010", "1008"]
        }
    }
}


  

3-14 附：DSL搜索 - multi_match/boost
multi_match
满足使用match在多个字段中进行查询的需求

POST     /shop/_doc/_search
{
    "query": {
        "multi_match": {
                "query": "皮特帕克",
                "fields": ["desc", "nickname"]

        }
    }
}
权重，为某个字段设置权重，权重越高，文档相关性得分就越高。通畅来说搜索商品名称要比商品简介的权重更高。

POST     /shop/_doc/_search
{
    "query": {
        "multi_match": {
                "query": "皮特帕克",
                "fields": ["desc", "nickname^10"]

        }
    }
}


```
