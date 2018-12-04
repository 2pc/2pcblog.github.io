---
layout: post
title: "Etcd log replication"
keywords: ["distributed","etcd"]
description: "Etcd log replication"
category: "distributed"
tags: ["etcd"]
---

过半数逻辑的优化pr: 

![maybeCommit](https://raw.githubusercontent.com/2pc/2pc.github.io/master/images/maybeCommit.jpg)

r.quorum()取半数以上

```
func (r *raft) quorum() int { return len(r.prs)/2 + 1 }
```
