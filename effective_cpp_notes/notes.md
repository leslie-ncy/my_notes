## 1. 视C++为联合体语言
可将C++视为：
- **C-like**（C语言）：区块、语句、预处理器、内置数据类型、数组、指针等；
- **Object-Oriented C++**（面向对象）：构造析构函数、封装继承多态、动态绑定等；
- **Template C++**（泛型编程）：
- **STL**：容器、迭代器、算法、函数对象等。

内置类型（C-like）值传递高效；   
自定义类（Object-C++）`const`引用传递更好；  
Template C++ `const`引用传递更好;    
STL迭代器和函数对象建立在指针上，因此值传递更好
## 2. 尽量以`const`, `enum`, `inline`替换`#define`
## 3. 尽可能使用`const`
![??](./3const.png)
- 令函数返回常量值，例如：
```C++
class A { ... };
const A operator*(const A& lhs, const B& rhs);
```
否则将会出现错误：
```C++
// (a * b) = c 给乘积赋值
A a, b, c;
if (a * b = c) { ... } 
```
- 将`const`用于成员函数，两点好处：
    - 可直接得知函数是否可以直接改动对象内容
    - 能操作`const`对象，例如：
 ```C++
class A {
    const A& operator[](int i) const;
    A& operator[](int i);
};

A a, b;
std::cout << a[0]; // call non-const func
a[0] = b; // √ 若返回类型非引用，则该句不成立

const A ca;
std::cout << ca[0]; // call const func
ca[0] = b; // ×
```
- 可以在`non-const`成员函数中利用两次转型调用`const`成员函数：
```C++
class A {
    const A& operator[](int i) const {
        ...
        return A;
    }

    A& operator[](int i) {
        return const_cast<A&> (
            static_cast<const A&>(*this)[i]
        );
    }
};
```
但不可以在`const`成员函数中调用`non-const`成员函数，因为存在改变对象的可能

## 4. 使用前确定对象被初始化
![?](4initialization.png)

## 5. C++自动编写的默认函数
1. 默认构造函数  
2. 默认拷贝构造函数  
3. 默认拷贝赋值函数
   
## 6. 明确拒绝不想要的默认函数
1. 可采用private声明法阻止自动生成

## 7. 为多态基类声明virtual析构函数
1. 若多态基类不声明virtual析构函数，则：当有一个基类指针指向派生类对象时，通过delete该指针只能析构基类部分，出现“局部销毁”。

## 8. 别让异常跑出析构函数
1. 析构函数应该吞下异常或结束程序。否则可能发生未知错误
2. 可以通过一个接口类的普通方法（函数）抛出异常

## 9. 在构造和析构期间不要调用virtual函数

## 10. 令operator=返回一个reference to *this

## 11. operator=中的“自我赋值”
1. 对比两个地址是否一致
2. 合理安排命令语句（例如保存原来的，然后先new再delete）
3. copy-and-swap方法

## 12. 复制对象时切勿忘记每个成分
1. 注意在派生类中的基类部分的复制


## 13. 以对象管理资源
1. 为防止资源泄漏，请使用RAII（资源管理机制）对象
2. 智能指针就是该条款例子

## 14. 在资源管理类中注意coping行为
1. 常见的RAII class的copying行为：抑制copying || 引用计数法

## 15. RAII类中提供对原始资源的访问
1. 可以提供get()函数or隐式类型转换
2. 隐式类型转换可能发生错误

## 16. new和delete前后一致（是否加[]）

## 17. 以独立语句将new的对象放入智能指针
1. 例如调用函数时不能在调用时传入`std::shared_ptr<A>(new A)`，否则若有异常发生则new的资源不会被放入智能指针，就很可能不会被及时释放

## 18. 让接口容易被正确使用、不易被误用
1. 容易被正确使用：保持接口一致性（如STL各类容器的方法一样）、与内置类型行为兼容
2. 不易被误用：消除用户资源管理责任、束缚对象值、建立新类型以限制类型

## 19. 设计class需要考虑的n个问题

## 20. 对于非内置类型，尽量使用const引用传递

## 21. 函数返回对象时，轻易不能返回reference
1. 绝对不要返回
   - pointer或reference指向的local stack对象
   - reference指向的heap-allocated对象
   - pointer或reference指向的local static对象

## 22. 将成员变量声明为private
1. 将成员变量封装的原因：
   - 语法一致性：public接口内的每个东西都是函数
   - 读写权限控制
   - 成员变量的封装性 与 它改变时造成的代码破坏量 成反比
2. protected 不比 public更具封装性

## 23. 以non-member、non-friend函数替换member函数
1. 原因：non-member、non-friend函数的封装性比后者高（不能访问类内的private等）
2. 强调是在member和non-member、non-friend之间抉择
3. 可采用工具类的方式。而C++的常用做法是命名空间，将类和non-member、non-friend函数置于同一命名空间。（命名空间可以跨越多个源码文件而class不同）

## 24. 若函数参数（包括this）可能需要类型转换，则需要采用non-member函数
- 只适用与C++面向对象编程，不适用于泛型编程

## 25. swap()函数
1. swap函数有三类：①non-member ②member ③std::swap。其中STL容器的std::swap()特化版本就是通过调用其member swap()实现的。
2. 如果需要改善默认swap()函数（意味着使用了pimpl(pointer to inplementation)手法）:
   - 提供一个public member swap() 让它高效置换。且该函数不能抛出任何异常
   - 在该class或template所在命名空间内提供一个non-member swap()，令它调用member swap()
   - 若你正编写一个class而非class template，则可为该class特化std::swap()，并令它调用member swap()；若是class template，则没办法特化，而且没办法重载（因为STL不允许增加template）
3. 当使用swap()函数时，先使用`using std::swap;`语句，再调用不加std的swap()函数
   ```C++
    template<typename T>
    void doSomething(T& obj1, T& obj2) {
        using std::swap;
        swap(obj1, obj2);
    }
   ```
   原因：通过using让std::swap()在定义域曝光，然后调用swap()时编译器为我们挑选最合适的函数。   
   特化版std::swap() > non-member swap() > default std::swap()
4. member swap()不应抛出任何异常。高效swap()往往是对内置类型的置换，这种置换不会抛出异常


