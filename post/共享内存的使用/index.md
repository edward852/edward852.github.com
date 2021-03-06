
本文主要介绍共享内存的使用。  
<!--more-->

# 原理
简单来说就是不同进程的虚拟内存区域映射到相同的物理内存区域。  
映射到进程内存空间的位置在栈和堆之间。  

# 优缺点

## 优点

- 高效  
  可以减少数据拷贝，是进程间通信极快(最快?)的方式。  
- 可常驻  
  不主动detach的情况下，即使进程结束，共享内存仍在(某T厂的用法)。  
  利用这个特点，把数据存到共享内存，这样即使进程重启或升级也不会对大量用户造成影响。  

## 缺点
不方便扩展，使用不当容易弄乱数据。通常需要配合信号量或者其他同步机制使用。  

# 注意事项
- 内存数据结构添加保留字段  
  这样可以提供一定的扩展性，方便后续添加新的字段。  
- 通过编译器确保内存数据结构总大小不变，旧字段位置不变  
  ```c
  #define CAT_TOKEN_1(t1,t2) t1##t2
  #define CAT_TOKEN(t1,t2) CAT_TOKEN_1(t1,t2)

  #define COMPILE_ASSERT(x)  \
					enum {CAT_TOKEN (comp_assert_at_line_, __LINE__) = 1 / !!(x) };

  #ifndef CHECK_OFFSET
  #define CHECK_OFFSET(type, member, value) \
      extern int offset_of_##member##_in_##type##_is_##value \
      [!!(__builtin_offsetof(type,member)==((size_t)(value))) - 1]
  #endif //CHECK_OFFSET
  
  CHECK_OFFSET( UsrRecord, shSrvId, 126 );
  COMPILE_ASSERT( sizeof(UsrRecord) == 512);
  ```

# 实现方式
## shm
通过 `shmget`, `shmat`, `shmdt`, `shmctl` 管理。  
可以通过ipcrm命令手动清共享内存。  

> A shared memory object is only removed after all currently attached processes have detached (shmdt(2)) the object from their virtual address space. -- man ipcrm

## mmap
通过 `mmap`, `munmap`, `msync` 管理。  
mmap会映射到普通文件，所以不仅进程重启还在，机器重启也还在。  

