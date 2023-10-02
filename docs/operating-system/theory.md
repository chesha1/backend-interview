# 操作系统理论
## 线程独占的资源有哪些?
1. **线程栈**：每个线程都有其自己的调用栈。这个栈保存了函数调用的历史、局部变量、返回地址等信息。

2. **线程局部存储（Thread Local Storage, TLS）**：这是一种机制，允许每个线程拥有其自己的数据实例，即使该数据是静态存储期或全局的。在C++11及更高版本中，可以使用 `thread_local` 关键字来声明线程局部变量。

3. **寄存器内容**：一些寄存器，如程序计数器、栈指针等，都是线程特有的。

4. **线程的优先级和调度信息**：这些信息由操作系统维护，用于决定线程的执行顺序。

5. **线程的特定属性**：例如线程ID、线程的状态（运行、就绪、阻塞等）。

6. **线程的信号掩码**：在某些操作系统（如Unix-like系统）中，线程可以有其自己的信号掩码，决定哪些信号可以递送给它。

7. **线程的错误状态**：某些库函数，如 `errno` 在POSIX系统中，可能为每个线程提供独立的错误状态。

## CPU 是如何将虚拟地址翻译成物理地址的？
内容较长，建议直接查看文章：https://zhuanlan.zhihu.com/p/636718404

## Linux 系统如何使用共享内存？有哪些方式？
在 Linux 中，共享内存是进程间通信 (IPC) 的一种机制，允许两个或更多进程访问同一块内存空间。以下是使用共享内存的几种方式：

1. **System V 共享内存**:
    - System V IPC 机制提供了一个传统的共享内存解决方案。使用 `shmget()`, `shmat()`, `shmdt()`, 和 `shmctl()` 这些函数可以创建、附加、分离和控制共享内存段。
    - 示例：
      ```c
      int shmid = shmget(key, size, IPC_CREAT | 0666);
      char* shared_mem = (char*)shmat(shmid, NULL, 0);
      ```

2. **POSIX 共享内存**:
    - POSIX 提供了另一种方式来创建共享内存，通常使用在 `/dev/shm` 虚拟文件系统上的文件。
    - 使用 `shm_open()`, `ftruncate()`, 和 `mmap()` 可以创建和映射共享内存。
    - 示例：
      ```c
      int fd = shm_open("/my_shm", O_CREAT | O_RDWR, 0666);
      ftruncate(fd, size);
      void* ptr = mmap(0, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
      ```


3. **共享内存对象**:
    - 这主要与 C++ 相关，Boost 和其他第三方库提供了更高级的、封装好的共享内存解决方案。


可以参考：https://cloud.tencent.com/developer/article/1551288