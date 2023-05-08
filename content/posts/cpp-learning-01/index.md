---
title: "C++ 类与类继承"
date: "2018-03-29T20:28:00+08:00"
categories:
- c-cpp
tags:
- c-cpp
- notes
---

### 构造函数与析构函数

#### 构造函数

- 普通构造函数
    
    普通构造函数用来创建对象，如果该类位定义构造函数，则编译器会生成一个默认的空参数构造函数。

- 复制构造函数

    复制构造函数，顾名思义，用于复制一个对象，定义如下:

    ```cpp
    Test(const Test &t);
    ```

- 转换构造函数

    转换构造函数，将某个类型转为这个类的类型，定义如下:

    ```cpp
    Test(const int &a);
    ```

    使用转换构造函数可以实现类型的转换

    ```cpp
    // 调用转换构造函数，实例化一个 Test 对象
    Test a = 42;
    ```

    如果不希望构造函数被用于隐式转换，可用 `explicit` 关键字阻止隐式转换。

使用类的默认构造函数创建对象时，可能会出现下面的问题:

<!-- more -->

```cpp
// 创建一个 Test 类型的变量 a
Test test;

// 声明了一个返回值为 Test 的函数
// 在 调用 a.fun() 时，会报错: 
// error: request for member 'fun' in 'a', which is of non-class type 'Test()'
Test a();

// 动态创建对象
Test *tp = new Test();
// 销毁动态创建的对象
delete tp;
```

#### 析构函数

正好与构造函数相反，用来在该对象被销毁时，释放资源。定义如下:

```cpp
~Test();
```

非动态创建的对象会在合适的时刻由编译器自动调用析构函数销毁该对象，而由 `new` 关键字创建的对象则需要手动使用 `delete` 销毁对象释放内存。


### 运算符重载

C++ 支持运算符重载，可以实现更加直观的语法，比如分数对象与分数对象的四则运算，或者在 `std::cout <<` 后直接跟着一个对象(重载 `<<`)。

定义如下:

```
<return_type> operator<op>([param1, ...]);
```


### 类的继承

C++ 属性函数的权限修饰符有三种:

1. public

    公开的，在任何地方都可以访问

2. private 

    私有的，只有在该类内可以访问，在子类中不可访问

3. protected

    保护的，只有在类内和子类内可访问


C++ 类的继承有三种模式:

1. public

    在子类外可以访问父类的所有 public 属性函数；
    在子类内可以访问所有非 private 的父类属性函数。

2. private

    与 public 相反，子类外不可以访问任何父类的属性及函数；
    private 使父类所有成员在子类中变为 private。

3. protected

    使父类所有 public 的成员变为 protected，类外不可访问。


|           | public 继承 | private 继承 | protected 继承 |
| :-------: | :---------: | :----------: | :------------: |
| public    | public      | private      | protected      |
| private   | -           | -            | -              |
| protected | protected   | private      | protected      |

 *父类的 private 属性子类永远都无权访问*


#### 虚函数与纯虚函数

#### 虚函数

看如下的代码

```cpp
class Base {
public:
    void fun() {
        cout << "from base" << endl;
    }
};

class Test : public Base {
public:
    void fun() {
        cout << "from test" << endl;
    }
};
int main() {
    Test t;
    Base &b = t;
    b.fun();
}
```

如果熟悉 Java 的话，上述代码应该会输出 `from test`，
但在 C++ 中，结果是 `from base`。

如果希望它自动调用这个对象所属类对应的函数，则需要使用虚函数

```cpp
class Base {
public:
    virtual void fun() {
        cout << "from base" << endl;
    }
};
```

为什么呢？

程序调用函数时，编译器负责将这个函数与对应的代码块联系起来，在 C++ 中，类的继承多态导致编译器无法在编译时期确定应该调用哪个函数，所以只能简单的根据 引用/指针 类型来决定调用哪个函数。这被称为 **静态联编**；

当这个函数被声明为 `virtual` 后，编译器将使用 **动态联编**，在运行时期决定调用哪个函数。

