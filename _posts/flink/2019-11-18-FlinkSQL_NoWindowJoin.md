---
title: Flink NoWindowJoin原理
tagline: ""
category : flink
layout: post
tags : [flink, streamsets, realtime]
---

示例：

```
object StreamTableExample {

  // *************************************************************************
  //     PROGRAM
  // *************************************************************************

  def main(args: Array[String]): Unit = {

    // set up execution environment
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val tEnv = StreamTableEnvironment.create(env)

    val orderA = env.fromCollection(Seq(
      OrderA(1L, "beer", 3),
      OrderA(1L, "diaper", 4),
      OrderA(3L, "rubber", 2)))
      //.toTable(tEnv,'aId,'product1,'amount1)

    val orderB = env.fromCollection(Seq(
      OrderB(1L, "pen", 3),
      OrderB(2L, "rubber", 3),
      OrderB(4L, "beer", 1)))
      //.toTable(tEnv,'bId,'product2,'amount2)

    tEnv.registerDataStream("orderA",orderA)
    tEnv.registerDataStream("orderB",orderB)
    val result = tEnv.sqlQuery("select orderA.aId as aId,orderB.bId as bId, product1, amount1 from orderA  Inner join orderB on orderA.aId = orderB.bId" )
//    tEnv.registerDataStream("retTable",result.toRetractStream[OrderRet]);
//    // union the two tables
//    val result: DataStream[OrderRet] = orderA.join(orderB).where('aId === 'bId)
//      .select( 'aId,'bId,'product1, 'amount1)
//     // .where('amount > 2)
//      .toAppendStream[OrderRet]
//
//    tEnv.sqlQuery("select aId,bId,product1,amount1 from retTable where amount1>3")
    println(tEnv.explain(result))
    result.toRetractStream[OrderRet].print()

    env.execute()
  }

  // *************************************************************************
  //     USER DATA TYPES
  // *************************************************************************

  case class OrderA(aId: Long, product1: String, amount1: Int)

  case class OrderB(bId: Long, product2: String, amount2: Int)

  case class OrderRet(aId: Long,bId: Long, product1: String, amount1: Int)

}
```

sql语句：

```
select orderA.aId as aId,orderB.bId as bId, product1, amount1 from orderA  Inner join orderB on orderA.aId = orderB.bId
```
执行计划：

```
== Abstract Syntax Tree ==
LogicalProject(aId=[$0], bId=[$3], product1=[$1], amount1=[$2])
  LogicalJoin(condition=[=($0, $3)], joinType=[inner])
    LogicalTableScan(table=[[default_catalog, default_database, orderA]])
    LogicalTableScan(table=[[default_catalog, default_database, orderB]])

== Optimized Logical Plan ==
DataStreamCalc(select=[aId, bId, product1, amount1])
  DataStreamJoin(where=[=(aId, bId)], join=[aId, product1, amount1, bId], joinType=[InnerJoin])
    DataStreamScan(id=[1], fields=[aId, product1, amount1])
    DataStreamCalc(select=[bId])
      DataStreamScan(id=[2], fields=[bId, product2, amount2])

== Physical Execution Plan ==
Stage 1 : Data Source
	content : collect elements with CollectionInputFormat

Stage 2 : Data Source
	content : collect elements with CollectionInputFormat

	Stage 3 : Operator
		content : from: (aId, product1, amount1)
		ship_strategy : REBALANCE

		Stage 4 : Operator
			content : from: (bId, product2, amount2)
			ship_strategy : REBALANCE

			Stage 5 : Operator
				content : select: (bId)
				ship_strategy : FORWARD

				Stage 8 : Operator
					content : InnerJoin(where: (=(aId, bId)), join: (aId, product1, amount1, bId))
					ship_strategy : HASH

					Stage 9 : Operator
						content : select: (aId, bId, product1, amount1)
						ship_strategy : FORWARD

```
这里debug可以看下Optimized Logical Plan   
![No_Window_Join](https://raw.githubusercontent.com/2pc/2pc.github.io/master/images/no_window_join.png)

在经过sql解析生成AST以及各种逻辑/物理计划优化后最后调用toDataStream

这里会包装一串transformations，于StreamGraph生成有关

```
  private def toDataStream[T](
      table: Table,
      modifyOperation: OutputConversionModifyOperation)
    : DataStream[T] = {
    val transformations = planner
      .translate(Collections.singletonList(modifyOperation))
    val streamTransformation: Transformation[T] = getTransformation(
      table,
      transformations)
    scalaExecutionEnvironment.getWrappedStreamExecutionEnvironment.addOperator(streamTransformation)
    new DataStream[T](new JDataStream[T](
      scalaExecutionEnvironment
        .getWrappedStreamExecutionEnvironment, streamTransformation))
  }
```
这个addOperator添加到transformations
```
public void addOperator(Transformation<?> transformation) {
    Preconditions.checkNotNull(transformation, "transformation must not be null.");
    this.transformations.add(transformation);
}
```
transformations在后续会用来生成StreamGraph

```
   public StreamGraph getStreamGraph(String jobName) {
        return this.getStreamGraphGenerator().setJobName(jobName).generate();
    }

    private StreamGraphGenerator getStreamGraphGenerator() {
        if (this.transformations.size() <= 0) {
            throw new IllegalStateException("No operators defined in streaming topology. Cannot execute.");
        } else {
            return (new StreamGraphGenerator(this.transformations, this.config, this.checkpointCfg)).setStateBackend(this.defaultStateBackend).setChaining(this.isChainingEnabled).setUserArtifacts(this.cacheFile).setTimeCharacteristic(this.timeCharacteristic).setDefaultBufferTimeout(this.bufferTimeout);
        }
    }
```
这里展示了示例sql的transformations：   
[transformation1](https://raw.githubusercontent.com/2pc/2pc.github.io/master/images/transformation2.png)   
[transformation2](https://raw.githubusercontent.com/2pc/2pc.github.io/master/images/transformation1.png)
