# `channel`
## `channel` 的底层实现？
1. **数据结构**：
   Go 语言中的 channel 是用结构体 hchan 表示的，底层结构主要包括以下几个部分：

    - 一个用于存储数据的环形缓冲区（对于带缓冲的 channels）。
    - 一个互斥锁，用于保护对 channel 的并发访问。
    - 两个等待队列：一个用于等待数据的发送 goroutines，另一个用于等待数据的接收 goroutines。
    - 一些状态信息，如缓冲区大小、当前存储的元素数量、下一次发送（接收）数据的下标位置等。

2. **发送数据**：
   
    当一个 goroutine 尝试发送数据到一个 channel 时，以下几种情况可能发生：

    - 如果有一个或多个正在等待的接收 goroutine，发送操作将直接把数据传递给一个接收者并立即返回。
    - 如果 channel 有空闲的缓冲区，数据将被存放到缓冲区并且发送操作立即返回。
    - 否则，发送 goroutine 会被置于等待队列中，并被阻塞，直到数据可以被接收或存放到缓冲区为止。

3. **接收数据**：

    当一个 goroutine 尝试从一个 channel 接收数据时：

    - 如果有一个或多个正在等待的发送 goroutine，接收操作将直接从发送者那里获取数据并立即返回。
    - 如果 channel 的缓冲区中有数据，数据会被移除并返回给接收者。
    - 否则，接收 goroutine 会被置于等待队列中，并被阻塞，直到有数据可接收为止。

4. **关闭 channel**：

    当一个 channel 被关闭时，所有等待的接收 goroutines 会被唤醒并接收到一个零值和一个 false 的第二返回值，表示 channel 已关闭并且没有更多数据。

5. **同步与并发**：

    channel 的底层实现确保了在任何时候只有一个 goroutine 可以执行发送或接收操作。这是通过内部的互斥锁和等待队列来实现的。

总的来说，channel 的底层实现是围绕环形缓冲区、等待队列、互斥锁以及相关状态进行的。其设计使得 goroutines 可以安全、有效地同步并通信。为了完全理解其底层工作原理，建议直接查看 Go 的源代码，特别是 `runtime` 包中与 channel 相关的部分。

## `channel` 有缓冲区和无缓冲区的区别？
1. **创建方式**：

    - **无缓冲区的 channel**：`ch := make(chan int)`
    - **有缓冲区的 channel**：`ch := make(chan int, n)` 其中 `n` 是缓冲区大小。

2. **行为**：

    - **无缓冲区的 channel**：发送和接收操作都是同步的。当一个值被发送到一个无缓冲的 channel 上时，发送操作会阻塞，直到另一个 Go 协程从这个 channel 上接收这个值。类似地，当一个 Go 协程试图从一个无缓冲的 channel 接收值时，如果 channel 为空，那么接收操作会阻塞，直到另一个 Go 协程发送一个值到这个 channel。
    - **有缓冲区的 channel**：发送和接收操作可能是异步的，只要缓冲区不满或不为空。当缓冲区满时，发送操作会阻塞，直到有空间可用；当缓冲区为空时，接收操作会阻塞，直到有值可用。

3. **应用场景**：

    - **无缓冲区的 channel**：经常被用作同步点，例如等待一个任务完成或等待数据准备好。无缓冲的 channel 是确保两个协程同步执行的一种方式。
    - **有缓冲区的 channel**：当你希望解耦生产数据的速度和消费数据的速度时，有缓冲的 channel 是有用的。它们还可以用于实现工作队列，其中发送操作不需要立即由接收操作来匹配。

4. **注意事项**：

    - 尽管有缓冲区的 channel 可以提高某些情况下的性能，但如果缓冲区太大，它可能会隐藏程序中的竞争条件或死锁，使问题更难以调试。
    - 使用无缓冲的 channel 可以更容易地确保程序的确定性，因为发送和接收操作是同步的。

## channel 什么时候会发生阻塞
在 Go 中，channel 的操作（如发送数据和接收数据）在某些情况下会导致 goroutine 阻塞。以下是引起阻塞的常见情况：

1. **发送数据到已满的缓冲 channel**：
    - 对于带缓冲的 channel，如果缓冲区已满，尝试发送数据会导致发送 goroutine 阻塞，直到有其他 goroutine 从 channel 中接收数据并为新数据腾出空间。

2. **从空的缓冲 channel 接收数据**：
    - 如果缓冲区为空，尝试从 channel 接收数据会导致接收 goroutine 阻塞，直到有其他 goroutine 向 channel 发送数据。

