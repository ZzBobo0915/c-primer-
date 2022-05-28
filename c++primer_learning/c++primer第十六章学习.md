# c++primer学习

## 第16章 模板与泛型编程

#### 16.1 定义模板

##### 1.函数模板

​    我们可以定义一个通用的函数模板，而不是为每个类型都定义一个新函数。一个函数模板就是一个公式，可用来生成针对特定类型的函数版本。对比两个数大小的compare的模板版本可能像下面一样：

```c++
template<typename T>
int compare(const T &v1, const T &v2){
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}
```

​    模板定义以关键字template开始，后跟一个模板参数列表，这是一个逗号分隔开的一个或多个模板参数的列表，用<>包围起来。

​    **ps：在模板定义中，模板参数列表不能为空。**

###### 1.实例化函数模板

​    当我们调用一个函数模板时，编译器通常用函数实参来推断模板实参。

```c++
cout << compare(1, 0) << endl;  //T为int
```

​    编译器用推断出的模板参数来为我们实例化一个特定版本的函数。

###### 2.模板类型参数

​    我们的compare函数有一个模板类型参数。一般来说，我们可以将类型参数当作类型说明符，就像内置类型或类类型说明符一样使用。特别是，类型参数可以用来指定返回类型或函数的参数类型，以及在函数体内用于变量声明或类型转换。

```c++
//错误：U之前必须加上class或typename
template<typename T, U>
T calc(cons T&, const U&);
//正确：在模板参数列表中，typename和class没什么不同
template<typename T, class U>
T calc(cons T&, const U&);  
```

###### 3.非类型模板参数

​    除了定义类型参数，还可以在模板中定义非类型参数。一个非类型参数表示一个值而非一个类型。我们通过一个特定的类型名而非关键字class或typename来指定非类型参数。

​    当一个模板被实例化时，非类型参数被一个用户提供的或编译器推断出的值所代替。这些值必须是常量表达式：

```c++
template<unsigned N, unsigned M>
int compare(const char(&p1)[N], const char(&p2)[M]){
    return strcmp(p1, p2);
}
```

###### 5.inline和constexpr的函数版本

​    函数模板可以声明为inline或constexpr的，如同非模板函数一样：

```c++
//inline说明符跟在模板参数列表之后
template<typename T> inline T min(const T&, const T&);
```

###### 6.模板编译

​    当编译器遇到一个模板定义时，它并不生成代码。只有我们实例化出模板的一个特定版本时，编译器才会生成代码。当我们使用（而不是定义）模板时，编译器才生成代码，这一特性影响了我们如何组织代码以及错误何时被检测到。

​    **ps：函数模板和类模板成员的定义通常放在头文件中。**

##### 2.类模板

​    类模板是用来生成来的蓝图的。与函数模板不同，编译器不能为类模板推断模板参数类型。

###### 1.定义类模板

​    与函数模板类似，类模板以关键字template开始，后跟模板参数列表。在类模板（及其成员）的定义中，我们将模板参数当作替身，代替使用模板时用户需要提供的类型或值。

```c++
template<typename T>
class Blob{
public:
    typedef T value_type;
    typefed typename std::vector<T>::size_type size_type;
    ...
private:
    std::shared_ptr<std::vector<T>> data;
    void check(size_type i, const std::string& msg) const;
};
```

###### 2.实例化模板

​    我们已经多次见到，当使用一个类模板时，我们必须提供额外信息。我们现在知道这些额外信息是显示模板实参列表，他们被绑定到模板参数。编译器使用这些模板实参来实例化出特定的类：

```c++
Blob<int> ia;
Blob<int> ia2 = {0, 1, 2, 3, 4};
```

​    ia和ia2使用相同的特定类型版本的Blob，从这两个定义中，编译器会实例化出一个与下面等价的类：

```c++
template<>
class Blob<int>{
    typedef typename std::vector<int>::size_type size_type;
    Blob();
    Blob(std::initialize_list<int> il);
    //...
    int& operator[](size_type i);
private:
    std::shared_ptr<std::vector<int>> data;'
    void check(size_type i, const std::string& msg) const;
};
```

​    当编译器从我们的Blob模板实例化出一个类时，他会重写Blob模板，将模板参数T的每个实例替换为给定的模板参数。

​    **ps：一个类模板的每个实例都形成一个独立的类。类型Blob<int>与其他任何Blob类型都没有关联，也不会对任何其他Blob类型的成员有特殊访问权限。**

###### 3.在模板作用域中引用模板类型

​    一个模板类如果使用了另外一个模板，通常不将一个实际类型（或值）的名字用作其模板实参，相反的我们通常将模板自己的参数当作被使用模板的实参。

###### 4.类模板的成员函数

​    与其他任何类相同，我们既可以在类模板内部，亦可以在类模板外部为其定义成员函数，且定义在类模板内的成员函数被隐式地声明为内联函数：

```c++
template<typename T>
ret-type Blob<T>::member-name(param-list)
```

###### 5.check和元素访问成员

```c++
template<typename T>
void Blob<T>::check(size_type i, const std::string& smg) const{
    if (i >= data->size())
        throw std::out_of_range(msg);
}
template<typename T>
T& Blob<T>::back(){
    check(0, "back on empty Blob");
    return data->back();
}
template<typename T>
T& Blob<T>::operator[](size_type i){
    check(i, "subscript out of range");
    return (*data)[i];
}
```

###### 6.Blob的构造函数

​    构造函数的定义要以模板参数开始：

```c++
//此构造函数分配一个空的vector，并将指向vector的指针保存在data中
template<typename T>
Blob<T>::Blob() : data(std::make_shared<std::vector<T>>()){}
```

​    类似的，接受一个initializer_list参数的构造函数将其类型参数T作为initializer_list参数的元素类型：

