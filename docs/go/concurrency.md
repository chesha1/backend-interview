# 并发
## 有哪些东西是协程安全的？
在 Go 语言中，某些基本的组件和工具被设计为协程安全（goroutine-safe），这意味着在多个 goroutine 并发访问时，它们能够正确并安全地工作，而不引发数据竞争或其他并发问题。以下是一些 Go 中的并发安全组件：

1. **`sync/atomic` 包**：提供了原子操作函数，如 `AddInt32`、`LoadPointer` 等，这些函数可以在并发环境中安全地操作变量。

5. **`sync.Map`**：一个并发安全的 map 实现，特别适合于读多写少的场景。

6. **`sync.Pool`**：一个并发安全的对象池，可以被多个 goroutine 同时使用，用于存储和复用临时对象。

7. **`channel`**：Go 的内置通道数据结构是并发安全的。它用于在 goroutines 之间发送和接收消息。

8. **`net/http` 的部分组件**：例如，`http.DefaultClient` 和 `http.DefaultTransport` 都是并发安全的，可以从多个 goroutines 安全地访问。

