---
layout: post
title: "Elasticsearch"
keywords: ["distributed","Search"]
description: "Elasticsearch"
category: "distributed"
tags: ["distributed","Search","Elasticsearch"]
---

### 安装配置
这里使用的2.3.3版本，这个版本默认不能用root直接启动。
具体参考[Bootstrap.java](https://github.com/elastic/elasticsearch/blob/93de1ed6068e8e9f35897f623efe00aa3cfafeea/core/src/main/java/org/elasticsearch/bootstrap/Bootstrap.java#L89)中的代码

```
public static void initializeNatives(Path tmpFile, boolean mlockAll, boolean seccomp, boolean ctrlHandler) {
  final ESLogger logger = Loggers.getLogger(Bootstrap.class);

  // check if the user is running as root, and bail
  if (Natives.definitelyRunningAsRoot()) {
      if (Boolean.parseBoolean(System.getProperty("es.insecure.allow.root"))) {
          logger.warn("running as ROOT user. this is a bad idea!");
      } else {
          throw new RuntimeException("don't run elasticsearch as root.");
      }
  }
```
可以通过参数-Des.insecure.allow.root=true 来实现
```
bin/elasticsearch  -Des.insecure.allow.root=true 
```

先给Elasticsearch启动，建立一个账户

```
Elasticsearch运行账户：Elasticsearch
Elasticsearch环境路径：/data/elasticsearch
```

新增用户并将/data/elasticsearch目录权限给Elasticsearch用户组(mac os新增用户elasticsearch，加入elasticsearch组)

```
mkdir  /data/elasticsearch/
useradd elasticsearch
passwd  elasticsearch
chown -R elasticsearch:elasticsearch  /data/elasticsearch/
cd /data/elasticsearch/
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.3/elasticsearch-2.3.3.tar.gz
tar zxvf elasticsearch-2.3.3.tar.gz 
chown -R elasticsearch:elasticsearch elasticsearch-2.3.3
su elasticsearch
cd /data/elasticsearch/elasticsearch-2.3.3
bin/elasticsearch
```
安装ik分词,[medcl](https://github.com/medcl)大神更新很快

```
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v1.9.3/elasticsearch-analysis-ik-1.9.3.zip
unzip  elasticsearch-analysis-ik-1.9.3.zip  -d  ./plugins/ik
```

create an index,pretty 返回数据格式不一样
```
curl -XPUT 'localhost:9200/indexname'
curl -XPUT 'localhost:9200/indexname?pretty'
```

Index and Query a Document

index a simple indexname document，indextype

```
bash-3.2$ curl -XPUT 'localhost:9200/indexname/indextype/1?pretty' -d '
{
  "name": "John Doe"
}'
{
  "_index" : "indexname",
  "_type" : "indextype",
  "_id" : "1",
  "_version" : 3,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : false
}
```
retrieve that document

```
curl -XGET 'localhost:9200/indexname/indextype/1?pretty'
{
  "_index" : "indexname",
  "_type" : "indextype",
  "_id" : "1",
  "_version" : 3,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```
list All indices

```
bash-3.2$ curl 'localhost:9200/_cat/indices?v'
health status index     pri rep docs.count docs.deleted store.size pri.store.size
yellow open   index       5   1          0            0       795b           795b
yellow open   customer2   5   1          0            0       795b           795b
yellow open   twitter     5   1          0            0       795b           795b
yellow open   db_news     5   1          1            0      4.2kb          4.2kb
yellow open   indexname   5   1          1            0      7.1kb          7.1kb
yellow open   customer    5   1          1            0      3.5kb          3.5kb
```
Delete an Index

```
bash-3.2$ curl -XDELETE 'localhost:9200/customer?pretty'
{
  "acknowledged" : true
}
bash-3.2$ curl -XDELETE 'localhost:9200/indexname?pretty'
{
  "acknowledged" : true
}
bash-3.2$ curl -XDELETE 'localhost:9200/customer2?pretty'
{
  "acknowledged" : true
}
```

ik分词测试sense建立type,news

```
curl -XPUT 'localhost:9200/index'
```
mapping

```
POST /index/news/_mapping -d’
{
  fulltext: {
    "_all": {
      "analyzer": "ik_max_word",
      "search_analyzer": "ik_max_word",
      "term_vector": "no",
      "store": "false"
    },
    "properties": {
      "content": {
        "type": "string",
        "store": "no",
        "term_vector": "with_positions_offsets",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_max_word",
        "include_in_all": "true",
        "boost": 8
      }
    }
  }
}
```
索引

```
bash-3.2$ curl -XPOST http://localhost:9200/index/news/1 -d '{
"content":"权力的游戏琼恩雪诺的身世彩蛋你有发现吗 琼恩雪诺的真实身世"}'

bash-3.2$ curl -XPOST http://localhost:9200/index/news/1 -d '
{"content":"iPhone 7为啥取消64GB？苹果太有心计"}'

bash-3.2$ curl -XPOST http://localhost:9200/index/news/1 -d '{"content":"马克飞象是一款专为印象笔记（Evernote）打造的Markdown编辑器，通过精心的设计与技术实现，配合印象笔记强大的存储和同步功能，带来前所未有的书写体验。特点概述"}'

```

查询

boolean查询must,should,must_not
>
1. must: 文档必须完全匹配条件
2. should: should下面会带一个以上的条件，至少满足一个条件，这个文档就符合should
3. must_not: 文档必须不匹配条件


[2.3版本的参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/index.html)，

可以看看Breaking changes

>
1. [2.3-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-2.3.html)
2. [2.2-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-2.2.html)
3. [2.1-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-2.1.html)
4. [2.0-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-2.0.html)
5. [1.6-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-1.6.html)
6. [1.4-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-1.4.html)
7. [1.0-Breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/breaking-changes-1.0.html)

参考

>
* [Elasticsearch 权威指南](http://learnes.net/index.html)
* [Elasticsearch Reference2.3](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/index.html)
