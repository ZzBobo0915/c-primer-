### STL序列式容器有哪些？

- array
- vector
- list
- deque
- forward_list

### array底层原理

​    array本质就是一个**大小固定的数组**，无法动态改变，只允许访问或替换数组中的元素。

### vector底层原理

​    vector底层是一个**动态数组**，包含三个迭代器，start, finish, end_of_storage；当数组空间装不下时，会自动申请另一篇更大的空间（1.5倍或2倍），把原来的数据拷贝带新的内存空间，然后释放原来的那片空间。

​    vector调用clear()函数只是清空了里面的数据，其存储空间不释放。

### vector中reserver与resize

​    vector中reverse()是直接扩充到确定的大小下，可以减少多次开辟、释放空间的问题；resize()可以改变有效空间的大小，也有改变默认值的功能。

### vector中size与capacity

​    vector中size表示当前有多少元素，capacity表示当前分配的内存能容纳多少元素。

### 如何正确释放vector内存

- vec.clear(); vec.shrink_to_fit();  清空内容并且释放内存
- vector(Vec).swap(Vec); 清除内存
- vector().swap(Vec); 清除内存

### list底层原理

​    list底层是一个**双向链表**，以节点为单位存储数据，节点的地址在内存中不一定连续，每次插入或者删除一个元素，就配置或释放一个元素空间。

​    list不支持随机存取，适合需要大量的插入和删除，而不关心随机存取的应用场景。

### deque底层原理

​    deque底层是一个**双向开口的线性连续空间（双端队列**），在头部和尾部插入、删除元素有理想的复杂度；并且deque也不保证存储元素所有元素都存储在连续的空间内。

### vector、list与deque都适用于什么情况？

​    vector可以随机存储元素，不需要挨个查找，适合随机存取的场景使用，除非必要情况尽量都使用vecter而非deque；

​    list适用于对象大、对象数量频繁变化，插入和删除频繁的场景；

​    deque适合在头尾两端进行操作的场景。

### forwrad_list底层原理

​    froward_list和list差不多，只不过他是一个**单向链表**，节点的地址在内存中不一定连续。

### STL关联容容器是什么？有哪些？

​    关联式容器是指存储元素是会为元素配备一个键Key，整体以键值对的方式存储在容器中；另外，关联式容器在存储元素时会根据元素的大小进行排序(less<T\>, greater<T\>)。

- map
- multimap
- set
- multiset

### 上述几个关联式容器的的低层原理是什么？

​    关联式容器的底层都是根据**红黑树**实现的，epoll模型的低层也是红黑树。红黑树具有以下特点：

1. 是一个棵平衡二叉树
2. 节点分为两种颜色：红色和黑色
3. 根节点和叶子节点为黑色
4. 红色结点的子节点必须是黑色节点
5. 根节点到任意一个叶子节点的路径上黑色节点个数相同

### map

​    map容器存储的都是pair对象（键值对），默认情况下会自定根据键的大小按照升序排序std::less<T\>。map中存储的键不能重复也不能被修改（注意是键不能修改！值可以）。

### multimap

​    multimap与map一样按照键值对存储，不相同的是multimap可以存入多个相同键的键值对。

### set

​    与两个map容器不同，使用set容器存储的各个键值对的key和value都相等。set还要求存储的键不能重复，并且默认情况下也会根据键的大小按升序排序。

### multeset

​     multiset与set一样按照键值对存储key和value都相等，不同的是multiset可以存入多个相同的键值对。

### map和set的插入删除速度是多少？为什么每次insert之后以前保存的iterator不会失效？

​    map和set的插入删除速度都是logN。

​    由于红黑树存储的是节点，不需要内存拷贝和内存移动，插入和删除操作也只是将节点的指针换来换去，节点内存没有改变。

### 为何map和set不能像vector一样有个reserve函数来预分配数据?

​    因为在map和set内部存储的已经不是元素本身了，而是包含元素的结点。也就是说map内部使用的Alloc并不是map<Key, Data, Compare, Alloc>声明的时候从参数中传入的Alloc。

### map的插入方式有几种？

- 用insert插入pair数据 `mapM.insert(pair<char, int>('a', 1));`
- 用insert插入value_type数据 `mapM.insert(map<char, int>::value_type('a', 1));`
- 在insert函数中使用make_pair()函数 `mapM.insert(make_pair('a', 1));`
- 用数组下标方式插入 `mapM['a'] = 1;`

### STL无序式关联容器有哪些？

​    无序式关联容器，又称哈希容器，采用的存储结构为哈希表。**哈希表可以根据关键字直接找到数据的存储位置，不需要进行任何比较，大大降低数据查找和存储消耗的时间，但是需要比较多的内存。**

- unordered_map
- unordered_set    
- unordered_multimap
- unordered_multiset 

### unordered_map底层原理、unordered_set底层原理

​    unordered_map的底层是一个防冗余的哈希表（采用除留余数法），还是采用键值对的方式存储，键值对互不相等并且不会对存储内容进行排序。

 unordered_set低层也是一个防冗余的哈希表，不以键值对存储，直接存储数据的值，元素值互不相同并且不会对内容进行排序。

​    使用一个下标范围较大的数组来存储元素，设计一个哈希函数(散列函数)，使得每一个元素的key都与一个函数值(数组下表，hash值)相对应，用这个数组单元来存储这个元素，这个数组单元也被成为桶。

​    对于不同元素计算出相同哈希值(即发生冲突)的时候，一般采用开链法解决冲突。

### STL的容器适配器有哪些？

​    容器适配器是一个封装了序列容器的模板。

- stack
- queue
- priority_queue

### stack底层原理

​    stack适配器是一个**单端开口的容器**，模拟栈存储结构，无论存数据还是读数据，都只能从这个一个开口操作。满足后进先出LIFO准则。

### queue底层原理

​    queue适配器有**两个开口的容器，一个专门输入数据，一个专门输出数据**，模拟单向队列存储结构。满足先入先出FIFO准则。

### priority_queue底层原理

​    priority_queue适配器模拟的也是队列这种存储结构，只能一端进，另一段出，且每次访问只能访问队头元素，本质是一个堆，又被称为优先队列(最大堆/最小堆)，队头元素永远都是优先级最大的元素。可以在创建优先队列对象时指明是最大堆less<T\>还是最小堆greater<T\>，这里less与greater与其他容器不同，priority_queue默认最大堆。



   



