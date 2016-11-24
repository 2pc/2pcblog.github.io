---
layout: post
title: "From RankNet to LambdaRank to LambdaMART: An Overview"
keywords: ["ML","Datascience","LIBSVM","","LIBLINEAR","classification"]
description: "LIBSVM"
category: "LambdaMART"
tags: ["LambdaMART","Learning To Rank"]
---

对于文档i与j，设打分函数为F(X)（X=w1v1+w2v2+w3v3+...+wnvn）,则F(Xi)-F(Xj)越大，i排在j前面的概率越高,即F(Xi)-F(Xj)表示文档i排在j前面的概率

但是概率的范围应该是[0,1]之间，参考逻辑斯蒂回归的归一化函数归一化得

$$ 
P_ij=\frac e^{F(x_i)-F(x_j)} {1+e^{F(x_i)-F(x_j)} 
$$


[From RankNet to LambdaRank to
LambdaMART: An Overview](http://research.microsoft.com/en-us/um/people/cburges/tech_reports/MSR-TR-2010-82.pdf)
