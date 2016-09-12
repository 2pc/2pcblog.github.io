---
layout: post
title: "Zookeeper"
keywords: ["distributed"]
description: "Paxos协议"
category: "distributed"
tags: ["distributed","Paxos","Zookeeper"]
---

### Zookeepeer 通信以及心跳机制

### Zookeepeer Leader 选举

#### 如何确定两张选票的大小

注释已经写的很清楚了

>
1. 选举轮数epoch的比较，这个大的，选票就大
2. 选举轮数相同的话，比较(事务号)zxid，事务号大，选票也大
3. 选举轮数，事务号都一样，比较节点的id,id大的选票也大


```
protected boolean totalOrderPredicate(long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {
    LOG.debug("id: " + newId + ", proposed id: " + curId + ", zxid: 0x" +
            Long.toHexString(newZxid) + ", proposed zxid: 0x" + Long.toHexString(curZxid));
    if(self.getQuorumVerifier().getWeight(newId) == 0){
        return false;
    }
    
    /*
     * We return true if one of the following three cases hold:
     * 1- New epoch is higher
     * 2- New epoch is the same as current epoch, but new zxid is higher
     * 3- New epoch is the same as current epoch, new zxid is the same
     *  as current zxid, but server id is higher.
     */
    
    return ((newEpoch > curEpoch) || 
            ((newEpoch == curEpoch) &&
            ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId)))));
}
```




### Zookeepeer 读写流程

