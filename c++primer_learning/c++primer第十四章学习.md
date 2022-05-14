# c++primer学习

## 第14章 重载运算与类型转换

#### 14.1 基本概念

​    重载的运算符是具有特殊名字的函数：他们的名字由关键字operator和其后要定义的运算符号共同组成。

​    **ps：当一个重载的运算符是成员函数时，this绑定到左侧运算对象。成员运算符将函数的（显式）参数比运算对象的数量少一个。**

​    我们只能重载已有的运算符，无权发明新的运算符号。

可以重载的运算符如下：                

| +    | -    | *    | /     | %      | ^        |
| ---- | ---- | ---- | ----- | ------ | -------- |
| &    | \|   | ~    | !     | ,      | =        |
| <    | >    | <=   | >=    | ++     | --       |
| <<   | >>   | ==   | !=    | &&     | \|\|     |
| +=   | -=   | /=   | %=    | ^=     | &=       |
| !=   | *=   | <<=  | >>=   | []     | ()       |
| ->   | ->*  | new  | new[] | delete | delete[] |

不能重载的运算符(4个)：

| ：：   | .*   | .    | ?:   |
| ---- | ---- | ---- | ---- |
|      |      |      |      |

###### 1.选择作为成员或者非成员

​    当我们定义重载的运算符时，必须首先决定是将其声明为类的成员函数还是声明为一个普通的非成员函数。下面的准则有助于我们在运算符定义为成员函数还是非成员函数做出抉择：

-   赋值=，下标[]，调用()和成员访问箭头->运算符必须是成员。
-   复合赋值运算符一般来说应该是成员，但并非必须，这一点与赋值运算符略有不同。
-   改变对象状态的运算符或者与给定类型密切相关的运算符，如递增，递减和解引用运算符通常应该是成员。
-   具有对称性的运算符可能转换任意一端的运算对象，例如算数、相等性、关系和位运算符等，因此他们通常应该是普通的非成员函数。

#### 14.2 输入和输出运算符

##### 1.重载输出运算符<<

​    通常情况下，输出运算符的第一个形参是一个非常量ostream对象的引用。第二个形参一般来说是一个常量的引用，该常量是我们想要打印的类类型。

```c++
ostream& operator<<(ostream& os, const Sales_data& item){
    os << item.isbn() << " " << item.units_sold;
    ...
    return os;
}
```

​    **ps：通常，输出运算符应该主要负责打印对象的内容而非控制格式，输出运算符不应该打印换行符。**

​    输入输出运算符必须是非成员函数。因此如果我们需要为类自定义IO运算符，则必须将其定义为非成员函数。当然IO运算符通常要读写类的非公有数据成员，所以IO运算符一般被声明为友元。

##### 2.重载输入运算符>>

​    通常情况下，输入运算符的第一个形参是运算符将要读取的流的引用，第二个形参是将要读入到的（非常量）的对象的引用。

```c++
istream& operator>>(istream& is, Sales_data& item){
    double price;
    is >> item.bookNo >> item.units_sold >> price;
    if (is)  //检查输入是否成功
        item.revenue = item.units_sold * price;
    else
        item = Sales_data();  //输入失败，对象被赋予默认的状态       
    return is;
}
```

​    **ps：输入运算符必须处理可能失败的情况，而输出运算符不需要。**

​    在执行输入运算符时可能发生下列错误：

-   当流含有错误类型的数据时读取操作可能失败。
-   当读取操作到达文件末尾或者遇到输入流的其他错误时也会失败。

​    通过将对象置为合法的状态，我们能（略微）保护使用者免于收到输入错误的影响。

#### 14.3 算术和关系运算符

​    通常情况下，我们把算数和关系运算符定义为非成员函数以允许对左侧或右侧的运算对象进行转换。因为这些运算符一般不需要改变运算对象的状态，所以形参都是常量的引用。

```c++
Sales_data operator+(cosnt Sales_data& lhs, const Sales_data& rhs){
    Sales_data sum = lhs;
    sum += rhs;
    return sum;
}
```

​    **ps：如果类同时定义了算术运算符和相关的复合赋值运算符，则通常情况下应该使用复合赋值运算符来实现算术运算符。**

##### 1.相等运算符

​     相等运算符==得设计准则：

