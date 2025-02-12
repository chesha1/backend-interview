# 智能指针
## shared_ptr 存了什么？
`std::shared_ptr` 内部通常包含以下部分：

1. **指向对象的指针**：这是 `shared_ptr` 管理的实际对象的指针，也就是 `get()` 所返回的指针。

2. **指向控制块的指针**：控制块是一个动态分配的对象，其中包含：  

    - 指向被管理对象的指针或被管理对象本身
    - 删除器（类型擦除）
    - 分配器（类型擦除）
    - 占有被管理对象的 shared_ptr 的数量（强引用计数）
    - 涉及被管理对象的 weak_ptr 的数量（弱引用计数）

## 申请 shared_ptr 需要额外占多少空间？
https://stackoverflow.com/questions/22295665/how-much-is-the-overhead-of-smart-pointers-compared-to-normal-pointers-in-c

## shared_ptr 是线程安全的吗？
`std::shared_ptr` 的某些操作是线程安全的，但并不是全部。让我们重新概括其线程安全性：

1. **引用计数的更新**：
   `shared_ptr` 的引用计数更新是原子的，因此是线程安全的。这意味着多个线程可以同时创建、复制或销毁指向同一个对象的 `shared_ptr`，而不需要额外的同步机制。它内部使用原子操作来递增或递减引用计数。

2. **读取 `shared_ptr` 实例**：
   在多个线程中同时读取同一个 `shared_ptr` 实例（例如，获取它所指向的对象的地址）是线程安全的。

3. **赋值和重新指向**：
   同时赋值（例如，使用 `operator=`）或重新指向（例如，使用 `reset` 方法）一个已经存在的 `shared_ptr` 实例是不线程安全的。如果存在这种可能性，你需要使用适当的同步机制。

4. **访问管理的对象**：
   `shared_ptr` 保证了引用计数的线程安全性，但并不保证所管理的对象的线程安全性。多个线程访问和修改同一个对象仍然需要适当的同步。

简而言之，尽管 `shared_ptr` 的引用计数机制是线程安全的，但当涉及到赋值、重新指向或直接访问其管理的对象时，还是需要考虑线程同步的问题。

## `weak_ptr` 是不是线程安全的？
- `std::weak_ptr` 和 `std::shared_ptr` 都有控制块，对控制块（引用计数）是线程安全的。
- `std::weak_ptr::lock` 是一个成员函数，用于尝试“升级” `weak_ptr` 到一个 `shared_ptr`，这个接口也是线程安全的。

## 智能指针相互引用的冲突问题怎么解决？
当两个智能指针对象相互引用时，可能会导致循环引用的问题。例如，如果两个 `std::shared_ptr` 对象互相引用，那么这两个对象的引用计数永远不会下降到0，因此涉及的对象永远不会被销毁，导致内存泄漏。

为了解决这个问题，可以使用 `std::weak_ptr` 来打破循环。`std::weak_ptr` 本质上是一个观察者，它可以观察 `std::shared_ptr` 所管理的对象，但不会增加对象的引用计数。
