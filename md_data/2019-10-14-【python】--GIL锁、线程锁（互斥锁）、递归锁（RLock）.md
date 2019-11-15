---
layout:     post
title:      【python】--GIL锁、线程锁（互斥锁）、递归锁（RLock）
subtitle:   
date:       2019-10-14
author:     P
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - python
---
# GIL锁

计算机有4核，代表着同一时间，可以干4个任务。如果单核cpu的话，我启动10个线程，我看上去也是并发的，因为是执行了上下文的切换，让看上去是并发的。但是单核永远肯定时串行的，它肯定是串行的，cpu真正执行的时候，因为一会执行1，一会执行2.。。。。正常的线程就是这个样子的。但是，在python中，无论有多少核，永远都是假象。无论是4核，8核，还是16核.......不好意思，同一时间执行的线程只有一个(线程)，它就是这个样子的。这个是python的一个开发时候，设计的一个缺陷，所以说python中的线程是假线程。

**1、[全局解释器锁(GIL)](http://www.dabeaz.com/python/UnderstandingGIL.pdf)**

　无论你启多少个线程，你有多少个cpu, Python在执行的时候会淡定的在同一时刻只允许一个线程运行

**2、GIL存在的意义？**

　因为python的线程是调用操作系统的原生线程，这个原生线程就是C语言写的原生线程。因为python是用C写的，启动的时候就是调用的C语言的接口。因为启动的C语言的远程线程，那它要调这个线程去执行任务就必须知道上下文，所以python要去调C语言的接口的线程，必须要把这个上限问关系传给python，那就变成了一个我在加减的时候要让程序串行才能一次计算。就是先让线程1，再让线程2.......

　每个线程在执行的过程中，python解释器是控制不了的，因为是调的C语言的接口，超出了python的控制范围，python的控制范围是只在python解释器这一层，所以python控制不了C接口，它只能等结果。所以它不能控制让哪个线程先执行，因为是一块调用的，只要一执行，就是等结果，这个时候4个线程独自执行，所以结果就不一定正确了。有了GIL，就可以在同一时间只有一个线程能够工作。虽然这4个线程都启动了，但是同一时间我只能让一个线程拿到这个数据。其他的几个都干等。python启动的4个线程确确实实落到了这4个cpu上，但是为了避免出错。这也是Cpython的一个缺陷，其他语言没有，仅仅只是Cpython有。

**3、GIL锁关系图**

GIL(全局解释器锁)是加在python解释器里面的，效果如图：

<img src="https://images2017.cnblogs.com/blog/1088183/201709/1088183-20170926125009651-1449713172.png" alt="" />

**如上图，为什么GIL锁要加在python解释器这一层，而却不加在其他地方？**

因为你python调用的所有线程都是原生线程。原生线程是通过C语言提供原生接口，相当于C语言的一个函数。你一调它，你就控制不了了它了，就必须等它给你返回结果。只要已通过python虚拟机，再往下就不受python控制了，就是C语言自己控制了。你加在python虚拟机以下，你是加不上去的。同一时间，只有一个线程穿过这个锁去真正执行。其他的线程，只能在python虚拟机这边等待。

** 总结：**

需要明确的一点是`GIL`并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。就好比C++是一套语言（语法）标准，但是可以用不同的编译器来编译成可执行代码。有名的编译器例如GCC，INTEL C++，Visual C++等。Python也一样，同样一段代码可以通过CPython，PyPy，Psyco等不同的Python执行环境来执行。像其中的JPython就没有GIL。然而因为CPython是大部分环境下默认的Python执行环境。所以在很多人的概念里CPython就是Python，也就想当然的把`GIL`归结为Python语言的缺陷。所以这里要先明确一点：GIL并不是Python的特性，Python完全可以不依赖于GIL。

#  

# 线程锁（互斥锁）

** 1、多进程共享数据（python2.7中实验）**
<td class="gutter">1234567891011121314151617181920212223242526</td><td class="code">`import` `threading``import` `time`  `def` `run(n):``    ``global` `num   ``# 把num变成全局变量``    ``time.sleep(``1``)  ``# 注意了sleep的时候是不占有cpu的，这个时候cpu直接把这个线程挂起了，此时cpu去干别的事情去了``    ``num ``+``=` `1`   `# 所有的线程都做+1操作` `num ``=` `0`   `# 初始化num为0``t_obj ``=` `list``()``for` `i ``in` `range``(``100``):``    ``t ``=` `threading.Thread(target``=``run, args``=``(``"t-{0}"``.``format``(i),))``    ``t.start()``    ``t_obj.append(t)` `for` `t ``in` `t_obj:``    ``t.join()`  `print``(``"--------all thread has finished"``)``print``(``"num:"``, num)   ``# 输出最后的num值` `#执行结果``-``-``-``-``-``-``-``-``all` `thead has finished``(``'num:'``, ``97``)  ``#输出的结果`</td>

最后输出的结果怎么会是 97 呢？应该是100才对啊，不是有GIL(全局解释器锁)已经控制了，为什么最后的输出结果还是错误？

其实这种情况只能在python2.x 中才会出现的，python3.x里面没有这种现象，下面我们就用一张图来解释一下这个原因。如图：

**<img src="https://images2017.cnblogs.com/blog/1088183/201709/1088183-20170926140930839-80064182.png" alt="" />**

**解释：**

1. 到第5步的时候，可能这个时候python正好切换了一次GIL(据说python2.7中，每100条指令会切换一次GIL),执行的时间到了，被要求释放GIL,这个时候thead 1的count=0并没有得到执行，而是挂起状态，count=0这个上下文关系被存到寄存器中.
1. 然后到第6步，这个时候thead 2开始执行，然后就变成了count = 1,返回给count，这个时候count=1.
1. 然后再回到thead 1，这个时候由于上下文关系，thead 1拿到的寄存器中的count = 0，经过计算，得到count = 1，经过第13步的操作就覆盖了原来的count = 1的值，所以这个时候count依然是count = 1，所以这个数据并没有保护起来。

**2、添加线程锁**
<td class="gutter">1234567891011121314151617181920212223242526</td><td class="code">`import` `threading``import` `time`  `def` `run(n):``    ``lock.acquire()  ``# 添加线程锁``    ``global` `num   ``# 把num变成全局变量``    ``time.sleep(``0.1``)  ``# 注意了sleep的时候是不占有cpu的，这个时候cpu直接把这个线程挂起了，此时cpu去干别的事情去了``    ``num ``+``=` `1`   `# 所有的线程都做+1操作``    ``lock.release()  ``# 释放线程锁`  `num ``=` `0`   `# 初始化num为0``lock ``=` `threading.Lock()  ``# 生成线程锁实例``t_obj ``=` `list``()``for` `i ``in` `range``(``10``):``    ``t ``=` `threading.Thread(target``=``run, args``=``(``"t-{0}"``.``format``(i),))``    ``t.start()``    ``t_obj.append(t)` `for` `t ``in` `t_obj:``    ``t.join()   ``# 为join是等子线程执行的结果，如果不加，主线程执行完，下面就获取不到子线程num的值了，共享数据num值就错误了`  `print``(``"--------all thread has finished"``)``print``(``"num:"``, num)   ``# 输出最后的num值`</td>

** 小结：**

1. 用theading.Lock()创建一个lock的实例。
1. 在线程启动之前通过lock.acquire()加加锁，在线程结束之后通过lock.release()释放锁。
1. 这层锁是用户开的锁，就是我们用户程序的锁。跟我们这个GIL没有关系，但是它把这个数据相当于copy了两份，所以在这里加锁，以确保同一时间只有一个线程，真真正正的修改这个数据，所以这里的锁跟GIL没有关系，你理解就是自己的锁。
1. 加锁，说明此时我来去修改这个数据，其他人都不能动。然后修改完了，要把这把锁释放。这样的话就把程序编程串行了。

**3、使用场景**

　在用户层面加锁，使程序变成串行了，那我们在什么情况下用呢？

　　1、我们在程序中间不能有sleep，因为程序变成串行，这样你再sleep，程序执行的时间就会变长。

　　2、我们使用的时候确保数据量不是特别大，如果数据量大，也会影响我们的执行效率。

　　3、如果你程序结束时，不释放锁的话，而且程序又是串行的，则就是占着坑，那永远在那边等着，所以最后需要释放锁。

#  

# 递归锁（RLock）

**1、线程死锁**

在线程间共享多个资源的时候，如果两个线程分别占有一部分资源并且同时等待对方的资源，就会造成死锁，因为系统判断这部分资源都

正在使用，所有这两个线程在无外力作用下将一直等待下去
<td class="gutter">12345678910111213141516171819202122232425262728293031323334353637383940414243</td><td class="code">`import` `threading`  `def` `run1():``    ``print``(``"grab the first part data"``)``    ``lock.acquire()  ``# 修改num前加锁``    ``global` `num``    ``num ``+``=` `1``    ``lock.release()   ``# 释放锁``    ``return` `num`  `def` `run2():``    ``print``(``"grab the second part data"``)``    ``lock.acquire()   ``# 修改num2前加锁``    ``global` `num2``    ``num2 ``+``=` `1``    ``lock.release()   ``# 释放锁``    ``return` `num2`  `def` `run3():``    ``lock.acquire()  ``# 加锁``    ``res ``=` `run1()   ``# 执行run1函数``    ``print``(``'--------between run1 and run2-----'``)``    ``res2 ``=` `run2()  ``# 执行run2函数``    ``lock.release()  ``# 释放锁``    ``print``(res, res2)`  `if` `__name__ ``=``=` `'__main__'``:``    ``num, num2 ``=` `0``, ``0``    ``lock ``=` `threading.Lock()  ``# 设置锁的全局变量``    ``for` `i ``in` `range``(``10``):``        ``t ``=` `threading.Thread(target``=``run3)``        ``t.start()`  `while` `threading.active_count() !``=` `1``:  ``# 判断是否只剩主线程了``    ``print``(threading.active_count())``else``:``    ``print``(``'----all threads done---'``)``    ``print``(num, num2)`</td>

上面的执行结果，是无限的进入死循环，所以不能这么加，这个时候就需要用到递归锁。

**2、递归锁(RLock)**
<td class="gutter">12345678910111213141516171819202122232425262728293031323334353637383940414243</td><td class="code">`import` `threading`  `def` `run1():``    ``print``(``"grab the first part data"``)``    ``lock.acquire()  ``# 修改num前加锁``    ``global` `num``    ``num ``+``=` `1``    ``lock.release()   ``# 释放锁``    ``return` `num`  `def` `run2():``    ``print``(``"grab the second part data"``)``    ``lock.acquire()   ``# 修改num2前加锁``    ``global` `num2``    ``num2 ``+``=` `1``    ``lock.release()   ``# 释放锁``    ``return` `num2`  `def` `run3():``    ``lock.acquire()  ``# 加锁``    ``res ``=` `run1()   ``# 执行run1函数``    ``print``(``'--------between run1 and run2-----'``)``    ``res2 ``=` `run2()  ``# 执行run2函数``    ``lock.release()  ``# 释放锁``    ``print``(res, res2)`  `if` `__name__ ``=``=` `'__main__'``:``    ``num, num2 ``=` `0``, ``0``    ``lock ``=` `threading.RLock()  ``# 只用修改这里，把线程锁lock()更改成递归锁RLock()的全局变量``    ``for` `i ``in` `range``(``10``):``        ``t ``=` `threading.Thread(target``=``run3)``        ``t.start()`  `while` `threading.active_count() !``=` `1``:  ``# 判断是否只剩主线程了``    ``print``(threading.active_count())``else``:``    ``print``(``'----all threads done---'``)``    ``print``(num, num2)`</td>

递归锁原理其实很简单：就是每开一把门，在字典里面存一份数据，退出的时候去到door1或者door2里面找到这个钥匙退出，如图：

<img src="https://images2017.cnblogs.com/blog/1088183/201709/1088183-20170926144841604-1032923525.png" alt="" />
<td class="gutter">1234</td><td class="code">`lock ``=` `{``door1：key1，``door2：key2``}`</td>

转载：[https://www.cnblogs.com/Keep-Ambition/p/7596098.html](https://www.cnblogs.com/Keep-Ambition/p/7596098.html)
