# c++primer学习

## 第15章 面向对象程序设计

#### 15.1 OPP：概述

​    面向对象程序设计（object-oriented-programming）的核心思想是数据抽象、继承和动态绑定。

###### 1.继承

​    通过继承联系在一起的类构成一种层次关系。通常在层次关系的根部有一个基类，其他类成为派生类。

​    在c++语言中，基类将类型相关的函数与派生类不做改变直接继承的函数区分对待。对于某些函数，基类希望它的派生类各自定义适合自身的版本，此时基类就将这些函数声明为虚函数：

```c++
class Quote{
public:
    std::string isbn() const;
    virtual double net_price(std::size_t n) const;
}
```

​    派生类必须通过使用类派生列表明确指出他是从哪个（哪些）基类继承来的：

```c++
class Bulk_quote : public Quote{
public:
    double net_price(std::size_t n) const override;
}
```

​    派生类必须在其内部对所有重新定义的虚函数进行声明。c++11允许派生类显式地注明他将使用哪个成员函数改写基类的虚函数，具体措施是在该函数的形参列表之后增加一个override关键字。

###### 2.动态绑定

​    通过使用动态绑定，我们能用同一段代码分别处理Quote和Bulk_quote的对象：

```c++
double print_total(ostream& os, const Quote& item, size_t n){
  	//根据item形参的对象类型调用
    double ret = item.net_price(n);
    os << "ISBN: " << item.isbn() << " # sold: " << n << " total due: " << ret << endl;
    return ret;
}
```

​    **ps：在c++中，当我们使用基类的引用（或指针）调用一个虚函数时将发生动态绑定。**

#### 15.2 定义基类和派生类

##### 1.定义基类

​    基类通常都应该定义一个虚析构函数，即使该函数不执行任何实际操作也是如此：

```c++
class Quote{
public:
    Quote() = default;
    Quote(const std::string& book, double sales_price) : bookNo(book), price(sales_price) {}
    std::string isbn() const { return bookNo; }
    virtual double net_price(std::size_t n) const { return n * price; }
    virtual ~Quote() = default;
private:
    std::string bookNo;
protected:
    double price = 0.0;
}
```

###### 1. 成员函数与继承

​    派生类有时需要对基类的成员提供自己的新定义以覆盖从基类继承而来的旧定义。

​    c++中基类必须将他的两种成员函数区分开来：一种是希望派生类进行覆盖的函数；另一种是基类希望派生类直接继承而不改变的函数。对于前者基类通常将其定义为虚函数。基类通过在成员函数的声明语句前加关键字virtual来使该函数执行动态绑定。

​    任何构造函数之外的非静态函数都可是虚函数，如果基类把一个函数声明为虚函数，则该函数在派生类中隐式地也是虚函数。成员函数如果没有被声明成虚函数，则其析构过程发生在编译时而并非运行时。

###### 2.访问控制与继承

​    派生类能访问公有成员，而不能访问私有成员。但是有时基类希望它的派生类有权访问该成员同时禁止其他用户访问，我们这是就要用protected。

##### 2.定义派生类

​    派生类必须通过使用类派生列表明确指出他是从哪个（哪些）基类继承而来的：

```c++
//public private protected
class Bulk_quote : public Quote{
public:
    Bulk_quote() = default;
    Bulk_quote(const std::string&, double, std::size_t, double);
    double net_price(std::size_t) const override;
private:
    std::size_t min_qty = 0;
    double discount = 0.0;
}
```

​    大多数类都只继承自一个类，这种形式称为单继承。

###### 1.派生类中的虚函数

   派生类可以在他覆盖的函数前使用virtual关键字，但不是非得这么做。c++11中直接用关键字override来表示覆盖了函数。

###### 2.派生类对象及派生类向基类的类型转换

​    在一个对象中，继承自基类的部分和派生类自定义的部分不一定是连续存储的。

​    因为在派生类对象中含有与其基类对象的组成部分，所以我们能把派生类的对象当成基类对象来使用，而且我们也能将基类的指针或引用绑定到派生类对象中的基类部分上：

```c++
Quote item;        //基类对象
Bulk_quote bulk;   //派生类对象
Quote* p = &item;  //p指向Quote对象
p = &bulk;         //p指向bulk的Quote部分
Quote& r = bulk;   //r绑定到bulk的Quote部分
```

