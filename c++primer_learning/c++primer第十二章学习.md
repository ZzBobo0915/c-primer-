# c++primer学习

## 第12章 动态内存

​    除了静态内存和栈内存，每个程序还拥有一个内存池。这部分内存被称作自由空间或堆。程序用堆来存储动态分配的对象--即那些在程序运行时分配的对象。动态对象的生存期由程序来控制，也就是说，当动态对象不再使用时，我们的代码必须显式地销毁他们。

#### 12.1 动态内存与智能指针

​    c++中，动态内存的管理是通过一对运算符来完成的：new，在动态内存中为对象分配空间并返回一个指向该对象的指针，我们可以选择对象进行初始化：delete，接受一个动态对象的指针，销毁该对象，并释放与之关联的内存。

​    新的标准库提供了两种智能指针类型来管理动态对象。智能指针最重要的区别就是她负责自动释放所指向的对象：shared_ptr允许多个指针指向同一个对象；unique_ptr则“独占”所指向的对象。标准库还定义了一种名为weak_ptr的伴随类，它是一种弱引用，指向shared_ptr所管理的对象。这三种类型都定义在memeory头文件中。

##### 1.shared_ptr类

​    类似vector，智能指针也是模板，默认初始化的智能指针中保存着一个空指针：

```c++
shared_ptr<string> p1;
shared_ptr<list<int>> p2;
```

![12-1](E:\朱智博-文件杂项\c++\c++primer学习图片\12-1.png)

###### 1.make_shared函数

​    最安全的分配和使用动态内存的方法时调用一个名为make_shared的标准库函数：

```c++
shared_ptr<int> p3 = make_shared<int>(42);
```

​    如果我们不传递任何参数，对象会进行值初始化，我们也可用auto定义一个对象来保存make_shared的结果：

```c++
auto p6 = make_shared<vector<string>>();
```

###### 2.shared_ptr的拷贝和赋值

​    当进行拷贝或赋值操作时，每个shared_ptr都会记录有多少个其他shared_ptr指向相同的对象：

```c++
auto p = make_shared<int>(42);
auto q(p);  //p和q指向相同的对象，此对象有两个引用者
```

​    我们认为每个shared_ptr都有一个关联的计数器，通常称其为引用计数。无论何时我们拷贝一个shared_ptr，计数器都会递增。当我们给shared_ptr赋予一个新值或是shared_ptr被销毁时，计数器会递减。一旦一个shared_ptr的计数器变为0，它会自动释放自己所管理的对象：

```c++
auto r = make_shared<int>(42);
r = q;
//给r赋值，令它指向另一个地址
//递增q指向的对象的引用计数
//递减r原来指向的对象的引用计数
//r原来指向的对象已没有引用者，会自动释放
```

###### 3.shared_ptr自动销毁所管理的对象

​    shared_ptr通过析构函数完成销毁工作。shared_ptr的析构函数会递减他所指向对象的引用计数，如果引用计数为0，shared_ptr的析构函数就会销毁对象，并释放他占用的内存。

###### 4.shared_ptr还会自动释放相关联的内存

   如果你将shared_ptr存放于一个容器中，而后不再需要全部元素，而只是用其中一部分，要记得用erase删除不再需要的那些元素。

###### 5.使用了动态生存周期的资源的类

​    主要有三种原因会使用动态内存：

1. 程序不知道自己需要使用多少对象
2. 程序不知道所需对象的准确类型
3. 程序需要在多个对象间共享数据

​    **ps：使用动态内存的一个常见原因时允许多个对象共享相同的状态。**

##### 2.直接管理内存

   c++定义了两个运算符来分配和释放动态内存。运算符new分配内存，delete释放new分配的问题。

###### 1.使用new动态分配和初始化对象

​    在自由空间分配的内存是无名的，因此new无法为其分配的对象命名，而是返回一个指向该对象的指针：

