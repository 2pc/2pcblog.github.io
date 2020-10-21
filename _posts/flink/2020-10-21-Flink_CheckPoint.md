---
title: Flink CheckPoint实现
tagline: ""
category : flink
layout: post
tags : [flink, streamsets, realtime]
---



//TaskExecutor triggerCheckpointBarrier

```
public CompletableFuture<Acknowledge> triggerCheckpoint(
    ExecutionAttemptID executionAttemptID,
    long checkpointId,
    long checkpointTimestamp,
    CheckpointOptions checkpointOptions,
    boolean advanceToEndOfEventTime) {
  log.debug("Trigger checkpoint {}@{} for {}.", checkpointId, checkpointTimestamp, executionAttemptID);

  final CheckpointType checkpointType = checkpointOptions.getCheckpointType();
  if (advanceToEndOfEventTime && !(checkpointType.isSynchronous() && checkpointType.isSavepoint())) {
    throw new IllegalArgumentException("Only synchronous savepoints are allowed to advance the watermark to MAX.");
  }

  final Task task = taskSlotTable.getTask(executionAttemptID);

  if (task != null) {
    task.triggerCheckpointBarrier(checkpointId, checkpointTimestamp, checkpointOptions, advanceToEndOfEventTime);

    return CompletableFuture.completedFuture(Acknowledge.get());
  } else {
    final String message = "TaskManager received a checkpoint request for unknown task " + executionAttemptID + '.';

    log.debug(message);
    return FutureUtils.completedExceptionally(new CheckpointException(message, CheckpointFailureReason.TASK_CHECKPOINT_FAILURE));
  }
}

```
