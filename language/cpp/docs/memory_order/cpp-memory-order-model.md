# c++ memory order model

本文档主要简述c++11引入的内存顺序模型。

## Synchronized-with 与 happens-before 关系

* Happens-before 关系

    通常定义为:
   > Let A and B represent operations performed by a multithreaded process. If A happens-before B, then the memory effects of A effectively become visible to the thread performing B before B is performed.

    对于单线程而言,这很明了:如果一个操作 A 排列在另一个操作 B 之前,那么这个操作 A happens-before B. 但如果多个操作发生在一条声明中(statement),那么通常他们是没有 happens-before 关系的,因为他们是未排序的。当然这也有例外,比如逗号表达式。

    对于多线程而言,如果一个线程中的操作 A inter-thread happens-before 另一个线程中的操作 B, 那么 A happens-before B。

    [refer-blog-1](https://preshing.com/20130702/the-happens-before-relation/)

* Synchronized-with 关系

    >”Synchronizes-with” is a term invented by language designers to describe ways in which the memory effects of source-level operations – even non-atomic operations – are guaranteed to **become visible** to other threads.

    >In every synchronizes-with relationship, you should be able to identify two key ingredients, which I like to call **the guard variable** and **the payload**. The payload is the set of data being propagated between threads, while the guard variable protects access to the payload.

    ![synchronize-with](/language/cpp/images/memory-order/synchronize-with.png)

    [refer-blog-2](https://preshing.com/20130823/the-synchronizes-with-relation/)

## C++ Memory order model

C++ 11 在标准中提出了6种原子同步操作： memory_order_relaxed, memory_order_acquire, memory_order_consume, memory_order_release, memory_order_acq_rel, memory_order_seq_cst。

这6种表示三种内存模型:

### sequential consistent

对应memory_order_seq_cst: SC作为默认的内存序，是因为它意味着将程序看做是一个简单的序列。如果对于一个原子变量的操作都是顺序一致的，那么多线程程序的行为就像是这些操作都以一种特定顺序被单线程程序执行

### relaxed

对应memory_order_relaxed: 在原子变量上采用 relaxed ordering 的操作不参与 synchronized-with 关系。在同一线程内对同一变量的操作仍保持happens-before关系，但这与别的线程无关。
在 relaxed ordering 中唯一的要求是在同一线程中，对同一原子变量的访问不可以被重排。

### acquire release

对应memory_order_acquire & memory_order_release & memory_order_consume & memory_order_acq_rel

Acquire与Release是无锁编程中最容易混淆的两个原语，它们是线程之间合作进行数据操作的关键步骤。在这里，借助前面对memory barrier的解释，对acquire与release的语义进行阐述。
* acquire本质上是read-acquire，它只能应用在从RAM中read数据这种操作上，它确保了所有在acquire之后的语句不会被调整到它之前执行
* release本质上是write-release，它只能应用在write数据到RAM中，它确保了所有在release之前的语句不会被调整到它之后执行

memory_order_consume 是 acquire-release 顺序模型中的一种，但它比较特殊，它为 inter-thread happens-before 引入了数据依赖关系：dependency-ordered-before 和 carries-a-dependency-to.

dependency-ordered-before 可以应用于线程与线程之间。这种关系通过一个原子的被标记为 memory_order_consume 的 load 操作引入。这是memory_order_acquire 内存序的特例:它将同步数据限定为具有直接依赖的数据。一个被标记为 memory_order_release, memory_order_acq_rel 或memory_order_seq_cst 的原子 store 操作 A dependency-ordered-before 一个被标记为 memory_order_consume 的欲读取刚被改写的值的原子 load 操作 B. 同时，如果 A dependency-order-beforeB, 那么 A inter-thread happens-before B。

如果一个 store 操作被标记为 memory_order_release, memory_order_acq_rel或 memory_order_seq_cst, 一个 load 操作被标记为 memory_order_cunsume,memory_order_acquire 或 memory_order_seq_sct, 一连串 load 中的每个操作，读取的都是之前操作写下的值，那么这一连串操作将构成 release sequence,最初的store 操作 synchronized-with 或 dependency-ordered-before 最后的 load。
