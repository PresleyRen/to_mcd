---
layout:     post
title:      【python】--队列（Queue）、生产者消费者模型
subtitle:   
date:       2019-10-14
author:     P
header-img: img/post-bg-mma-6.jpg
catalog: true
tags:
    - python
---
# 队列（Queue）

在多个线程之间安全的交换数据信息，队列在多线程编程中特别有用

队列的好处：

1. 提高双方的效率，你只需要把数据放到队列中，中间去干别的事情。
1. 完成了程序的解耦性，两者关系依赖性没有不大。

## 一、队列的类型：

### 1、lass queue.Queue(maxsize=0)

先进先出，后进后出
<td class="gutter">12345678</td><td class="code">`import` `queue``q ``=` `queue.Queue()   ``# 生成先入先出队列实例``q.put(``1``)  ``# 先放进1，再放入2``q.put(``2``)``print``(q.get())  ``# ` `# 输出``1`</td>

### 2、class queue.LifoQueue(maxsize=0)

是先进后出，后进新出规则，last in fisrt out
<td class="gutter">12345678</td><td class="code">`import` `queue``q ``=` `queue.LifoQueue()   ``# 生成后入先出队列实例``q.put(``1``)  ``# 先放进1，再放入2``q.put(``2``)``print``(q.get())  ``#` `# 输出``2`</td>

### 3、class queue.PriorityQueue(maxsize=0)

根据优先级来取数据。存放数据的格式  : Queue.put((priority_number,data))，priority_number越小，优先级越高，data代表存入的值
<td class="gutter">12345678910111213</td><td class="code">`import` `queue``q ``=` `queue.PriorityQueue()``q.put((``1``, ``"d1"``))``q.put((``-``1``, ``"d2"``))``q.put((``6``, ``"d3"``))``print``(q.get())``print``(q.get())``print``(q.get())` `#执行结果``(``-``1``, ``'d2'``)``(``1``, ``'d1'``)``(``6``, ``'d3'``)`</td>

注：maxsize代表这个队列最大能够put的长度

##  二、队列（Queue）的内置方法
<td class="gutter">123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156</td><td class="code">`1``、exception queue.Empty``当队列中的数据为空时，就会抛出这个异常。` `>>> ``import` `queue``>>> q ``=` `queue.Queue()``>>> q.get(block``=``False``)   ``#获取不到的时候``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``161``, ``in` `get``    ``raise` `Empty``queue.Empty`  `###############################################``2``、 exception queue.Full``当队列中满了以后，再放数据的话，就会抛出此异常。` `>>> ``import` `queue``>>> q ``=` `queue.Queue(maxsize``=``1``)  ``#创建队列实例，并且设置最大值为1``>>> q.put(``1``)``>>> q.put(``1``,block``=``False``)``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``130``, ``in` `put``    ``raise` `Full``queue.Full`  `###############################################``3``、Queue.qsize()``查看队列的大小` `>>> ``import` `queue``>>> q ``=` `queue.Queue()``>>> q.put(``1``)``>>> q.qsize()   ``#查看队列的大小``1`  `###############################################``4``、Queue.empty()``队列如果为空返回``True``，不为空返回``False` `>>> ``import` `queue``>>> q ``=` `queue.Queue()``>>> q.put(``1``)``>>> q.empty()  ``#队列不为空``False``>>> q.get()``1``>>> q.empty() ``#队列为空``True`  `###############################################``5``、Queue.full()``队列如果满了，返回``True``，没有满返回``False` `>>> ``import` `queue``>>> q ``=` `queue.Queue(maxsize``=``1``)  ``#设置队列的大小为1``>>> q.full()   ``#队列没有满``False``>>> q.put(``1``)``>>> q.full()   ``#队列已满``True`  `###############################################``6``、Queue.put(item,block``=``True``,timeout``=``None``)``把数据插入队列中。block参数默认为true，timeout默认值是``None``。如果blcok为false的话，那么在put时候超过设定的maxsize的值，就会报full 异常。如果timeout设置值得话，说明put值得个数超过maxsize值，那么会在timeout几秒之后抛出full异常。` `>>> ``import` `queue``>>> q ``=` `queue.Queue(maxsize``=``1``)  ``#是定队列的大小为1``>>> q.put(``1``)``>>> q.put(``1``,block``=``False``)   ``#block不会阻塞，会full异常``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``130``, ``in` `put``    ``raise` `Full``queue.Full``>>> q.put(``1``,timeout``=``1``)    ``#超过1秒，则会报full异常``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``141``, ``in` `put``    ``raise` `Full``queue.Full`  `###############################################``7``、Queue.put_nowait(item)``这个其实等同于Queue.put(item,block``=``False``)或者是Queue.put(item,``False``)` `>>> ``import` `queue``>>> q ``=` `queue.Queue(maxsize``=``1``)``>>> q.put(``1``)``>>> q.put_nowait(``1``)   ``#等同于q.put(1,block=False)``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``184``, ``in` `put_nowait``    ``return` `self``.put(item, block``=``False``)``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``130``, ``in` `put``    ``raise` `Full``queue.Full`  `###############################################``8``、Queue.get(block``=``True``,timeout``=``None``)``移除并返回队列中的序列。参数block``=``true并且timeout``=``None``。如果block``=``false的话，那么队列为空的情况下，就直接Empty异常。如果timeout有实际的值，这个时候队列为空，执行get的时候，则时隔多长时间则报出Empty的异常。` `>>> ``import` `queue``>>> q ``=` `queue.Queue()``>>> q.put(``1``)``>>> q.get()``1``>>> q.get(block``=``False``)    ``#获取不到值，直接抛Empty异常``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``161``, ``in` `get``    ``raise` `Empty``queue.Empty``>>> q.get(timeout``=``1``)    ``#设置超时时间，抛出Empty异常``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``172``, ``in` `get``    ``raise` `Empty``queue.Empty`  `###############################################``9``、Queue.get_nowait(item)``其实这个等同于Queue.get(block``=``False``)或者Queue.get(``False``)` `>>> ``import` `queue``>>> q ``=` `queue.Queue()``>>> q.put(``1``)``>>> q.get()``1``>>> q.get_nowait()   ``#等同于q.get(block=False)``Traceback (most recent call last):``  ``File` `"<input>"``, line ``1``, ``in` `<module>``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``192``, ``in` `get_nowait``    ``return` `self``.get(block``=``False``)``  ``File` `"D:\Python\Python35\lib\queue.py"``, line ``161``, ``in` `get``    ``raise` `Empty``queue.Empty`  `###############################################``10``、Queue.task_done()``get()用于获取任务，task_done()则是用来告诉队列之前获取的任务已经处理完成`  `###############################################``11``、Queue.join()``block(阻塞)直到queue（队列）被消费完毕``如果生产者生产``10``个包子，那么要等消费者把这个``10``个包子全部消费完毕，生产者才能继续往下执行。`</td>

　　

