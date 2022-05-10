# c++primer学习

## 第9章 顺序容器

#### 9.1 顺序容器概述

![9-1](E:\朱智博-文件杂项\c++\c++primer学习图片\9-1.png)

​    **ps：通常使用vector是最好的选择，除非你有很好的理由选择其他容器。**

#### 9.2 容器库概览

​    一般来说，每个容器都定义了一个在头文件中，文件名和类型相同。

```c++
#include<vector>
vector<int> v;
```

​    对容器可以保存的元素类型的限制：几乎可以保存任何类型的元素。

```c++
list<Sales_data> lst;
deque<double> de;
vector<vector<int>> v;
```

![9-2](E:\朱智博-文件杂项\c++\c++primer学习图片\9-2.png)

![9-3](E:\朱智博-文件杂项\c++\c++primer学习图片\9-3.png)

##### 1.迭代器

​    与容器一样，迭代器有着公共的接口：如果一个迭代器提供某个操作，那么所有提供相同操作的迭代器对这个操作的实现方式都是相同的。

​    迭代器范围的概念是标准库的基础。一个迭代器范围有一对迭代器显示，两个迭代器分别指向同一个容器中的元素或者尾后位置。通常被称作begin,end，表示为[begin, end)左闭右开区间。

##### 2.容器类型成员

​    我们已经使用过其中三种：size_type、iterator、const_iterator。使用这些类型成员需要显式使用其类名：

```c++
list<string>::iterator iter;
vector<int>::difference_type count;
```

##### 3.begin和end成员

​    begin和end操作生成值相同其中第一个元素和尾元素之后位置的迭代器。

​    begin和end有多个版本：带r的版本返回反向迭代器、以c开头的版本返回const迭代器

```c++
list<string> a(10, "abc");
auto it1 = a.begin();    //iterator
auto it2 = a.rbegin();   //reverse_iterator
auto it3 = a.cbegin();   //const_iterator
auto it4 = a.crbegin();  //const_reverse_iterator
```

##### 4.容器定义和初始化

​    每个容器类型都定义了一个默认构造函数。除array外其他容器的默认构造函数都会创建一个指定类型的空容器，且都可以接受指定容器大小和元素初始值的参数。

![9-4](E:\朱智博-文件杂项\c++\c++primer学习图片\9-4.png)

​    **ps：当一个容器初始化为另一个容器的拷贝时，两个容器的容器类型和元素类型都必须相同。**

###### 1.列表初始化

​    我们可以对一个容器进行列表初始化：

```c++
list<string> authors = {"abc", "def", "ghi"};
```

###### 2.与顺序容器大小相关的勾构造函数

​    除array外，顺序容器加提供一个接受一个容器大小和元素初始值(可选)的构造函数：

```c++
vector<int> v(10, -1);
deque<string> deq(10);
```

​    **ps：只有顺序容器的构造函数才接受函数大小参数，关联容器并不支持。**

###### 3.标准库array具有固定大小

​    当定义一个array时，除了指定元素类型，还有指定容器大小：

```c++
array<int, 42>;
array<string, 10>;
```

​    为了使用array，我们还必须同时指定元素类型和大小：

```c++
array<int, 10>::size_type i;
```

​    如果对array进行列表初始化，初始值的数目必须小于或等于array的大小，所有剩余的元素都会进行值初始化：

```c++
array<int, 10> ia1;
array<int, 10> ia2 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};  //列表初始化
array<int, 10> ia3 = {42};  //ia3[0]为42，其他为0
```

​    array可以进行拷贝和赋值操作，前提是类型匹配：

```c++
int digs[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
int cpy[10] = digs;            //错误
array<int, 10> digits = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
array<int, 10> copy = digits;  //正确
```

##### 5.赋值和swap    ![9-5](E:\朱智博-文件杂项\c++\c++primer学习图片\9-5.png)

###### 1.使用assign(仅顺序容器)

​    assign允许我们从一个不同但相容的类型赋值，或者从容器的一个子序列赋值。

