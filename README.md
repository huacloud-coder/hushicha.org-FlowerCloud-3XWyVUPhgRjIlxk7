
### std::optional


1. 背景
在编程时，我们经常会遇到可能会返回/传递/使用一个确定类型对象的场景。也就是说，这个对象可能有一个确定类型的值也可能没有任何值。因此，我们需要一种方法来模拟类似指针的语义：指针可以通过 nullptr来表示没有值。解决方法是定义该对象的同时再定义一个附加的 bool类型的值作为标志来表示该对象是否有值。std::optional\<\>提供了一种类型安全的方式来实现这种对象。
2. 占用内存大小
可选对象所需的内存等于内含对象的大小加上一个 bool类型的大小。因此，可选对象一般比内含对象大一个字节（可能还要加上内存对齐的空间开销）。可选对象不需要分配堆内存，并且对齐方式和内含对象相同。



```
#include 
#include 

// 定义一个没有默认构造函数的类
class MyClass {
public:
    explicit MyClass(int value) : data(value) {}
    ~MyClass() {}

    int getData() const {
        return data;
    }

private:
    int data;
};

// 输出 std::optional 是否包含值
void check_optional_value(std::optional& opt) {
    if (opt) {
        std::cout << "Value present: " << opt->getData() << std::endl;
    } else {
        std::cout << "No value present." << std::endl;
    }
}

int main() {
    // 创建一个没有值的 std::optional
    std::optional opt1;
    check_optional_value(opt1);

    // 创建一个有值的 std::optional
    std::optional opt2{MyClass(42)};
    check_optional_value(opt2);

    // 尝试通过 emplace 添加值
    opt1.emplace(24);
    check_optional_value(opt1);

    // 尝试通过 operator= 添加值
    opt1 = MyClass(56);
    check_optional_value(opt1);

    return 0;
}

```

输出：
Size of i: 4 bytes
Size of St8optionalIiE: 8 bytes
Size of 7MyClass: 4 bytes
Size of St8optionalI7MyClassE: 8 bytes


然而，可选对象并不是简单的等价于附加了bool标志的内含对象。例如，在没有值的情况下，将不会调用内含对象的构造函数（通过这种方式，没有默认构造函数的内含类型也可以处于有效的默认状态）。


3\.语义
和 std::variant\<\>、std::any一样，可选对象有值语义。也就是说，拷贝操作会被实现为深拷贝：将创建一个新的独立对象，新对象在自己的内存空间内拥有原对象的标记和内含值（如果有的话）的拷贝。拷贝一个无内含值的 std::optional\<\>的开销很小，但拷贝有内含值的 std::optional\<\>的开销约等于拷贝内含值的开销。另外，std::optional\<\>对象也支持 move语义。


4\.应用
（1）std::optional\<\>模拟了一个可以为空的任意类型的实例。它可以被用作成员、参数、返回值等。
下面的示例程序展示了将 std::optional\<\>用作返回值的一些功能：



```
#include 
#include 
#include 

// 如果可能的话把string转换为int：
std::optional<int> asInt(const std::string& s)
{
    try {
        return std::stoi(s);
    }
    catch (...) {
        return std::nullopt;
    }
}

int main()
{
    for (auto s : {"42", "  077", "hello", "0x33"}) {
        // 尝试把s转换为int，并打印结果：
        std::optional<int> oi = asInt(s);
        if (oi.has_value()) {
            std::cout << "convert '" << s << "' to int: " << oi.value() << "\n";
        }
        else {
            std::cout << "can't convert '" << s << "' to int\n";
        }
    }
}

```

（2） 另一个使用 std::optional\<\>的例子是传递可选的参数和设置可选的数据成员：



```
#include 
#include 
#include 

class Name
{
private:
    std::string first;
    std::optional middle;
    std::string last;
public:
    Name (std::string f, std::optional m, std::string l)
          : first{std::move(f)}, middle{std::move(m)}, last{std::move(l)} {
    }
    friend std::ostream& operator << (std::ostream& strm, const Name& n) {
        strm << n.first << ' ';
        if (n.middle) {
            strm << *n.middle << ' ';
        }
        return strm << n.last;
    }
};

int main()
{
    Name n{"Jim", std::nullopt, "Knopf"};
    std::cout << n << '\n';

    Name m{"Donald", "Ervin", "Knuth"};
    std::cout << m << '\n';
}

```