```c++
templete<typename T>
Blob<T>::Blob(std::initialize_list<T> il) : data(std::make_shared<std::vector<T>>(il)) { }
```

###### 7.类模板成员函数的实例化

​    默认情况下，一个类模板的成员函数只有当程序用到它时才进行实例化：

```c++
Blob<int> squares = {0, 2, 3, 4, 5, 6, 7, 8, 9};
for(size_t i = 0; i != squared.size(); ++i)
    squares[i] = i * i;  //实例化Blob<int>::operator[](size_t)
```

###### 8.在类代码内简化模板类名的使用

​    当我们使用一个类模板类型时必须提供模板实参，但这一规则有一个例外。在类模板自己的作用域中，我们可以直接使用模板名而不提供实参：

```c++
template<typename T> class BlobPtr{
public:
    BlobPtr() : curr(0) { }
    BlobPtr(Blob<T> &a, size_t sz = 0) : wptr(a.data), curr(sz) { }
    T& operator*() const{
        auto p = check(curr, "dereference past end");
        return (*p)[curr];
    }
    //可以返回BlobPtr&而不是BlobPtr<T>&
    BlobPtr& operator++();
    BolbPtr& operator--();
private:
    std::shared_ptr<std::vector<T>> check(std::size_t, const std::string&) const;
    std::weak_ptr<std::vector<T>> wptr;
    std::size_t curr;
}
```

###### 9.在类模板外使用类模板名

​    当我们在类模板外定义成员时，必须记住，我们并不在类的作用域中，直到遇到类名才表示进入类的作用域。

###### 10.类模板和友元

​    当一个类包含一个友元声明时，类与友元各自是否模板是互相无关的。如果一个类模板包含一个非模板友元，则友元被授权可以访问所有模板实例。如果友元自身是模板，类可以授权给所有友元模板实例，也可以之授权给特定实例。

###### 11.一对一友好关系

​    类模板与另一个（类或函数）模板间友好关系的最常见的形式是建立对应实例及友元间的友好关系。

​    为了引用（类或函数）模板的一个特定实例，我们必须首先声明模板自身。一个模板声明包括模板参数列表：

```c++
//前置恒明
template<typename> class BlobPtr;
template<typename> class Blob;
template<typename T>
bool operator==(const Blob<T>&, const Blob<T>&);
template<typename T> class Blob{
    friend class BlobPtr<T>;
    friend bool operator==<T> (const Blob<T>&, const Blob<T>&);
};
```

###### 12.通用和特定模板友好关系

​    一个类也可以将另一个模板的每个实例都声明为自己的友元，或者限定特定的实例为友元：

```c++
template<typename T> class Pal;
class C {
    friend class Pal<C>;  //用C实例化的Pal是C的一个友元
    template<typename T> friend class Pal2;  //Pal2的所有实例都是C的友元
};
template<typename T> class C2{
    friend class Pal<T>;
    template<typename X> friend class Pal2;
    friend class Pal3;
}
```

​    **ps：为了让所有实例成为友元，友元声明中必须使用与类模板本身不同的模板参数。**

###### 12.令模板自己的类型参数变成友元

​    在新标准中，我们可以将模板类型参数声明为友元：

```c++
template<typename Type> class Bar{
    friend Type;
    //...
};
```

​    此处我们将用来实例化Bar的类型声明为友元。比如对于某个类型Foo，Foo将成为Bar<Foo>的友元。

​    值得注意的是，虽然友元通常来说应该是一个类或是一个函数，但我们完全可以用一个内置类型来实例化Bar。这种与内置类型的友好关系是允许的，以便我们能用内置类型来实例化Bar这样的类。

###### 13.模板类型别名

​    类模板的一个实例定义了一个类类型，与其他任何类型一样，我们可以定义一个typedef来引用实例化的类：

```c++
typedef Blob<string> StrBlob;
```

​    我们无法定义一个typedef引用Blob<T>，但是新标准允许我们为模板定义一个类型别名：

```c++
template<typename T> using twin = pair<T, T>;
twin<string> authors;  //authors是一个pair<string, string>
```

​    当我们定义一个模板别名时，可以固定一个或多个模板参数：

```c++
template<typename T> using partNo = pair<T, unsigned>;
partNo<string> books;  //books是一个pair<string, unsigned>  
```

###### 14.类模板的static成员

​    与其它类相同，类模板可以声明static成员：

```c++
template<typename T> class Foo{
public:
    static std::size_t count() { return ctr; }
private:
    static std::size_t ctr;
};
```

​    所有Foo<X>类型的对象共享相同的ctr对象和count()函数：

```c++
Foo<int>, fi, fi2, fi3;  //fi,fi2,fi3共享Foo<int>::count()和Foo<int>::ctr
```

​    与其他的static数据成员相同，模板类的每个static数据成必须有且仅有一个定义。但是模板类的每个实例都有一个独有的static对象，因此，与定义模板的成员函数相似，我们将static数据成员也定义为模板：

```c++
template<typename T>
size_t Foo<T>::ctr = 0;
```

​    与非模板类的静态成员函数相同，我们可以通过类类型对象来访问一个类模板的static成员，也可以使用作用域运算符直接访问成员。当然，为了通过类来直接访问static成员，我们必须引用一个特定的实例：

```c++
Foo<int> fi;
auto ct = Foo<int>::count();  //实例化Foo<int>::count
ct = fi.count();              //使用Foo<int>count
ct = Foo::count();            //错误：没指定使用哪个模板实例的count
```

##### 3.模板参数

​    类似函数参数的名字，一个模板参数的名字也没有任何内在含义。我们通常将类型参数命名为T，但实际上我们可以使用任何名字：

```c++
template<typename Foo> Foo clac(const Foo& a, const Foo& b){
    Foo tmp = a;
    //...
    return tmp;
}
```

