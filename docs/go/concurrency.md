# 并发
## 有哪些东西是协程安全的？
在 Go 语言中，某些基本的组件和工具被设计为协程安全（goroutine-safe），这意味着在多个 goroutine 并发访问时，它们能够正确并安全地工作，而不引发数据竞争或其他并发问题。以下是一些 Go 中的并发安全组件：

1. **`sync/atomic` 包**：提供了原子操作函数，如 `AddInt32`、`LoadPointer` 等，这些函数可以在并发环境中安全地操作变量。

5. **`sync.Map`**：一个并发安全的 map 实现，特别适合于读多写少的场景。

6. **`sync.Pool`**：一个并发安全的对象池，可以被多个 goroutine 同时使用，用于存储和复用临时对象。

7. **`channel`**：Go 的内置通道数据结构是并发安全的。它用于在 goroutines 之间发送和接收消息。

8. **`net/http` 的部分组件**：例如，`http.DefaultClient` 和 `http.DefaultTransport` 都是并发安全的，可以从多个 goroutines 安全地访问。

## Go 的协程是怎么调度的？
Go 语言的协程调度由 Go 运行时的调度器（scheduler）管理，这个调度器是设计用来有效利用 CPU 资源的同时保持足够的简单性和可维护性。以下是 Go 协程调度的几个关键点：

1. **M, P, G 模型**：
    - **Goroutine（G）**：它代表 Go 中的一个协程，是一个可调度的实体，类似于线程但更轻量。
    - **Machine（M）**：代表一个操作系统线程。
    - **Processor（P）**：是一种资源，可以认为是调度 Goroutine 的上下文。每个 P 都有自己的运行队列，P 数量通常由 GOMAXPROCS 环境变量控制，这个值默认等于机器的 CPU 核心数。

2. **工作窃取**：
    - 每个 P 都有自己的本地运行队列，其中存放了待运行的 Goroutines。
    - 当一个 M（线程）空闲时，它会尝试从其他P的运行队列中"窃取"一些 Goroutines 来执行。
    - 这种工作窃取机制可以减少 CPU 的空闲时间，并提高整体效率。

3. **调度点**：
    - Go 的调度器是协作式的，这意味着它需要在特定的点进行调度。这些点通常是系统调用或阻塞操作，如 I/O 操作、channel 操作、锁操作等。
    - 在这些调度点，当前正在运行的 Goroutine 可能会被暂停，调度器将选择另一个 Goroutine 来运行。

4. **非抢占式调度**：
    - Go 调度器基本上是非抢占式的，意味着正在执行的 Goroutine 将持续运行，直到它达到一个调度点。
    - 从 Go 1.14 开始，引入了基于信号的抢占调度，这允许调度器更有效地处理长时间运行的 Goroutines。

5. **网络轮询器**：
    - Go 的网络轮询器是调度器的一部分，专门用于网络 I/O 操作。
    - 当Goroutine 进行网络I/O时，它会被放入网络轮询器等待，而调度器可以在这期间运行其他 Goroutines。

6. **Goroutine 的轻量级**：
    - Goroutines 比操作系统线程更轻量，启动更快，且在内存占用和上下文切换成本方面都更高效。
    - 这使得Go程序可以轻松创建成千上万的 Goroutines。

## Goroutine 和线程的区别？
Goroutine 和线程是两种不同的并发执行单位，它们在设计和功能上有显著的区别：

1. **定义和抽象级别**：
    - **Goroutine**：是 Go 语言特有的并发执行体，比线程更轻量级。它是由 Go 的运行时（runtime）管理的，而非直接由操作系统管理。
    - **线程**：是操作系统提供的并发执行体，是操作系统调度的基本单位。线程拥有独立的执行堆栈和局部存储，但共享数据段、代码段和其他操作系统资源。

2. **资源消耗和扩展性**：
     - **Goroutine**：需要的资源（内存、初始化时间等）远小于线程，通常只需要几 KB 的堆栈空间。因此，在相同的内存条件下，可以创建数十万个 Goroutine。
     - **线程**：每个线程通常需要较大的内存（通常为 MB 级别）以及显著的创建和销毁开销。

3. **调度方式**：
    - **Goroutine**：由Go语言的运行时进行调度，采用的是 M:N 调度模型（多个 Goroutine 映射到多个线程）。Go 的调度器可以在用户态进行调度，避免了内核态切换的开销。
    - **线程**：由操作系统内核进行调度，采用 1:1 模型（一个线程对应一个内核线程）。线程切换需要在用户态和内核态之间转换，带来较大开销。

4. **创建和销毁的开销**：
    - **Goroutine**：创建和销毁的开销小，因为它们是由 Go 运行时在用户空间管理。
    - **线程**：创建和销毁的开销相对较大，因为这些操作需要操作系统内核介入。

5. **并发模型**：
    - **Goroutine**：鼓励使用基于通道（channel）的并发模型，通过通道进行数据通信，避免了传统的共享内存并发所带来的复杂性。
    - **线程**：通常使用共享内存进行通信，这可能导致竞争条件和死锁等问题，需要谨慎处理同步和互斥。

6. **堆栈管理**：
    - **Goroutine**：拥有动态增长的堆栈，初始时只占用很小的内存，随着需要可以增长和缩减。
    - **线程**：拥有固定大小的堆栈，通常在创建时就已经确定，不会根据需求变化。

总结来说，Goroutine 在 Go 语言中提供了一种更轻量级、更容易扩展、调度更高效的并发执行方式，相比传统的线程，它们更适合构建高并发和高性能的应用程序。

## 说一下 Go 的 GMP 模型
Go 语言的并发模型是其核心特性之一，这个模型主要是通过 GMP 调度机制来实现的。GMP 模型包含三个关键部分：Goroutine（G），Machine（M），和Processor（P）。它们共同工作，实现了高效的并发处理。

