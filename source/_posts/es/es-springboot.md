---
title: es-springboot
date: 2021-09-12 19:23:56
tags: es
---

## es-springboot
```xml
    版本协调
    目前springboot-data-elasticsearch中的es版本贴合为es-6.4.3，如此一来版本需要统一，把es进行降级。等springboot升级es版本后可以在对接最新版的7.4。
    
    Netty issue fix
    @Configuration
    public class ESConfig {
    
        /**
         * 解决netty引起的issue
         */
        @PostConstruct
        void init() {
            System.setProperty("es.set.netty.runtime.available.processors", "false");
        }
    
    }

```

## logstatch 数据同步到es
```xml
布式搜索引擎-ES
  

7-4 附：logstash数据同步 - 数据同步配置
下载logstatch
logstash同步数据库配置
上传并解压logstash，位置放在如下：

创建文件名：logstash-db-sync.conf，后缀为conf，文件名随意，位置也随意（可以参考课程）

把数据库驱动拷贝：

配置内容如下：

input {
    jdbc {
        # 设置 MySql/MariaDB 数据库url以及数据库名称
        jdbc_connection_string => "jdbc:mysql://192.168.1.6:3306/foodie-shop-dev?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true"
        # 用户名和密码
        jdbc_user => "root"
        jdbc_password => "root"
        # 数据库驱动所在位置，可以是绝对路径或者相对路径
        jdbc_driver_library => "/usr/local/logstash-6.4.3/sync/mysql-connector-java-5.1.41.jar"
        # 驱动类名
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        # 开启分页
        jdbc_paging_enabled => "true"
        # 分页每页数量，可以自定义
        jdbc_page_size => "10000"
        # 执行的sql文件路径
        statement_filepath => "/usr/local/logstash-6.4.3/sync/foodie-items.sql"
        # 设置定时任务间隔  含义：分、时、天、月、年，全部为*默认含义为每分钟跑一次任务
        schedule => "* * * * *"
        # 索引类型
        type => "_doc"
        # 是否开启记录上次追踪的结果，也就是上次更新的时间，这个会记录到 last_run_metadata_path 的文件
        use_column_value => true
        # 记录上一次追踪的结果值
        last_run_metadata_path => "/usr/local/logstash-6.4.3/sync/track_time"
        # 如果 use_column_value 为true， 配置本参数，追踪的 column 名，可以是自增id或者时间
        tracking_column => "updated_time"
        # tracking_column 对应字段的类型
        tracking_column_type => "timestamp"
        # 是否清除 last_run_metadata_path 的记录，true则每次都从头开始查询所有的数据库记录
        clean_run => false
        # 数据库字段名称大写转小写
        lowercase_column_names => false
    }
}
output {
    elasticsearch {
        # es地址
        hosts => ["192.168.1.187:9200"]
        # 同步的索引名
        index => "foodie-items"
        # 设置_docID和数据相同
        document_id => "%{id}"
    }
    # 日志输出
    stdout {
        codec => json_lines
    }}

 SELECT
     i.id as itemId,
     i.item_name as itemName,
     i.sell_counts as sellCounts,
     ii.url as imgUrl,
     tempSpec.price_discount as price,
     i.updated_time as updated_time
 FROM
     items i
 LEFT JOIN
     items_img ii
 on
     i.id = ii.item_id
 LEFT JOIN
     (SELECT item_id,MIN(price_discount) as price_discount from items_spec GROUP BY item_id) tempSpec
 on
     i.id = tempSpec.item_id
 WHERE
     ii.is_main = 1
     and
     i.updated_time >= :sql_last_value

                                代码块                                                            
启动logstatsh
./logstash -f /usr/local/logstash-6.4.3/sync/logstash-db-sync.conf


 
 

7-7 附：Logstash数据同步 - 自定义模板配置中文分词
引子
目前的数据同步，mappings映射会自动创建，但是分词不会，还是会使用默认的，而我们需要中文分词，这个时候就需要自定义模板功能来设置分词了。

查看Logstash默认模板
GET     /_template/logstash
1
                                代码块                                                            
修改模板如下
{
    "order": 0,
    "version": 1,
    "index_patterns": ["*"],
    "settings": {
        "index": {
            "refresh_interval": "5s"
        }
    },
    "mappings": {
        "_default_": {
            "dynamic_templates": [
                {
                    "message_field": {
                        "path_match": "message",
                        "match_mapping_type": "string",
                        "mapping": {
                            "type": "text",
                            "norms": false
                        }
                    }
                },
                {
                    "string_fields": {
                        "match": "*",
                        "match_mapping_type": "string",
                        "mapping": {
                            "type": "text",
                            "norms": false,
                            "analyzer": "ik_max_word",
                            "fields": {
                                "keyword": {
                                    "type": "keyword",
                                    "ignore_above": 256
                                }
                            }
                        }
                    }
                }
            ],
            "properties": {
                "@timestamp": {
                    "type": "date"
                },
                "@version": {
                    "type": "keyword"
                },
                "geoip": {
                    "dynamic": true,
                    "properties": {
                        "ip": {
                            "type": "ip"
                        },
                        "location": {
                            "type": "geo_point"
                        },
                        "latitude": {
                            "type": "half_float"
                        },
                        "longitude": {
                            "type": "half_float"
                        }
                    }
                }
            }
        }
    },
    "aliases": {}
}

                                代码块                                                            
新增如下配置，用于更新模板，设置中文分词
# 定义模板名称
template_name => "myik"
# 模板所在位置
template => "/usr/local/logstash-6.4.3/sync/logstash-ik.json"
# 重写模板
template_overwrite => true
# 默认为true，false关闭logstash自动管理模板功能，如果自定义模板，则设置为false
manage_template => false

                                代码块                                                            
重新运行Logstash进行同步
./logstash -f /usr/local/logstash-6.4.3/sync/logstash-db-sync.conf

```