-   如果一个类含有判断两个对象是否相等的操作，则它显然应该把函数定义为operator==而非一个普通得命名函数
-   如果类定义了operator==，则该运算符应该能判断一组给定的对象中是否含有重复数据。
-   通常情况下，相等运算符应该具有传递性，换句话说，如果a==b和b==c都为真，则a==c也应该为真。
-   如果类定义了operator==，则也应该定义operator!=。
-   相等运算符和不相等运算符中的一个应该把工作委托给另外一个，这意味着其中一个运算符应该负责实际比较对象的工作，而令一个运算符则只是调用那个真正工作的运算符。

##### 2.关系运算符

​    定义相等运算符的类通常已包含关系运算符。特别是因为关联容器和一些算法要用到<小于运算符，所以定义operator<会比较有用。

​    通常情况下关系运算符应该：

-   定义顺序关系，令其与关联容器中对关键字的要求一致
-   如果类同时也包含==运算符的话，则定义一种关系令其与==保持一致，特别是如果两个对象是!=的，那么一个对象应该<另外一个。

#### 14.4 赋值运算符

​    类还可以定义其他赋值运算符以使用别的类型作为右侧运算对象。

```c++
class StrVec{
public:
	StrVec& operator=(std::initializer_list<std::string>);
};
StrVec& StrVec::operator=(initializer_list<string> il){
  	//alloc_n_copy分配内存空间并从给定范围内拷贝元素
    auto data = alloc_n_copy(il.begin(), il.end()); 
    free();  //销毁对象中的元素并释放空间
    elements = data.first;  //更新数据成员使其指向新空间
    first_free = cap = data.second;
    return* this;
}
```

​    和拷贝赋值和移动赋值运算符一样，其他重载的赋值运算符也必须是先释放当前内存空间，在创建一片新空间。不同之处是，这个运算符无需检查对象向自身的赋值。

​    **ps：不论形参的类型是什么，赋值运算符都必须定义为成员函数。**

###### 1.复合赋值运算符

​    复合赋值运算符不非得是类的成员。为了与内置类型的复合赋值保持一致，类中的复合赋值运算符也要返回其左侧运算对象的引用。

#### 14.5 下标运算符

​    表示容器的类通常可以通过元素在容器中的位置访问元素，这些类一般都会定义下标运算符operator[]。

​    **ps：下标运算符必须是成员函数。如果一个类包含下标运算符，则它通常会定义两个版本：一个返回普通引用，另一个是类的常量成员并且返回常量引用。**

```c++
class StrVec{
public:
    std::string& operator[](std::size_t n) { return elements[n]; }
    const std::string& operator[](std::size_t n) const { return elements[n]; }
private:
	std::string* elements;
}
```

#### 14.6 递增和递减运算符

​    c++语言并不要求递增和递减运算符必须是类的成员，但是因为他们改变的正好是所操作对象的状态，所以建议将其设定为成员函数。

###### 1.定义前置递增/递减运算符

```c++
class StrBlobPtr{
public:
    StrBlobPtr& operator++();
    StrBlobPtr& operator--();
}
```

​    为了内置版本保持一致，前置运算符应该返回递增或递减后对象的引用。

###### 2.区分前置和后置运算符

​    后置版本接受一个额外的（不被使用）int类型的形参。当我们使用后置运算符时，编译器为这个形参提供一个值为0的实参。这个形参的唯一作用就是区分前置版本和后置版本的函数。

```c++
class StrBlobPtr{
public:
	StrBlobPtr operator++(int);
	StrBolbPtr operator--(int);
};
```

​    **ps：为了与内置版本保持一致，后置运算符应该返回对象的原值（递增或递减之前的值），返回的形式是一个值而非引用。**

###### 3.显式地调用后置运算符

```c++
StrBlobPtr p(a1);
p.operator++(0);  //调用后置版本
p.operator++();   //调用前置版本
```

#### 14.7 成员访问运算符

​    在迭代器类及智能指针类中常常用到解引用符(*)和箭头运算符(->)。

```c++
class StrBlobPtr{
public:
	string& operator*() const{
        auto p = check(curr, "dereference past end");
        return (*p)[curr];
	}
	string* operator->() const{
        return& this->operator*();
	}
}
```

​    **ps：箭头运算符必须是类的成员，解引用运算符通常也是类的成员，尽管并非必须如此。**

###### 1.对箭头运算符返回值的限定

​    对如形如point->men的表达式来说，point必须是指向类对象的指针或者是一个重载了operator->类的对象。根据point类型的不同，point->mem等价于：

