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

## 26. 尽可能延后变量定义式的出现

## 27. 尽量少做转型
1. 提倡使用C++新式转型，容易辨识，且功能分开：
   - static_cast: 
   - const_cast: 常量去除
   - dynamic_cast: 安全向下转型（成本较大）
   - reinterpret_cast: 低级转型
2. 如果可以，请避免转型。尤其是dynamic_cast
3. 如果转型必要，可以试着将其隐藏在函数后

## 28. 避免返回handles(引用、指针、迭代器)
1. 增加封装性，使const方法更像个const
2. 返回const-reference的函数可能会发生 悬空指针

## 29. 异常安全
1. 异常安全函数即使发生异常也不会泄露资源或破坏任何数据结构。分为三种可能的保证：
   - 基本承诺：若异常抛出，则程序内任何事物仍然保持在有效状态下
   - 强烈保证：如非完全成功，程序就会恢复到调用函数前的状态
   - 不抛掷（nothrow）保证
2. “强烈保证”往往能够以copy-and-swap实现。但该保证并非对所有情况都适用
3. 函数提供的保证以其调用的函数保证的最弱者为标准

## 30. inline函数
1. 将inline函数用在小型频繁被调用的函数上。避免调试麻烦和潜在的代码膨胀问题
2. inline难以调试，因此调试模式多使用非inline（现在呢？）

## 31. 将文件间的编译依存关系最小化
1. 编译依存性最小化的一般构想：相依于声明式，而非定义式；手段分别是Handle classes和Interface classes
2. 程序库头文件应仅有声明式

----------
# 六、继承与面向对象设计
## 32. public继承意味着is-a(是一个or一种)
1. 适用于base classes的每件事也一定都适用于derived classes。
2. 每个derived对象也都是一个base对象，反之不是。
3. 正例：人和学生；鸟和企鹅，不考虑飞行；
4. 反例：鸟和企鹅，飞行；矩形和正方形，增加面积

## 33. 避免遮掩继承
1. derived内与base内相同的名称（无论函数还是变量）都会遮掩class内的名称，不符合public “is-a”的设计原则
2. 可用using声明 或是 转交函数

## 34. 区分 接口继承 和 实现继承
1. 成员函数可以分为三类：
   - pure virtual 纯虚函数：接口继承，强制derived实现接口
   - impure virtual （非纯）虚函数：接口继承和缺省实现继承
   - non-virtural 非虚函数：接口继承和强制性的实现继承
2. 通过在base class中定义pure virtual，既能强制dirived实现接口又能通过调用base的实现减少代码重复量

## 35. 考虑virtual函数以外的其他选择
1. 替代方案包括NVI（non-virtual interface）方法和Strategy设计模式的多种形式。NVI本身是一种特殊的Template Method设计模式
2. Strategy设计模式：
   - 通过将函数指针作为变量实现
   - 通过tr1::function函数对象实现
   - 古典strategy，将virtual函数替换为另一个继承体系的virtual函数

## 36. 禁止重新定义继承而来的non-virtual函数

## 37. 禁止重新定义继承而来的缺省参数值（函数默认参数）
1. 如果是virtual函数说明要动态绑定，而函数默认参数是静态绑定

## 38. 通过复合（组合或聚合）构造“has-a”和“根据某物实现”的关系
1. 复合（composition）与public继承的“is-a”完全不同
2. 在应用领域，意味“has-a”;在实现方面，意味“is-implemented-in-terms-of”（根据某物实现）

## 39. 慎用private继承
1. private继承意味着“is-implemented-in-terms-of”，但通常不如复合合适
2. 但当derived需要访问protected成员或重写virtual函数时，就需要private继承
3. 若继承空类（无成员变量），private继承的好处是使该空类不占空间，复合由于将依赖的类作为成员变量，需要占1字节空间。

## 40. 慎用多重继承
1. 多重继承可能导致歧义性，比如两个父类拥有相同名称的函数。
2. “钻石型多呈继承”：继承的不同base classes派生自相同的父类 或是 继承体系中某个base和某个derived之间有一条或以上的相同路线。这样会产生的问题是：若被多次继承的base中有个成员变量，则在derived中有多少这个成员变量？
   - 缺省做法是复制，即两个都有
   - 通过virtual继承可以实现derived对象中只有一个成员变量。
3. virtual继承会增加编译和运行成本，但如果virtual base class不带任何数据，则将是最具实用价值的情况
4. 多重继承的正当用途：比如，public继承某个Interface和private继承某个协助实现的class

-----------
# 七、模板与泛型编程

--------------
# 八、定制new和delete
