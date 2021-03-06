
本文主要记录内存管理相关的一些笔记。  
<!--more-->

# 布局
## 系统
### 32位系统  
![](https://i.loli.net/2020/04/17/jKXnRoviOJFz4es.png)  
```
0xFFFFFFFF
    内核空间 1G
0xC0000000
...
    栈
    共享内存(动态库等)
    堆
    bss段
    data段
    text段
...
0x00000000
```
32位系统地址空间是32位，最大是4GB。  
除去内核空间1GB，用户空间最大3GB。  
考虑内存碎片、系统限制等因素，用户实际可申请内存大小在min(虚拟内存大小, 3GB)以内。  
一般来说虚拟内存大小大于3GB，所以说最大可申请内存在3GB附近(有人实测2.8GB左右)。  

### 64位系统  
64位系统地址空间是64位，但是系统可能只支持几百GB(Windows)或者几百TB(Linux)。  
一般来说64位系统最大可申请内存主要取决于虚拟内存大小(物理内存+swap大小)。  

## 进程
```
<--- 高地址
栈 (向下增长)
共享内存(动态库等)
堆 (向上增长)
bss段
data段
rodata段
text段
<--- 低地址
```
text段存放代码，bss段存放未初始化的全局变量，data段存放已初始化的全局变量。  

# 分配器
- jemalloc  
  https://github.com/jemalloc/jemalloc
- tcmalloc  
  https://github.com/google/tcmalloc

对于稳定性要求比较高的场合(如嵌入式、服务后台)可以采用内存池静态分配。  

# 回收策略
- 手动  
  通过malloc,free和new,delete等手动申请、释放内存。  
  优点是速度快，缺点是容易遗漏。  
- GC自动回收  
  由GC(Garbage Collector)自动扫描回收不再使用的内存。  
  优点是不需要程序员介入，缺点是有性能开销，不适合实时的场合(有STW停顿扫描时间)。
- 其他  
  引用计数回收(需要避免循环引用)、Rust检查所有权和生命周期由编译器管理内存。  

# 日常使用
## 共享内存
具体参考 [共享内存使用](/post/共享内存的使用) 的说明。  

## 缓存淘汰
一般是被动和主动扫描结合淘汰过期缓存。具体可以参考[redis](https://redis.io/commands/expire#how-redis-expires-keys)的做法。  
主动扫描可以定时扫描部分，结合时间轮/最小堆淘汰过期缓存。  
redis的实现是从存储带超时key的哈希表里面随机淘汰。  

不建议使用(加锁)全局扫描这种方式，无用功太多。业务高峰还会增加CPU使用率，很容易造成恶性循环。  

# 常见问题

## 泄露与OOM
当申请内存大小多于剩余可用内存时就会发生OOM(out of memory)。  

有可能是特殊请求导致申请过大的内存，一般需要抓包复现定位解决。  
另外的可能就是内存泄露，特别是运行时间比较长才出现的情况。  
具体可以参考 [内存泄露定位](/post/内存泄露定位/) 的说明。  

## 溢出

### 栈溢出
一般就是栈上的局部缓冲区(数组)发生越界访问。  

### 堆溢出
一般就是堆上的缓冲区(数组)的越界访问。  

### 全局数据溢出
一般就是data/bss段的全局缓冲区(数组)的越界访问。  

## 访问非法地址
一般是解引用了空指针或者野指针，触发SIGSEGV信号。  
通过coredump文件或者注册SIGSEGV信号处理函数进行回溯定位。  

## 访问未初始化地址
一般是栈上分配变量没有初始化就使用。  
valgrind, clang支持检查。  

# 定位工具
- AddressSanitizer  
  GCC4.8以上版本可以通过AddressSanitizer定位，具体参考 [AddressSanitizer使用](/post/addresssanitizer定位内存问题/) 的说明。  
- valgrind  
  ```sh
  valgrind --tool=memcheck --leak-check=full --log-file=valgrind.log 待定位程序 参数列表
  ```
- gdb  
  GCC版本较低但不方便升级的话可以通过GDB+watch命令定位。  
  ```sh
  gdb
  
  # 加载程序
  file ./程序
  # 设置参数
  set args 参数
  
  # 全局变量
  watch *((char *)&g_var)
  
  # 局部变量 (先断点运行到所在函数)
  break 函数
  run
  watch var
  ```
  定位coredump，参考 [gdb定位coredump](/post/使用gdb定位死锁coredump问题#coredump) 的说明。  

