# 容器
## vector 容器动态扩展的原理？
当插入新元素的时候，如果空间不足，那么 vector 会重新申请更大的一块内存空间，将原空间数据拷贝到新空间，释放旧空间数据，再把新元素插入新申请空间。  

在这个过程中，vector 会根据某种策略确定新的容量，常见的策略是倍增，即新的容量是当前容量的 2 倍。但具体的扩展因子可能因库的实现而异。  

但是因为 2 倍增长对于缓存并不友好，新开辟的空间无法重复利用之前释放的空间，所以也有会库选择 1.5 倍增长，比如 folly

## vector 容器插入元素发生了什么？
1. **对空 vector 插入一个元素**： 

     如果采用无参构造，vector 并不会申请内存，所以对空 vector 插入一个元素时，会先申请 1 个元素的空间，并把这个元素插入到 vector

2. **当前内存用完时插入**：

     扩容后，复制或移动旧内存块中的元素到新的内存块，再插入新元素

3. **在中间插入一个元素**：

     从插入点开始，后面的所有元素都需要向后移动一个位置以为新元素腾出空间，再插入新元素

在所有插入的情况下，都需要更新内部指针，如 `begin`、`end` 和 `end_of_capacity`

## vector 的迭代器什么时候会失效？
1. **改变容量的操作**：
     - **扩展**：当 `std::vector` 的容量不足以容纳更多的元素时（例如，在 `push_back` 或 `insert` 操作中），它可能需要重新分配内存并扩展其容量。在这种情况下，所有之前的迭代器都将失效，因为它们指向的内存块可能已被释放。
     - **收缩**：使用 `shrink_to_fit` 或 `resize`（减少大小）可以导致容器收缩。这可能导致重新分配内存，从而使迭代器失效。

2. **删除或插入元素**：
     - 使用 `erase` 删除元素将导致指向已删除元素及其之后元素的所有迭代器失效，但 `erase` 也会返回当前位置的新的、正确的迭代器。
     - 使用 `insert` 插入元素将导致指向已元素元素及其之后元素的所有迭代器失效（如果不涉及容量改变时）

3. **调用 `clear` 方法**：这将删除容器中的所有元素，使所有迭代器失效。

## vector 的 emplace_back 和 push_back?
`std::vector` 的 `emplace_back` 和 `push_back` 都是用于在容器的尾部插入元素的方法，但它们的工作方式和使用场景有所不同：

1. **push_back**:
      - 这个方法直接接受一个元素作为参数，该元素必须与 `std::vector` 所存储类型兼容。
      - `push_back` 会将这个元素复制或移动到容器的尾部。
      - 示例：
        ```cpp
        std::vector<int> vec;
        int value = 42;
        vec.push_back(value);  // Copies the value into the vector
        ```

2. **emplace_back**:
      - 这个方法接受一组参数，并直接在容器的尾部构造一个元素。这意味着它可以避免不必要的临时对象的创建和复制。
      - `emplace_back` 的参数是用来构造 `std::vector` 所存储类型的元素的。换句话说，这些参数将被传递给元素类型的构造函数。
      - 示例：
        ```cpp
        std::vector<std::pair<int, std::string>> vec;
        vec.emplace_back(42, "meaning");  // Constructs the pair directly in the vector
        ```

3. **优势**：
     - 使用 `emplace_back` 通常更为高效，因为它直接在容器内存中进行构造，避免了不必要的复制或移动操作。在某些情况下，这可以提供明显的性能优势，特别是当元素类型的复制或移动操作代价较高时。
     - 但要注意，`emplace_back` 的效率提升并不总是明显，尤其是对于简单的数据类型（如 `int`、`double` 等）。

4. **适用场景**：
     - 当你已经有一个完整的对象并想将其添加到 `std::vector` 中时，使用 `push_back` 是合适的。
     - 当你希望直接在 `std::vector` 中构造一个对象（而不是先在外部构造然后复制或移动）时，使用 `emplace_back` 是更好的选择。


## 使用 vector 需要注意一些什么？
- 使用 `reserve` 来避免不必要的重新分配
- 避免使用 `vector<bool>`

## stack 的底层实现？
`std::stack` 在 C++ 标准库中并不是一个完全独立的容器，而是一个容器适配器。它提供了栈操作（如 `push()`, `pop()`, 和 `top()`），但底层的数据存储和实际的内存管理是由另一个容器类型提供的。

默认情况下，`std::stack` 使用 `std::deque` 作为其底层容器，但也可以使用其他容器，如 `std::vector` 或 `std::list`，只要这些容器提供了 `back()`, `push_back()`, 和 `pop_back()` 这些操作。

示例：
```cpp
std::stack<int> s1; // 默认使用 std::deque<int> 作为底层容器

std::stack<int, std::vector<int>> s2; // 使用 std::vector<int> 作为底层容器

std::stack<int, std::list<int>> s3; // 使用 std::list<int> 作为底层容器
```

`std::stack` 只提供有限的接口，它隐藏了底层容器的大部分功能，确保只能按照后进先出 (LIFO) 的方式访问元素。

## unordered_map 的底层原理等？
`unordered_map` 是 STL 中的一个容器，它实现了一个哈希表（hash table）。与 `std::map` 不同，它不保证其中的元素有任何特定的顺序，但它的插入、删除和查找操作通常具有常数平均时间复杂度（尽管在最坏情况下，这些操作可能是线性的）。

以下是 `unordered_map` 的底层原理和实现的简要描述：

1. **存储结构**：`unordered_map` 的基本结构是一个动态数组（bucket array）。数组的每一个位置（称为桶）通常包含一个链表，链表中存放的是哈希值相同的所有元素。

