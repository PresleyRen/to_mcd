---
layout:     post
title:      关于socket的setsockopt的使用
subtitle:   
date:       2019-04-08
author:     P
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - python
---
关于setsockopt的使用

学习python的时候学习到了socket，其中有个setsockopt方法的使用，于是乎整理一下关于这个方法的一些内容。

**本节目录**

- [一 功能描述](#part_1)
- [二 用法(getsockopt\setsockopt)](#part_2)
- [三 参数及参数详细说明](#part_3)
- [四 缓冲区](#part_4)
- [五 setsockopt的用法](#part_5)

### 一 功能描述

获取或者设置与某个套接字关联的选 项。选项可能存在于多层协议中，它们总会出现在最上面的套接字层。当操作套接字选项时，选项位于的层和选项的名称必须给出。为了操作套接字层的选项，应该 将层的值指定为SOL_SOCKET。为了操作其它层的选项，控制选项的合适协议号必须给出。例如，为了表示一个选项由TCP协议解析，层应该设定为协议 号TCP。

### 二 用法(getsockopt\setsockopt)

int getsockopt(int sock, int level, int optname, void *optval, socklen_t *optlen);

int setsockopt(int sock, int level, int optname, const void *optval, socklen_t optlen);

python中的使用：

{% raw %}
```
import socket
from socket import SOL_SOCKET,SO_REUSEADDR,SO_SNDBUF,SO_RCVBUF

sk = socket.socket()
sk.setsockopt(SOL_SOCKET,SO_SNDBUF,32*1024)
print('>>>>',sk.getsockopt(SOL_SOCKET,SO_SNDBUF))
print('>>>>',sk.getsockopt(SOL_SOCKET,SO_RCVBUF))从结果来看，貌似我们是更改了缓冲区的大小，但是并没有更改，这个缓冲区大小是由系统来控制的，也就是说socket在系统的配置中，如果想让其生效，需要修改系统中的配置文件，windows下socket的配置文件大家可以去百度一下。
```
{% endraw %}

### 三 参数及参数详细说明

参数：

参数详细说明：
SOL_SOCKET:通用套接字选项.IPPROTO_IP:IP选项.IPPROTO_TCP:TCP选项.
optname指定控制的方式(选项的名称),我们下面详细解释

optval获得或者是设置套接字选项.根据选项名称的数据类型进行转换

 

### 四 缓冲区

SO_RCVBUF和SO_SNDBUF每个套接口都有一个发送缓冲区和一个接收缓冲区，使用这两个套接口选项可以改变缺省缓冲区大小。

注意：当设置TCP套接口接收缓冲区的大小时，函数调用顺序是很重要的，因为TCP的窗口规模选项是在建立连接时用SYN与对方互换得到的。对于客 户，O_RCVBUF选项必须在connect之前设置；对于服务器，SO_RCVBUF选项必须在listen前设置。

{% raw %}
```
1 1.每个套接口都有一个发送缓冲区和一个接收缓冲区。 接收缓冲区被TCP和UDP用来将接收到的数据一直保存到由应用进程来读。 TCP：TCP通告另一端的窗口大小。 TCP套接口接收缓冲区不可能溢出，因为对方不允许发出超过所通告窗口大小的数据。 这就是TCP的流量控制，如果对方无视窗口大小而发出了超过窗口大小的数据，则接 收方TCP将丢弃它。 UDP：当接收到的数据报装不进套接口接收缓冲区时，此数据报就被丢弃。UDP是没有流量控制的；快的发送者可以很容易地就淹没慢的接收者，导致接收方的 UDP丢弃数据报。
2         2.我们经常听说tcp协议的三次握手,但三次握手到底是什么，其细节是什么，为什么要这么做呢?
3         第一次:客户端发送连接请求给服务器，服务器接收;
4         第二次:服务器返回给客户端一个确认码,附带一个从服务器到客户端的连接请求,客户机接收,确认客户端到服务器的连接.
5         第三次:客户机返回服务器上次发送请求的确认码,服务器接收,确认服务器到客户端的连接.
6         我们可以看到:
7         1. tcp的每个连接都需要确认.
8         2. 客户端到服务器和服务器到客户端的连接是独立的.
9         我们再想想tcp协议的特点:连接的,可靠的,全双工的,实际上tcp的三次握手正是为了保证这些特性的实现.
```
{% endraw %}

### 五 setsockopt的用法

1.closesocket（一般不会立即关闭而经历TIME_WAIT的过程）后想继续重用该socket：BOOL bReuseaddr=TRUE;setsockopt(s,SOL_SOCKET ,SO_REUSEADDR,(const char*)&bReuseaddr,sizeof(BOOL));　　2. 如果要已经处于连接状态的soket在调用closesocket后强制关闭，不经历TIME_WAIT的过程：BOOL bDontLinger = FALSE;setsockopt(s,SOL_SOCKET,SO_DONTLINGER,(const char*)&bDontLinger,sizeof(BOOL));　　3.在send(),recv()过程中有时由于网络状况等原因，发收不能预期进行,而设置收发时限：　　int nNetTimeout=1000;//1秒　　//发送时限　　setsockopt(socket，SOL_S0CKET,SO_SNDTIMEO，(char *)&nNetTimeout,sizeof(int));　　//接收时限　　setsockopt(socket，SOL_S0CKET,SO_RCVTIMEO，(char *)&nNetTimeout,sizeof(int));　　4.在send()的时候，返回的是实际发送出去的字节(同步)或发送到socket缓冲区的字节(异步);系统默认的状态发送和接收一次为8688字节(约为8.5K)；在实际的过程中发送数据和接收数据量比较大，可以设置socket缓冲区，而避免了send(),recv()不断的循环收发：　　// 接收缓冲区　　int nRecvBuf=32*1024;//设置为32K　　setsockopt(s,SOL_SOCKET,SO_RCVBUF,(const char*)&nRecvBuf,sizeof(int));　　//发送缓冲区　　int nSendBuf=32*1024;//设置为32K　　setsockopt(s,SOL_SOCKET,SO_SNDBUF,(const char*)&nSendBuf,sizeof(int));　　5. 如果在发送数据的时，希望不经历由系统缓冲区到socket缓冲区的拷贝而影响程序的性能：　　int nZero=0;　　setsockopt(socket，SOL_S0CKET,SO_SNDBUF，(char *)&nZero,sizeof(nZero));　　6.同上在recv()完成上述功能(默认情况是将socket缓冲区的内容拷贝到系统缓冲区)：　　int nZero=0;　　setsockopt(socket，SOL_S0CKET,SO_RCVBUF，(char *)&nZero,sizeof(int));　　7.一般在发送UDP数据报的时候，希望该socket发送的数据具有广播特性：　　BOOL bBroadcast=TRUE;　　setsockopt(s,SOL_SOCKET,SO_BROADCAST,(const char*)&bBroadcast,sizeof(BOOL));　　8.在client连接服务器过程中，如果处于非阻塞模式下的socket在connect()的过程中可以设置connect()延时,直到accpet()被呼叫(本函数设置只有在非阻塞的过程中有显著的作用，在阻塞的函数调用中作用不大)　　BOOL bConditionalAccept=TRUE;　　setsockopt(s,SOL_SOCKET,SO_CONDITIONAL_ACCEPT,(const char*)&bConditionalAccept,sizeof(BOOL));　　9.如果在发送数据的过程中(send()没有完成，还有数据没发送)而调用了closesocket(),以前我们一般采取的措施是"从容关 闭"shutdown(s,SD_BOTH),但是数据是肯定丢失了，如何设置让程序满足具体应用的要求(即让没发完的数据发送出去后在关闭 socket)？　　struct linger {　　　　u_short l_onoff;　　　　u_short l_linger;　　};　　linger m_sLinger;　　m_sLinger.l_onoff=1;//(在closesocket()调用,但是还有数据没发送完毕的时候容许逗留)　　// 如果m_sLinger.l_onoff=0;则功能和2.)作用相同;　　m_sLinger.l_linger=5;//(容许逗留的时间为5秒)　　setsockopt(s,SOL_SOCKET,SO_LINGER,(const char*)&m_sLinger,sizeof(linger));

 

 [回到顶部](#top) 
