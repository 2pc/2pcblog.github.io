---
title: Spark SQL QueryExecution
tagline: "spark"
category : spark
layout: post
tags : [Spark Sql]
---

```
  override def addBatch(batchId: Long, data: DataFrame): Unit = {
    val resolvedEncoder = encoder.resolveAndBind(
      data.logicalPlan.output,
      data.sparkSession.sessionState.analyzer)
    val rdd = data.queryExecution.toRdd.map[T](resolvedEncoder.fromRow)(encoder.clsTag)
    val ds = data.sparkSession.createDataset(rdd)(encoder)
    batchWriter(ds, batchId)
  }
```
toRdd
```
  /** Internal version of the RDD. Avoids copies and has no schema */
  lazy val toRdd: RDD[InternalRow] = executedPlan.execute()
```
 executedPlan: SparkPlan
```
  // executedPlan should not be used to initialize any SparkPlan. It should be
  // only used for execution.
  lazy val executedPlan: SparkPlan = prepareForExecution(sparkPlan)
```
sparkPlan: SparkPlan
```
  lazy val sparkPlan: SparkPlan = {
    SparkSession.setActiveSession(sparkSession)
    // TODO: We use next(), i.e. take the first plan returned by the planner, here for now,
    //       but we will implement to choose the best plan.
    planner.plan(ReturnAnswer(optimizedPlan)).next()
  }
```
optimizedPlan: LogicalPlan
```
  lazy val optimizedPlan: LogicalPlan = sparkSession.sessionState.optimizer.execute(withCachedData)
```
withCachedData: LogicalPlan
```
  lazy val withCachedData: LogicalPlan = {
    assertAnalyzed()
    assertSupported()
    sparkSession.sharedState.cacheManager.useCachedData(analyzed)
  }

```
analyzed: LogicalPlan
```
  lazy val analyzed: LogicalPlan = {
    SparkSession.setActiveSession(sparkSession)
    sparkSession.sessionState.analyzer.executeAndCheck(logical)
  }

```
