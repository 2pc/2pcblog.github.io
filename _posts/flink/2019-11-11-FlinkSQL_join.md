---
title: Flink join原理
tagline: ""
category : flink
layout: post
tags : [flink, streamsets, realtime]
---

StreamExecWindowJoin


```
/**
 * A CoProcessFunction to execute time-bounded stream inner-join.
 * Two kinds of time criteria:
 * "L.time between R.time + X and R.time + Y" or "R.time between L.time - Y and L.time - X"
 * X and Y might be negative or positive and X <= Y.
 */
abstract class TimeBoundedStreamJoin extends CoProcessFunction<BaseRow, BaseRow, BaseRow> {}
 ```
