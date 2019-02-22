
---
title: Spark stage划分
tagline: "spark"
category : spark
layout: post
tags : [Spark thrift]
---
rdd-->job-->stage-->task
```
sc.parallelize(1 to 10000, 2).map { i => Thread.sleep(10); i }.count()
def parallelize[T: ClassTag](
    seq: Seq[T],
    numSlices: Int = defaultParallelism): RDD[T] = withScope {
  assertNotStopped()
  new ParallelCollectionRDD[T](this, seq, numSlices, Map[Int, Seq[String]]())
}
//SparkContext.scala
private[spark] class ParallelCollectionRDD[T: ClassTag](
    sc: SparkContext,
    @transient private val data: Seq[T],
    numSlices: Int,
    locationPrefs: Map[Int, Seq[String]])
    extends RDD[T](sc, Nil) {
//RDD.scala
def count(): Long = sc.runJob(this, Utils.getIteratorSize _).sum
```
