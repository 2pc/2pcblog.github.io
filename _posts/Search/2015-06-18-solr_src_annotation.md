---
layout: post
title: "Solr Code Annotation"
keywords: ["distributed","Search","SolrCloud"]
description: "SolrCloud Leader Elect"
category: "distributed"
tags: ["distributed","Search","SolrCloud"]

Solr Query Syntax and Parsing 查询语法
>
1. (Common Query Parameters)[https://cwiki.apache.org/confluence/display/solr/Common+Query+Parameters]
2. (The Standard Query Parser)[https://cwiki.apache.org/confluence/display/solr/The+Standard+Query+Parser]
3. (The DisMax Query Parser)[https://cwiki.apache.org/confluence/display/solr/The+DisMax+Query+Parser]
4. (The Extended DisMax Query Parser)[https://cwiki.apache.org/confluence/display/solr/The+Extended+DisMax+Query+Parser]
5. (Function Queries)[https://cwiki.apache.org/confluence/display/solr/Function+Queries]
6. (Local Parameters in Queries)[https://cwiki.apache.org/confluence/display/solr/Local+Parameters+in+Queries]
7. (Other Parsers)[https://cwiki.apache.org/confluence/display/solr/Other+Parsers]

查询关键流程

```
SearchHandler.handleRequestBody( c.prepare(rb)--QueryComponent.prepare())--> SearchHandler.handleRequestBody( c.process(rb)--QueryComponentprocess())-->SearchHandler.handleRequestBody( c.finishStage(rb)--QueryComponent.finishStage())
```
QueryComponent.prepare中进行参数解析，得到QParser，Query,FilterQuery等

```

```
>
1. [Solr dismax 源码详解以及使用方法](http://www.wxdl.cn/index/solr-dismax.html)
2. [Solr 的edismax与dismax比较与分析](http://www.linuxidc.com/Linux/2012-10/72373.htm)
