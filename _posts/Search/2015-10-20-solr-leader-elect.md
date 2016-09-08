---
layout: post
title: "SolrCloud Leader Elect"
keywords: ["distributed","Search","SolrCloud"]
description: "SolrCloud Leader Elect"
category: "distributed"
tags: ["distributed","Search","SolrCloud"]
---
SolrCloud 领导选举，最小seq,这点有点同Kafka seq(0)

主要有两部分，一个是集群的leader选举过程，一个是collection的shard的leader选举过程

集群的leader选举
>
1.主要相关的类：OverseerElectionContext
2.相关节点：/overseer_elect/election可以说leader候选节点，/overseer_elect/election是选举出来的leader

```
[zk: 172.16.113.253:2182,172.16.113.254:2182/galaxy2/solr(CONNECTED) 56] ls /overseer_elect/election
[166405419031556844-172.17.52.154:8080_solr-n_0000000117, 166405419031556837-172.17.52.155:8080_solr-n_0000000115]
[zk: 172.16.113.253:2182,172.16.113.254:2182/galaxy2/solr(CONNECTED) 57] get  /overseer_elect/election
null
cZxid = 0x36002c1fe2
ctime = Fri May 06 06:02:06 EDT 2016
mZxid = 0x36002c1fe2
mtime = Fri May 06 06:02:06 EDT 2016
pZxid = 0x36002d5efe
cversion = 234
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
```

collection的shard的leader选举
>
1.主要相关的类：OverseerElectionContext
2.相关节点：/collections/collection/leaders/shard1(shardN表示这个collection得shard) 可以说某个shard对应的leader候选节点，/collections/collection/leader_elect/shard1/election是选举出来的leader

```
[zk: 172.16.113.253:2182,172.16.113.254:2182/galaxy2/solr(CONNECTED) 8] ls   /collections/song/leader_elect/shard1/election
[166405419031556844-core_node1-n_0000000028, 166405419031556837-core_node2-n_0000000026]
[zk: 172.16.113.253:2182,172.16.113.254:2182/galaxy2/solr(CONNECTED) 9] get  /collections/song/leaders/shard1              
{
  "core":"song_shard1_replica1",
  "node_name":"172.17.52.155:8080_solr",
  "base_url":"http://172.17.52.155:8080/solr"}
cZxid = 0x36002d5ea2
ctime = Tue Sep 06 05:18:13 EDT 2016
mZxid = 0x36002d5ea2
mtime = Tue Sep 06 05:18:13 EDT 2016
pZxid = 0x36002d5ea2
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x24f30d8d21372e5
dataLength = 125
numChildren = 0
```

SolrCloud Group

```
Caused by: java.lang.OutOfMemoryError: Java heap space
        at org.apache.lucene.codecs.blocktree.SegmentTermsEnumFrame.<init>(SegmentTermsEnumFrame.java:53)
        at org.apache.lucene.codecs.blocktree.SegmentTermsEnum.<init>(SegmentTermsEnum.java:78)
        at org.apache.lucene.codecs.blocktree.FieldReader.iterator(FieldReader.java:156)
        at org.apache.lucene.index.ExitableDirectoryReader$ExitableTerms.iterator(ExitableDirectoryReader.java:141)
        at org.apache.lucene.index.TermContext.build(TermContext.java:94)
        at org.apache.lucene.search.TermQuery.createWeight(TermQuery.java:191)
        at org.apache.lucene.search.IndexSearcher.createWeight(IndexSearcher.java:851)
        at org.apache.lucene.search.BooleanWeight.<init>(BooleanWeight.java:57)
        at org.apache.lucene.search.BooleanQuery.createWeight(BooleanQuery.java:184)
        at org.apache.lucene.search.IndexSearcher.createWeight(IndexSearcher.java:851)
        at org.apache.lucene.search.BooleanWeight.<init>(BooleanWeight.java:57)
        at org.apache.lucene.search.BooleanQuery.createWeight(BooleanQuery.java:184)
        at org.apache.lucene.search.FilteredQuery.createWeight(FilteredQuery.java:81)
        at org.apache.lucene.search.IndexSearcher.createWeight(IndexSearcher.java:851)
        at org.apache.lucene.search.IndexSearcher.createNormalizedWeight(IndexSearcher.java:834)
        at org.apache.lucene.search.IndexSearcher.search(IndexSearcher.java:485)
        at org.apache.solr.search.Grouping.searchWithTimeLimiter(Grouping.java:456)
        at org.apache.solr.search.Grouping.execute(Grouping.java:407)
        at org.apache.solr.handler.component.QueryComponent.process(QueryComponent.java:492)
        at org.apache.solr.handler.component.SearchHandler.handleRequestBody(SearchHandler.java:255)
        at org.apache.solr.handler.RequestHandlerBase.handleRequest(RequestHandlerBase.java:143)
        at org.apache.solr.core.SolrCore.execute(SolrCore.java:2064)
        at org.apache.solr.servlet.HttpSolrCall.execute(HttpSolrCall.java:654)
        at org.apache.solr.servlet.HttpSolrCall.call(HttpSolrCall.java:450)
        at org.apache.solr.servlet.SolrDispatchFilter.doFilter(SolrDispatchFilter.java:227)
        at org.apache.solr.servlet.SolrDispatchFilter.doFilter(SolrDispatchFilter.java:196)
        at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1652)
        at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:585)
        at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
        at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:577)
        at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
        at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
```
