---
title: dolphinscheduler调度系统源码
tagline: ""
category : Java
layout: post
tags : [Java, File, tools，OOM]
---
## 1.架构设计
### 1.1 老版(1.2.1为例)架构
![1.2.1设计](https://raw.githubusercontent.com/2pc/2pc.github.io/master/_posts/images/ds2.png)


### 1.2 新版(2.x github dev)架构  
![1.3设计](https://raw.githubusercontent.com/2pc/2pc.github.io/master/_posts/images/ds1.png)

## 2.任务分发
### 2.1 老版任务分发
#### 2.1.1 master将任务提交给zk
```
//ProcessDao.java
public Boolean submitTaskToQueue(TaskInstance taskInstance) {

    try{
        if(taskInstance.isSubProcess()){
            return true;
        }
        if(taskInstance.getState().typeIsFinished()){
            logger.info(String.format("submit to task queue, but task [%s] state [%s] is already  finished. ", taskInstance.getName(), taskInstance.getState().toString()));
            return true;
        }
        // task cannot submit when running
        if(taskInstance.getState() == ExecutionStatus.RUNNING_EXEUTION){
            logger.info(String.format("submit to task queue, but task [%s] state already be running. ", taskInstance.getName()));
            return true;
        }
        if(checkTaskExistsInTaskQueue(taskInstance)){
            logger.info(String.format("submit to task queue, but task [%s] already exists in the queue.", taskInstance.getName()));
            return true;
        }
        logger.info("task ready to queue: {}" , taskInstance);
        boolean insertQueueResult = taskQueue.add(DOLPHINSCHEDULER_TASKS_QUEUE, taskZkInfo(taskInstance));
        logger.info(String.format("master insert into queue success, task : %s", taskInstance.getName()) );
        return insertQueueResult;
    }catch (Exception e){
        logger.error("submit task to queue Exception: ", e);
        logger.error("task queue error : %s", JSONUtils.toJson(taskInstance));
        return false;
    }
}
//TaskQueueZkImpl.java
@Override
public boolean add(String key, String value){
    try {
        String taskIdPath = getTasksPath(key) + Constants.SINGLE_SLASH + value;
        zookeeperOperator.persist(taskIdPath, value);
        return true;
    } catch (Exception e) {
        logger.error("add task to tasks queue exception",e);
        return false;
    }

}
```
#### 2.1.2 work从zk获取任务
```
//FetchTaskThread.java
@Override
public void run() {
    logger.info("worker start fetch tasks...");
    while (Stopper.isRunning()){
        InterProcessMutex mutex = null;
        String currentTaskQueueStr = null;
        try {
            //......省略部分代码......
            //whether have tasks, if no tasks , no need lock  //get all tasks
            List<String> tasksQueueList = taskQueue.getAllTasks(Constants.DOLPHINSCHEDULER_TASKS_QUEUE);
            if (CollectionUtils.isEmpty(tasksQueueList)){
                Thread.sleep(Constants.SLEEP_TIME_MILLIS);
                continue;
            }
            // creating distributed locks, lock path /dolphinscheduler/lock/worker
            mutex = zkWorkerClient.acquireZkLock(zkWorkerClient.getZkClient(),
                    zkWorkerClient.getWorkerLockPath());


            // task instance id str
            List<String> taskQueueStrArr = taskQueue.poll(Constants.DOLPHINSCHEDULER_TASKS_QUEUE, taskNum);
            //......省略部分代码......
                // submit task
                workerExecService.submit(new TaskScheduleThread(taskInstance, processDao));

                // remove node from zk
                removeNodeFromTaskQueue(taskQueueStr);
            }

        }catch (Exception e){
            processErrorTask(currentTaskQueueStr);
            logger.error("fetch task thread failure" ,e);
        }finally {
            AbstractZKClient.releaseMutex(mutex);
        }
    }
}
//TaskQueueZkImpl
@Override
public List<String> poll(String key, int tasksNum) {
    try{
        List<String> list = zookeeperOperator.getChildrenKeys(getTasksPath(key));

        if(list != null && list.size() > 0){}
//......省略部分代码......
}
```

### 2.2 新版任务分发
#### 2.2.1  master任务分发
```

```
#### 2.2.2  master任务分发




### link
![图片来源](https://mp.weixin.qq.com/s/xHviN-2KW7Jwgq5hqhwQSQ)

