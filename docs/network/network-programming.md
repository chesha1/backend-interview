# 网络编程

## `recv` 失败返回 `-1`，怎么得到错误的具体信息
当 `recv` 调用失败并返回 `-1` 时，你可以使用全局变量 `errno` 来获取错误代码。Linux 中系统调用的错误都存储于 `errno` 中，`errno` 由操作系统维护，存储就近发生的错误，即下一次的错误码会覆盖掉上一次的错误。

为了得到此错误代码对应的具体错误消息，你可以使用 `perror()` 或 `strerror()` 函数。

以下是如何使用这些函数的示例：

1. **使用 `perror()`：**
    ```c
    #include <stdio.h>
    #include <sys/socket.h>
    #include <errno.h>

    // ... 其他代码 ...

    ssize_t bytes_received = recv(socket_fd, buffer, sizeof(buffer), 0);
    if (bytes_received == -1) {
        perror("recv");
    }
    ```

    `perror` 函数会打印你提供的字符串，后跟一个冒号和具体的错误消息。

2. **使用 `strerror()`：**
    ```c
    #include <stdio.h>
    #include <sys/socket.h>
    #include <string.h>
    #include <errno.h>

    // ... 其他代码 ...

    ssize_t bytes_received = recv(socket_fd, buffer, sizeof(buffer), 0);
    if (bytes_received == -1) {
        fprintf(stderr, "recv error: %s\n", strerror(errno));
    }
    ```

    `strerror` 函数接受一个 `errno` 值并返回一个指向描述此错误的字符串的指针。