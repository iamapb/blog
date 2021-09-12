---
title: es-dsl
date: 2021-09-12 14:58:11
tags: es
---

## es-DSL查询
```xml

    请求参数的查询(QueryString)
    查询[字段]包含[内容]的文档
    
    GET     /shop/_doc/_search?q=desc:慕课网
    GET     /shop/_doc/_search?q=nickname:慕&q=age:25
                                                                
    text与keyword搜索对比测试(keyword不会被倒排索引，不会被分词)
    
    GET     /shop/_doc/_search?q=nickname:super
    GET     /shop/_doc/_search?q=username:super
    GET     /shop/_doc/_search?q=username:super hero
   
                                                                                     
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
                                    代码块                                                            
    语法格式为一个json object，内容都是key-value键值对，json可以嵌套。
    key可以是一些es的关键字，也可以是某个field字段，后面会遇到
    搜索不合法问题定位
    DSL查询的时候经常会出现一些错误查询，出现这样的问题大多都是json无法被es解析，他会像java那样报一个异常信息，根据异常信息去推断问题所在，比如json格式不对，关键词不存在未注册等等，甚至有时候不能定位问题直接复制错误信息到百度一搜就能定位问题了。
    
    3-6 附：DSL搜索 - 查询所有与分页
    match_all
    在索引中查询所有的文档
    
    GET     /shop/_doc/_search
    
    POST     /shop/_doc/_search
    {
        "query": {
            "match_all": {}
        },
        "_source": ["id", "nickname", "age"]
    }
    
      

3-8 附： DSL搜索 - term/match
term精确搜索与match分词搜索
搜索的时候会把用户搜索内容，比如“慕课网强大”作为一整个关键词去搜索，而不会对其进行分词后再搜索

POST     /shop/_doc/_search
{
    "query": {
        "term": {
            "desc": "慕课网"
        }
    }
}
对比
{
    "query": {
        "match": {
            "desc": "慕课网"
        }
    }
}
                            
注：match会对慕课网先进行分词（其实就是全文检索），在查询，而term则不会，直接把慕课网作为一个整的词汇去搜索。
head 可视化操作对比：

terms 多个词语匹配检索
相当于是tag标签查询，比如慕课网的一些课程会打上前端/后端/大数据/就业课这样的标签，可以完全匹配做类似标签的查询

POST     /shop/_doc/_search
{
    "query": {
        "terms": {
            "desc": ["慕课网", "学习", "骚年"]
        }
    }
}

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
            "desc": "慕课网"
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
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
                                代码块                                                            
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
1
                                                            
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
                "query": "皮特帕克慕课网",
                "fields": ["desc", "nickname"]

        }
    }
}

权重，为某个字段设置权重，权重越高，文档相关性得分就越高。通畅来说搜索商品名称要比商品简介的权重更高。

POST     /shop/_doc/_search
{
    "query": {
        "multi_match": {
                "query": "皮特帕克慕课网",
                "fields": ["desc", "nickname^10"]

        }
    }
}

阶段三 · 分布式搜索引擎-ES
  

3-16 附：DSL搜索 - 布尔查询
可以组合多重查询

must：查询必须匹配搜索条件，譬如 and
should：查询匹配满足1个以上条件，譬如 or
must_not：不匹配搜索条件，一个都不要满足
实操1：

POST     /shop/_doc/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "multi_match": {
                        "query": "慕课网",
                        "fields": ["desc", "nickname"]
                    }
                },
                {
                    "term": {
                        "sex": 1
                    }
                },
                {
                    "term": {
                        "birthday": "1996-01-14"
                    }
                }
            ]
        }
    }
}

{
    "query": {
        "bool": {
            "should（must_not）": [
                {
                    "multi_match": {
                        "query": "学习",
                        "fields": ["desc", "nickname"]
                    }
                },
                {
                	"match": {
                		"desc": "游戏"
                	}	
                },
                {
                    "term": {
                        "sex": 0
                    }
                }
            ]
        }
    }
}


{
    "query": {
        "bool": {
            "must": [
                {
                	"match": {
                		"desc": "慕"
                	}	
                },
                {
                	"match": {
                		"nickname": "慕"
                	}	
                }
            ],
            "should": [
                {
                	"match": {
                		"sex": "0"
                	}	
                }
            ],
            "must_not": [
                {
                	"term": {
                		"birthday": "1992-12-24"
                	}	
                }
            ]
        }
    }
}


为指定词语加权
特殊场景下，某些词语可以单独加权，这样可以排得更加靠前。

POST     /shop/_doc/_search
{
    "query": {
        "bool": {
            "should": [
            	{
            		"match": {
            			"desc": {
            				"query": "律师",
            				"boost": 18
            			}
            		}
            	},
            	{
            		"match": {
            			"desc": {
            				"query": "进修",
            				"boost": 2
            			}
            		}
            	}
            ]
        }
    }
}





3-18 附：DSL搜索 - 过滤器
对搜索出来的结果进行数据过滤。不会到es库里去搜，不会去计算文档的相关度分数，所以过滤的性能会比较高，过滤器可以和全文搜索结合在一起使用。
post_filter元素是一个顶层元素，只会对搜索结果进行过滤。不会计算数据的匹配度相关性分数，不会根据分数去排序，query则相反，会计算分数，也会按照分数去排序。
使用场景：

query：根据用户搜索条件检索匹配记录
post_filter：用于查询后，对结果数据的筛选
实操：查询账户金额大于80元，小于160元的用户。并且生日在1998-07-14的用户

gte：大于等于
lte：小于等于
gt：大于
lt：小于
（除此以外还能做其他的match等操作也行）
POST     /shop/_doc/_search

{
	"query": {
		"match": {
			"desc": "慕课网游戏"
		}	
    },
    "post_filter": {
		"range": {
			"money": {
				"gt": 60,
				"lt": 1000
			}
		}
	}	
}



  

3-20 附： DSL搜索 - 排序
es的排序同sql，可以desc也可以asc。也支持组合排序。

实操：

POST     /shop/_doc/_search
{
	"query": {
		"match": {
			"desc": "慕课网游戏"
		}
    },
    "post_filter": {
    	"range": {
    		"money": {
    			"gt": 55.8,
    			"lte": 155.8
    		}
    	}
    },
    "sort": [
        {
            "age": "desc"
        },
        {
            "money": "desc"
        }
    ]
}
                                                         
对文本排序
由于文本会被分词，所以往往要去做排序会报错，通常我们可以为这个字段增加额外的一个附属属性，类型为keyword，用于做排序。

创建新的索引
POST        /shop2/_mapping
{
    "properties": {
        "id": {
            "type": "long"
        },
        "nickname": {
            "type": "text",
            "analyzer": "ik_max_word",
            "fields": {
                "keyword": {
                    "type": "keyword"
                }
            }
        }
    }
}

插入数据
POST         /shop2/_doc
{
    "id": 1001,
    "nickname": "美丽的风景"
}
{
    "id": 1002,
    "nickname": "漂亮的小哥哥"
}
{
    "id": 1003,
    "nickname": "飞翔的巨鹰"
}
{
    "id": 1004,
    "nickname": "完美的天空"
}
{
    "id": 1005,
    "nickname": "广阔的海域"
}
                                                         
排序
{
    "sort": [
        {
            "nickname.keyword": "desc"
        }
    ]
}

3-22 附：DSL搜索 - 高亮highlight
高亮显示
POST     /shop/_doc/_search
{
    "query": {
        "match": {
            "desc": "慕课网"
        }
    },
    "highlight": {
        "pre_tags": ["<tag>"],
        "post_tags": ["</tag>"],
        "fields": {
            "desc": {}
        }
    }
}




4-2 附：深度分页
分页查询
POST     /shop/_doc/_search
{
    "query": {
        "match_all": {}
    },
    "from": 0,
    "size": 10
}
                                                        
深度分页
深度分页其实就是搜索的深浅度，比如第1页，第2页，第10页，第20页，是比较浅的；第10000页，第20000页就是很深了。

使用如下操作：

{
    "query": {
        "match_all": {}
    },
    "from": 9990,
    "size": 10
}

{
    "query": {
        "match_all": {}
    },
    "from": 9999,
    "size": 10
}


4-4 附：深度分页 - 提升搜索量
提升搜索量
“changing the [index.max_result_window] index level setting”

通过设置index.max_result_window来突破10000数据

GET     /shop/_settings

PUT     /shop/_settings
{ 
    "index.max_result_window": "20000"
    
}


4-6 附：scroll 滚动搜索
一次性查询1万+数据，往往会造成性能影响，因为数据量太多了。这个时候可以使用滚动搜索，也就是 scroll。
滚动搜索可以先查询出一些数据，然后再紧接着依次往下查询。在第一次查询的时候会有一个滚动id，相当于一个锚标记，随后再次滚动搜索会需要上一次搜索的锚标记，根据这个进行下一次的搜索请求。每次搜索都是基于一个历史的数据快照，查询数据的期间，如果有数据变更，那么和搜索是没有关系的，搜索的内容还是快照中的数据。

scroll=1m，相当于是一个session会话时间，搜索保持的上下文时间为1分钟。
POST    /shop/_search?scroll=1m
{
    "query": { 
    	"match_all": {
    	}
    },  
    "sort" : ["_doc"], 
    "size":  5
}

POST    /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "your last scroll_id"
}


阶段三 · 分布式搜索引擎-ES
  

4-11 附：批量操作 bulk
基本语法
bulk操作和以往的普通请求格式有区别。不要格式化json，不然就不在同一行了，这个需要注意。

{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
1
2
3
4
5
                                代码块                                                            
{ action: { metadata }}代表批量操作的类型，可以是新增、删除或修改
\n是每行结尾必须填写的一个规范，每一行包括最后一行都要写，用于es的解析
{ request body }是请求body，增加和修改操作需要，删除操作则不需要
批量操作的类型
action 必须是以下选项之一:

create：如果文档不存在，那么就创建它。存在会报错。发生异常报错不会影响其他操作。
index：创建一个新文档或者替换一个现有的文档。
update：部分更新一个文档。
delete：删除一个文档。
metadata 中需要指定要操作的文档的_index 、 _type 和 _id，_index 、 _type也可以在url中指定

实操
create新增文档数据，在metadata中指定index以及type

POST    /_bulk
{"create": {"_index": "shop2", "_type": "_doc", "_id": "2001"}}
{"id": "2001", "nickname": "name2001"}
{"create": {"_index": "shop2", "_type": "_doc", "_id": "2002"}}
{"id": "2002", "nickname": "name2002"}
{"create": {"_index": "shop2", "_type": "_doc", "_id": "2003"}}
{"id": "2003", "nickname": "name2003"}
                                         
create创建已有id文档，在url中指定index和type

POST    /shop/_doc/_bulk
{"create": {"_id": "2003"}}
{"id": "2003", "nickname": "name2003"}
{"create": {"_id": "2004"}}
{"id": "2004", "nickname": "name2004"}
{"create": {"_id": "2005"}}
{"id": "2005", "nickname": "name2005"}
                                        
index创建，已有文档id会被覆盖，不存在的id则新增

POST    /shop/_doc/_bulk
{"index": {"_id": "2004"}}
{"id": "2004", "nickname": "index2004"}
{"index": {"_id": "2007"}}
{"id": "2007", "nickname": "name2007"}
{"index": {"_id": "2008"}}
{"id": "2008", "nickname": "name2008"}

                                                            
update跟新部分文档数据

POST    /shop/_doc/_bulk
{"update": {"_id": "2004"}}
{"doc":{ "id": "3004"}}
{"update": {"_id": "2007"}}
{"doc":{ "nickname": "nameupdate"}}
                                                       
delete批量删除
POST    /shop/_doc/_bulk
{"delete": {"_id": "2004"}}
{"delete": {"_id": "2007"}}

                                                            
综合批量各种操作

POST    /shop/_doc/_bulk
{"create": {"_id": "8001"}}
{"id": "8001", "nickname": "name8001"}
{"update": {"_id": "2001"}}
{"doc":{ "id": "20010"}}
{"delete": {"_id": "2003"}}
{"delete": {"_id": "2005"}}

```
