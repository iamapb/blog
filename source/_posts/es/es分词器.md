---
title: es分词器
date: 2022-06-21 17:05:14
tags: es
---
 ## es分词器
### 1 介绍
```xml
   分词(Analysis)是指将一段全文本转换为一系列单词(term)的过程，转换后的单词可用于构建倒排索引。
```
> 分词器: 执行分词过程的插件叫做分词器，由三部分组成:

1 字符过滤器(Character filter) : 对文本中的特殊字符进行转换或过滤，对 html标签过滤掉
2 单词切分器(Tokenizer): 将一段全文切分成多个单词(term),对于英文，可以用空格作为分隔符
3 单词过滤器(Token filter) 切好的单词要经过一系列的token filter的处理


### 2 内置分词器
> 1 standard 分词器:
> Elasticsearch中的默认分词器，当你没有明确为一个索引字段指定分词器时，那么该字段的分词器就是standard

分词后的效果
![stadard分词器](../../static/es/standard分词器.jpg)

