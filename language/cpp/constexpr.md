# constexpr

## const 语义

C++中的const可用于修饰变量、函数，且在不同的地方有着不同的含义。

### const 变量

用const修饰的变量要求编译器阻止一切对变量的修改。const修饰变量必须在初始化时提供初始值。

    const int i = 0; //ok
    
    const int j;
    j = 5; //error
    
    const int k = cin.get(); //ok
    

### const 指针 & 引用
    
const 修饰引用时，与修饰变量相同。const修饰指针情况比较复杂。

指针变量的类型按变量名左边最近的‘*’分成两部分，右边的部分表示指针变量自己的性质，而左边的部分则表示它指向元素的性质。
指针自身为const表示不可对该指针进行赋值，而指向物为const则表示不可对其指向进行赋值。因此可以将引用看成是一个自身为const的指针，而const引
用则是const Type * const指针。

指向为const的指针是不可以赋值给指向为非const的指针，const引用也不可以赋值给非const引用，但反过来就没有问题了，这也是为了保证const语义不
被破坏。
可以用const_cast来去掉某个指针或引用的const性质，或者用static_cast来为某个非const指针或引用加上const性质：

C++类中的this指针就是一个自身为const的指针，而类的const方法中的this指针则是自身和指向都为const的指针。

### 类中的const成员变量

类中的const成员变量可分为两种：非static常量和static常量。

* 非static常量
类中的非static常量必须在构造函数的初始化列表中进行初始化，因为类中的非static成员是在进入构造函数的函数体之前就要构造完成的，而const常量在
构造时就必须初始化，构造后的赋值会被编译器阻止。

    
    class B {
    public:
        B(): name("aaa") {
            name = "bbb";
        } 
    private:   
        const std::string name;
    
    };
    
* static常量 static常量是在类中直接声明的，但要在类外进行唯一的定义和初始值，常用的方法是在对应的.cpp中包含类的static常量的定义：
    
    
    class A {
    
        ...
    
        static const std::string name;
    
    };
    const std::string A::name("aaa");
    

一个特例是，如果static常量的类型是内置的整数类型，如char、int、size_t等，那么可以在类中直接给出初始值，且不需要在类外再进行定义了。编译
器会将这种static常量直接替换为相应的初始值，相当于宏替换。但如果在代码中我们像正常变量那样使用这个static常量，如取它的地址，而不是像宏一样
只使用它的值，那么我们还是需要在类外给它提供一个定义，但不需要初始值了（因为在声明处已经有了）。

    
    class A {
    
        ...
    
        static const int SIZE = 50;
    
    };
    
    const int A::SIZE = 50;
    
* const修饰函数 C++中可以用const去修饰一个类的非static成员函数，其语义是保证该函数所对应的对象本身的const性。在const成员函数中，所有可
能违背this指针const性（const成员函数中的this指针是一个双const指针）的操作都会被阻止，如对其它成员变量的赋值以及调用它们的非const方法、
调用对象本身的非const方法。但对一个声明为mutable的成员变量所做的任何操作都不会被阻止。这里保证了一定的逻辑常量性。
     
另外，const修饰函数时还会参与到函数的重载中，即通过const对象、const指针或引用调用方法时，优先调用const方法。

    
    class A {
    
    public:
    
        int &operator[](int i) {
    
            ++cachedReadCount;
    
            return data[i];
    
        }
    
        const int &operator[](int i) const {
    
            ++size;
    
            --size;
    
            ++cachedReadCount;
    
            return data[i];
    
        }
    
    private:
    
        int size;
    
        mutable cachedReadCount;
    
        std::vector<int> data;
    
    };
    
    A &a = ...;
    
    const A &ca = ...;
    
    int i = a[0];
    
    int j = ca[0];
    
    a[0] = 2;
    
    ca[0] = 2;

这个例子中，如果两个版本的operator[]有着基本相同的代码，可以考虑在其中一个函数中去调用另一个函数来实现代码的重用（参考Effective C++）。
这里我们只能用非const版本去调用const版本。

    
    int &A::operator[](int i) {
    
        return const_cast<int &>(static_cast<const A &>(*this).operator[](i));
    
    }
    
其中为了避免调用自身导致死循环，首先要将*this转型为const A &，可以使用static_cast来完成。而在获取到const operator[]的返回值后，还要手
动去掉它的const，可以使用const_cast来完成。一般来说const_cast是不推荐使用的，但这里我们明确知道我们处理的对象其实是非const的，那么这里
使用const_cast就是安全的。

## constexpr

