---
title: Flink On Yarn
tagline: ""
category : flink
layout: post
tags : [flink, streamsets, realtime]
---

### 配置环境变量HADOOP_CONF_DIR,将yarn的配置文件解压在此目录

```
export HADOOP_CONF_DIR=/opt/soft/yarn-conf
```
### yarn-session 启动

```
./yarn-session.sh -n 8 -s 8 -jm 1024 -tm 1024 -nm flink –d
```