###### 3.派生类构造函数

​    派生类对象的基类部分与派生类对象自己的数据成员都是在构造函数的初始化阶段执行初始化操作。首先初始化基类部分，然后按照声明的顺序依次初始化派生类的成员。

###### 1.派生类使用基类的成员

​    派生类可以访问基类的公有成员和保护成员

###### 2.继承与静态成员

​    如果基类定义了一个静态成员，则在整个继承体系中只存在该成员的唯一定义。静态成员遵循通用的访问控制规则，如果基类中的成员使private，则派生类无权访问，protected和private类似。

###### 3.派生类的声明

​    派生类的声明中包含类名但不是包含他的派生列表：

```c++
class Bulk_quote : public Quote;  //错误
class Bulk_quote;                 //正确
```

###### 4.被用作基类的类

​    如果我们想使用某个类作为基类，则该类必须已经定义而非声明。

###### 5.防止继承的发生

​    有时我们会定义一种类，我们不希望其他类继承他，或者不想考虑它是否适合作为一个基类，c++11中使用final来防止继承：

```c++
class NoDerived final {};    //Boderived不能作为基类
class Base {};            
class Last final : Base {};  //Last不能作为基类
class Bad : Noderived {};    //错误
class Bad2 : Last {};        //错误
```

##### 3.类型转换与继承

​    通常情况下，如果我们像把引用或指针绑定到一个对象上，则引用或指针的类型应与对象的类型一致，或者对象的类型含有一个可接受const类型转换规则。存在继承关系的类是一个例外：我们可以将基类的指针或引用绑定到派生类对象上。

​    **ps：和内置指针一样，智能指针类也支持派生类向基类的类型转换，这意味着我们可以将一个派生类对象的指针存储在一个基类的智能指针内。**

###### 1.静态类型与动态类型

​    当我们使用存在继承关系的类型时，必须将一个变量或其他表达式的静态类型与该表达式表示对象的动态类型区分开。表达式的静态类型在编译时总是出错的，他是变量声明时的类型或表达式生成的类型；动态类型则是变量或表达式的内存中的对象的类型。动态类型运行时可知。

​    **ps：如果表达式既不是指针也不是引用，则它的动态类型和静态类型一致。**

###### 2.不存在从基类向派生类的隐式类型转换

​    之所以存在派生类向基类的类型转换是因为每个派生类都包含一个基类部分。

```c++
Quote base;
Bulk_quote* bulkP = &base;   //错误
Bulk_quote& bulkRef = base;  //错误
```

​    除此之外还有一种情况显得有点特别，即使一个基类指针或引用绑定在一个派生类对象上，我们也不能执行从基类向派生类的转换：

```c++
Bulk_quote bulk;
Quote* itemP = &bulk;       //正确
Bulk_quote* bulkP = itemP;  //错误，不能从基类转换为派生类
```

​    如果在基类中含有一个或多个虚函数，我们可以使用dynamic_cast请求一个类型转换；如果我们已知某个基类向派生类转换是安全的，则我们可以使用static_cast来强制覆盖掉编译器的检查工作。

###### 3.。。。在对象之间不存在类型转换

​    派生类向基类自动转换类型只对指针或引用类型有效，在派生类类型和基类类型之间不存在这种转换。注意，当我们初始化或赋值一个类类型的对象时，实际上是在调用某个函数：

```c++
Bulk_quote bulk;
Quote item(bulk);  //使用Quote::Quote(const Quote&)构造函数
item = bulk;       //使用Quote::operator=(const Quote&)拷贝赋值函数
```

​    上述过程中会忽略Bulk_quote部分。

#### 15.3 虚函数

​    我们必须为每一个虚函数都提供定义，而不管他是否被用到了，因为连编译器都不确定会用哪个函数。

###### 1.对虚函数的调用可能在运行时才被解析

​    当某个虚函数通过指针或引用调用时，编译器产生的代码直到运行时才能确定应该调用哪个版本的函数。被调用的函数是与绑定到指针或引用上的对象的动态类型相匹配的那一个。

###### 2.派生类中的虚函数