```c++
(*point).mem;           //point是一个内置的指针类型
point.operator()->mem;  //point是类的一个对象
```

​    除此之外，代码都将发生错误。point->mem的执行过程如下：

-   如果point是指针，则我们应该用内置的箭头运算符，表达式等价于(*point).mem。
-   如果point是定义了operator->类的一个对象，则我们使用point.operator()->mem。

#### 14.8 函数调用运算符

​    如果类重载了函数调用运算符，则我们可以像使用函数一样使用该类的对象，因为这样的类同时也能存储状态。

```c++
struct absInt{
    int operator() (int val) const{
        return val < 0 ? -val : val;
    }
}
int i = 42;
absInt absObj;
int ui = absObj(i);
```

​    **ps：函数调用运算符必须是成员函数。一个类可以定义多个不同版本的调用运算符，相互之间应该在参数数量或类型上有所区别。如果类定义了调用运算符，则该类的对象称作函数对象。**

##### 1.lambda是函数对象

​    在lambda表达式产生的类中还有一个重载的函数调用运算符：

```c++
stable_sort(words.begin(), words.end(), 
		    [](const string& a, const string& b)
		      { return a.size() < b.size(); });
//上面的行为和下面的类似
class ShorterString{
public:
    bool operator() (const string& s1, const string& s2) const{
        return s1.size() < s2.size();
    }
};
stable_sort(words.begin(), words.end(), ShorterString());
```

###### 1.表示lambda及相应捕获行为的类

​    如我们所知，当一个lambda表达式通过引用捕获变量时，将由程序负责确保lambda执行时引用所用的对象确实存在。因此，编译器可以直接使用该引用而不需在lambda产生的类中将其存储为数据成员。

​    想反，通过值捕获的变量被拷贝到lambda中。因此这种lambda产生的类必须为每个值捕获的变量建立对象的数据成员，同时创建构造函数，令其使用捕获的变量的值来初始化数据成员。

```c++
auto wc = find_if(words.begin(), words.end(),
                  [sz](const string& a) { return a.size() < sz;})
//上面的行为和下面的类似
class SizeComp{
    SizeComp(size_t n) : sz(n) {}
    bool operator() (const string& s) const{
        return s.size() < sz;
    }
private:
    size_t sz;
}
auto wc = auto wc = find_if(words.begin(), words.end(), SizeComp(sz));
```

​    lambda表达式产生的类不含有默认的构造函数、赋值运算符及默认的析构函数；他是否有默认的拷贝/移动构造函数则通常要视捕获的数据类型成员类型而定。

##### 2.标准库定义的函数对象

​    标准库定义了一组表示算术运算符、关系运算符和逻辑运算符的类，每个类分别定义了一个执行命名操作的调用运算符。定义在头文件functional中：

|        算术        |         关系          |        逻辑         |
| :--------------: | :-----------------: | :---------------: |
|    plus<Type>    |   equal_to<Type>    | logical_and<Type> |
|   minus<Type>    | not_equal_to<Type>  | logical_or<Type>  |
| multiplies<Type> |    greater<Type>    | logical_not<Type> |
|  divides<Type>   | greater_equal<Type> |                   |
|  modulus<Type>   |     less<Type>      |                   |
|   negate<Type>   |  less_equal<Type>   |                   |

```c++
plus<int> intAdd;                 //可执行int加法的函数对
negate<int> intNegate;            //可执行int值取反的函数对象
int sum = intAdd(10, 20);         //sum=30;
sum = intNegate(intAdd(10, 20));  //sum=-30
sum = intAdd(10, intNegate(10));  //sum=0
```

​    表示运算符的函数对象常用来替换算法中的默认运算符。

```c++
//传入一个临时函数对象用于执行两个string对象>比较操作
sort(vec.begin(), vec.end(), greater<string>());
```

##### 3.可调用对象与function

​    c++语言中有几种可调用的对象：函数、函数指针、lambda表达式、bind创建的对象以及重载了函数调用运算符的类。和其他对象一样，可调用的对象也有类型。然而，两个不同类型的可调用对象却可能共享同一中调用形式，调用形式指明了调用返回的类型以及传递给调用你的实参类型。

###### 1.不同类型可能具有相同的调用形式