###### 1.模板参数与作用域

​    模板参数遵循普通的作用域规则。一个模板参数名的可用范围是在其声明之后，至模板声明或定义结束前。与其他名字一样，模板参数会隐藏外层作用域中声明的相同名字。但是，与大多数其他上下文不同，在模板内不能重用模板参数名。但是，与大多数其他上下文不同，在模板内不能重用模板参数名：

```c++
typedef double A;
template<typename A, typename B> void f(A a, B b){
    A temp = a;  //tmp的类型为模板参数A的类型，而非double
    double B;    //错误：重声明模板参数为B
}
template<typename A, typename A>  //错误：重用模板名A
```

###### 2.模板声明

​    模板声明必须包含模板参数：

```c++
template<typename T> int compare(const T&, const T&);
template<typename T> class Blob;
```

​    与函数参数相同，声明中的模板参数的名字不必与定义相同：

```c++
//两个calc指向的相同的模板
template<typename T> T calc(const T&, const T&);
template<typename Type>
Type calc(const Type&, const Type&) { }
```

​    **ps：一个特定文件所需要的所有模板的声明通常一起放置在文件开始的位置。**

###### 3.使用类的类型成员

​    如果我们希望使用一个模板类型参数的类型成员，就必须显式告诉编译器该名字是一个类型：

```c++
template<typename T>
typename T::value_type top(const T& c){
    if (!c.empty())  return c.back();
    else return typename T::value_type();
}
```

###### 4.默认模板实参

​    我们也可以提供默认模板实参：

```c++
template<typename T, typename F = less<T>>
int compare(const T& v1, const T& v2, F f = F ()){
    if (f(v1, v2)) return -1;
    if (f(v2, v1)) return 1;
    return 0;
}
```

​    **ps：与函数默认实参一样，对于一个模板参数，只有当它右侧的所有参数都有默认实参时，他才可以有默认实参。**

###### 5.模板默认实参与模板类

​    无论何时使用一个类模板，我们都必须在模板名之后接上尖括号。尖括号指出类必须从一个模板实例化而来。特别是，如果一个类模板为其所有模板参数都提供了默认实参，且我们希望使用这些默认实参，就比系在模板名之后跟一个空尖括号对：

```c++
template<class T = int> class Numbers{
public:
    Numbers(T v = 0) : val(v) { }
private:
    T val;
};
Numbers<long double> lots_of_precision;
Numbers<> average_precision;  //<>表示我们希望使用默认类型
```

##### 4.成员模板

​    一个类（无论是普通类还是类模板可以包含本身是模板的成员函数）。这种成员被称为成员模板。成员模板不能是虚函数。

###### 1.普通（非模板）类的成员模板

​    作为普通类包含成员模板的例子，我们定义一个类，类似unique_ptr所使用的默认删除其类型：

```c++
class DebugDelete{
public:
    DebugDelete(std::ostream& s = std::cerr) : os(s) {}
    //与任何函数模板相同，T的类型由编译器推断
    template<typename T> void operator()(T *p) const{
        os << "deleting unique_ptr" << std::endl;
        delete p;
    }
private:
    std::ostream& os;
};
delete *p = new double;
DebugDelete d;  //可像delete表达式一样使用的对象
d(p);  //调用DebugDelete::operator()(double*),释放p
int *ip = new int;
//在一个临时DebugDelete对象上调用operator()(int*)
DebugDelete()(ip);
//销毁p指向的对象
//实例化DebugDelete::operator()<int>(int*)
unique_ptr<int, DebugDelete> p(new int, DebugDelete());
//销毁sp指向的对象
//实例化DebugDelete::operator()<string>(string*)
unique_ptr<string, DebugDelete> p(new string, DebugDelete());
```

######  2.类模板的成员函数

​    对于类模板，我们也可以为其定义成员模板。在此情况下，类和成员各自有各自的、独立的模板参数：

```
template<typename T> class Blob{
    tempalte<typename It> Blob(It b, It e);
    //...
};
```

   与类模板的普通函数不同，成员模板是函数模板。当我们在类模板外定义一个成员模板时，必须同时为类模板和成员模板提供模板参数列表。类模板的参数列表在前，后跟成员自己的模板参数列表：

```c++
template<typename T>
template<typename It>
Blob<T>::Blob(It b, It e) : data(std::make_shared<std::vector<T>>(b, e)) { }
```

###### 3.实例化与成员模板

​    为了实例化一个类模板的成员模板，我们必须同时提供类和函数模板的实参。与往常一样，我们在哪个对象上调用成员模板，编译器就根据该对象的类型来推断类模板参数的实参。与普通函数模板相同，编译器通常根据传递给成员模板的函数实参来推断它的模板实参：

```c++
int ia[] = {0, 1, 2, 3 ,4 ,5 ,6 , 7, 8, 9};
vector<long> vi = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
list<const char*> w = {"now", "is", "the", "time"};
//实例化Blob<int>类及其接受两个int*参数的构造函数
Blob<int> a1(begin(ia), end(ia));
//实例化Blob<int>类及其接受两个vector<long>参数的构造函数
Blob<int> a2(vi.begin(), vi.end());
//实例化Blob<string>及其接受两个list<const char*>::iterator参数的构造函数
Blob<string> a3(w.begin(), w.end());
```

###### 4.控制实例化

​    在大系统中，在多个文件中实例化相同模板的额外开销可能会非常严重。在新标准下，我们可以通过显式实例化来避免这种开销。一个显式实例化有如下形式：

```c++
extern template decaration;  //声明
template declaration;        //定义
extern template class Blob<string>;
template int compare(const int&, const int&);
```

​    将一个实例化声明为extern就表示承诺在程序其他位置有该实例化的一个非extern声明（定义）。对于一个给定实例化版本，可能有多个extern声明，但必须只有一个定义。