3. **发送数据到没有接收者的非缓冲 channel**：
    - 对于不带缓冲的 channel（或称为同步 channel），发送操作要求有一个接收操作与之匹配。如果没有可用的接收者，发送 goroutine 会阻塞。

4. **从没有发送者的非缓冲 channel 接收数据**：
    - 同样，对于非缓冲的 channel，接收操作要求有一个发送操作与之匹配。如果没有数据被发送到 channel，接收 goroutine 会阻塞。

5. **尝试从已关闭且为空的 channel 接收数据**：
    - 如果一个 channel 已经关闭并且所有数据都已被接收，那么进一步的接收操作不会阻塞。相反，它会立即返回相应类型的零值。但这种情况并不会导致阻塞。

6. **使用 select 语句时没有可用的 case**：
    - 当使用 `select` 语句进行多路选择时，如果没有一个 case 可以执行（即所有 channel 操作都会阻塞），并且没有提供 `default` case，那么整个 `select` 会阻塞。

## 我能不能在写入 channel 的时候，先判断它阻不阻塞再写？
是的，你可以使用 Go 的 `select` 语句在尝试写入 channel 之前先检查它是否会阻塞。使用 `select` 的非阻塞模式，你可以先尝试一个操作，并在该操作无法立即完成时执行其他操作或回退。

以下是一个简单的示例，展示了如何检查 channel 是否可以无阻塞地写入：

```go
ch := make(chan int, 2)
ch <- 1  // 填充channel，使其快速满

select {
case ch <- 2:
    fmt.Println("Wrote to channel successfully.")
default:
    fmt.Println("Channel was full, did not write.")
}
```

在上面的例子中，`ch` 是一个带有容量为 2 的缓冲 channel。首先，我们写入一个值使其部分填满。然后我们尝试再次写入。`select` 语句允许我们尝试写入，如果 channel 已满（即写入操作会阻塞），则立即回退到 `default` case。

需要注意的是，非阻塞的 `select` 应该谨慎使用，因为不当的使用可能会导致忙等待，从而消耗 CPU 资源。此外，根据具体的使用场景，有时最佳的做法是让 goroutine 阻塞，而不是频繁地检查 channel 是否可用。

## 读取空 channel 会发生什么？
1. **未初始化的channel**:
    如果你尝试从一个未初始化的channel（nil channel）中读取或写入数据，你的goroutine将会阻塞，但这种阻塞是永久的，因为没有其他的goroutine会来初始化或关闭这个channel。在一个主goroutine中这样做会导致死锁。

    ```go
    var ch chan int // ch 是 nil
    <-ch            // 会永久阻塞
    ```

2. **已初始化但没有数据的channel**:
    如果你从一个已经初始化但是没有数据的channel读取数据，goroutine会阻塞，直到有其他的goroutine向该channel写入数据，或者该channel被关闭。

    ```go
    ch := make(chan int)
    go func() {
        time.Sleep(2 * time.Second)
        ch <- 42
    }()
    fmt.Println(<-ch) // 这会阻塞，直到上面的goroutine发送数据
    ```

3. **关闭的channel**:
    如果channel已经被关闭，那么从这个channel读取数据不会阻塞，它会立即返回该类型的零值。同时，你还可以通过第二个返回值来检测channel是否已关闭。

    ```go
    ch := make(chan int)
    close(ch)
    val, ok := <-ch
    fmt.Println(val, ok) // 输出 "0 false"
    ```

    其中，`val` 是从channel读取的值（对于int类型channel，关闭的channel读取会返回0），而 `ok` 是一个布尔值，当channel已经关闭并且所有的值都已经被读取时，它会返回false。这可以用来检测channel是否已经关闭。

以下是一个表格，描述了在不同channel状态下进行读、写和关闭操作的结果：

| 状态\操作   | 读取 (`<-ch`)            | 写入 (`ch <-`)                  | 关闭 (`close(ch)`) |
|------------|------------------------|-------------------------------|---------------------|
| 未初始化 (nil) | 永久阻塞                   | 永久阻塞                          | panic: close of nil chan |
| 空但已初始化     | 阻塞，直到有数据或 channel 关闭   | 阻塞，直到有读取或 channel 关闭          | 可以关闭，后续读操作会立即返回零值 |
| 有数据        | 立即返回数据                 | 可能阻塞，直到有足够空间或有读取              | 可以关闭，后续读操作会返回已有数据，然后返回零值 |
| 已关闭        | 如果还有数据，返回数据；无数据则立即返回零值 | panic: send on closed channel | panic: close of closed channel |

