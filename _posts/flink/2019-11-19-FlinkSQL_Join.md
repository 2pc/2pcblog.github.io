---
title: Flink join
tagline: ""
category : flink
layout: post
tags : [flink, streamsets, realtime]
---

## Unbounded JOIN
### Regular join/Equi-join
```
SELECT *
FROM Orders INNER JOIN Product ON Orders.productId = Product.id
SELECT *
FROM Orders LEFT JOIN Product ON Orders.productId = Product.id

SELECT *
FROM Orders RIGHT JOIN Product ON Orders.productId = Product.id

SELECT *
FROM Orders FULL OUTER JOIN Product ON Orders.productId = Product.id
```
## bounded JOIN
### Time-windowed Join  

A time-windowed join requires at least one equi-join predicate and a join condition that bounds the time on both sides

. ltime = rtime   
. ltime >= rtime AND ltime < rtime + INTERVAL '10' MINUTE   
. ltime BETWEEN rtime - INTERVAL '10' SECOND AND rtime + INTERVAL '5' SECOND   

```
SELECT *
FROM Orders o, Shipments s
WHERE o.id = s.orderId AND
      o.ordertime BETWEEN s.shiptime - INTERVAL '4' HOUR AND s.shiptime
```   
[Flink Sql Join](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sql.html)

## Window Join&Interval Join

### Window Join

```
val orangeStream: DataStream[Integer] = ...
val greenStream: DataStream[Integer] = ...

orangeStream.join(greenStream)
    .where(elem => /* select key */)
    .equalTo(elem => /* select key */)
    .window(TumblingEventTimeWindows.of(Time.milliseconds(2)))
    .apply { (e1, e2) => e1 + "," + e2 }
```

### Interval Join   
. b.timestamp ∈ [a.timestamp + lowerBound; a.timestamp + upperBound]   
. or a.timestamp + lowerBound <= b.timestamp <= a.timestamp + upperBound

```
val orangeStream: DataStream[Integer] = ...
val greenStream: DataStream[Integer] = ...

orangeStream
    .keyBy(elem => /* select key */)
    .intervalJoin(greenStream.keyBy(elem => /* select key */))
    .between(Time.milliseconds(-2), Time.milliseconds(1))
    .process(new ProcessJoinFunction[Integer, Integer, String] {
        override def processElement(left: Integer, right: Integer, ctx: ProcessJoinFunction[Integer, Integer, String]#Context, out: Collector[String]): Unit = {
         out.collect(left + "," + right); 
        }
      });
    });
```
[Window Join&Interval Join](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/joining.html)
