---
layout:     post
title:      python获取linux系统内存、cpu、网络使用情况
subtitle:   
date:       2019-07-08
author:     P
header-img: img/post-bg-mma-1.jpg
catalog: true
tags:
    - python
---
# python获取linux系统内存、cpu、网络使用情况

做个程序需要用到系统的cpu、内存、网络的使用情况，百度之后发现目前使用python获取这些信息大多是调用系统命令（top、free等）。其实多linux命令也是读取/proc下的文件实现的，索性不如自己写一个。

一、计算cpu的利用率

要读取cpu的使用情况，首先要了解/proc/stat文件的内容，下图是/proc/stat文件的一个例子：

<img src="http://m2.img.srcdd.com/farm5/d/2013/0704/21/5EBEF5943578422AD175D84AF9A6FB9C_B500_900_500_188.PNG" alt="" width="500" height="188" />

cpu、cpu0、cpu1每行数字的意思相同，从左到右分别表示user、nice、system、idle、iowait、irq、softirq。根据系统的不同，输出的列数也会不同，比如ubuntu 12.04会输出10列数据，centos 6.4会输出9列数据，后边几列的含义不太清楚，每个系统都是输出至少7列。没一列的具体含义如下：

user：用户态的cpu时间（不包含nice值为负的进程所占用的cpu时间）

nice：nice值为负的进程所占用的cpu时间

system：内核态所占用的cpu时间

idle：cpu空闲时间

iowait：等待IO的时间

irq：系统中的硬中断时间

softirq：系统中的软中断时间

 

以上各值的单位都是jiffies，jiffies是内核中的一个全局变量，用来记录自系统启动一来产生的节拍数，在这里我们把它当做单位时间就行。

 

intr行中包含的是自系统启动以来的终端信息，第一列表示中断的总此数，其后每个数对应某一类型的中断所发生的次数

ctxt行中包含了cpu切换上下文的次数

btime行中包含了系统运行的时间，以秒为单位

processes/total_forks行中包含了从系统启动开始所建立的任务个数

procs_running行中包含了目前正在运行的任务的个数

procs_blocked行中包含了目前被阻塞的任务的个数

 

由于计算cpu的利用率用不到太多的值，所以截图中并未包含/proc/stat文件的所有内容。

 

知道了/proc文件的内容之后就可以计算cpu的利用率了，具体方法是：先在t1时刻读取文件内容，获得此时cpu的运行情况，然后等待一段时间在t2时刻再次读取文件内容，获取cpu的运行情况，然后根据两个时刻的数据通过以下方式计算cpu的利用率：100 - (idle2 - idle1)*100/(total2 - total1),其中total = user + system + nice + idle + iowait + irq + softirq。python代码实现如下：

{% raw %}
```
import time
           
           
           
def readCpuInfo():
           
    f = open('/proc/stat')
           
    lines = f.readlines();
           
    f.close()
           
           
           
    for line in lines:
           
        line = line.lstrip()
           
        counters = line.split()
           
        if len(counters) < 5:
           
            continue
           
        if counters[0].startswith('cpu'):
           
            break
           
    total = 0
           
    for i in xrange(1, len(counters)):
           
        total = total + long(counters[i])
           
    idle = long(counters[4])
           
    return {'total':total, 'idle':idle}
           
           
           
def calcCpuUsage(counters1, counters2):
           
    idle = counters2['idle'] - counters1['idle']
           
    total = counters2['total'] - counters1['total']
           
    return 100 - (idle*100/total)
           
           
           
if __name__ == '__main__':
           
    counters1 = readCpuInfo()
           
    time.sleep(0.1)
           
    counters2 = readCpuInfo()
           
    print calcCpuUsage(counters1, counters2):

```
{% endraw %}

　　

二、计算内存的利用率

计算内存的利用率需要读取的是/proc/meminfo文件，该文件的结构比较清晰，不需要额外的介绍，需要知道的是内存的使用总量为used = total - free - buffers - cached，python代码实现如下：

<img src="http://m3.img.srcdd.com/farm4/d/2013/0704/21/4C4640EFB3CDEFEB726904D2265110FD_B500_900_500_63.PNG" alt="" width="500" height="63" />

{% raw %}
```
def readMemInfo():
         
    res = {'total':0, 'free':0, 'buffers':0, 'cached':0}
         
    f = open('/proc/meminfo')
         
    lines = f.readlines()
         
    f.close()
         
    i = 0
         
    for line in lines:
         
        if i == 4:
         
            break
         
        line = line.lstrip()
         
        memItem = line.lower().split()
         
        if memItem[0] == 'memtotal:':
         
            res['total'] = long(memItem[1])
         
            i = i +1
         
            continue
         
        elif memItem[0] == 'memfree:':
         
            res['free'] = long(memItem[1])
         
            i = i +1
         
            continue
         
        elif memItem[0] == 'buffers:':
         
            res['buffers'] = long(memItem[1])
         
            i = i +1
         
            continue
         
        elif memItem[0] == 'cached:':
         
            res['cached'] = long(memItem[1])
         
            i = i +1
         
            continue
         
    return res
         
         
         
def calcMemUsage(counters):
         
    used = counters['total'] - counters['free'] - counters['buffers'] - counters['cached']
         
    total = counters['total']
         
    return used*100/total
         
         
         
if __name__ == '__main__':
         
    counters = readMemInfo()
         
    print calcMemUsage(counters)

```
{% endraw %}

　　

 

三、获取网络的使用情况

获取网络使用情况需要读取的是/proc/net/dev文件，如下图所示，里边的内容也很清晰，不做过的的介绍，直接上python代码，取的是eth0的发送和收取的总字节数：

<img src="http://m1.img.srcdd.com/farm4/d/2013/0704/21/8593F0E33D984EEEB3B6B01BF4A7CD5D_B500_900_362_291.PNG" alt="" width="362" height="291" />

{% raw %}
```
def readNetInfo(dev):
        
    f = open('/proc/net/dev')
        
    lines = f.readlines()
        
    f.close()
        
    res = {'in':0, 'out':0}
        
    for line in lines:
        
        if line.lstrip().startswith(dev):
        
            # for centos
        
            line = line.replace(':', ' ')
        
            items = line.split()
        
            res['in'] = long(items[1])
        
            res['out'] = long(items[len(items)/2 + 1])
        
    return res
        
if __name__ == '__main__':
        
    print readNetInfo('eth0')

```
{% endraw %}

　　 

 

 

四、获取网卡的ip地址和主机的cpu个数

这两个函数是程序的其它地方需要用到，就顺便写下来：

{% raw %}
```
import time
     
import socket
     
import fcntl
     
import struct
     
     
     
def getIpAddress(dev):
     
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
     
    a = s.fileno()
     
    b = 0x8915
     
    c = struct.pack('256s', dev[:15])
     
    res = fcntl.ioctl(a, b, c)[20:24]
     
    return socket.inet_ntoa(res)
     
     
     
def getCpuCount():
     
    f = open('/proc/cpuinfo')
     
    lines = f.readlines()
     
    f.close()
     
    res = 0
     
    for line in lines:
     
        line = line.lower().lstrip()
     
        if line.startswith('processor'):
     
            res = res + 1
     
    return res
     
     
     
if __name__ == '__main__':
     
    print getCpuCount()
     
    print getIpAddress('eth0')

```
{% endraw %}

　　
