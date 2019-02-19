---
title: float&double&decimal精度损失
tagline: "spark"
category : java
layout: post
tags : [Spark,scala,java]
---
起初是同事在用spark sql时候反馈的一个问题，sql如下

```
%hive
select 
10900*now_pocket_limit_rate as d
from 
dp_fk_tmp.dszx_88w_v2
where uid='41195' 
```
这个now_pocket_limit_rate在表里的值为0.7，返回结果为：
```
7629.999999999999
```
此时脑海居然浮现出之前decimal group by不准确的问题，好吧先让用cast as decimal(38,1)) 替换

```
select 
cast(10900*now_pocket_limit_rate  as decimal(38,1)) as d
from 
dp_fk_tmp.dszx_88w_v2
where uid='41195' 
```
结果肯定没问题

回想起来以为是spark sql的问题，搜索无果，然后用scala跑了下

```
scala> 10900*0.7
res0: Double = 7629.999999999999

scala> 10900*0.5
res1: Double = 5450.0
```

这个难道是scala的问题，直到我又用java跑了下

```
 System.out.println(10900 * 0.7);
```
输出也是7629.999999999999

卧槽，脑回路才反应过来，这不是浮点精度问题么,经典的还有1-0.9

```
scala> 1.0 - 0.9
res6: Double = 0.09999999999999998
```

shit  疑问了半天，是个精度问题。。。

这个mysql也是这样的吧

```
mysql> create table tt(num double);
Query OK, 0 rows affected (0.07 sec)

mysql> insert into tt(num)values(0.7);
Query OK, 1 row affected (0.00 sec)

mysql> select *  from tt;
+------+
| num  |
+------+
|  0.7 |
+------+
1 row in set (0.00 sec)

mysql> select num*10900 from tt;
+-------------------+
| num*10900         |
+-------------------+
| 7629.999999999999 |
+-------------------+
1 row in set (0.00 sec)

mysql> 
```