​    当我们在派生类中覆盖了某个虚函数，则它的形参类型必须与被覆盖的基类函数完全一样。同样，返回类型也要匹配。有一个例外，就是当类的虚函数返回类型是类本身的指针或引用时，上述规则无效。

###### 3.final和override说明符

​    我们使用override关键字来说明派生类中的虚函数，如果使用override标记了某个函数，但函数没有覆盖已存在的虚函数，此时编译器会报错：

```c++
struct B{
   virtual void f1(int) const;
   virtual void f2();
   void f3();
};
struct D1 : B{
    void f1(int) const override;  //正确
    void f2(int) override;        //错误：B中没有f2(int)的函数
    void f3() override;           //错误：f3不是虚函数
    void f4() override;           //错误：B没有名为f4的函数
}
```

​    我们还能把某个函数指定为final，如果我们已经把函数定义为final了，则之后任何尝试覆盖该函数的操作都将引发错误：

```c++
struct D2 : B{
    void f1(int) const final;
};
struct D3 : D2{
    void f2();           //正确：覆盖D2从B继承来的f2()
    void f1(int) const;  //错误：D2中f1(int)为final，不能继承
}
```

​    final和override说明符出现在形参列表（包括任何const或引用修饰）以及尾置返回类型->type之后。

###### 4.虚函数与默认实参

​    和其他函数一样，虚函数也有默认实参。如果我们通过基类的引用或指针调用函数，则使用基类中定义的默认实参。即使实际运行的是派生类中的函数版本也是如此。此时，传入派生类函数的将是基类函数定义的默认实参。

   **ps：如果虚函数使用默认实参，则基类和派生类定义的默认实参最好一致。**

###### 5.回避虚函数的机制

​    在某些情况下我们希望对虚函数的调用不要动态绑定，而是强迫执行虚函数的某个特定版本。使用作用域运算符可以实现这个目的：

```c++
double undiscounted = baseP->Quote::net_price(42);
```

​    **ps：通常情况下，只有成员函数（或友元）中的代码才需要使用作用域运算符来回避虚函数的机制。**

#### 15.4 抽象基类

###### 1.纯虚函数

​    和普通函数不一样，一个纯虚函数无须定义。我们通过在函数体的位置（即在声明语句的分号之前）书写=0就可以将一个虚函数说明为纯虚函数。其中，=0只能出现类内部的虚函数声明语句处：

```c++
class Disc_quote : public Quote{
public:
    Disc_quote() = default;
    Disc_quote(const std::string& book, double price, std::size_t qty, double disc) : Quote(book, price), quantity(qty), discount(disc) {}
    double net_price(std::size_t) const = 0;
protected:
    std::size_t quantity = 0;  //折扣适用的购买量
    double discount = 0.0;     //表示折扣的小数值
}
```

​    值得注意的是，我们可以为纯虚函数提供定义，不过函数题必须定义在类的外部。也就是说我们不能在类的内部为一个=0的函数提供函数体。

###### 2.含有纯虚函数的类是抽象基类

​    含有（或未经覆盖直接继承）纯虚函数的类是抽象基类。抽象类负责定义接口，而后续的其他类可以覆盖该接口。我们不能（直接）创建一个抽象基类的对象。

###### 3.派生类构造函数只能初始化他的直接基类

```c++
class Bulk_quote : public Disc_quote{
public:
    Bulk_quote() = default;
    Bulk_quote(const std::string& book, double price, std::size_t qty, double disc) : Disc_quote(book, price, qty, disc) {}
    double net_price(std::size_t) const override;
};
```

#### 15.5 访问控制与继承

​    每个类还分别控制着其成员对于派生类来说是否可访问。

###### 1.受保护的成员

​    protected说明符可以看作是private和public中和后的产物：

-   和私有成员类似，受保护的成员对于类的用户来说是不可访问的
-   和公有成员类似，受保护的成员对于派生类的成员和友元来说时刻访问的
-   派生类的成员或友元只能通过派生类对象来访问基类的受保护成员。派生类对于一个基类对象中的受保护成员没有任何直接访问特权

###### 2.共有、私有和受保护继承

​    某个类对其继承而来的成员的访问权限受两个因素影响：一是在基类中该成员的访问说明符，二是在派生类的派生列表中的访问说明符。

