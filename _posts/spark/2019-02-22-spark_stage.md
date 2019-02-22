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
////SparkContext.scala
def runJob[T, U: ClassTag](rdd: RDD[T], func: Iterator[T] => U): Array[U] = {
runJob(rdd, func, 0 until rdd.partitions.length)
}
...
def runJob[T, U: ClassTag](
  rdd: RDD[T],
  func: (TaskContext, Iterator[T]) => U,
  partitions: Seq[Int],
  resultHandler: (Int, U) => Unit): Unit = {
if (stopped.get()) {
  throw new IllegalStateException("SparkContext has been shutdown")
}
val callSite = getCallSite
val cleanedFunc = clean(func)
logInfo("Starting job: " + callSite.shortForm)
if (conf.getBoolean("spark.logLineage", false)) {
  logInfo("RDD's recursive dependencies:\n" + rdd.toDebugString)
}
dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, resultHandler, localProperties.get)
progressBar.foreach(_.finishAll())
rdd.doCheckpoint()
}
//DAGScheduler.scala
def submitJob[T, U](
  rdd: RDD[T],
  func: (TaskContext, Iterator[T]) => U,
  partitions: Seq[Int],
  callSite: CallSite,
  resultHandler: (Int, U) => Unit,
  properties: Properties): JobWaiter[U] = {
// Check to make sure we are not launching a task on a partition that does not exist.
val maxPartitions = rdd.partitions.length
partitions.find(p => p >= maxPartitions || p < 0).foreach { p =>
  throw new IllegalArgumentException(
    "Attempting to access a non-existent partition: " + p + ". " +
      "Total number of partitions: " + maxPartitions)
}

val jobId = nextJobId.getAndIncrement()
if (partitions.size == 0) {
  // Return immediately if the job is running 0 tasks
  return new JobWaiter[U](this, jobId, 0, resultHandler)
}

assert(partitions.size > 0)
val func2 = func.asInstanceOf[(TaskContext, Iterator[_]) => _]
val waiter = new JobWaiter(this, jobId, partitions.size, resultHandler)
eventProcessLoop.post(JobSubmitted(
  jobId, rdd, func2, partitions.toArray, callSite, waiter,
  SerializationUtils.clone(properties)))
waiter
}

//EventLoop.scala
def post(event: E): Unit = {
eventQueue.put(event)
}

private val eventThread = new Thread(name) {
setDaemon(true)

override def run(): Unit = {
  try {
    while (!stopped.get) {
      val event = eventQueue.take()
      try {
        onReceive(event)
      } catch {
        case NonFatal(e) =>
          try {
            onError(e)
          } catch {
            case NonFatal(e) => logError("Unexpected error in " + name, e)
          }
      }
    }
  } catch {
```
onReceive的实现类

```
//DAGScheduler.scala-->DAGSchedulerEventProcessLoop
override def onReceive(event: DAGSchedulerEvent): Unit = {
val timerContext = timer.time()
try {
  doOnReceive(event)
} finally {
  timerContext.stop()
}
}

private def doOnReceive(event: DAGSchedulerEvent): Unit = event match {
case JobSubmitted(jobId, rdd, func, partitions, callSite, listener, properties) =>
  dagScheduler.handleJobSubmitted(jobId, rdd, func, partitions, callSite, listener, properties)

```
