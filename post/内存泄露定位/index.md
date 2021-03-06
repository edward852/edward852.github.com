
本文主要介绍如何定位内存泄露。  
<!--more-->

# 用户态
## valgrind
```sh
valgrind --tool=memcheck --leak-check=full --log-file=memleak.log 你的程序
```
valgrind生成报告需要程序正常退出，所以不可以使用 `kill -9` 等方式强行退出。  
可以修改程序接管信号SIGUSR1或SIGUSR2支持正常退出，然后通过该信号让程序退出。  

## AddressSanitizer
GCC4.9以上可以参考 [AddressSanitizer使用](/post/addresssanitizer定位内存问题/) 的说明。  

## malloc_stats对比
可以统计进程内存使用情况，精确到字节。  
可以采用二分法，逐步缩小内存泄露代码的范围。  

# 内核态
一般是第三方驱动引起的泄露，之前定位过芯片供应商SDK里面的驱动内存泄露。  
开启内核内存泄露检查选项，包括slab调试信息。  
观察 `/proc/slabinfo` 是否有异常，`kmemleak` 是否有信息。  
假设 `kmalloc-128` 有异常，查看 `/sys/kernel/slab/kmalloc-128/alloc-calls` 定位具体调用。  

