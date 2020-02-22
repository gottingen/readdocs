# c++ memory order model

本文档主要简述c++11引入的内存顺序模型。

## C++ Memory order model

C++ 11 在标准中提出了6种同步操作： memory_order_relaxed, memory_order_acquire, memory_order_consume, memory_order_release, memory_order_acq_rel, memory_order_seq_cst。
