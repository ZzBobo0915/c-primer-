# c++primer学习

## 第11章 关联容器

​    关联容器支持高效的关键字查找和访问。两个主要的关联容器类型map、set。

![11-1](E:\朱智博-文件杂项\c++\c++primer学习图片\11-1.png)

#### 11.1 使用关联容器

​    map是关键字-值对的集合。map类型通常被成为关联数组。

​    与之相对，set就是关键字的简单集合。

##### 1.使用map

​    一个经典的使用map的例子：

```c++
map<string, size_t> word_count;
string word;
while (cin >> word)
    ++word_count[word];
for (const auto& w : word_count)
    cout << w.first << " occurs " << w.second << ((w.second > 1) ? " times" : " time") << endl;  
```

​    map所使用pair(一个模板)用first保存关键字，用second成员保存对应的值。

##### 2.使用set

​    一个经典的使用set的例子：

```c++
map<string, size_t> word_count;
set<string> exClude = {"The","But","And","Or","An","A","the","but","and","or","an","a")};
string word;
while (cin >> word)
    if (exclude.find(word) == exclude.end()) ++word_count[word];
```

#### 11.2 关联容器概述

##### 1.定义关联容器

​    每个关联容器都定义了默认构造函数，他创建一个指定类型的空容器。我们也可以将关联容器初始化为零一个同类型的容器的拷贝，或是从一个值范围来初始化关联容器。在新标准下，我们也可以对关联容器进行值初始化。

​    一个map 或 set 中的关键字必须是唯一的，即，对于一个给定的关键字，只能有-个元素的关键字等于它。容器 multimap 和multiset没有此限制，它们都允许多个元素具有相同的关键字。例如，在我们用来统计单词数量的 map 中，每个单词只能有一个元素。另一方面，在一个词典中，一个特定单词则可具有多个与之关联的词义。 

##### 2.关键字类型的要求

​    为了指定使用自定义的操作，必须在定义关联容器类型时提供此操作的类型。用尖括号制定出要定义那种类型的容器，自定义的操作类型必须在尖括号中紧跟着元素类型的输出。

​    例如，我们不能直接定义Sales_data的multiset，因为Sales_data不包含<运算符。但是我们可以用一个函数定义multiset：

```c++
bool compareIsbn(const Sales_data& lhs, const Sales_data& rhs){
    return lhs.isbn() < rhs.isbn();
}
multiset<Sales_data, decltype(compareIsbn)*> bookstore(compareIsbn);
```

​    **ps：当我们用decltype来过的一个函数指针类型时，必须加上一个*来指出我们要使用一个给定函数类型的指针。**

##### 3.pair类型

​    他定义在头文件utility中。一个pair保存两个数据成员。pair的默认构造函数对数据成员进行初始化且数据成员时public的，两个成员分别命名为first和second。

![11-2](E:\朱智博-文件杂项\c++\c++primer学习图片\11-2.png)

#### 11.3 关联容器操作

​    ![11-3](E:\朱智博-文件杂项\c++\c++primer学习图片\11-3.png)

```c++
set<string>::valte_type v1;        //v1是一个string
set<string>::key_type v2;          //v2是一个string
map<string, int>::value_type v3;   //v3是一个pair<const string, int>
map<string, int>::key_type v4;     //v4是一个string
map<string, int>::mapped_type v5;  //v5是一个int
```

##### 1.关联容器迭代器

​    当解引用一个关联容器迭代器时，我们会得到一个类型为容器的value_type的值的引用。

​    **ps：一个map的value_pair是一个pair，我们可以改变pair的值，但不能改变关键字成员的值。**

##### 2.添加元素                                                       

###### 1.向set添加元素：

```c++
vector<int> ivec = {2, 4, 5, 8, 2, 4, 6, 8};
set<int> st;
st.insert(ivec.begin(), ivec.end());  //st中有4个元素
st.insert({1, 3, 5, 7, 1, 3, 5, 7});  //st现在有8个元素{1,2,3,4,5,6,7,8}
```