```c++
int* pi = new int;        //pi指向一个未初始化的无名对象
int* pi = new int(1024);  //pi指向对象值为1024
int* pi = new int();      //pi指向一个值初始化为0的对象
```

​    如果我们提供一个括号包围的初始化器，就可以使用auto来推断想要分配的内存：

```c++
auto p1 = new auto(obj);
```

###### 2.动态分配的const对象

​    用new分配const对象是合法的，且一个动态分配的const对象必须进行初始化：

```c++
const int* pci = new const int(1024);
```

###### 3.内存耗尽

​    一旦一个程序用光了它所有可用的内存，new表达式就会失败。如果new失败，会抛出一个bad_alloc的异常。我们可以改变使用new的方式来阻止他抛出异常：

```c++
int* p1 = new int;            //如果分配失败，抛出std::bad_alloc异常
int* p2 = new (nothrow) int;  //如果分配失败，会返回一个空指针
```

​    nothrow和bad_alloc都定义在头文件new中。

###### 4.释放动态内存

​    我们通过delete将动态内存归还给系统：

```c++
delete p;  //p必须指向一个动态分配的对象或是一个空指针
```

​    delete也执行两个动作：销毁给定的指针指向的对象、释放对应的内存。

###### 5.指针值和delete

​    我们传递给delete的指针必须指向动态分配的内存，或者是一个空指针。释放一块并非new分配的内存，或者将相同的指针值释放多次，其行为是未定义的：

```c++
int i, *pil = &i, *pi2 = nullptr;
double* pd = new double(33), *pd2 = pd;
delete i;    //错误：i不是一个指针
delete pil;  //未定义：pil指向一个局部变量
delete pd;   //正确
delete pd2;  //未定义：pd2指向的内存已经被释放了
delete pi2;  //正确，释放一个空指针总是没有错误的
```

###### 6.动态对象的生存期直到被释放为止

​    对于一个由内置指针管理的动态对象，直到被显式释放之前他都是存在的。返回动态内存的指针的函数给其调用者增加了一个额外负担--调用者必须记得释放内存。

```c++
Foo* factory(T arg){
    return new Foo(arg);
}
void use_factory(T arg){
    Foo* p = factory(arg);
    ...
    delete(p);  //记得释放p
}
```

###### 7.delete之后重置指针值

​    当我们delete一个指针后，指针值就变为无效了。虽然指针无效，但很多机器上指针依然保存的(已经释放了的)动态内存的地址。在delete后，指针就变成了空悬指针。

​    有一种方法可以避免空悬指针，即指向一块曾经保存数据对象但现在已经无效的内存的指针。在指针即将要离开其作用域之前释放掉他关联的内存。如果我们需要保留指针，可以在delete之后将nullptr赋予：

```c++
int* p(new int(42));
auto q = p;  //p.q指向相同的内存
delete p;    //p.q都无效
p = nullptr; //指出p不在绑定任何对象
```

##### 3.shared_ptr和new结合使用

​    我们可以使用new返回的指针来初始化智能指针，因为接受指针参数的智能指针构造函数是explicit的，因此我们不能隐式转换，必须使用直接初始化的形式：

```c++
shared_ptr<double> p1 = new int(42);  //错误
shared_ptr<int> p2(new int(42));      //正确
```

​    下表是定义和改变shared_ptr的其他方法：

![12-2](E:\朱智博-文件杂项\c++\c++primer学习图片\12-2.png)

![12-3](E:\朱智博-文件杂项\c++\c++primer学习图片\12-3.png)

###### 1.不要混合使用普通指针和智能指针

​    当将一个shared_ptr绑定到一个普通指针时，我们就将内存的管理责任交给了这个shared_ptr。一旦这样做了，我们就不应该在使用内置指针来访问shared_ptr所指向的内存的。

###### 2.也不要使用get初始化另一个智能指针或者为智能指针赋值

​    智能指针定义了一个get的函数，他返回一个内置指针，指向智能指针管理的对象。get用来将指针的访问权限传递给代码，你只有确定代码不会delete指针的情况下，才能使用get。