2. **哈希函数**：为了确定一个元素应该存储在哪个桶中，`unordered_map` 使用一个哈希函数将元素的键转换为一个整数，然后使用这个整数的模数值来确定桶的索引。

3. **碰撞处理**：当两个不同的键产生相同的哈希值时，会发生所谓的“碰撞”。`unordered_map` 通常使用开链法（chaining）来处理碰撞，这意味着在同一个桶中的所有元素都存储在一个链表中。

4. **动态调整**：为了保持操作的高效，当元素数量增加到使得每个桶的平均大小超过一个阈值时，`unordered_map` 会增加桶的数量并重新哈希所有的元素（即重新分布所有元素）。这是一个相对昂贵的操作，但是它发生的频率是有限的，所以平均情况下，插入操作仍然是常数时间的。


## map 和 unordered_map 区别？
`map` 和 `unordered_map` 都是 STL 中的关联容器，用于存储键值对。以下是它们之间的主要区别：

1. **内部实现**：
    - `map`：通常使用红黑树（一种平衡二叉搜索树）来实现。
    - `unordered_map`：使用哈希表来实现。

2. **迭代顺序**：
    - `map`：由于是基于红黑树，键值对按键的升序排序。因此，迭代器按键的升序遍历元素。
    - `unordered_map`：如其名所示，它不保证任何特定的顺序。哈希表的迭代顺序基于桶和哈希值，与键的实际值无关。

3. **时间复杂度**：
     - `map`：插入、删除、查找：平均和最坏情况都是 \(O(\log n)\)。
     - `unordered_map`：插入、删除、查找：平均情况下是 \(O(1)\)，但在最坏情况下（例如，当所有元素都哈希到同一个桶时）是 \(O(n)\)。

4. **功能**：
    - `map`：由于是有序的，支持一些与顺序相关的操作，例如 `lower_bound` 和 `upper_bound`。
    - `unordered_map`：不支持与顺序相关的操作。

5. **空间复杂度**：
    - `map`：通常比 `unordered_map` 使用更少的空间，因为它不需要存储额外的哈希信息。
    - `unordered_map`：由于哈希表的性质，它可能使用更多的空间，特别是为了保持低负载因子。

6. **函数需求**：
    - `map`：需要键类型定义了 `<` 运算符或提供了自定义的比较函数。
    - `unordered_map`：需要一个哈希函数（可以使用 C++11 提供的 `std::hash`）和键类型的 `==` 运算符来处理碰撞。

## map 为什么用红黑树不用 AVL 树？
1. **插入和删除的效率**：
    - 插入导致的 rebalance，红黑树和 AVL 树都最多需要 2 次旋转。
    - 删除导致的 rebalance，红黑树最多需要 3 次旋转，而在最坏的情况下，AVL 树可能需要 \(O(\log n)\) 次旋转。
    - 其次，AVL 树的结构相较红黑树来说更为平衡，插入和删除节点更容易引起不平衡，因此在大量数据需要插入或者删除时，AVL 树需要 rebalance 的频率会更高
   
2. **查询效率**：
    - 红黑树的查询效率略低于 AVL 树，但是仍为 \(O(\log n)\)。

3. **实现复杂性**：
    - 虽然 AVL 树为了保持严格的平衡性需要进行额外的维护（例如，跟踪每个节点的平衡因子），但红黑树的平衡要求相对宽松，使得其实现起来可能更简单。

4. **CPU 友好性**：
    - 红黑树的平衡条件可能使其在实际硬件上更有效。由于 AVL 树的严格平衡性，它可能需要更多的旋转和内存访问。更多的内存访问可能会导致更多的 CPU 缓存未命中，这可能会对性能产生负面影响。

5. **空间考虑**：
    - 红黑树通常只需要存储节点颜色信息（红或黑），而 AVL 树则需要存储额外的平衡信息（例如每个节点的平衡因子或左右子树的高度差）。这可能使得红黑树的空间需求略小一些。

所以，选择红黑树是考虑了多种性能下的折衷

## 优先队列的底层实现？
优先队列是一种抽象的数据结构，它允许插入元素、检索并删除优先级最高（或最低）的元素。优先队列不直接支持访问所有内容或按顺序访问每个元素，它只提供对优先级最高的元素的访问。

在 C++ 标准库中，`std::priority_queue` 是一个容器适配器，它提供了优先队列的语义。其底层实现通常基于以下数据结构：

1. **二叉堆（Binary Heap）**：这是实现优先队列的最常见的数据结构。二叉堆是一个完全二叉树，它可以有效地用数组表示。`std::priority_queue` 默认使用 `std::vector` 作为其底层容器，但也可以使用其他容器，如 `std::deque`。二叉堆支持如下操作的时间复杂度：
    - `push` (插入)：\(O(\log n)\)
    - `pop` (删除最大/最小元素)：\(O(\log n)\)
    - `top` (查看最大/最小元素)：\(O(1)\)

2. **平衡二叉搜索树**：如红黑树或AVL树也可以用来实现优先队列，它们提供了对所有操作O(log n)的时间复杂度。不过，由于常数因子和实际的实现细节，它们通常不如二叉堆那么高效。

在实际使用中，如果你不需要修改优先队列的底层实现或性能特性，`std::priority_queue` 提供的默认二叉堆实现通常已经足够了。但如果有特定的性能需求，了解底层的数据结构和其特性是很有帮助的。

## 容器的线程安全？
https://www.zhihu.com/question/577529159

[//]: # (TODO)