###### 4.实例化定义会实例化所有成员

​    我们用来显式实例化一个类模板的类型，必须能用于模板的所有成员。

#### 16.2 模板实参推断

​    从函数实参来确定模板实参的过程被称为模板实参推断。在模板实参推断过程中，编译器使用函数调用中的实参类型来寻找模板实参，用这些模板实参生成的函数版本与给定的函数调用最为匹配。

##### 1.类型转换与模板类型参数

​    与非模板函数一样，我们在一次调用中传递给函数模板的实参被用来初始化函数的形参。如果一个函数形参的类型使用了模板类型参数，那么它采用特殊的初始化规则。只有很有限的几种类型转换会自动地应用于这些实参。编译器通常不是对实参进行类型转换，而是生成一个新的模板实例。

​    能在调用中应用于函数模板的包括以下两项：

-   const转换，可以将一个非const对象的引用（指针）传递给一个const的引用（或指针）的形参
-   数组或函数指针转换：如果函数形参不是引用类型，则可以对数组或函数类型的实参应用正常的指针转换。一个数组实参可以转换为一个指向其首元素的指针。类似的，一个函数实参可以转换为一个该函数类型的指针。

​    其他类型转换，如算数转换、派生类向基类转换，用户定义的转换都不能应用于函数模板。

```c++
template<typename T> T fobj(T, T);
template<typename T> T fref(const T&, const T&);
string s1("a value");
const string s2("another value");
fobj(s1, s2);      //调用fobj(string, string),const被忽略
fref(s1, s2);      //调用fref(const string&, const string&)
int a[10], b[42];  
fobj(a, b);        //调用f(int*, int*)
fref(a, b);        //错误：数组类型不匹配
```

###### 1.使用相同模板参数类型的函数形参

​    一个模板类型参数可以用作多个函数形参的类型，由于只允许有限的几种类型转换，因此传递给这些形参的实参必须具有相同的类型。如果希望允许对函数实参进行正常的类型转换，我们可以将函数模板定义为两个类型参数：

```c++
template<typename T>
int compare(const T& v1, const T& v2){
    if (v1 > v2) return 1;
    if (v1 < v2) return -1;
    return 0;
}
long lng = 10;
compare(lng, 1024);  //错误：不能实例化compare(long, ing)
template<typename A, typename B>
int flexiableCompare(const A& v1, const B& v2){
    if (v1 > v2) return 1;
    if (v1 < v2) return -1;
    return 0;
}
long lng = 10;
flexiableCompare(lng, 1024);  //正确
```

###### 2.正常类型转换应用于普通函数实参

​    函数模板可以用普通类型定义的参数，即，不涉及模板类型参数的类型。这种函数实参不进行特殊处理：他们正常转换为对应形参的类型：

```c++
template<typename T> 
ostream& print(ostream& os, const T& obj){
    return os << obj;
}
print(cout, 42);
ofstream f("out put");
print(f, 10);
```

##### 2.函数模板显式实参

​    在某些情况下，编译器无法推出的模板实参的类型。其他一些情况下，我们希望允许用户控制模板实例化。当函数返回类型与参数列表中任何类型都不相同时，这两种情况最常出现。

###### 1.指定显式模板类型

​    我们可以定义表示返回类型的第三个模板参数，从而允许用户控制返回类型：

```c++
//编译器无法推断T1，它未出现在产生列表中
template<typename T1, typename T2, typename T3>
T1 sum(T2, T3);
```

​    这时我们需要对T1提供一个显式模板实参，显式模板实参再尖括号中给出，位于函数名之后，实参列表之前：

```c++
//T1显式指定long long，T2和T3是从函数实参类型推断而来的
auto val3 = sum<long long>(i, lng);
```

​    显式模板类型按由左至右的顺序与对应模板参数匹配；第一个模板实参与第一个模板参数匹配，以此类推。只有尾部的参数的显示模板实参才可以忽略。 

###### 2.正确类型转换应用于显式指定的实参

​    对于模板类型参数已经显式制定了的函数实参，也进行正常的类型转换：

```c++
long lng;
compare(lng, 1024);        //错误：模板参数不匹配
compare<long>(lng, 1024);  //正确：实例化compare<long, int>
compare<int>(lng, 1024);   //正确：实例化compare<int, int>
```

##### 3.尾置返回类型与类型转换

​    例如，我们希望编写一个函数，表示接受序列的一对迭代器和返回序列中一个元素的引用：

```c++
template<typename It>
??? &fcn(It beg, It end){
    //...
    return *beg;
}
```

​    我们并不知道返回结果的准确类型，但知道所需类型是所处理的序列的元素类型：

```c++
vector<int> vi = {1, 2, 3, 4, 5};
Blob<string> ca = { "hi", "bye"};
auto &i = fcn(vi.begin(), vi.end());  //fcn应该返回int&
auto &s = fcn(ca.begin(), ca.end());  //fcn应该返回string&
```

​    为了定义这样的函数，我们必须使用尾置返回类型：

```c++
template<typename T>
auto fcn(It beg, It end) -> decltype(*beg){
    ...
    return *beg;  //返回序列中一个元素的引用
}
```

###### 1.进行类型转换的标准库模板类

​    有时我们无法直接获得所需要的类型。在编写这个函数的过程中，我们面临着一个问题，对于传递的参数的类型，我们几乎一无所知，再次函数中，我们知道唯一可以使用的操作是迭代器操作，而所有迭代器操作都不会产生元素，只能生成元素的引用。

​    为了获得元素类型，我们可以使用标准库的类型转换模板。这些模板定义在头文件type_traits中。

```c++
template<typename T>
auto fcn2(It beg, It end) ->
    typename remove_reference<decltype(*beg)>::type
{
    //...
    return *beg;
}
```