## es 复习路线
![](/../../static/es/es复习.jpg)
```xml
  

8-7 阶段复习
阶段复习
那么到这里，Elasticsearch部分讲完啦~学完后可能很多初次接触es的同学会觉得搜索起来要写json会很陌生，而且隔了一段时间再看可能也会忘记，其实没有关系，这些内容是没有必要去死记硬背的，在学习的过程中做好相关笔记，在用到的时候去查一下，或者百度谷歌一下就都有了。那么咱们针对本阶段的学习做个简短的总结。来看一下下方思维脑图来梳理内容：


本阶段主要针对es的讲解，首先我们讲了什么是分布式搜索，以及lucene、solr与es的对比，他们都是全文检索，而且底层基于lucene，目前已es使用最为流行；随后我们又讲了什么是倒排索引，这是搜索引擎里的核心。此外呢我们也在最一开始就讲了集群原理，因为es的使用主要以集群为主，此外还包含了主分片与副本分片的概念。
学习es必须要安装，首先我们在linux中安装了es7，另外呢，我们也安装了head插件，这个插件是基于es的可视化操作，使用起来非常方便，当然我们也通过使用postman对es做了一些基本的操作。
接下来我们对es做了一个快速入门，主要是创建索引与映射，文档的CRUD以及es乐观锁的讲解。
在es中，默认的分词是基于英文的，毕竟是老外发明的，对中文支持的不好，所以我们需要自己去安装中文分词器，如此就能对中文词语做分词了，安装好之后，还能对特定的词汇去设置自定义词库，比如慕课网不是词语，我们就能为他自定义。
DSL搜索是es中的复杂查询，而且也是使用的最多的，其中主要包含了term/match/multi_match/boost/bool/sort/highlight等，这些搜索可以满足日常使用，如果语法忘记，百度上搜一下即可，json没必要手写，直接复制修改就能达到效果。另外我们也讲了分页，分页的话有一点需要注意，那就是深度分页，分页太深，会造成性能的影响。此外我们还讲了滚动搜索，这个相当于是快照分页功能，可以大批量的把数据查询出来。
批量操作在redis中我们也提到过，在es中也有，因为批量操作可以提高性能与吞吐量，查询使用mget，增加删除修改使用bulk就能达到效果。
es集群是非常重要的，在企业里es往往以集群的形式存在，一般以3个节点居多，在老版本中，es集群有一个需要注意的就是脑裂问题，一定要设置半数master节点以上的投票，才能选举成为一个master，否则会出现脑裂现象，这个在老版本es中需要注意。
es提供了rest-api，所以我们可以使用es结合springboot来实现检索功能，主要是通过elasticsearchTemplate来进行相关检索，比如创建索引，文档的crud，分页，高亮，排序等。
要查询商品数据，那么我们必须要把数据同步到es中，这里我们使用了logstash，需要注意，版本要和es的安装版本一致，同步好以后就能检索相关数据了。自动同步以后，logstash会使用自定义的模板为我们创建mappings映射，这样的话中文分词就不行了，我们可以手动创建或者使用自定义logstash模板来设置field的中文分词就行了。
最后我们把es整合到了项目中，实现了基于商品的全文检索，并且还有分页高亮以及排序。



```