5\.std::optional\<\>类型和操作
（1）std::optional\<\>类型标准库在头文件 中以如下方式定义了 std::optional\<\>类：
namespace std {
template class optional;
}
另外还定义了下面这些类型和对象：
• std::nullopt\_t类型的 std::nullopt，作为可选对象无值时候的“值”。
• 从 std::exception派生的 std::bad\_optional\_access异常类，当无值时候访问值将会抛出该异常。
可选对象还使用了 头文件中定义的 std::in\_place对象（类型是 std::in\_place\_t）来支持用多个参数初始化可选对象（见下文）。
（2）std::optional\<\>的操作
表std::optional的操作列出了 std::optional\<\>的所有操作：
![](https://img2024.cnblogs.com/blog/1249620/202409/1249620-20240913104211813-690798813.png)



```
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 


// 使用命名空间简化代码
using namespace std::string_literals;

// 示例 1：构造 std::optional
void construct_optional() {
    std::optional<int> o1; // 不含有值
    assert(!o1.has_value());

    std::optional<int> o2(std::nullopt); // 显式表示不含有值
    assert(!o2.has_value());

    std::optional o3{42}; // 推导出 std::optional
    assert(o3.has_value());
    assert(*o3 == 42);

    std::optional o4{"hello"}; // 推导出 std::optional
    assert(o4.has_value());
    assert(*o4 == "hello");

    std::optional o5{"hello"s}; // 推导出 std::optional
    assert(o5.has_value());
    assert(*o5 == "hello");

    // 用多个参数初始化可选对象
    std::optionaldouble>> o6{std::in_place, 3.0, 4.0};
    assert(o6.has_value());
    assert(o6->real() == 3.0 && o6->imag() == 4.0);

    // 使用 std::make_optional
    auto o13 = std::make_optional(3.0); // std::optional
    assert(o13.has_value());
    assert(*o13 == 3.0);

    auto o14 = std::make_optional("hello"); // std::optional
    assert(o14.has_value());
    assert(*o14 == "hello");

    auto o15 = std::make_optionaldouble>>(3.0, 4.0);
    assert(o15.has_value());
    assert(o15->real() == 3.0 && o15->imag() == 4.0);
}

// 示例 2：访问值
void access_optional_value() {
    std::optionalint, std::string>> o{std::make_pair(42, "hello")};
    assert(o.has_value());
    assert(o->first == 42);
    assert(o->second == "hello");

    std::optional o2{"hello"};
    assert(o2.has_value());
    assert(*o2 == "hello");

    // 当没有值时访问会导致未定义行为
    o2 = std::nullopt;
    assert(!o2.has_value());
    // std::cout << *o2 << std::endl; // 未定义行为
}

// 示例 3：使用 value_or
void use_value_or() {
    std::optional o{"hello"};
    std::cout << o.value_or("NO VALUE") << std::endl; // 输出 "hello"

    o = std::nullopt;
    std::cout << o.value_or("NO VALUE") << std::endl; // 输出 "NO VALUE"
}

// 示例 4：比较
void compare_optionals() {
    std::optional<int> o0;
    std::optional<int> o1{42};
    assert(o0 == std::nullopt);
    assert(!(o0 == 42));
    assert(o0 < 42);
    assert(!(o0 > 42));
    assert(o1 == 42);
    assert(o0 < o1);
    assert(!(o0 > o1));

    std::optional<unsigned> uo;
    assert(uo < 0);
    assert(uo < -42);

    std::optional<bool> bo;
    assert(bo < false);

    std::optional<int> o2{42};
    std::optional<double> o3{42.0};
    assert(o2 == 42);
    assert(o3 == 42);
    assert(o2 == o3);
}

// 示例 5：修改值
void modify_optional_value() {
    std::optionaldouble>> o; // 没有值
    std::optional<int> ox{77}; // optional，值为77
    o = 42; // 值变为 complex(42.0, 0.0)
    assert(o.has_value());
    assert(o->real() == 42.0 && o->imag() == 0.0);

    o = std::complex<double>{9.9, 4.4}; // 值变为 complex(9.9, 4.4)
    assert(o.has_value());
    assert(o->real() == 9.9 && o->imag() == 4.4);

    o = ox; // OK，因为 int 转换为 complex
    assert(o.has_value());
    assert(o->real() == 77.0 && o->imag() == 0.0);

    o = std::nullopt; // o 不再有值
    assert(!o.has_value());

    o.emplace(5.5, 7.7); // 值变为 complex(5.5, 7.7)
    assert(o.has_value());
    assert(o->real() == 5.5 && o->imag() == 7.7);

    o.reset(); // o 不再有值
    assert(!o.has_value());

    o = std::complex<double>{88.0, 0.0}; // OK：值变为 complex(88.0, 0.0)
    assert(o.has_value());
    assert(o->real() == 88.0 && o->imag() == 0.0);

    o = std::complex<double>{1.2, 3.4}; // OK：值变为 complex(1.2, 3.4)
    assert(o.has_value());
    assert(o->real() == 1.2 && o->imag() == 3.4);
}

// 示例 6：使用 lambda 初始化 set
void initialize_set_with_lambda() {
    auto sc = [](int x, int y) {
        return std::abs(x) < std::abs(y);
    };

    std::optionalint, decltype(sc)>> o8{std::in_place,
                                                   std::initializer_list<int>{4, 8, -7, -2, 0, 5},
                                                   sc};
    assert(o8.has_value());
    assert(o8->size() == 6);
}

int main() {
    construct_optional();
    access_optional_value();
    use_value_or();
    compare_optionals();
    modify_optional_value();
    initialize_set_with_lambda();
    return 0;
}

```

6\.注意
（1）value()和 value\_or()
value()和 value\_or()之间有一个需要考虑的差异：4 value\_or()返回值，而 value()返回引用。这意味着如下调用：
std::cout \<\< middle.value\_or("");
和：
std::cout \<\< o.value\_or("fallback");
都会暗中分配内存，而 value()永远不会。
然而，当在临时对象 (rvalue)上调用 value\_or()时，将会移动走内含对象的值并以值返回，而不是调用拷贝函数构造。这是唯一一种能让 value\_or()适用于 move\-only的类型的方法，因为在左值 (lvalue)上调用的 value\_or()的重载版本需要内含对象可以拷贝。
因此，上面例子中效率最高的实现方式是：
std::cout \<\< o ? o‐\>c\_str() : "fallback";
而不是：
std::cout \<\< o.value\_or("fallback");
value\_or()是一个能够更清晰地表达意图的接口，但开销可能会更大一点。
（2）bool 类型或原生指针的可选对象
将可选对象用作 bool值时使用比较运算符会有特殊的语义。如果内含类型是 bool或者指针类型的话这可能导致令人迷惑的行为。例如：
std::optional ob{false}; // 值 为false
if (!ob) ... // 返 回false
if (ob \=\= false) ... // 返 回true
std::optional op{nullptr};
if (!op) ... // 返 回false
if (op \=\= nullptr) ... // 返 回true


 本博客参考[豆荚加速器](https://yirou.org)。转载请注明出处！