```c++
list<string> names;
vector<const char*> oldstyle;
names = oldstyle;                                  //错误：容器类型不匹配
names.assign(oldstyle.cbegin(), oldstyle.cend());  //正确，可以将const char*转为string
```

​    **ps：由于其旧元素被替换，因此传递给assign的迭代器不能只想调用assign的容器。**

###### 2.使用swap

​    swap操作交换两个相同类型容器的内容，swap只是交换了两个容器的内部数据结构，元素本身并未产生交换没所以交换操作可以在常数时间内完成。

```c++
vector<string> v1(10);
vector<string> v2(20);
swap(v1, v2);
```

##### 6.容器大小操作

   除了一个例外，每个容器类型都有按个与大小相关的操作。size、empty、max_size。

forward_list不支持size。

##### 7.关系运算符

​    每个容器类型都支持相等运算符(==和!=)，除了无需关联容器外的所有容器都支持关系运算符(>,>=,<,<=)。关系运算符左右两边的运算对象必须是保存相同类型元素的相同类型的容器。

​    **ps：如果保存的元素类型不支持所需运算符，那么保存这种元素的容器就不能使用对应的关系运算符。**

```c++
vector<Sales_data> v1, v2;  //已知Sales_data未定义<运算符
if (v1 < v2)  //错误
```

#### 9.3 顺序容器操作

##### 1.向顺序容器中添加元素

![9-6](E:\朱智博-文件杂项\c++\c++primer学习图片\9-6.png)

###### 1.使用push_back

```c++
vector<int> v;
v.push_back(0);
```

​    **ps：当我们用一个对象来初始化容器或者将一个对象插入到容器中时，实际上放入的是对象值的一个拷贝。**

###### 2.使用push_front

​    list、forward_list、deque容器还支持名为push_front的类似操作。此操作将元素插入到容器头部：

```c++
list<int> lst;
lst.push_front(1);
```

###### 3.在容器中的特定位置添加元素

​    insert成员允许我们在容器中任意位置插入0个或多个元素。

```c++
list<int> lst;
lst.insert(lst.begin(), 0);  //等价于 lst.push_front(0);
```

​    **ps：将元素插入到vector、deque和string中的任何位置都是合法的。然后速度可能会很慢。**

###### 4.插入范围内元素

​    除了第一个迭代器参数之外，insert还可以接受更多的参数，与容器的构造函数类似。

```c++
vector<string> v = {"zzb", "msy", "zr", "lgh"};
list<string> lst;
//将v的最后两个元素添加到lst的开始位置
lst.insert(lst.begin(), v.end()-2, v.end());
```

###### 5.使用insert的返回值

​    通过使用insert的返回值，我们可以在容器中一个特定的位置反复插入元素：

```c++
list<string> lst;
auto iter = lst.begin();
string word;
while (cin >> word){
    iter = lst.insert(iter, word);  //等价于调用 lst.push_front();
}
```

###### 6.使用emplace操作

​    新标准引入了三个新成员：emplace_front、emplace、emplace_back，这些操作构造而不是拷贝元素。

```c++
//Sales_data需要三个元素
list<Sales_data> lst;
lst.emplace_back("123123", 25, 1.99);  //正确
lst.push_back("123123", 25, 1.99);     //错误，vector没有三个参数的版本
```

​    **ps：emplace函数在容器中直接构造元素。传递给emplace函数的参数必须与元素类型的构造函数相匹配。**

##### 2.访问元素

![9-7](E:\朱智博-文件杂项\c++\c++primer学习图片\9-7.png)

###### 1.访问成员函数返回的是引用

​    在容器中访问元素的成员函数返回的都是引用。如果容器是一个const对象，则返回值是const引用；如果容器不是cosnt的，则返回值是普通应用，我们可以改变元素的值。

```c++
vecter<int> v(10, 0);
v[0] = 1;
v.at(1) = 2;
```

###### 2.下标操作和安全的随机访问

​    提供快速随机访问的容器(string, vector, deque, array)也提供下标运算符。给定的下标必须早范围内。如果我们需要确保下标是合法的，可以使用at成员函数。at在越界时，会抛出一个out_of_range异常。