​    标准类型转换模板如下：

| 对Mod<T>，其中Mod为       | 若T为           | 则Mod<T>::type为 |
| -------------------- | ------------- | -------------- |
| remove_reference     | X&或X&&        | X              |
|                      | 否则            | T              |
| add_const            | X&、const X或函数 | T              |
|                      | 否则            | const T        |
| add_lvalue_reference | X&            | T              |
|                      | X&&           | X&             |
|                      | 否则            | T&             |
| add_rvalue_reference | X&或X&&        | T              |
|                      | 否则            | T&&            |
| remove_pointer       | X*            | X              |
|                      | 否则            | T              |
| add_pointer          | X&或X&&        | X*             |
|                      | 否则            | T*             |
| make_signed          | unsigned X    | X              |
|                      | 否则            | T              |
| make_unsigned        | 带符号类型         | unsigned X     |
|                      | 否则            | T              |
| remove_extent        | X[n]          | X              |
|                      | 否则            | T              |
| remove_all_extents   | X[n1]...      | X              |
|                      | 否则            | T              |

##### 4.函数指针和实参推断

​    当我们用一个函数模板初始化一个函数指针或作为一个函数指针赋值时，编译器使用指针的类型来推断模板实参。

```c++
template<typename T> int compare(const T&, const T&);
//pf1指向实例int compare(const int&, const int&)
int (*pf1)(const int&, const int&) = compare;
```

​    如果不能从函数指针类型确定模板实参，则产生错误：

```c++
void func(int(*)(const string&, const string&));
void func(int(*)(const int&, const int&));
func(compare);  //错误：使用compare的哪个实例
```

​    我们可以通过使用显式模板损失惨来消除func调用的歧义：

```c++
func(compare<int>);  //传递compare(const int&, const int&)
```

##### 5.模板实参推断和引用

###### 1.从左值引用函数参数推断类型

​    当一个函数参数是模板参数的一个普通（左值）引用时，绑定规则告诉我们，只能传递给他一个左值。实参可以是const类型也可以不是。如果实参是const，则T将被推断为const类型：

```c++
template<typename T> void f1(T&);  //实参必须是一个左值
f1(i);   //i是一个int，模板参数类型T是int
f1(ci);  //ci是一个const int，模板参数T是const int
f1(5);   //错误：传递给一个&参数的实参必须是一个左值
```

​    如果一个函数参数的类型是const T&，正常的绑定规则告诉我们可以传递给它任何类型的实参--一个对象（const或非const）、一个临时对象或是一个字面值常量。当参数本身是const时，T的类型推断的结果不会是一个const类型。const已经是函数参数类型的一部分，因此她不会是模板参数类型的一部分：

```c++
compare<typename T> void f2(const T&);  //可以接受一个右值
f2(i);   //i是一个int，模板参数是const int&
f2(ci);  //ci是一个const int，但模板参数T是int
f2(5);   //一个const&参数可以绑定到一个右值上，T是int
```

###### 2.从右值引用函数参数推断类型

​    当一个函数参数是一个右值引用时，正常绑定规则告诉我们可以传递给他一个右值，当我们这样做时，类型推断过程类似普通左值引用函数参数的推断过程：

```c++
template<typename T> void f3(T&&);
f3(42);  //实参是一个int的右值，模板参数T是int
```

###### 3.引用折叠和右值引用参数

​    通常我们不能将一个右值绑定到一个左值上。但是c++在正常绑定规则之外还定义了两个例外规则，允许这种绑定。这两个例外规则是move这种标准库设施正确工作的基础：

​    第一个例外规则影响右值引用参数的推断如何进行。当我们将一个左值传递给函数的右值引用参数，且此右值引用指向模板类型参数（如T&&）时，编译器推断模板类型参数为实参的左值引用类型。因此我们调用f3(i)时，编译器推断T的类型为int&，而非int。

​    如果我们间接创建一个引用的引用，则这些引起形成了“折叠”。在所有情况下（除了上面的哪个），引用会折叠成一个普通的左值引用类型。在新标准中，止跌规则扩展到右值引用。只在一种特殊情况下引用会折叠成右值引用：右值引用的右值有引用：

-   X& &、X& &&和X&& &都将折叠为类型X&
-   类型X&& &&折叠成X&&

​    **ps：折叠引用只能引用与间接创建的引用的引用，如类型别名或模板参数。**

```c++
f3(i);   //实参是一个左值；模板参数T是int&
f3(ci);  //实参是一个措置；模板参数T是一个const int&
void f3<int&>(int&&);  //当T是int&&时，此函数参数折叠为int& &&
void f3<int&>(int&);   //当T是int&时，此函数参数折叠为int&
```

​    这两个规则导致了两个重要结果：

-   如果一个函数参数是一个指向模板类型参数的右值引用，如（T&&），则它可以被绑定到一个左值；且：
-   如果实参是一个左值，则推断出的模板实参类型将是一个左值引用，且函数参数将被实例化为一个（普通）左值引用参数（T&）

​    **ps：如果一个函数参数是指向模板参数类型的右值引用，则可以传递给它任何类型的实参。如果将一个左值传递给这样的参数，则该函数参数被实例化为一个普通的左值引用（T&）。**

###### 4.编写接受右值引用参数的模板参数

​    模板参数可以推断为一个引用类型，这一特性对模板内的代码可能有令人惊讶的影响：

```c++
template<typename T> void f3(T&& val){
    T t = val;         //拷贝还是绑定一个引用？
    t = fcn(t);        //赋值只能改变t还是既改变t有改变val？
    if (val == t) { }  //若T为引用类型，则一直为true
}
```