# 生产者消费者模型

并发编程中使用生产者和消费者模式能够解决绝大多数并发问题。该模式通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度。

### 1、**为什么要使用生产者和消费者模式**

在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式。

### 2、**什么是生产者消费者模式**

### 3、生成者消费者模型例子

**3.1、生产者生产完毕，消费者再消费例子：**
<td class="gutter">1234567891011121314151617181920212223242526272829303132</td><td class="code">`import` `threading``import` `queue`  `def` `producer():``    ``"""``    ``模拟生产者``    ``:return:``    ``"""``    ``for` `i ``in` `range``(``10``):``        ``q.put(``"骨头 %s"` `%` `i)` `    ``print``(``"开始等待所有的骨头被取走..."``)``    ``q.join()  ``# 等待这个骨头队列被消费完毕``    ``print``(``"所有的骨头被取完了..."``)`  `def` `consumer(n):``    ``"""``    ``模拟消费者``    ``:return:``    ``"""``    ``while` `q.qsize() > ``0``:``        ``print``(``"%s 取到"` `%` `n, q.get())``        ``q.task_done()  ``# 每去到一个骨头，便告知队列这个任务执行完了` `q ``=` `queue.Queue()` `p ``=` `threading.Thread(target``=``producer,)``p.start()` `c1 ``=` `consumer(``"QQ"``)`</td>

**3.2 边生产边消费的模型例子**
<td class="gutter">12345678910111213141516171819202122232425262728293031</td><td class="code">`import` `time,random``import` `queue,threading``q ``=` `queue.Queue()`  `def` `producer(name):``  ``count ``=` `0` `  ``while` `count < ``20``:``    ``time.sleep(random.randrange(``3``))``    ``q.put(count)  ``# 在队列里放包子``    ``print``(``'Producer %s has produced %s baozi..'` `%` `(name, count))``    ``count ``+``=` `1`  `def` `consumer(name):``  ``count ``=` `0``  ``while` `count < ``20``:``    ``time.sleep(random.randrange(``4``))``    ``if` `not` `q.empty():  ``# 如果还有包子``        ``data ``=` `q.get()  ``# 就继续获取保证``        ``print``(data)``        ``print``(``'\033[32;1mConsumer %s has eat %s baozi...\033[0m'` `%` `(name, data))``    ``else``:``        ``print``(``"-----no baozi anymore----"``)``    ``count ``+``=` `1` `p1 ``=` `threading.Thread(target``=``producer, args``=``(``'A'``,))``c1 ``=` `threading.Thread(target``=``consumer, args``=``(``'B'``,))``p1.start()``c1.start()`</td>

3.3、流程图

<img src="https://images2017.cnblogs.com/blog/1088183/201709/1088183-20170926165636964-877022578.png" alt="" />

图解：

1. 生产者生产，消费者消费。
1. 消费者每消费一次，都要去执行以下task_done()方法，来告诉消费者已经消费成功，相当于吃完饭，消费者应该给钱了。
1. 消费者每消费一次，则队列中计数器会做减1操作。
1. 当队列中的计数器为0的时候，则生产者不阻塞，继续执行，不为0的时候，则阻塞，直到消费者消费完毕为止。
