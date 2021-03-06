
本文总结后端开发面试常见的一些题型。  
<!--more-->

# 常用语言
考察常用语言(如cpp, go, java等)的掌握程度。  
比如说面向对象、闭包、多线程(协程)、多态实现、传值与传引用、字节对齐等等。  
另外对比同一概念不同语言的实现可以更深入地理解编程。  

## cpp
- 内存占用(sizeof)与存储(大小端)  
- extern "C"的作用，实现原理？  
- inline的作用，有什么缺点？虚函数是否可以加inline？  
- 虚函数怎么实现，虚表存放在哪里？  

## go
参考文章：[Go使用陷阱与易犯错误](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang) 。  

- 常见数据类型的底层结构  
- goroutine的调度  

# 数据结构
可以参考 [常用数据结构](/post/常用数据结构) 的说明。  
掌握各种数据结构的优缺点与时空复杂度，能够根据需求给出较优方案(比如说字符串、定时器、LRU实现)。  

# 算法
可以参考 [常见算法](/post/常见算法) 的说明。  
个人觉得下面3种比较常考：  
- 分治算法  
- 排序算法  
  归并排序、快速排序。  
- 动态规划  
  最长公共子字符串、最长公共子序列、最短路径。  

# 操作系统
## 进程、线程、协程
- 进程是资源分配基本单位  
- 线程是系统调度基本单位，共享进程资源，少量独占资源(线程信息、栈、寄存器等)  
- 协程是用户态线程，调度由程序控制  

## 内存
可以参考 [内存管理](/post/内存管理) 的说明。  
### 内存布局
进程的内存布局如何，malloc是否可以申请超过物理内存大小。  

### 内存问题与定位
- 段错误  
- 内存溢出  
- 内存泄露  

## CPU
可以参考 [CPU使用率高问题定位](/post/cpu使用率高定位) 的说明。  

## 进程间通信
可以参考 [进程间通信](/post/进程间通信) 的说明。  

## 并发与锁
无锁、原子操作、生产者消费者模式。  
死锁定位可以参考 [使用gdb定位死锁、coredump](/post/使用gdb定位死锁coredump问题) 的说明。  
竞态检测。  

# 网络
## 字节序
了解网络序、大小端，具体参考[字节序与对齐](/post/字节序与对齐/#字节序)。  

## 协议栈
经典问题：  
- 浏览器访问URL背后发生了什么?  
  涉及到HTTP(HTTPS)、TCP、IP、DNS、ARP、proxy(正向代理和反向代理)、web server等等。  
- 前端页面点击没反应，后端没收到请求如何定位问题？  
  主要涉及浏览器开发者工具、网络连通性检查、抓包软件等。  

### HTTP、HTTPS
HTTP请求的构成，GET与POST的区别。  
![http request](/images/http_req.png)  

常见错误码有哪些。  
HTTP首部有哪些，connection close和keep-alive的区别。  
cookie与session的区别。  

HTTPS一般在SSL或者TLS上面传输HTTP，包括证书验证、密钥交换、对称加密通讯等步骤。  
HTTPS是如何安全地交换密钥？  

### TCP、UDP
经典问题：  
- TCP为什么是三次握手，而不是两次或四次？  
  关键点是TCP通过SEQ和ACK保证可靠传输，三次足已交换各自起始序列号并确认对方已收到。  
  两次不能确保收到，四次不够高效。详见 [知乎回答](https://www.zhihu.com/question/24853633/answer/115173386) 。  
- TCP为什么是四次挥手？  
  TCP是全双工传输，四次挥手允许双方各自选择关闭连接的时机。  
  实际上三次挥手也是可以的，详见 [维基百科](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_termination) 的说明(关键词possible to terminate)。  
- TIME-WAIT状态的作用，为什么要等2MSL？  
  主动关闭方收到对端FIN后发出的ACK可能丢包，导致对端重传FIN。  
  为了能够处理该情况，需要等待2MSL以便接收FIN并再次ACK，最终双方正确关闭连接。  
- TCP和UDP的区别

### IP
IPv4与IPv6 socket如何兼容：  
一种是创建2种socket同时监听，分开处理。  
一种是只创建IPv6 socket并关闭`IPV6_V6ONLY`选项，IPv4连接的地址以[IPv4-mapped](http://en.wikipedia.org/wiki/IPv6#IPv4-mapped_IPv6_addresses)形式表示。  
另外`getaddrinfo`,`inet_pton`,`inet_ntop`同时支持IPv4和IPv6。  

## socket编程
- TCP  
  server: socket bind listen accept send recv close  
  client: socket connect send recv close  
- UDP  
  server: socket bind sendto recvfrom close  
  client: socket sendto recvfrom close  

另外还有一些选项设置超时、缓冲区大小、非阻塞。  

## IO多路复用
- select, poll, epoll的区别  
  select轮询fd集合、最大连接数小且不可配置、可移植性较好。  
  poll轮询fd集合、链表解决最大连接限制。  
  epoll直接处理就绪fd集合、最大连接数大且可配置、仅限linux平台。  
- epoll的ET和LT触发的区别  
  ET边缘触发模式就绪事件只通知一次，高效但边角情况(漏读漏写)处理不当容易死锁。  
  LT水平触发模式就绪事件没处理完会持续通知，效率低一点但不容易死锁。  
- Reactor与Proactor的区别  
  Reactor收到的是可读可写事件，需要程序完成读写操作后再处理业务逻辑。  
  Proactor收到的是读完写完事件，读写操作已经由系统(或框架)完成，程序处理业务逻辑。  
  大部分IO多路复用框架都是Reactor，包括libuv、Event Library(redis)、libev等。  
  Windows平台Boost Asio是通过IOCP实现的Proactor。  

## 安全
可以参考 [常见网络攻击](/post/常见网络攻击) 和 [信息安全](/post/信息安全基础)。  

## 调优
BBR、QUIC。  

# 工具使用
## 常用命令
top, lsof, netstat, tcpdump, du, df, free, kill, gdb等等。  

## git使用
具体参考 [Git使用笔记](/post/git使用笔记)。  

# 数据库
可以参考 [数据库之redis](/post/数据库之redis) 和 [MySQL基础](/post/mysql基础) 的说明。  

# web server
nginx与apache。  

# docker && k8s
可以参考 [docker使用笔记](/post/docker使用笔记) 说明。  

# 智力题
- 排队报数  
  100个人排队，按照1,2,3,4依次报数，奇数退出偶数留下。  
  重复上述过程直到剩下一人，那么最后留下的是谁？
- 找毒酒  
  1000瓶酒中有1瓶毒酒，如何通过10只小白鼠找到毒酒？  
- 海盗分金  
  都是通过逆推得到结果，不过**半数以上**和**超过半数**才能通过方案的分配情况是不一样的。  
  以5海盗分100金币为例，半数以上通过的分配方案为(98,0,1,0,1)。  
  超过半数通过的分配方案为(97,0,1,2,0)或(97,0,1,0,2)。  

# 系统设计
排行榜、好友表、点赞系统、秒杀系统。  