```c++
class Base{
public:
    void pub_mem();
private:
    char priv_mem();
protected:
    int prot_mem();
};
struct Pub_Derv : public Base{
    //正确
    int f() { return prot_mem; }
    //错误
    char g() { return priv_mem; }
};
//private不影响派生类的访问权限
struct Priv_Derv : private Base{
    int f1() const { return prot_mem; }
};
```

​    派生访问说明符对派生类成员(以及友元)能否访问其直接基类的成员没有什么影。对基类成员访问权限只与基类中的访问说明符有关。派生访问说明符的目的是控制派生类用户（包括派生类的派生来在内）对于基类成员的访问权限：

```c++
Pub_Derv d1;
Priv_Derv d2;
d1.pub_mem();  //正确：pub_mem在派生类中是public的
d2.pub_mem();  //错误：pub_mem在派生类中是private的
```

​    派生访问说明符还可以控制继承自派生类的新类的访问权限：

```c++
struct Derived_from_Public : public Pub_Derv{
    //正确
    int use_base() { return prot_mem; }
};
struct Derived_from_Private : public Priv_Derv{
    //错误，Base::prot_mem在Priv_Derv中是private的
    int use_base() { return prot_mem; }
}
```

###### 3.派生类向基类转换的可访问性

​    派生类向基类的转换是否可访问由使用该转换的代码决定，同时派生类的派生访问说明符也会有影响。假定D继承自B：

-   只有当D共有继承B时，用户代码才能使用派生类向基类的转换，两外两种继承不能转换
-   不论D怎么继承B，D的成员函数和友元都能使用派生类向基类的转换；派生类如果向其直接基类的类型转换对派生类的成员和友元来说永远是可访问的。
-   如果D继承B的方式是共有或保护，则D的派生类的成员和友元可以使用D向B的类型转换，繁殖不能

​    **ps：对于代码中的某个给定节点来说，如果基类的公有成员是可访问的，则派生类向向基类的类型转换也是可访问的；反之则不行。**

###### 4.友元与继承

​    就像友元关系不能传递一样，友元关系同样也不能继承。基类的友元在访问派生类成员时不具有特殊性，类似的，派生类的友元也不能随意访问基类成员。

###### 5.改变个别成员的可访问性

​    有时我们需要改变派生类继承的某个名字的访问级别，通过使用using声明：

```c++
class Base{
public:
    std::size_t size() const { return n; }
ptotected:
    std::size_t n;
};
class Derived : private Base{
public:
    //保持对象尺寸相关的成员的访问级别
    using Base::size;
protected:
    usint Base::n;
}
```

​    因为Derived是私有继承Base，所以继承的size()和n默认情况下是Derived的私有成员。然而我们使用using声明语句改变了这些成员的可访问性。改变之后，Derived的用户将可以使用size成员，而Derived的派生类将能使用n。

​    **ps：如果一条using声明语句出现在类的private部分，则该名字只能被类的成员和友元访问；如果using语句位于public部分，则类的所有用户都可以访问它；如果using位于protected部分，则该名字对成员、友元和派生类是可访问的。**

###### 6.默认的继承保护级别

​    默认情况下class关键字定义的派生类是私有继承，struct的派生类是公有继承。

#### 15.6 继承中的类作用域

​    当存在继承关系时，派生类的作用域嵌套在器基类的作用域之中。如果一个名字在派生类的作用域内无法解析，则编译器将继续在外层的基类作用域中寻找该名字的定义。

###### 1.在编译时进行名字查找

​    一个对象、引用或指针的静态类型决定了该对象的那些成员是可见的。

###### 2.名字冲突与继承

​    和其他作用域一样，派生类也能重用定义在其直接基类或间接基类中的名字，此时定义在内层作用域的名字将隐藏定义在外层作用域的名字。即派生类的成员将隐藏同名的基类成员。

###### 3.通过作用域运算符来使用隐藏的成员

​    我们可以通过作用域运算符来使用一个被隐藏的基类成员：

​    **ps：除了覆盖继承而来的虚函数，派生类最好不要重用其他定义在基类中的名字。**

###### 4.一如往常，名字查找优先于类型检查

​     如果派生类的成员与基类的某个成员同名，则派生类将在其作用域内隐藏该基类成员，即使派生类成员和基类成员的形参列表不一致，基类成员也仍然会被隐藏掉。