​    当我们对一个右值调用f3时，例如字面常量42，T为int，在此情况下，局部变量t的类型为int，val不会随着t的改变而改变；当对一个左值i调用f3时，T为int&，局部变量t的类型为int&，val会随着t的改变而改变，if会一直进行下去。

​    我们通常使用下面两种方式进行重载：

```c++
template<typename T> void f(T&&);       //绑定到非const右值
template<typename T> void f(const T&);  //左值和const右值
```

##### 6.理解std::move

​    标准库move函数是使用右值引用的模板的一个很好的例子。

###### 1.std::move是如何定义的

​    标准库是这样定义move的：

```c++
//在返回类型和类型转换中也要用到typename
template<typename T>
typename remove_reference<T>::type&& move(T&& t){
    return static_cast<typename remove_reference<T>::type&&>(t);
}
```

​    我们既可以传递给move一个左值，也可以传递一个右值：

```c++
string s1("hello"), s2;
s2 = std::move(string("bye!"));  //正确：从一个右值移动数据
s2 = std::move(s1);              //正确：但在赋值之后，s1的值是不确定的
```

###### 2.std::move是如何工作的

​    在第一个赋值中，我们传递一个右值，过程如下：

-   推断T的类型为string
-   因此，remove_reference用string实例化
-   remove_reference<string>的type成员是string
-   move的返回类型是string&&
-   move的函数参数t的类型是string&&

​    因此，调用实例化move<string>，即函数：

```c++
string&& move(string&& t);
```

​    在第二个赋值，我们传递一个左值，过程如下：

-   推断出T的类型为string&
-   因此，remove_reference用string&进行实例化
-   remove_reference<string&>的type成员是string
-   move的返回类型是string&&
-   move的函数参数t实例化为string& &&，会折叠为string&

​    因此，调用实例化move<string&>，即函数：

```c++
string&& move(string& t);
```

###### 3.从一个左值static_cast到一个右值引用是允许的

​    虽然不能隐式地将一个左值转换为右值引用，但是我们可以用static_cast显式地将一个左值转换为一个右值引用。

##### 7.转发

​    某些函数需要将一个或多个实参连同类型不变地转发给其他函数。在此情况下，我们需要保持被转发实参的所有性质，包括实参类型是否是const以及实参是左值还是右值。

```c++
//接受一个可调用对象和另外两个参数的模板
//对“翻转”的参数调用给定的可调用对象
//flip1是一个不完整的实现：顶层const和引用丢失了
template<typename F, typename T1, typename T2>
void flip1(F f, T1 t1, T2 t2){
    f(t2, t1);
}
```

​    这个函数一般情况下工作的很好，但当我们希望用它调用一个接受引用参数的函数时就会出现问题：

```c++
void f(int v1, int& v2){
    cout << v1 << " " << v2 << endl;
}
f(42, i);         //f改变了实参i
flip1(f, j, 42);  //通过flips调用f不会改变j
```

​    因此，这个flip1调用会实例化为：

```c++
void flip1(void (*fcn)(int, int&), int t1, int t2);
```

###### 1.定义能保持类型信息的函数参数

​    通过将一个函数参数定义为一个只想模板类型参数的右值引用，我们可以保持其对应实参的所有类型信息。而使用引用参数（无论是左值还是右值）使得我们可以保持const属性，因为在引用类型中const是最低层的。如果我们将函数定义为T1&&和T2&&，通过引用折叠就可以保持翻转实参的左值/右值属性：

```c++
template<typename F, typename T1, typename T2>
void flip2(F f, T1&& t1, T2&& t2){
    f(t2, t1);
}
```

​    **ps：如果一个函数参数是指向模板参数的右值引用（T&&），她对应的实参的const属性和左值/右值属性都将得到保持。**

​    这个版本的flip2只解决了对左值引用的函数工作，但不能用于接收右值引用参数的函数：

```c++
void g(int&& i, int&& j){
    cout << i << " " << j << endl;
}
flip2(g, i, 42);  //错误：不能从左值实例化int&&
```

###### 2.在调用中使用std::forward保持类型信息

​    与move不同，forward必须通过显式模板实参来调用。forward返回该显式实参类型的右值引用。即，forward<T>的返回类型是T&&。

​    通常情况下，我们使用forward传递那些定义为模板类型的右值引用的函数参数。通过其返回类型上的引用折叠，forward可以保持给定实参的左值/右值属性：

```c++
template<typename Type> intermediary(Type&& arg){
    finalFcn(std::forward<Type>(arg));
    //...
}
```

​    **ps：当用于一个指向模板参数类型的右值引用函数参数（T&&）时，forward会保持实参类型的所有细节。**

​    使用forward，我们可以再次重写翻转函数：

```c++
template<typename F, typename T1, typename T2>
void flip(F f, T1 t1, T2 t2){
    f(std::forward<T2>(t2), std::forward<T1>(t1));
}
```

#### 16.3 重载与模板

​    如果重载涉及函数模板，则函数匹配规则会在一下几方面收到影响：

-   对于一个调用，其候选函数包括所有模板实参推断成功的函数模板实例
-   候选的函数模板总是可行的，因为模板实参推断会排除任何不可行的模板
-   与往常一样，可行函数（模板与非模板）按类型转换来排序。当然，可以用于函数模板调用的类型转换是非常有限的
-   与往常一样，如果恰有一个函数提供比任何函数都跟好的匹配，则选择此函数。但是，如果有多个函数提供同样好的匹配，则：
    -   如果同样好的函数只有一个是非模板函数，则选择此函数
    -   如果同样好的函数中没有非模板函数，而有多个模板函数，且其中一个模板比其他模板更特例化，则选择此模板
    -   否则，此调用有歧义

###### 1.编写重载模板

```c++
//打印任何我们不能处理的类型
template<typename T> string debug_rep(const T& t){
    ostringstream ret;
    ret << t;
    return ret.str();  //返回ret绑定的string的一个副本
}
```