###### 3.其他shared_ptr操作

​    我们可以用reset将一个新的指针赋予一个shared_ptr：

```c++
p = new int(1024);       //错误：不能讲一个指针赋予shared_ptr
p.reset(new int(1024));  //正确：p指向一个新对象
```

​    与赋值类似，reset会更新引用计数，若果需要的话会释放p指向的对象：

```c++
if (!p.unique())
    p.reset(new string(*p));  //我们不是唯一用户，分配新的拷贝
*p += newVal;                 //我们知道自己是唯一的用户，可以改变对象的值
```

##### 4.智能指针和异常

######  1.智能指针和哑类

​    那些分配了资源而又没有定义析构函数来释放这些资源的类，可能会遇到与使用动态内存相同的错误--程序员非常容易忘记释放资源。类似的，如果在资源分配和释放之间发生了异常，程序也会发生资源泄露。

###### 2.使用我们自己的释放操作

​    为了使shared_ptr来管理，我们必须首先定义一个函数来替代delete。

##### 5.unique_ptr

​    一个unique_ptr拥有他所指向的对象，且某个时刻只能有一个unique_ptr指向一个给定对象。当我们定义一个unique_ptr时，需要将其绑定到一个new返回的指针上，了类似shared_ptr，初始化unique_ptr时必须采用直接初始化：

```c++
unique_ptr<string> p1(new string("zhuzhibo"));
unique_ptr<string> p2(p1);  //错误，unique_ptr不支持拷贝
unique_ptr<string> p3;
p3 = p2;                    //错误：不支持赋值
```

![12-4](E:\朱智博-文件杂项\c++\c++primer学习图片\12-4.png)

​    虽然我们不能拷贝或赋值unique_ptr，但可以通过调用release火reset将指针从一个非const的unique_ptr转移到另一个unique：

```c++
unique_ptr<string> p2(p1.release());            //将所有权从p1转为p2，p1置空
unique_ptr<string> p3(new string("zhuzhibo"));  
p2.reset(p3.release());                         //reset释放了p2原来指向的内存
```

###### 1.传递unique_ptr参数和返回unique_ptr

​    不能拷贝unique_ptr的规则有一条例外：我们可以拷贝或赋值一个将要被销毁的unique_ptr，最常见的例子就是从函数返回一个unique_ptr:

```c++
unique_ptr<int> clone(int p){
    return unique_ptr<int>(new int(p));
}
```

###### 2.向unique_ptr传递删除器

​    unique_ptr默认情况下用delete释放他指向的对象。与shared_ptr一样，我们可以重载一个unique_ptr中默认的删除器。

​    重载一个 unique_ptr 中的删除器会影响到 unique_ptr 类型以及如何构造（或reset）该类型的对象。与重载关联容器的比较操作类似，我们必须在尖括号中unique_ptr指向类型之后提供删除器类型。在创建或reset一个这种unique_ptr类型的对象时，必须提供一个指定类型的可调用对象（删除器）∶

```c++
//p指向一个类型为objT的对象，并使用一个类型为delT的对象释放objT对象
//它会调用一个名为fcn的delT类型对象
unique_ptr<objT, delT> p(new objT, fcn);
```

##### 6.weak_ptr

​    weak_ptr是一个不控制所指对象生存期的智能指针，他指向由一个shared_ptr管理的对象。将一个weak_ptr绑定到一个shared_ptr不会改变他的引用计数。

![12-5](E:\朱智博-文件杂项\c++\c++primer学习图片\12-5.png)

###### 1.核查指针类

​        由于对象可能不存在，我们不能使用weak_ptr直接访问对象，而必须调用lock。此函数检查指向的对象是否存在。

```c++
if (shared_ptr<int> np = wp.lock()){ }
```

#### 12.2 动态数组