1. **Goroutine（G）**：
    - Goroutines 是 Go 程序中并发执行的实体，类似于线程但更加轻量。
    - 它们代表着待执行的任务，可以快速创建和销毁，且初始时占用的内存更小。
    - Go 的调度器负责调度这些 Goroutines 到线程上执行。

2. **Machine（M）**：
    - Machine 代表一个操作系统级别的线程。
    - 它是与内核线程直接对应的实体，用于执行 Goroutines。
    - 一个 M 在任何时刻都在执行一个 Goroutine，或者在等待执行 Goroutine 的指令。

3. **Processor（P）**：
    - Processor 代表了运行 Goroutines 所需的资源，可以理解为 Goroutine 执行的上下文环境。
    - 每个 P 都有一个本地的可运行 Goroutine 队列。
    - P 的数量通常由 GOMAXPROCS 环境变量控制，这个值默认等于机器的 CPU 核心数。

**调度过程**：

 - 当一个 Goroutine 被创建时，它会被放置到一个 P 的本地队列中。如果队列满了，它可能会被移动到一个全局队列中。
 - 每个 M 都必须持有一个 P 才能执行 Goroutines。如果一个 M 失去了它的 P（比如执行了一个阻塞的系统调用），它会尝试从其他 M 那里窃取一个 P。
 - 为了保证负载均衡，空闲的 M 会尝试从其他 P 或全局队列中窃取 Goroutines 来执行。

**特点**：

- GMP 模型使得 Go 能够以更少的线程来处理大量的 Goroutines，减少了线程创建和切换的开销。
- Goroutines 有能力根据需要动态地增长或缩减其栈大小，这使得每个 Goroutine 的初始内存占用变得非常小。
- 从 Go 1.14 开始，Go 调度器引入了基于信号的抢占式调度，可以在长时间运行的 Goroutine 中断其执行，让其他 Goroutine 有机会运行，增强了调度的公平性和响应性。

## 为什么要有 P，只有 GM 可以吗？
感觉回答得不好，仅供参考

在 Go 语言的并发模型中，P（Processor）的存在对于调度器的效率和效能至关重要。虽然理论上仅有 G（Goroutine）和 M（Machine，即操作系统线程）也可以实现并发，但这种方式存在一些限制和效率问题。P 的引入主要解决了这些问题。

1. **负载均衡**：P 充当了 Goroutines 和 M 之间的调度单位，每个 P 维护自己的本地运行队列。这种设计允许多个线程（M）平衡地从这些队列中获取任务（G），从而实现了更有效的负载均衡。

2. **减少锁竞争**：如果仅有 G 和 M，所有 Goroutines 将共享一个全局队列，而所有线程将从这个全局队列中获取任务。这会导致高度的锁竞争，因为每个线程在访问和修改队列时都需要加锁，从而降低效率。P 通过提供本地队列减少了这种全局锁的需求。

3. **高效的协程调度**：P 的存在使得 Go 的运行时可以更加灵活地管理 Goroutines，例如通过工作窃取算法来动态分配任务。当一个 P 的本地队列为空时，它的线程可以尝试从其他 P 的队列中“窃取”任务，从而保持线程的忙碌并减少空闲时间。

4. **调度性能优化**：P 的数量通常被设置为与机器的 CPU 核心数相匹配。这种对应确保了程序可以充分利用 CPU 资源，同时避免过多的线程切换和资源竞争。

5. **充分利用多核处理器**：现代计算机通常具有多个 CPU 核心。通过将 P 的数量与 CPU 核心数匹配，Go 程序可以在多核上实现真正的并行执行，而不仅仅是并发执行。

总的来说，P 的存在增强了 Go 调度器的灵活性和效率，使得 Goroutines 的调度更加高效，减少了资源竞争，同时更好地利用了多核处理器的能力。在没有 P 的情况下，这些优点将难以实现。

## 用两个 goroutine 交替打印 1-100 的奇偶数
要实现两个 Goroutine 交替打印 1 到 100 内的奇偶数，可以使用 Go 语言中的 channel 来进行 Goroutines 之间的同步。下面是一个实现的例子：

首先，创建两个 Goroutine，一个用于打印奇数，另一个用于打印偶数。通过两个 channel —— `odd` 和 `even` —— 来控制打印的顺序。当一个 Goroutine 打印一个数后，它通过相应的 channel 发送信号给另一个 Goroutine，然后等待另一个 Goroutine 通过另一个 channel 发回信号。

```go
package main

import (
	"fmt"
)

func main() {
	odd := make(chan bool)
	even := make(chan bool)

	go printOdd(odd, even)
	go printEven(odd, even)

	// 启动打印奇数的 Goroutine
	odd <- true

	// 等待打印任务完成
	var input string
	fmt.Scanln(&input)
}

func printOdd(odd, even chan bool) {
	for i := 1; i <= 100; i += 2 {
		<-odd
		fmt.Println(i)
		even <- true
	}
}

func printEven(odd, even chan bool) {
	for i := 2; i <= 100; i += 2 {
		<-even
		fmt.Println(i)
		odd <- true
	}
}
```

在这个程序中：

- `printOdd` 函数负责打印奇数，它在接收到来自 `odd` channel 的信号后打印一个奇数，然后通过 `even` channel 发送信号。
- `printEven` 函数负责打印偶数，它在接收到来自 `even` channel 的信号后打印一个偶数，然后通过 `odd` channel 发送信号。
- 主 Goroutine 首先发送信号给 `printOdd` 函数开始打印序列，然后等待用户输入来防止程序立即退出。

这种方法通过 channel 实现了 Goroutine 之间的同步和通信，保证了奇数和偶数的交替打印。
