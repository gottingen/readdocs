# seastar

[项目首页](http://seastar.io)

[github地址](https://github.com/search?q=seastar)

## introduction

seastar 是一个优秀的c++ 网络框架。
## features

## code reading

### app

### cmake

### cmake-tests

### cmake-tools

### core

### debug

### docs

### docker

### http

### json

### kvm

### license

### net

### proto

### rpc

### scripts

### test

### util

#### [noncopyable_function](/rpc/seastar/seastar-18.08-branch/util/noncopyable_function.hh)
    
class noncopyable_function是std::function的兄弟版本，最大的区别之处在于。该类只调用函数对象额吗，move构造函数。

    
***feature***
1. 禁用拷贝构造
2. 禁用赋值运算符

***优点***
1. 通过移动赋值，减少了多引用问题，避免内存竞态生命周期管理的复杂性。
2. 源码实现简单，易读，可维护性高。libc++和stdc++用复杂的模板元编程实现了function/bind.一但出现问题等于宣布死刑，修改stl涉及到整个系统
，多个应用的协同等问题。
3. 性能方面，使用constexpr，在编译器确定内存位置。

***缺点***
1. 不能携带太多的参数，这点在后面的源码分析会持续涉及到。seastar设计使用tuple等处理这类问题。
    
***实现***

#### 

    
### root