​    new和delete运算符一次分配/释放一个对象，但某些应用需要一次为很多对象分配内存的功能。为了支持这种需求，c++和stl提供了两种一次分配一个对象数组的方法：可以new分配一个对象数组；可以用allocator类进行分配。

##### 1.new和数组

​    new分配一个对象数组，要在类型名后跟一对方括号，指明分配对象的数目：

```c++
int *pia = new int[get_size()];  //括号里的大小必须是整型，但不必是常量
//下面两条语句等价于 int *p = new int[42];
typedef int arrT[42];
int *p = new arrT;
```

​    我们通常称new T[]分配的内存为“动态数组”，但这种叫法有点误导。当我们new分配一个数组时，我们得到的是一个数组元素类型的指针。new返回的是一个元素类型的指针。

###### 1.初始化动态分配对象的数组

​    默认情况下，new分配的对象不管是单个还是数组，都是默认初始化的，新标准中我们还可以提供一个元素初始化器的花括号列表：

```c++
int *pia = new int[10];     //10个未初始化的int
int *pia2 = new int[10]();  //10个值初始化为0的int
int *pia3 = new int[10]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
```

​    虽然我们用空括号对数组中的元素进行值初始化，但是不能在括号中给出初始化器，这意味着不能用auto分配数组。

###### 2.动态分配一个空数组是合法的

​    当我们用new分配一个大小为0的数组时，new'返回一个合法的非空指针。我们可以把这个指针像尾后指针一样使用，可以+0，-0，但是不能解引用，因为他不指向任何元素。

###### 3.释放动态数组

​    我们使用一种特殊的delete--在指针前加上一个空方括号对：

```c++
delete p;
delete [] pa; 
```

​    数组中的元素按照逆序销毁。如果我们在delete一个数组指针时忘记加入了[]，或者在delete一个单一对象的指针时使用了方括号，编译器可能不会给出警告。

###### 4.智能指针和动态数组

​     标准库提供了一个可以管理分配的数组的unique_ptr版本：

```c++
unique_ptr<int[]> ip(new int[10]);
up.release();
```

![12-6](E:\朱智博-文件杂项\c++\c++primer学习图片\12-6.png)

​    与unique_ptr相比，shared_ptr不直接支持管理动态数组，必须提供自己定义的删除器：

```c++
shared_ptr<int> sp(new int[10], [](int *p) { delete [] p; } );
sp.reset();
```

##### 2.allocator类

###### 1.allocator类

​    标准库allocator类定义在头文件memory中，他提供一种类型感知的内存分配方法，他分配的内存是原始的、未构造的。类似vector，allocator也是一个模板，我们需要指明对象类型：

```c++
allocator<string> alloc;
auto const p = alloc.allocate(n);  //分配n个未初始化的string
```

![12-7](E:\朱智博-文件杂项\c++\c++primer学习图片\12-7.png)

###### 2.allocator分配未构造的内存

​    allocator分配的内存是未构造的。我们需要在此内存中构造对象。construct接受一个指针和额外参数：

```c++
auto q = p;
alloc.construct(q++);           //*q为字符串
alloc.construct(q++, 10, 'c');  //*q为cccccccccc
alloc.construct(q++, "hi");     //*q为hi
```

​    当我们用完对象后，必须对每个构造的元素调用destroy来销毁他们：

```c++
while(q != p) alloc.destroy(--q);
```

​    **ps：我们只能对真正构造了的元素进行destroy操作。**

​    一旦元素被销毁后，就可以重新使用这部分内存来保存其他string，也可以调用deallocate来释放内存。

###### 3.拷贝和填充未初始化内存的算法

​    以下两个伴随算法可以在未初始化内存中创建对象：

![12-8](E:\朱智博-文件杂项\c++\c++primer学习图片\12-8.png)

```c++
auto p = alloc.allocate(vi.size() * 2);
auto q = uninitialized_copy(vi.begin(), vi.end(), p);
uninitialized_fill_n(q, vi.size(), 42);
```