```c++
//普通函数
int add(int i, int j) { return i + j; }
//lambda
auto mod = [](int i, int j) { return i % j; }
//函数对象类
struct divide{
  int operator() (int denominator, int divisor){
      return denominator / divisor;
  }
}
```

​    虽然上述可调用对象执行了不同的算数运算，且类型（指可调用对象）各不相同，但是共享一种调用形式

```c++
int(int ,int)
```

​    我们可能希望使用这些可调用对象构建一个简单的桌面计算器。为了实现这一目的，我们需要定义一个函数表用于存储指向这些可调用对象的”指针“：

```c++
map<string, int*()(int, int)> binops;
```

​    我们可以按照下面的形式将add指针添加到binops中：

```c++
binops.insert({"+", add});  //{"+", add}是一个pair
```

​    但是我们不能将mod或divide存入指针中：

```c++
binops.insert({"%", mod});  //错误：mod不是一个函数指针
```

​    这个问题在于mod是个lambda表达式，每个表达式有他自己的类类型，该类型与binops的值的类型不匹配。

###### 2.标注库function类型

​    我们可以使用一个名为function的新的标准库类型解决上述问题，function定义在functional中：

| function<T> f;           | f是一个用来存储可调用对象的空function                  |
| ------------------------ | ---------------------------------------- |
| function<T> f(nullptr);  | 显式地构造一个空function                         |
| function<T> f(obj);      | 在f中存储可调用对象obj的副本                         |
| f                        | 将f作为条件，当f含有一个可调用对象时为真，否则为假               |
| f(args)                  | 调用f中的对象，参数时args                          |
| **定义为function<T>的成员的类型** |                                          |
| result_type              | 该function类型的可调用对象返回的类型                   |
| argument_type            | 当T有一个或两个实参时定义的类型，如果T只有一个实参，则argument_type是该类型的同义词；如果有两个，first和second分别代表两个实参 |
| first_argument_type      |                                          |
| second_argument_type     |                                          |

​    function是一个模板： `function<int(int, int)>`，我们声明了一个function类型，他可以表示接受两个int，返回一个int的可调用对象：

```c++
function<int(int, int)> f1 = add;  //函数指针
function<int(int, int)> f2 = divide();  //函数对象类的对象
function<int(int, int)> f3 = [](int i, int j) { return i * j; };  //lambda
cout << f1(4, 2) << endl;  //6
cout << f2(4, 2) << endl;  //2
cout << f3(4, 2) << endl;  //8
```

​    使用这个function类型我们可以重新定义map：

```c++
map<string, function<int(int, int)>> binops = {
	{"+", add},                                 //函数指针
	{"-", std::minus<int>()},                   //标准库函数对象
	{"/", divide()},                            //用户定义的函数对象
	{"*", [](int i, int j) { return i * j; }},  //未命名的lambda
	{"%", mod}                                  //命名了的函数对象
};
```

​    当我们索引map时将得到关联值的一个引用：

```c++
binops["+"](10, 5);  //调用add(10, 5)
....
binops["%"](10, 5);  //调用lambda函数对象
```

###### 3.重载的函数与function

​    我们不能直接将重载函数的名字存入function类型的对象中

```c++
int add(int i, int j) { return i + j; }
Sales_data add(const Sales_data&, const Sales_data&);
map<string, function<int(int, int)>> binops;
binops.insert({"+", add});  //到底添加哪个add？
```

​    解决上述二义性的问题的一条途径时存储函数指针而非函数的名字：

```c++
int (*fp)(int, int) = add;  //指针所指的add是接受俩个int的版本
binops.insert({"+", fp});   //正确：fp指向一个正确的add版本
```

​    同样我们可以使用lambda来消除二义性：

```c++
binops.insert({"+", [](int a, int b) { return add(a, b); } });
```

#### 14.9 重载、类型转换与运算符

​    我们同样能定义对于类类型的类型转换，转换构造函数和类型转换运算符共同定义了类类型转换，这样的转换有时也被称为用户定义的类型转换。

##### 1.类型转换运算符

​    类型转换运算符是类的一种特殊成员函数，她负责将一个类类型的值转换成其他类型。类型转换函数一般形式如下：

```c++
operator type() const;
```

​    其中type表示类型，类型转换可以面向除了void外的任何类型进行定义，只要该类型能作为函数的返回类型。因此我们不允许转换成数组或函数类型，但允许转换指针或引用类型。

​    **ps：一个类型转换函数必须是类的成员函数；他不能声明返回类型，形参列表也必须为空。类型转换函数通常应该是const。**