constexpr是C++11中新增的关键字，其语义是“常量表达式”，也就是在编译期可求值的表达式。最基础的常量表达式就是字面值或全局变量/函数的地址或
sizeof等关键字返回的结果，而其它常量表达式都是由基础表达式通过各种确定的运算得到的。constexpr值可用于enum、switch、数组长度等场合。

constexpr所修饰的变量一定是编译期可求值的，所修饰的函数在其所有参数都是constexpr时，一定会返回constexpr。

    
    constexpr int Inc(int i) {
    
        return i + 1;
    
    }
    
    constexpr int a = Inc(1);
    
    constexpr int b = Inc(cin.get());
    
    constexpr int c = a * 2 + 1;

constexpr还能用于修饰类的构造函数，即保证如果提供给该构造函数的参数都是constexpr，那么产生的对象中的所有成员都会是constexpr，该对象也就
是constexpr对象了，可用于各种只能使用constexpr的场合。注意，constexpr构造函数必须有一个空的函数体，即所有成员变量的初始化都放到初始化列
表中。

    
    struct A {
    
        constexpr A(int xx, int yy): x(xx), y(yy) {}
    
        int x, y;
    
    };
    
    constexpr A a(1, 2);
    
    enum {SIZE_X = a.x, SIZE_Y = a.y};
    
constexpr的好处：

* 是一种很强的约束，更好地保证程序的正确语义不被破坏。
* 编译器可以在编译期对constexpr的代码进行非常大的优化，比如将用到的constexpr表达式都直接替换成最终结果等。
* 相比宏来说，没有额外的开销，但更安全可靠。
* 允许一些计算只在编译时进行一次，而不是每次程序运行时；

## constexpr 变量

const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。所有constexpr对象都是const的，但是不是所有的const对象都
是constexpr的。

## constexpr 函数

constexpr函数限制持有和返回的类型为字面值类型（literal type），本质上就是一些在编译期间可确定值的类型。在C++中，除了void之外的内置类型
都是字面值类型，不过用户定义的类型也有可能是字面值类型，因为构造函数和其他成员函数可能是constexpr的；
如果实参都是常量表达式的话，那么它可以在编译时产生返回值；其它情况下，常量表达式函数跟普通函数一样，只有在运行时才能被调用，产生返回值；

对constexpr函数的基本要求：
* 常量表达式函数必须有返回值（不可以是void函数）
* 常量表达式函数体中只能有一条语句，且该语句必须是return语句。（可以使用?:、递归）但不产生实际代码的语句可以在常量表达式函数中使用，
如static_assert,using,typedef等（这条规定在C++14中大幅放松）
* return语句中，不能使用非常量表达式的变量、函数，且return的表达式也要是常量表达式
* 常量表达式函数在使用前，必须有定义。（普通函数在被调用前只要有函数声明就够了，不一定有定义）

常量构造函数的要求：
1. 成员变量只能通过初始化列表来初始化，函数体必须为空
2. 初始化列表只能由常量表达式来赋值

常量成员函数的要求：
   
1. 常量成员函数被隐式定义为const成员函数，不可以通过常量成员函数去修改成员变量。也就是说，常量成员函数往往是所谓的getter函数。
（c++14则不同，允许constexpr成员函数去修改成员变量）
2. 量成员函数不能是virtual的

在C++11与C++14的区别：
    在C++11标准中，对于constexpr修饰的函数给了及其苛刻的限定条件：函数的返回值类型及所有形参的类型都是字面值类型，而且函数体内必须有且只
    有一条return语句。这个条件显然是太苛刻了，以至于很多在constexpr的操作都要借助？：表达式，递归等办法实现。在C++14中，放宽了这一限定，
    只保留了“函数的返回值类型及所有形参的类型都是字面值类型”，也就是说，这些值都在编译期能确定了就行。
    

## constexpr与const的本质区别

const主要用于表达“对接口的写权限控制”，即“对于被const修饰的量名(例如const指针变量)，不得通过它对所指对象作任何修改”。(但是可以通过其他
接口修改该对象)。另外，把对象声明为const也为编译器提供了潜在的优化可能。具体来说就是，如果把一个量声明为const，并且没有其他地方对该量作取
址运算，那么编译器通常(取决于编译期实现)会用该量的实际常量值直接替换掉代码中所有引用该量的地方，而不用在最终编译结果中生成对该量的存取指令。
constexpr的主要功能则是让更多的运算可以在编译期完成，并能保证表达式在语义上是类型安全的。(译注：相比之下，C语言中#define只能提供简单的文
本替换，而不具任何类型检查能力)。与const相比，被constexpr修饰的对象则强制要求其初始化表达式能够在编译期完成计算。之后所有引用该常量对象的
地方，若非必要，一律用计算出来的常量值替换。