###### 5.虚函数与作用域

​    假如基类与派生类的虚函数接受的实参不同，则我们我i发通过基类的引用或指针调用派生类的虚函数了：

```c++
class Base{
public:
    virtual int fcn();
};
class D1 : public Base{
public:
    //隐藏基类中的fcn，这个fcn不是虚函数
    int fcn(int);       //形参列表与基类中的fcn不一致
    virtual void f2();  //是一个新的虚函数
};
class D2 : public D1{
public:
    int fcn(int);       //是一个非虚函数，隐藏了D1::fcn(int)
    int fcn();          //覆盖了Base的虚函数fcn
    void f2();          //覆盖了D1的虚函数f2
};
```

###### 6.通过基类调用隐藏的虚函数

```c++
Base bobj; D1 d1obj; D2 d2obj;
Base *bp1 = &bobj, *bp2 = &d1obj, *bp3 = &d2obj;
bp1->fcn();  //Base::fcn();
bp2->fcn();  //Base::fcn();
bp3->fcn();  //D2::fcn();
D1 *d1p = &bobj; D2 *d2p = &d2obj;
bp2->f2();  //错误 Base中无f2
d1p->f2();  //D1::f2();
d2p->f2();  //D2::f2();
```

###### 7.覆盖重载的函数

​    为重载的成员提供一条using声明语句，这样我们就无需覆盖基类中的每一个重载版本了，using声明语句指定一个名字而不指定形参列表，所以一条基类成员函数的using声明语句就可以把该函数的所有重载实例添加到派生类作用域中。

#### 15.7 构造函数与拷贝控制

##### 1.虚析构函数

​    当我们delete一个动态分配的对象的指针时将执行析构函数，如果该指针指向继承体系中的某个类型，则有可能出现指针的静态类型与被删除对象的动态类型不符情况。我们要通过在基类中将析构函数定义为虚析构以确保正确的析构函数版本。

```c++
class Quote{
public:
    ...
    //如果我们删除的是一个指向派生类的对象的基类指针，则需要虚析构函数
    virtual ~Quote () = default;
}
```

​    只要基类的析构函数是虚析构函数，我们就能确保当我们delete基类指针时将运行正确的析构函数版本。如果一个类需要析构函数，一般也会需要拷贝和赋值的操作，但虚析构并不是这样。

​    **ps：如果一个类定义了析构函数，即使它通过=default的形式使用了合成的版本，编译器也不会为这个类合成移动操作。**

##### 2.合成拷贝控制与继承

​    基类或派生类的合成拷贝控制成员的行为和其他合成的类似：对类本身的成员依次进行初始、赋值、销毁的操作。此外，这些合成的成员还负责使用直接基类中对应的操作对一个对象的直接基类部分进行初始化、赋值、销毁操作。

###### 1.派生类中删除的拷贝控制与基类的关系

​    某些定义基类的方式也可能造成派生类成员被定义为删除的函数：

-   如果基类中的默认构造函数、拷贝构造函数、拷贝赋值运算符或析构函数是被删除的函数或者不可访问，则派生类中对应的成员将是被删除的，原因是编译器不能使用基类成员来执行派生类对象基类部分的构造、赋值或销毁操作。
-   如果在基类中有一个不可访问或删除掉的析构函数，则派生类中合成的默认和拷贝构造函数将是被删除的，因为编译器无法销毁派生类对象的基类部分。
-   和过去一样，编译器将不会合成一个删除掉的移动操作。当我们使用=default 请求一个移动操作时，如果基类中的对应操作是删除的或不可访问的，那么派生类中该函数将是被删除的，原因是派生类对象的基类部分不可移动。同样，如果基类的析构函数是删除的或不可访问的，则派生类的移动构造函数也将是被删除的。

```c++
class B{
public:
    B();
    B(const B&) = delete;  //基类中的拷贝构造函数被定义为删除
};
class D : public B{
};
D d;                 //正确
D d2(d);             //错误：D的和合成拷贝构造函数是被删除的
D d3(std::move(d));  //错误：隐式地使用了拷贝构造函数
```

###### 2.移动操作与继承

​    因为基类缺少移动操作会阻止派生类拥有自己的合成移动，所以当我们确实需要执行移动操作时需要在基类中进行定义：