###### 1.定义含有类型转换运算符的类

```c++
calss SmallInt{
public: 
    SmallInt(int i = 0) : val(i) {
        if (i < 0 || i > 255)
            throw std::out_of_range("error");
    }
    operator int() const { return val; }
ptivate:
    std::size_t val;
}
```

​    我们的SmallInt即定义了向类类型的转换，也定义了从类类型向其他类型的转换。

```c++
SamllInt si;
si = 4;  //首先将4隐式地转换成SmallInt，然后调用SmallInt::operator=
si + 3;  //首先将si隐式地转换为int，然后执行整数的加法
```

​    因为类型转换运算符是隐式执行的，所以无法传递给这些函数参数，当然也不能定义中使用任何形参。同时，尽管类型转换函数不负责指定返回类型，但实际上每个类型转换函数都会返回一个对应类型的值。

###### 2.类型转换运算符可能产生意外结果

​    在实践中，类很少提供类型转换符。

###### 3.显式的类型转换运算符

​    为了防止异常情况发生，c++11新标准引入了显式的类型转换运算符explicit:

```c++
class SamllInt{
public:
    ...
    //编译器不会自动执行这一类型的转换
    explicit operator int() const { return val; }
}
```

​    和显式的构造函数一样，编译器通常也不会将一个显式的类型转换运算符用于隐式类型转换：

```c++
SmallInt si = 3;           //正确：SmallInt的构造函数不是显式的
si + 3;                    //错误：此处需要隐式的类型转换，但类的运算符是显式的
static_cast<int>(si) + 3;  //正确：显式地请求类型转换
```

​    该规定存在一个例外，即如果表达式被用作条件，则编译器会将显式的类型转换自动应用于它。换句话说，当表达式出现下列位置时，显示的类型转换将被隐式地执行：

-   if、while及di语句的条件部分
-   for语句头的条件表达式
-   逻辑非!、逻辑或||、逻辑与&&的运算对象
-   条件运算符 ? : 的条件表达式

###### 4.转换为bool

​    无论我们什么时候在条件下使用流对象，都会使用为IO类型定义的operator bool。

​    **ps：向bool的类型转换通常用在条件部分，因此operator bool一般定义成explicit的。**

##### 2.避免有二义性的类型转换

​    有两种情况下可能产生多重转换路径。第一种情况是两个类提供相同的类型转换，第二种情况是类定义了多个转换规则。

###### 1.实参匹配和相同的类型转换

```c++
//最好不要在两个类之间构建相同的类型转换
struct B;
struct A{
    A() = default;  //把一个B转换成A
    A(const B&);
};
struct B{
    operator A() const;  //也是把一个B转换成A
};
A f(const A&);
B b;
A a = f(b);  //二义性错误:到底是 f(B::operator A()) 还是 f(A::A(cosnt B&))
```

​    同时存在两种由B获得A的方法，所以造成编译器无法判断应该运行哪个类型转换，也就是说对f的调用存在二义性。如果我们想执行上述的调用，就不得不显式地调用转换运算符或转换构造函数：

```c++
A a1 = f(b.operator A());  //使用B的类型转换运算符
A a2 = f(A(b));            //使用A的构造函数
```

###### 2.二义性与转换目标为内置类型的多重类型转换

​    另外如果类定义了一组类型转换，他们的转换源（或者转换目标）类型本身可以通过其他类型转换联系在一起，则同样会造成二义性问题。

###### 3.重载函数与转换构造函数

​    当我们调用重载的函数时，从多个类型转换中进行选择将变得更加复杂，如果两个或多个类型转换都提供了同一种可行匹配，则这些类型转换一样好。

##### 3.函数匹配与重载运算符

​    重载的运算符也是重载的函数。因此，通用的函数匹配规则同样适用于判断在给定的表达式中到底应该使用内置运算符还是重载运算符。不过当运算符函数出现在表达式中，候选函数集的规模要比我们使用调用运算符调用函数时更大。如果a是一种类类型，则表达式a sym b 可能是：

```c++
a.operatorsym(b);   //operators是a的一个成员函数
operatorsym(a, b);  //operatorsym是一个普通函数
```

​    **ps：如果我们对同一个类及提供了转换目标是算数类型的类型转换，也提供了重载的运算符，则将会遇到重载运算符与内置运算符的二义性问题。**