编译器在需要动态联编的类中添加了一个隐藏的成员，指向函数的指针数组，称为 `虚函数表`。其中存储了该类的所有的虚函数的地址。

当子类继承父类后，子类也会有一个虚函数表，如果子类没有为父类的虚函数提供新的定义，则该虚函数还指向父类，否则，将指向子类的函数；当子类定义了新的虚函数，这个函数的地址也会被加入到虚函数表中。

在父类中声明为 `virtual` 的函数，在其所有子类中都将是虚函数。所以，如果一个类中的某个函数可以被子类重写，最好将这个函数声明为虚函数。

**另外，类的构造函数不可声明虚函数，但析构函数可以，并且最好将析构函数声明为虚函数，否则，当出现上述代码的情况时，子类的析构函数将不会被调用。**

#### 纯虚函数

Java 中有抽象类的概念，就是不能直接实例化的类，在 C++ 中将一个类中的一个或多个函数声明为纯虚函数，也可实现相同的效果。当一个函数在父类中被声明为纯虚函数时，表明在子类中必须定义这个函数。

纯虚函数的定义方法如下

```cpp
class BaseClass {
public:
    virtual void fun() = 0;
};
```


#### 继承中需要注意的问题

当在子类中声明了一个与父类中某个函数同名的函数（参数不同）时，会导致父类的同名函数被覆盖掉。

```cpp
class Base {
public:
    void fun(int a) {
        cout << "base fun: " << a << endl;
    }
};
class Test {
public:
    void fun(string &str) {
        cout << "test fun: " << str << endl;
    }
};
int main() {
    Test t;
    // no known conversion for argument 1 from 'int' to
    // 'std::__cxx11::string& {aka std::__cxx11::basic_string<char>&}'
    t.fun(123);
    return 0;
}
```

重新定义不会生成两个重载的版本，而是隐藏了父类的同名函数。

所以

- 当重新定义继承的函数时，应该保证函数的签名完全相同，但如果在父类中返回类型是父类的 引用/指针 则可以修改为子类的 引用/指针

- 如果父类中的被重新定义的函数在父类中有重载，则在子类中应该尽量重新定义所有版本


当一个类中使用了动态内存分配后，则最好重写析构函数(用来释放内存)、复制构造函数(深拷贝) 和 重载赋值运算符(深拷贝)。

如果这个类继承自另一个类，在析构函数和复制构造函数中，编译器会自动调用其父类对应的函数，但父类的复制运算符不会被自动调用，需要我们在子类中手动调用。

```cpp
class Child {
public:
    Child &operator=(const Child &ch) {
        // ...
        // 直接使用 operator=(ch); 将会进入递归死循环。
        Parent::operator=(ch);
        // ...
    }
};
```


### 多重继承

C++ 支持多重继承，即一个子类可以继承自多个父类。

```cpp
class Child : public ParentA : public ParentB {
    // ...
};
```

这会导致一个问题

```cpp
class Base {
public:
    void fun() {
        cout << "base" << endl;
    }
};
class ChildA : public Base {
};
class ChildB : public Base {
};
class Grandchild : public ChildA, public ChildB {
};
```

可以看到，类 `Grandchild` 同时继承自 `ChildA` 和 `ChildB`，而它们俩又继承自 `Base`，那么当我们直接调用 `GrandChild::fun()` 时会发生什么呢？

```
error: request for member 'fun' is ambiguous
```

因为 `Grandchild` 继承自“两个” `Base`，一个来自 `ChildA`，另一个来自 `ChildB`，所以出现了二义性，编译器不知道该调用哪个，所以报错。

解决办法是，引入 **虚基类**，使 `ChildA` 和 `ChildB` **虚继承** 于 `Base`。

```cpp 
class ChildA : virtual public Base {
};
// ChildB 同上
```

这样，当一个子类有多个相同父类时，它们中的所有从**虚继承**来的父类将会共用一个父类实例。