​    接下来，我们将定义打印指针的debug_rep版本：

```c++
template<typename T> string debug_rep(T* p){
    ostringstream ret;
    res << "pointer : " << p;  //打印指针本身
    if(p)
        ret << " " << debug_rep(*p);  //打印p指向的值
    else
        ret << " null pointer";
    return ret.str();  //返回ret绑定的string的一个副本
}
```

###### 2.多个可行模板

​    我们考虑下面的调用：

```c++
const string* sp = &s;
cout << debug_rep(sp) << endl;
```

​    此例中的两个模板都是可行的，而且两个都是精确匹配：

-   debug_ret(const string*&)，由第一个版本的debug_ret实例化而来，T绑定到string *
-   debug_ret(const string*)，由第二个版本的debug_ret实例化而来，T绑定到const string *

​    在这种情况下，正常函数匹配规则无法区分这两个函数。根据重载函数模板的特殊规则，此调用被解析为debug_rep(T*)，即，更特例化的版本。

​    模板debug_rep(const T&)本质上可以用与任何类型，包括指针类型，此模板比debug_ret(T*)更通用，后者只能用于指针类型。

​    **ps：当有多个重载模板对一个调用提供同样好的匹配时，应该选择更特例化的版本。**

###### 3.非模板和模板重载

​    作为下一个例子，我们将定义一个普通非模板的debug_rep来打印双引号包围的string：

```c++
string debug_rep(const string& s){
    return '"' + s + '"';
}
```

​    我们对一个string调用：

```c++
string s("hi");
cout << debug_rep(s) << endl;  //调用的是非模板的debug_rep(const string& s)
```

​    **ps：对一个调用，如果一个非函数模板和一个模板函数都提供同样好的匹配，则选择非模板版本。**

###### 4.重载模板与类型转换

​    还有一种情况我们目前为止还未讨论：C风格字符串指针和字符串字面常量：

```c++
cout << debug_rep("hello") << endl;  //调用debug_rep(T*)
```

​    本例中三个版本的debug_rep都是可行的：

-   debug_rep(const T&)，T被绑定到char[10]
-   debug_rep(T*)，T被绑定到const char
-   debug_rep(const string&) ，要求从const char*转换到string

​    虽然第二个模板函数需要进行一次类型转换，但是对于函数匹配来说，这种转换被认为是精确匹配；非模板版本需要进行一次用户定义的类型转换，因此没有精确匹配那么好，所以在两个模板函数中选择，和之前一样，T*更特例化，所以选择T *版本。

​    如果我们希望将字符指针按string处理，可以定义另外两个非模板重载版本：

```c++
string debug_rep(char* p){
    return debug_rep(string(p));
}
string debug_rep(const char* p){
    return debug_rep(string(p));
}
```

###### 5.缺少声明可能导致程序行为异常

​    为了使用char*版本的debug_rep，在定义此版本前，debug_rep(const string&)的声明必须在作用域中。

​    **ps：在定义任何函数前，记得声明所有重载的函数版本，这样就不必担心编译器由于遇到你希望调用的函数而实例化一个并非你需要的版本。**

#### 16.4 可变参数模板

​    一个可变参数模板就是接受一个可变数目参数的模板函数或模板类。可变数目的参数被称为参数包。存在两种参数包：

-   模板参数包：表示零个或多个模板参数
-   函数参数包：表示零个或多个函数参数

​    我们用一个省略号来指出一个模板参数或函数参数表示一个包。在一个模板参数列表中，class...或typename...指出接下来的参数表示零个或多个类型的列表。例如：

```c++
//Args是一个模板参数包，rest是一个函数参数包
template<typename T, typename... Args>;
void foo(const T& t, const Args& ... rest);
```

​    与往常一样，编译器从函数的实参推断模板参数类型，对于一个可变参数模板，编译器还会推断包中参数的数目：

```c++
int i = 0; double d = 3.14; string s = "how now brown cow";
foo(i, s, 42, d);  //包中右三个参数
foo(s, 42, "hi");
foo(d, s);
foo("hi");         //空包
```

​    编译器实例化出四个版本的foo：

```
void foo(const int&, const string&, const int&, const double&);
void foo(const string&, const int&, const char[3]&);
void foo(const double&, const string*);
void foo(const char[3]&);
```

​    在每个实例中，T都是根据第一个实参的类型推断出来的，剩下的实参（如果有的话）提供额外实参的数目和类型。

###### 1.sizeof...运算符

​    当我们需要知道包中有多少元素时，可以使用sizeof...运算符：

```c++
template<typename... Args> void g(Args... args){
    cout << sizeof...(Args) << endl;  //类型参数的数目
    cout << sizeof...(args) << endl;  //函数参数的数目
}
```

##### 1.编写可变参数函数模板 

​    可变函数通常是递归的。第一步调用处理包中的第一个实参，然后用剩余实参调用自身，我们print函数也是这样的模式，每次递归调用将第二个实参打印到第一个实参表示的流中。为了中止递归，我们还需要一个非可变参数的print函数，他接受一个流和一个对象：

```c++
template<typename T>
ostream& print(ostream& os, const T& t){
    return os << t;
}
//包中除了最后一个元素之外的其他元素都会调用这个版本的print
template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args&... rest){
    os << t << ", ";            //打印第一个实参
    return print(os, rest...);  //递归调用，打印其他实参 
}
```

​    **ps：当定义可变参数版本的print时，非可变参数版本的声明必须在作用域中。否则会变成无限递归。**

##### 2.包扩展

​    对于一个参数包，除了获取其大小外，我们能对他做的唯一的事情就是扩展，当扩展一个包时，我们还需要提供用于每个扩展元素的模式。扩展一个包就是将他分解为构成的元素，对每个元素应用模式，获得扩展后的列表，我们通过在模式邮编放一个省略号来触发扩展操作：

