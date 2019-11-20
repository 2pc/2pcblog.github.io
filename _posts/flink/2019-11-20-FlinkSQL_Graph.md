---
title: Flink Graph
tagline: ""
category : flink
layout: post
tags : [flink, streamsets, realtime]
---

transformations-->streamGraph(Operator)-->JobGraph-->ExecutionGraph

JobGraph-->ExecutionGraph LegacyScheduler
```
//JobMaster
schedulerAssignedFuture.thenRun(this::startScheduling);
//JobMaster
private void startScheduling() {
  checkState(jobStatusListener == null);
  // register self as job status change listener
  jobStatusListener = new JobManagerJobStatusListener();
  schedulerNG.registerJobStatusListener(jobStatusListener);

  schedulerNG.startScheduling();
}
//LegacyScheduler
public void startScheduling() {
  mainThreadExecutor.assertRunningInMainThread();

  try {
    executionGraph.scheduleForExecution();
  }
  catch (Throwable t) {
    executionGraph.failGlobal(t);
  }
}

```
看下executionGraph的构造
```
//LegacyScheduler构造函数
this.executionGraph = createAndRestoreExecutionGraph(jobManagerJobMetricGroup, checkNotNull(shuffleMaster), checkNotNull(partitionTracker));
//LegacyScheduler.createAndRestoreExecutionGraph
private ExecutionGraph createAndRestoreExecutionGraph(
    JobManagerJobMetricGroup currentJobManagerJobMetricGroup,
    ShuffleMaster<?> shuffleMaster,
    PartitionTracker partitionTracker) throws Exception {

  ExecutionGraph newExecutionGraph = createExecutionGraph(currentJobManagerJobMetricGroup, shuffleMaster, partitionTracker);

  final CheckpointCoordinator checkpointCoordinator = newExecutionGraph.getCheckpointCoordinator();

  if (checkpointCoordinator != null) {
    // check whether we find a valid checkpoint
    if (!checkpointCoordinator.restoreLatestCheckpointedState(
      newExecutionGraph.getAllVertices(),
      false,
      false)) {

      // check whether we can restore from a savepoint
      tryRestoreExecutionGraphFromSavepoint(newExecutionGraph, jobGraph.getSavepointRestoreSettings());
    }
  }

  return newExecutionGraph;
}
```

link:
[Flink Job提交流程(Dispatcher之后)](https://blog.csdn.net/lisenyeahyeah/article/details/100662367)
