---
title: Structured Streaming with ElasticSearch
tagline: "spark"
category : spark
layout: post
tags : [Spark]
---

依赖
```
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch-spark-20_2.11</artifactId>
    <version>6.2.4</version>
</dependency>
```
writeStream

```
  .writeStream
  .outputMode(OutputMode.Append())
 // .format("es") classNotFound es.DefaultSource
  .format("org.elasticsearch.spark.sql")
  .option("es.nodes", "localhost")
  .option("es.port", "9200")
  .option("checkpointLocation", "/tmp/checkpointLocation")
 // .option("es.mapping.id", "id")//没有mapping.id呢
  .trigger(Trigger.ProcessingTime(10, TimeUnit.SECONDS))
  .start("a/profile")
  .awaitTermination()
  
```