​    例如，我们print函数包括两个扩展：

```c++
template<typename T, typename... Args>
ostream& print(ostream& os, const T& t, const Args&... rest){  //扩展Args
    os << t << ", ";            
    return print(os, rest...);  //扩展rest
}
```

###### 1.理解包扩展

​    print中的函数参数包扩展仅仅将包扩展为其构成元素，c++语言还允许更复杂的扩展模式。例如，我们可以编写第二个可变参数函数，对每个实参调用debug_rep，然后调用print打印结果string：

```c++
template<typename... Args>
ostream& errorMsg(ostream& os, coust Args&... rest){
    //print(os, debug_rep(a1), debug_rep(a1)..., debug_rep(an))
    return print(os, debug_rep(rest)...);
}
```

​    下面的两个调用一样：

```c++
errorMsg(cerr, fcnName, code.num(), otherData, "other", item);
print(cerr, debug_rep(fcnName), ..., debug_rep(item));
```

##### 3.转发参数包

​    新标准下，我们可以组合使用可变参数模板和forward机制来编写函数，实现将其他参不变地传递给其他函数。

```c++
template<class... Args>
inline
void StrVec::emplace_check(Args&&... args){
    chk_n_alloc();
    alloc.construct(first_free++, std::forward<Args>(args)...);
}
```

#### 16.5 模板特例化

​    当我们不能（或不想）使用模板版本时，可以定义类或函数模板的一个特例化版本。

```c++
//第一个版本：可以比较任意两个类型
template<typename T> int compare(const T&, const T&);
//第二个版本处理字符串字面常量
template<size_t N, size_t M>
int compare(const char& [N], const char& [M]);
```

​    但是，只有我们传递给compare一个字符串字面值常量或是一个数组时，编译器才会调用第二个版本，如果传递给它字符指针，会调用第一个版本，因为我们无法将一个指针转换为一个数组的引用。

​    为了处理字符指针（而不是数组），可以为第一个版本的compare定义一个模板特例化版本。

###### 1.定义函数模板特例化

​    当我们特例化一个函数模板时，必须为原模版中的每个模板参数都提供实参。为了指出我们正在实例化一个模板，应使用关键字template后跟一个空尖括号对。空间括号对指出我们为原模版的所有模板参数提供实例：

```c++
template<>
int compare(const char* const& p1, const char* const& p2){
    return strcmp(p1, p2);
}
```

​    当我们定义一个特例化版本时，函数参数类型必须与一个先声明的模板中对应的类型匹配。本例中我们特例化：

```c++
template<typename T> int compare(const T&, const T&);
```

###### 2.函数重载与模板特例化

​    一个特例化版本本质是一个实例，而非函数名的一个重载版本，不影响函数匹配。

###### 3.类模板特例化

​    我们需要在原模版定义所在的命名空间中特例化它：

```c++
namespace std{  //打开std命名空间
template<>
struct hash<Sales_data>{
    typedef size_t result_type;
    typedef Sales_data argument_type;
    size_t operator()(const Sales_data& s) const;
};
size_t hash<Sales_data>::operator()(const Sales_data& s) const{
    return hash<string>()(s.bookNo) ^
           hash<unsigned>()(s.utils_sold) ^
           hash<double>()(s.revenue);
}
}  //关闭std空间，没有分号
```

​    由于hash<Sales_data>使用了Sales_data的私有成员，我们必须将他声明为Sales_data的友元：

```c++
template<class T> class std::hash;
class Sales_data{
    friend class std::hash<Sales_data>;
}
```

###### 4.类模板部分特例化

​    与函数模板不同，类模板的特例化不必为所有模板参数提供实参。我们可以至指定一部分而非所有模板参数，或是参数的一部分而非全部特性。一个类模板的部分特例化本身是一个模板，使用它的用户还必须为那些特例化版本中未指定的模板参数提供实参。

​    **ps：我们只能部分特例化类模板，而不能部分特例化函数模板。**

​    标准库remove_reference类型，该模板是通过一系列特例化版本来完成其他功能了：

```c++
//原始的、最通用的版本
template<class T> struct remove_reference{
    typedef T type;
};
//部分特例化版本，将用于左值引用和右值引用
template<class T> struct remove_reference<T&> 
{ typedef T type; };
template<class T> struct remove_reference<T&&>
{ typedef T type; };
```

​    由于一个部分特例化版本本质是一个模板，与往常一样，我们首先定义模板参数。部分特例化版本的模板参数列是原始模板的参数列表的一个子集或者是一个特例化版本。在本例中，特例化版本的模板参数的数目和原始模板相同，但是类型不同。两个特例化版本分别用于左值引用和右值引用类型：

```c++
int i;
remove_reference<decltype(42)>::type a;  //原始模板
remove_reference<decltype(i)>::type b;   //使用T&特例化模板
remove_reference<decltype(std::move(i))>::type c;  //使用T&&特例化模板
```

###### 5.特例化成员而不是类

​    我们可以之特例化特定成员函数而不是特例化整个模板。例如，如果Foo是一个模板类，包含一个成员Bar，我们可以之特例化该成员：

```c++
template<typename T> struct Foo{
    Foo(const T& t = T()) : mem(t) {}
    void Bar() { }
    T mem;
};
template<>
void Foo<int>::Bar(){
}
```

​    本例中我们只特例化Foo<int>类的一个成员，其他成员由Foo模板提供：

```c++
Foo<string>fs;  //实例化Foo<string>::Foo()
Fs.Bar();       //实例化Foo<string>::Bar()
Foo<int> fi;    //实例化Foo<int>::Foo()
fi.Bar();       //使用我们特例化版本的Foo<int>::Bar()
```