###### 2.向map添加元素：

```c++
//插入元素的四种方法
mp.insert({word, 1});
mp.insert(make_pair(word, 1));
mp.insert(pair<string, size_t>(word, 1));
mp.insert(map<string, size_t>::value_type(word, 1));
```

![11-4](E:\朱智博-文件杂项\c++\c++primer学习图片\11-4.png)

###### 3.检测insert的返回值

​    insert(或emplace)返回的值依赖于容器类型和参数。添加单一元素的insert和emplace版本返回一个pair，first是一个迭代器，指向具有定关键字的元素，second是一个bool值，指出是否插入成功。

```c++
map<string, size_t> word_count;
string word;
while (cin >> word){
    auto ret = word.count.insert({word, 1});
    if (!ret.second) ++ret.first->second;
}
```

​    对于允许重复关键字的容器，接受单个元素的insert操作返回一个指向新元素的迭代器，无需返回一个bool。

```c++
multimap<string, string> authors;
authors.insert({"abc", "123"});
authors.insert({"abc", "456"});
```

##### 3.删除元素

​    关联容器定义了三个版本的erase：

![11-5](E:\朱智博-文件杂项\c++\c++primer学习图片\11-5.png)

##### 4.map的下标操作

​    map和unordered_map容器提供了下表运算符和一个对应的at函数，set不支持下标操作。

![11-6](E:\朱智博-文件杂项\c++\c++primer学习图片\11-6.png)

##### 5.访问元素

![11-7](E:\朱智博-文件杂项\c++\c++primer学习图片\11-7.png)

![11-8](E:\朱智博-文件杂项\c++\c++primer学习图片\11-8.png)

###### 1.对map使用find代替下标操作

​    使用下标有一个严重的副作用：如果关键字未在map中，下标操作会插入一个具有给定关键字的元素。

###### 2.在multimap或multiset中查找元素

​    如果一个multimap或multiset中有多个元素具有给定关键字，则这些元素在容器中会相存储。一般用三个方法来进行：

- ```c++
  string search_item("abc");
  auto entries = authors.count(search_item);
  auto iter = authors.finid(search_item);
  while (entries){
      cout << iter->second << endl;
      ++iter;
      --entries;
  }
  ```


- ```c++
  for (auto beg = authors.lower_bound(search_item); beg != authors.upper_bound(search_item); ++beg){
      cout << beg->second << endl;
  }
  ```

- ```c++
  for (auto pos = authors.equal_range(search_item); pos.first != pos.second; ++pos.fisrt){
      cout << pos.first->second << endl;
  }
  ```


#### 11.4 无序容器

​    新标准定义了4个无序关联容器。这些容器是不能用比比较运算来组织元素，而是使用一个哈希函数和关键字类型的==运算符。

##### 1.使用无序容器

​    除了哈希管理操作之外，无序容器还提供了与有序容器相同的操作。因此，通常可以用一个无需容器替换对应的有序容器，反之亦然。

##### 2.管理桶

​    无序容器在存储上组织为一组桶，每个桶保存零个或多个元素。无序容器使用一个哈希函数将元素映射到桶。为了访问一个元素，容器首先计算元素的哈希值，它指出应该搜索哪个桶。容器将具有一个特定哈希值的所有元素都保存在相同的桶中。如果容器允许重复关键字，所有具有相同关键字的元素也都会在同一个桶中。因此，无序容器的性能依赖于哈希函数的质量和桶的数量和大小。

![11-9](E:\朱智博-文件杂项\c++\c++primer学习图片\11-9.png)

##### 3.无序容器对关键字类型的要求

​    默认情况下，无序容器使用关键字类型的==运算符来比较元素，他们还使用一个hash<key_value>类型的对象来生成每个元素的哈希值。