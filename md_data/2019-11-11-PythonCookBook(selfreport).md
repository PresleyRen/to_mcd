---
layout:     post
title:      PythonCookBook(selfreport)
subtitle:   
date:       2019-11-11
author:     P
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - python
---
# Python CookBook

中文版：[https://python3-cookbook.readthedocs.io/zh_CN/latest/copyright.html](https://python3-cookbook.readthedocs.io/zh_CN/latest/copyright.html)

英文版：[https://d.cxcore.net/Python/Python_Cookbook_3rd_Edition.pdf](https://d.cxcore.net/Python/Python_Cookbook_3rd_Edition.pdf)

# Data Structures and Algorithms

### start expressions

{% raw %}
```
items = [1, 10, 7, 4, 5, 9] 
def sum(items):
... head, *tail = items
... return head + sum(tail) if tail else head
...
>>> head, *tail = items
>>> head
1
>>> tail
[10, 7, 4, 5, 9]
>>> sum(items)
36
```
{% endraw %}

### Deque

Using deque(maxlen=N) creates a fixed-sized queue. When new items are added and the queue is full, the oldest item is automatically removed. 

{% raw %}
```
from collections import deque
>>> q = deque(maxlen=3)
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3], maxlen=3)
>>> q.append(4)
>>> qdeque([1, 2, 3]) >>> q.appendleft(4) >>> q deque([4, 1, 2, 3]) >>> q.pop() 3 >>> q deque([4, 1, 2]) >>> q.popleft() 4 　
```
{% endraw %}

★ Adding or popping items from either end of a queue has O(1) complexity. This is unlike a list where inserting or removing items from the front of the list is O(N).

### Heapq

{% raw %}
```
The heapq module has two functionsnlargest() and nsmallest()that do exactly what you want. For example:
import heapq
nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums)) # Prints [42, 37, 23]
print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]
Both functions also accept a key parameter that allows them to be used with more
complicated data structures. For example:
portfolio = [
 {'name': 'IBM', 'shares': 100, 'price': 91.1},
 {'name': 'AAPL', 'shares': 50, 'price': 543.22},
 {'name': 'FB', 'shares': 200, 'price': 21.09},
 {'name': 'HPQ', 'shares': 35, 'price': 31.75},
 {'name': 'YHOO', 'shares': 45, 'price': 16.35},
 {'name': 'ACME', 'shares': 75, 'price': 115.65}
]
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])
```
{% endraw %}

{% raw %}
```
声明：如果涉及侵权请联系本人删除，谢谢~
```
{% endraw %}