##### 3.删除元素

![9-8](E:\朱智博-文件杂项\c++\c++primer学习图片\9-8.png)

###### 1.pop_front和pop_back成员函数

​    pop_front和pop_back分别对应删除首元素和尾元素。与元素访问成员函数类似，不能对一个空容器执行弹出操作。这些操作会返回void，如果需要弹出元素，要在删除前保存。

```c++
list<int> lst(10, 0);
int temp = lst.front();  //保存弹出元素
lst.pop_front();
```

###### 2.从容器内部删除元素

​    成员函数erase从容器中指定位置删除元素。我们可以删除有一个迭代器指定的单个元素，也可以删除由一对迭代器指定的区间内所有元素。

```c++
lst.erase(lst.begin());
lst.erase(lst.begin(), lst.end());  //等价于lst.clear();
```

##### 4.特殊的forward_list操作

​    有一个特殊的before_begin，指向首前迭代器。

![9-9](E:\朱智博-文件杂项\c++\c++primer学习图片\9-9.png)

##### 5.改变容器大小

​    我们可以使用resize来增加或缩小容器。

![9-10](E:\朱智博-文件杂项\c++\c++primer学习图片\9-10.png)

##### 6.容器操作可能使迭代器失效

​    向容器中添加元素和从容器中删除元素的操作可能会使指向容器元素的指针、引用或迭代器失效。一个失效的指针、引用或迭代器将不再表示任何元素。使用失效的指针、引用或迭代器是一种严重的程序设计错误，很可能引起与使用未初始化指针一样的问题。

###### 1.编写改变容器的循环程序

```c++
//删除偶数元素，复制每个奇数元素
vector<int> v = {0, 1, 2, 3, 4, 5 , 6, 7, 8, 9};
auto iter = v.begin();
while (iter != v.end()){
    if (*iter % 2) {
        iter = v.insert(iter, *iter);  //复制当前元素
        iter += 2;  //向前移动迭代器，跳过当前元素以及插入到它之前的元素
    }else {
        iter = v.erase(iter);  //删除偶数元素，不向前移动迭代器
    }
}
```

###### 2.不要保存end返回的迭代器

​    如果在一个循环中插入或删除deque,string或vector中的元素，不要缓存end返回的迭代器。

#### 9.4 vector对象是如何增长的

​    如果没有空间容纳新元素，容器不可能简单地将他添加到内存中其他位置——因为元素必须连续存储。容器必须分配新的内存空间来保存已有元素和新元素，将已有元素从旧位置移动到新空间中，然后添加新元素，释放旧存储空间。每添加一个新元素，vector就要执行这样的一次操作。

##### 1.管理容量的成员函数

![9-11](E:\朱智博-文件杂项\c++\c++primer学习图片\9-11.png)

​    **ps：reserve并不改变容器中元素的数量，他仅影响vector预先分配多大的内存空间。**

##### 2.capacity和size

​    size表示他已保存的元素数目，capacity则是不分配新的内存空间下最多可以保存多少元素。  

```c++
vector<int> v(24, 0);
cout << v.size();      //等于24
cout << v.capacity();  //应该大于24
v.reserve(50);
cout << v.size();      //等于24
cout << v.capacity();  //应该等于50
```

​    **ps：每个vector实现都可以选择自己的内存空间分配策略。但是必须遵循的一条原则是：只有当迫不得已时才可以重新分配新的空间。**

#### 9.5 额外的string操作

##### 1、构造string的其他方法

![9-12](E:\朱智博-文件杂项\c++\c++primer学习图片\9-12.png)

​    substr返回一个string，他是原始string的一部分或全部的拷贝。

```c++
string s("Hello World!");
string s1 = s.substr(0, 5);   //hello
string s2 = s.substr(6);      //world
string s3 = s.substr(6, 11);  //world
string s4 = s.substr(12);     //抛出out_of_range异常
```

##### 2.改变string的其他方法

​    string还提供了接受下表的insert和erase版本：

