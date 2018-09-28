---
layout: post
title: "Python Iterator & Iterable"
keywords: ["python"]
description: "python"
category: "python"
tags: ["python","python"]
---

在计算的len的时候碰到

```
TypeError: object of type 'listiterator' has no len()
```

说明这个对象是个iterator，是没有len的，翻了下google

普遍的做法是使用len(list(iterator))

```
>>> l1 = [1, 2, 3]
>>> it = iter(l1)
>>> it
<listiterator object at 0x1013089d0>
>>> type(it)
<type 'listiterator'>
>>> l2 = list(it)
>>> l2
[1, 2, 3]
>>> list(it)
[]
>>> l2
[1, 2, 3]
>>> l1
[1, 2, 3]
>>> 
>>> l1 = [1, 2, 3]
>>> it = iter(l1)
>>> len(list(it))
3
>>> list(it)
[]
```
