# c++primer学习

## 第10章 泛型算法

#### 10.1 概述

​    大多数算法都定义在头文件algorithm中，标准库还在头文件numeric中定义了一组数值泛型算法。一般这些算法一般不直接操作容器，而是遍历由两个迭代器指定的一个元素范围来进行操作。

```c++
int val = 42;
auto res = find(vec.begin(), vec.end(), val);
int ia = {1, 2 ,3, 4, 5, 6, 7, 8, 9, 0};
int val = 8;
int* result = find(begin(ia), end(ia), val);
```

​    **ps：迭代器零算法不依赖于容器，但算法依赖于元素类型的操作。**

#### 10.2 初识泛型算法

##### 1.只读算法

​    一些算法只会读取其输入范围内的元素，而不改变元素。包含find、count、accumulate、equal等。accumulate定义在头文件numeric中：

```c++
#include<numeric>
int sum = accumulate(vec.cbegin(), vec.cend(), 0);  //和的初值为0
string num = accumulate(v.cbegin(), v.cend(), string(""));  //和的初值为""
```

​    **ps：accumulate的第三个参数类型决定了函数中使用哪个加法运算符以及返回值的类型。**

​    equal用于确定两个序列是否保存相同的值：

```c++
//v2中的元素数目应该至少与v1一样多
equal(v1.cbegin(), v1.cend(), v2.cbegin());
```

##### 2.写容器元素的算法

​    当我们使用这些算法时，必须确保序列原大小至少不小于我们要求算法写入的元素数目。记住：算法不会执行容器操作，因此他们自身不可能改变容器的大小。比如，算法fill接受一对迭代器表示一个范围，还接受一个值作为第三个参数：

```c++
fill(vec.begin(), vec.end(), 0);
fill(vec.begin(), vec.end()+vec.size()/2, 10);
```

###### 1.算法不检查写操作

​    向目的位置迭代器写入数据的算法假定目的位置足够大，能容纳要写入的元素。

```c++
vector<int> vec;
fill_n(vec.begin(), vec.end(), 0);  //合法
fill_n(vec.begin(), 10, 0);  //不合法，因为vec中不存在元素
```

###### 2.介绍back_inserter

​    一种保证算法有足够的元素空间来容纳输出数据的方法是使用插入迭代器。插入迭代器是一种向容器中添加元素的迭代器：

```c++
vector<int> vec;
auto it = back_inserter(vec);
*it = 42;

vector<int> vec;
fill_n(back_inserter(vec), 10, 0);  //每一步迭代中fill_n向给定序列的一个元素赋值
```

###### 3.拷贝算法

​    拷贝算法是另一个向目的位置的迭代器指向的输出序列中的元素写入数据的算法。

```c++
int a1[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
int a2[sizeof(a1)/sizeof(&ai)];  //a2与a1大小一样
auto ret = copy(begin(a1), end(a1), a2);  //copy返回的是目的迭代器递增后的值
```

##### 3.重排容器元素的算法

​    某些算法会重排容器中元素的顺序。比如sort、unique：

```c++
//排序words
sort(words.begin(), words.end());
//unique重排输入范围，使得每一个单词只出现一次
auto end_unique = unique(word.begin(), words.end());
words.erase(emd_unique, words.end());
```

#### 10.3 定制操作

​    标准库还为这些算法定义了额外的版本，允许我们提供自己定义的操作来代替默认运算符。

##### 1.向算法传递函数

###### 1.谓词

​    谓词是一个可调用的表达式，去返回结果是一个能用做条件的值。标准库算法所使用的谓词分为两种：一元谓词、二元谓词。

​    接受一个二元谓词的sort版本用这个谓词替代<来比较元素：

```c++
bool isShorter(const string& s1, const string& s2){
    return s1.size() < s2.size();
}
sort(words.begin(), words.end(), isShorter);
```

###### 2.排序算法

​    stable_sort排序算法维持想等元素的原有位置：

```c++
//按长度重新排序，长度相同的单词维持字典序
stable_sort(words.begin(), words.end(), isShorter);
```

##### 2.lambda表达式

​    我们可以像一个算法传递任何类别的可调用对象。对一个对象或一个表达式，如果可以对其使用调用运算符，则称他为可调用的。

###### 1.介绍lambda

​    一个lambda表达式表示一个可调用的代码单元。我们可以将其理解为一个未命名的内联函数。与任何函数类似，一个lambda具有一个返回类型、一个参数列表和一个函数体。但与函数不同，lambda可能定义在函数内部。一个lambda表达式具有如下的形式：

`[capture list] (parameter list) -> return type { function body }`

​    我们可以忽略参数列表和返回类型，但必须永远包含捕获列表和函数体：

```c++
auto f = [] { return 42; };
cout << f() << endl;  //打印42
```

​    **ps：如果lambda的函数体包含单一的return语句之外的内容，且未指定返回类型，则返回void。**

###### 2.向lambda传递参数