```c++
s.insert(s.size(), 5, '!');
s.erase(s.szie() - 5, 5);
```

​    string还接受C风格字符数组的insert和assign版本：

```c++
const char* cp = "Hello World!";
s.assign(cp, 7);           //"Hello W"
s.insert(s.size(), cp+7);  //"Hello World!"
```

###### 1.append和replace函数

​    append是在string末尾进行插入操作的一种简写形式：

```c++
string s("Hello World!");
s.append(" hello world!");  //等价于 s.insert(s.size(), " hello world!");
```

​    replace操作时调用erase和insert的一种简写形式：

```c++
string s("Hello World!");
s.replace(13, 3, " ab");  //等价于 s.insert(s.size(), " ab");
```

![9-13](E:\朱智博-文件杂项\c++\c++primer学习图片\9-13.png)

##### 3.string搜索操作

​    c++提供了6个不同的搜索函数，每个函数都有4个重载版本。每个搜索操作都返回一个string::size_type值，是一个unsigned类型，表示匹配发生位置的下标，如果匹配失败，则返回一个名为string::npos的static成员。

![9-14](E:\朱智博-文件杂项\c++\c++primer学习图片\9-14.png)

![9-15](E:\朱智博-文件杂项\c++\c++primer学习图片\9-15.png)

###### 1.指定在哪里搜索

​    我们可以传递给find操作一个可选的开始位置。这个可选的参数指出从哪个位置开始搜索。默认情况下，此位置被置为0。

```c++
string::size_type pos = 0;
while ((pos = name.find_first_of(numbers, pos)) != string::npos){
    ...
    ++pos;
}
```

######  2.逆向搜索

​    rfind成员函数搜索最后一个匹配，即字符串最靠右的出现位置：

```c++
string river("Mississippi");
auto first_pos = river.find("is");  //返回1
auto last_pos = river.rfind("is");  //返回4
```

##### 5.数值转换

​    新标准引入了多个函数，可以实现数值数据于标准库string之间的转换。

```c++
int i = 42;
string s = to_string(i);
double d = stod(s);
int i_0 = stoi(s);
//要转换为数值的string中第一个非空字符必须是数值中可能出现的字符
string s2 = "pi = 3.14";
d = stod(s2.substr(s2.find_first_of("+-.0123456789")));  //d=3.14
```

![9-16](E:\朱智博-文件杂项\c++\c++primer学习图片\9-16.png)

#### 9.6 容器适配器

​    除了顺序容器外，标准库还定义了三个顺序容器适配器：stack、queue、priority_queu	e。本质上，适配器是一种机制，一个容器适配器就受一种已有的容器类型，使其行为看起来像是一种不同的类型。

![9-17](E:\朱智博-文件杂项\c++\c++primer学习图片\9-17.png)

##### 1.定义一个适配器

​    每个适配器都定义了两个构造函数：默认构造函数和接受一个容器的构造函数拷贝该容器来初始化适配器。

```c++
stack<int> stk;
stack<int> stk(deq);  //deq是一个deque容器
```

​    默认情况下，stack和queue是基于deque实现的，priority_queue是在从vector之上实现的。我们可以在创建一个适配器的时候将一个命名的顺序容器作为第二个类型参数，来重载默认容器类型：

```c++
stack<string, vector<string>> str_stk;
```

###### 1.栈适配器

​    stack定义在stack头文件中。

![9-18](E:\朱智博-文件杂项\c++\c++primer学习图片\9-18.png)

​    **ps：stack因为只要求push_back,pop_back和back操作，因此可以使用除array和forward_list之外的任何容器类型来构造。**

###### 2.队列适配器

​    queue和priority_queue适配器定义在头文件queue中。

![9-19](E:\朱智博-文件杂项\c++\c++primer学习图片\9-19.png)

![9-20](E:\朱智博-文件杂项\c++\c++primer学习图片\9-20.png)

​    **ps：queue适配要求push_back,front和push_front，因此他可以构造于list或deque，但不能基于vector构造。priority_queue可以构造于vector或deque上，但不能基于list。**