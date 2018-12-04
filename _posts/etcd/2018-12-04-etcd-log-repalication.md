---
layout: post
title: "Etcd log replication"
keywords: ["distributed","etcd"]
description: "Etcd log replication"
category: "distributed"
tags: ["etcd"]
---

![maybeCommit](https://raw.githubusercontent.com/2pc/2pc.github.io/master/images/maybeCommit.jpg)

r.quorum()取半数以上

```
func (r *raft) quorum() int { return len(r.prs)/2 + 1 }
```