​    与普通函数类似，调用一个lambda时给定的实参被用来初始化形参；又与普通函数不同，lambda不能有默认参数：

```c++
[] (const string& a, const string& b){ return a.size() < b.size(); }
```

###### 3.使用捕获列表

​    一个lambda通过将局部变量包含在捕获列表中指出将来会使用这些变量：

```c++
[sz] (const string& a) { return a.size() >= sz; }
```

​    **ps：一个lambda只有在其捕获列表中捕获一个他所在函数中的局部变量，才能在函数体内时使用该变量。**

###### 4.调用find_if

```c++
auto wc = find_if(words.begin(), words.end(), 
    [sz](const string& a) { return a.size() >= sz; });
```

###### 5.for_each算法

```c++
for_each(wc, words.end(),
    [](const string& s) { cont << s << " "; });
cout << endl;
```

​    **ps：捕获列表只用于局部非static变量，lambda可以直接使用局部static变量和他所在函数之外生命的名字。**

##### 3.lambda捕获和返回

​    当定义一个lambda时，编译器生成一个与lambda对应的新的(未命名的)类类型。

###### 1.值捕获

​    采用值捕获的前提是变量可以拷贝。与参数不同，被捕获的变量的值是在lambda创建时拷贝，而不是调用时拷贝：

```c++
void fun1(){
    size_t v1 = 42;
    auto f = [v1] { return v1; };
    v1 = 0;
    auto j = f();  //j=42，因为保存的是创建时的v1的拷贝
}
```

###### 2.引用捕获

​    我们定义lambda时可以采用引用方式捕获：

```c++
void func2(){
    size_t v1 = 42;
    auto f2 = [&v1] { return v1; };
    v1 = 0;
    auto j = f2();  //j=0
}
```

​    如果我们采用引用捕获的方式捕获一个变量，就必须确保被引用的对象在lambda执行的时候是存在的。

###### 3.隐式捕获

​    可以在捕获列表中写一个&或=，&表示采用引用捕获，=表示采用值捕获。

```c++
//os隐式捕获，引用捕获；c显示捕获，值捕获
for_each(words.begin(), words.end(), 
    [&, c](const string& s) { os << s << c; });
//os显式捕获，引用捕获；c隐式捕获，值捕获
for_each(words.begin(), words.end(), 
    [=, &o's](const string& s) { os << s << c; });
```

​    **ps：当我们混合使用隐式捕获和显示捕获时，捕获列表中的第一个元素必须是一个&或者=。**

###### 4.可变lambda

​    如果我们希望能改变一个被捕获的变量的值，就必须在参数列表首加上关键字mutable。因此，可变lambda能省略参数列表：

```c++
size_t v1 = 42;
auto f = [v1]() mutable { return ++v1; };
v1 = 0;
auto j = f();  //j=43
```

###### 5.指定lambda返回类型

​    当我们需要为一个lambda定义返回类型时，必须是使用尾置返回类型：

```c++
transform(v.begin(), v.end(), v.begin(), 
    [](int i) -> int { if (i < 0) return -i; else return i; });
```

##### 4.参数绑定

###### 1.标准库bind函数

​    他定义在头文件functional中，可以将bind看作一个通用的函数适配器，他接受一个可调用对象，生成一个新的可调用对象来”适应“原对象的参数列表。

`auto newCallable = bind(callable, arg_list);`其中newCallable本身是一个可调用对象，arg_list是一个逗号分隔的参数列表，对应给定的callabel参数。即当我们调用newCallable时，newCallabel会调用callable，并传递给arg_list中的参数。

###### 2.绑定check_size的sz参数

```c++
bool check_size(const string& s, string::size_type sz){
    return s.size() >= sz;
}
//check6是一个可调用对象，接受一个string类型的参数
//并用此string和值6来调用check_size
auto check6 = bind(check_size, _1, 6);
string s = "Hello";
bool b1 = check6(s);
```

###### 3.使用placeholders名字

​    名字_n都定义在一个名为palceholders的命名空间中，而这个命名空间本身定义在std命名空间中。 `using std::placeholders_1`

###### 4.用bind重排参数顺序

​    下面是用bind重排参数顺序的一个具体例子：

```c++
sort(words.begin(), words.end(), bind(isShorter, _2, _1));
```

###### 5.绑定引用参数

​    默认情况下，bind的那些不是占位符的参数呗拷贝到bind返回的可调用对象中。例如，为了替换一个引用方式捕获的ostream的lambda：

```c++
//错误：不能拷贝os
for_each(words.begin(), words.end(), bind(print, os, _1, ' '));
//使用ref函数,ref函数返回一个对象，包含给定的引用，此对象是可以拷贝的
for_each(words.begin(), words.end(), bind(print, ref(os), _1, ' '));
```

#### 10.4 再探迭代器

​    标准库在头文件iterator中还定义了额外几种迭代器。这些迭代器包括以下几种：

- 插入迭代器
- 流迭代器
- 反向迭代器
- 移动迭代器