```c++
class Quote{
public:
    Quote() = default;
    Quote(const Quote&) = default;
    Quote(Quote&&) = default;
    Quote& operator=(const Quote&) = default;
    Quote& operator=(Quote&&) = default;
    virtual ~Quote() = default;
}
```

##### 3.派生类的拷贝控制成员

​    派生类的拷贝和移动构造函数在拷贝和移动自有成员的同时，也要拷贝和移动基类部分成员，类似的，派生类赋值运算符也必须维奇基类部分的成员赋值。

###### 1.定义派生类的拷贝或移动构造函数

​    当我们为派生了定义拷贝或移动构造函数时，我们通常使用对应的基类构造函数初始化对象的基类部分：

```c++
class Base{ };
class D : public Base{
public:
    D(const D& d) : Base(d) { }
    D(D&& d) : Base(std::move(d)) { }
}
```

​    **ps：在默认情况下，基类默认构造函数初始化派生类对象的基类部分。如果我们向拷贝（或移动）基类部分，则必须在派生类的构造函数初始值列表中显式地用基类的拷贝（或移动）构造函数。**

###### 2.派生类赋值运算符

​    和拷贝/移动构造函数一样，派生类的赋值运算符也必须显式地为其基类部分赋值：

```c++
//Base::operator=(const Base&)不会被自动调用
D& D::operator=(const D &rhs){
    Base::operator=(rhs);
    ...
    return *this;
}
```

###### 3.派生类析构函数

​    和构造函数与赋值运算符不同的是，派生类析构函数只负责销毁由派生类分配的资源：

```c++
class D : public Base{
public:
    ...
    //Base::~Base被自动调用执行
    ~D() { /*该处由用户定义清除派生类成员的操作*/ }
};
```

###### 4.在构造函数和析构函数中调用虚函数

​    如果构造函数或析构函数调用了某个虚函数，则我们应该执行与构造函数或析构函数所属类型对应的虚函数版本。

##### 4.继承的构造函数

​    在c++11中，派生类能够重用其直接类定义的构造函数。一个类只初始化他的直接基类，处于同样的原因，一个类也只继承其直接基类的构造函数。类不能继承默认、拷贝和移动构造函数。如果派生类没有直接定义这些构造函数，则编译将为派生类合成他们。

​    派生类继承基类构造函数的方式是提供了一条注明了（直接）基类名的using声明语句：

```c++
class Bulk_quote : public Disc_quote{
public:
    using Disc_quote::Disc_quote;  //继承Disc_quote的构造函数
    double net_price(std::size_t) const;
};
//using Disc_quote::Disc_quote; 等价以于：
Bulk_quote(const std::string &book, double price, std::size_t qty, double disc) : Disc_quote(book, price, qty, disc) {}
```

​    和普通的using声明不一样，一个构造函数的using声明不会改变该构造函数的访问级别：不管using声明出现在哪儿，基类的私有构造函数在派生类中还是一个私有构造函数，受保护的构造函数和共有构造函数也是同样规则。

​    而且，一个using声明语句不能指定explicit或constexpr：如果基类的构造函数是explicit或constexpr，则继承的构造函数也拥有相同的属性。

​    当一个基类构造函数含有默认实参时，这些实参并不会被继承，相反，派生类将获得多个继承的构造函数，其中每个构造函数分别省略掉一个含有默认实参的形参。

#### 15.8 容器与继承

​    当我们使用容器存放继承体系中的对象时，通常必须采取间接存储的方式。因为不允许在容器中保持不同类型的元素，所以我们不能把具有继承关系的多种类型的对象直接存放在容器当中。

```c++
vector<Quote> basket;
basket.push_back(Quote("0-201-82470-1", 50));
//正确，但是只能把对象的Quote部分拷贝给basket
basket.push_back(Bulk_quote("0-201-82470-1", 50, 10, .25));
```

​    当我们希望在容器中存放具有继承关系的对象时，我们实际上存放的通常是基类的指针（更好的选择是智能指针）：

```c++
vector<shared_ptr<Quote>> basket;
basket.push_back(make_shared<Quote>("0-201-82470-1", 50));
basket.push_back(make_shared<Bulk_quote>("0-201-82470-1", 50, 10, .25));
cout << basket.back()->net_price(15) << endl;
```

 