##### 1.插入迭代器

![10-1](E:\朱智博-文件杂项\c++\c++primer学习图片\10-1.png)

​    插入器有三种类型，差异在于插入的位置：

- back_inserter：创建一个push_back的迭代器
- front_inserter：创建使用一个push_font的迭代器
- inserter：创建使用一个insert的迭代器

##### 2.iostream迭代器 

​    虽然iostream类型不是容器，但是还是定义了对IO类型对象的迭代器。istream_iterator读取输入流；ostream_iterator像一个输出流写数据。

###### 1.istream_iterator

​    当创建一个流迭代器时，必须指定迭代器将要读写的对象类型。

```c++
istream_iterator<int> int_it(cin);  //从cin读取int
istream_iterator<int> int_eof;      //尾后迭代器
iftream in("afile");        
istream_iterator<string> str_it(in);//从“afile”读取字符串
```

![10-2](E:\朱智博-文件杂项\c++\c++primer学习图片\10-2.png)

###### 2.ostream_iterator

![10-3](E:\朱智博-文件杂项\c++\c++primer学习图片\10-3.png)

​    我们可以用ostream_iterator来输出值的序列：

```c++
ostream_iterator<int> out_iter(cout, " ");
for (auto e : vec)
    out_iter = e;  //向out_iter赋值时可以忽略解引用和递增运算符
cout << endl;
```

##### 3.反向迭代器

​    反向迭代器就是在容器中从尾元素向首元素反向移动的迭代器。++就移动到前一个元素，--就移动到后一个元素。除了forword_list之外，其他容器都支持反向迭代器，可以通过调用rbegin，rend,crbegin,crend成员函数来获得反向迭代器。

​    ps：反向迭代器的目的是表示元素范围，而这些范围是不对称的，这导致一个重要的结果：当我们从一个普通迭代器初始化一个反向迭代器，或是给一个反向迭代器赋值时，结果迭代器与原迭代器指向的并不是相同的元素。

#### 10.5 泛型算法结构

​    算法所要求的迭代器操作可以分为5个迭代器类别：

![10-4](E:\朱智博-文件杂项\c++\c++primer学习图片\10-4.png)

##### 1.5类迭代器

​    迭代器类别：

- 输入迭代器：可以读取序列中的元素。一个输入迭代器必须支持：
  - ==和!=
  - ++
  - *只会出现在赋值运算符的右侧
  - ->，等价于(*it).member


- 输出迭代器：可以看作是输入迭代器功能上的补集--只写而不读元素。输出迭代器必须支持：
  - ++
  - *只会出现在赋值运算符的左侧
- 向前迭代器：可以读写元素。这类迭代器只能在序列中沿着一个方向移动，并且支持所有输入和输出迭代器的操作，可以多次读写同一个元素。
- 双向迭代器：可以正向/反向读写序列中的元素。除了支持前向迭代器的操作外，双向迭代器还支持前置和后置递减运算符(--)。
- 随机访问迭代器：提供在常量时间内访问序列中任意元素的能路。此类迭代器支持双向迭代器的所有操作，还支持以下操作：
  - <、<=、>和>=
  - +、+=、-和-=
  - -
  - 下标运算符

##### 2.算法形参模式

​    大多数算法具有如下4种形式之一：

```c++
alg(beg, end, other args);
alg(beg, end, dest, other args);
alg(beg, end, beg2, other args);
alg(beg, edn, beg2, end2, other args);
```

##### 3.算法命名规范

​    这些规范处理诸如：如何提供一个操作代替默认<或==运算符以及算法是将输出数据写入输入序列还是一个分离的目的位置等问题。

###### 1.一些算法使用重载形式传递一个谓词

​    函数的一个版本用元素类型的运算符来比较元素；另一个版本接受一个额外谓词参数，来替代<或=：

```c++
unique(beg, end);
unique(beg, end, comp);
```

###### 2._if版本的算法

​    接受一个元素值的算法通常有另一个不同命的(不是重载的)版本，该版本接受一个谓词代替元素值。接受谓词参数的算法都有附加的_if前缀：

```c++
find(beg, end, val);      //查找输入范围内val第一次出现得位置
find_if(beg, end, pred);  //查找第一个令pred为真的元素
```

###### 3、区分拷贝元素版本和不拷贝的版本

​    一般写到额外目的空间的算法都在名字后面附加一个_copy：

```c++
reverse(beg, end);
reverse_copy(beg, end, dest);
```

​    一些算法同时提供_copy和 _if版本。这些版本接受一个目的位置迭代器和一个谓词。

#### 10.6 特定容器算法

​    list和forward_list成员函数版本的算法：

![10-5](E:\朱智博-文件杂项\c++\c++primer学习图片\10-5.png)

![10-6](E:\朱智博-文件杂项\c++\c++primer学习图片\10-6.png)

​    链表类型还定义了splice算法：

![10-7](E:\朱智博-文件杂项\c++\c++primer学习图片\10-7.png)