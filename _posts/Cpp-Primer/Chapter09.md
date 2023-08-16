# Chapter9 顺序容器

容器是一些特定类型对象的集合。顺序容器中的元素与其加入容器时的位置相对应，与元素的值无关；之后Chapter11中会介绍关联容器，根据关键字的值来存储元素。

## 9.1 顺序容器概述

顺序容器都提供了快速按**顺序**访问元素的能力，但向容器中增加或删除元素，非顺序访问等有较大代价。

- `vector`: 可变大小的数组，支持快速随机访问（按下标访问），在除了尾部之外的位置插入或删除元素较慢；因为在中间插入某元素，需要将其后面的所有元素进行移动；是最常用的容器类型；
- `dueue`: 双端队列，支持快速随机访问，在头尾插入/删除很快，效率相当于链表；
- `list`: 双向链表，只支持双向顺序访问（不能按下标访问，访问某元素需要从头/尾开始遍历整个容器），在任意位置插入/删除元素都很快；额外内存开销比其他大；没有`size`操作；
- `forward_list`: 单向链表，只支持单向顺序访问，在任意位置插入/删除元素都很快; 额外内存开销比其他大；没有size操作；
- `array`: 固定大小数组，支持快速随机访问，不能插入/删除元素；**除了`array`，其他几个类别都提供了高效灵活的内存管理**；`array`较为安全，更易使用；
- `string`: 与`vector`相似，但只保存字符；

## 9.2 容器库概览

容器的操作包括三个层次：

- 所有容器（包括顺序容器，关联容器，无序容器等）共有；
- 上述三类容器各自共有；
- 一小部分容器特有；

常见的所有容器共有的操作包括：

- 迭代器

  - `iterator`: 容器的迭代器类型
  - `const_iterator`: 只能读取元素，不能修改元素的迭代器类型
  - `size_type`: 无符号整数，足够保存容器中元素的最大数量
  - `difference_type`: 带符号整数，足够保存两个迭代器之间的距离

- 类型别名

  - `value_type`: 容器中元素的类型
  - `reference`: 元素的左值类型，与`value_type&`相同
  - `const_reference`: 元素的`const`左值类型，与`const value_type&`相同

- 构造函数

  - `C c;` 默认构造函数
  - `C c2(c1);` 拷贝构造函数
  - `C c(b, e);` 拷贝迭代器`b`和`e`指定范围的元素，构造`c`（`array`不支持）
  - `C c{a, b, c...};` 列表初始化

- 赋值与swap

  - `c1 = c2` 将容器`c1`的元素替换为`c2`的元素
  - `c1 = {a, b, c...}` 将容器的元素替换为列表中的元素（不适用`array`）
  - `c1.swap(c2)` 交换`c1`和`c2`中的元素
  - `swap(c1, c2)` 交换`c1`和`c2`中的元素

- 大小

  - `c.size()`: `c`中元素的数量（不适用`forward_list`）
  - `c.max_size()`: `c`可保存的最大元素数目
  - `c.empty()`: `bool`值，c中是否有元素

- 添加/删除元素（不适用`array`）_不同容器类型的接口可能有所不同，用args统一指代_

  - `c.insert(args)`: 插入`args`指定的元素
  - `c.emplace(inits)`: 构造c中的元素（使用`inits`初始化）
  - `c.erase(args)`: 删除`args`指定的元素
  - `c.clear()`: 删除所有元素

- 关系运算符

  - `==, !=`: 所有容器都支持
  - `<, >, <=, >=`: 除了无序关联容器都支持

- 获取迭代器

  - `c.begin()`: 返回指向容器中首元素的迭代器
  - `c.end()`: 返回指向容器中尾后元素的迭代器
  - `c.cbegin()`: `const`的`c.begin()`
  - `c.cend()`: `const`的`c.cend()`

- 反向容器的额外成员（`forward_list`不支持）

  - `reverse_iterator`: 按逆序寻址元素的迭代器
  - `const_reverse_iterator`: `const`的`reverse_iterator`
  - `c.rbegin()`: 指向尾元素的逆序迭代器
  - `c.rend()`: 指向首元素之前一个元素（首前元素）的逆序迭代器
  - `c.crbegin()`: `const`的`c.rbegin()`
  - `c.crend()`: `const`的`c.rend()`

标准库中的所有容器都定义在与类型同名的头文件，均定义为模板类； 大多数容器定义时需要提供元素的类型；有些顺序容器要求提供容器的大小；

```c++
// 假设noDefault类没有默认初始化
vector<noDefault> vec1(10, init_value); // 正确，需要提供初始值 
vector<noDefault> vec2(10); // 会抛出异常
```

### 9.2.1 迭代器

标准库中的所有容器都支持迭代器通过解引用符访问容器中的元素的操作；也都定义了递增运算符，移动到下一个元素；

迭代器范围`[begin, end)`, `end`不能指向`begin`之前的位置；

使用`begin`和`end`可以很方便地遍历容器中的元素：

```c++
while (begin != end) {
    *begin = val;
    ++begin;
}
```

### 9.2.2 容器类型成员

迭代器在Chapter3中已经熟悉；类型别名`value_type`用于不清楚容器元素类型的情况;

使用这些类型需要显式使用容器类名与作用域运算符, 不然无法确认使用的是哪个类型的`iterator`, 因为所有容器都有`iterator`类型:

```c++
list<string>::iterator iter;
vector<int>::difference_type count;
```

### 9.2.3 `begin`和`end`成员

it.end() it.rbegin() it.rend()与下述it.begin()用法相同

```c++
vector<int> nums1;
const vector<int>::iterator it1 = nums1.begin(); // 合法，it1的指向是const的，不能移动；
                                                 // 发生从vector<int>::iterator到const vector<int>::iterator的强制类型转换；
vector<int>::const_iterator it2 = nums1.begin(); // 合法，it2所指的元素是const的，不能修改，但这种需求在nums1不是const的时候不常见；
                                                 // 发生从vector<int>::iterator到vector<int>::const_iterator的强制类型转换

const vector<int> nums2;                         // nums1.begin()和num2.begin()重载, 分别返回vector<int>::iterator 和 vector<int>::const_iterator类型
const vector<int>::iterator it3 = nums2.begin(); // 非法，nums2是const的，其迭代器只能是const_iterator;
vector<int>::const_iterator it4 = nums2.begin(); // 合法，it4所指的元素是const的，不能修改，与nums2的const属性相契合，没有发生强制类型转换；
```

it.cend()与下述用法相同

```c++
vector<int> nums3;
auto it5 = nums2.cbegin();                              // cbegin()只会返回vector<int>::const_iterator类型；
auto it6 = nums3.begin();                               // 对于非const的vector<int>，也会返回vector<int>::const_iterator
vector<int>::const_iterator it7 = nums3.cbegin();       // 合法，与上一行等效，但这种需求在nums1不是const的时候不常见；
const vector<int>::const_iterator it8 = nums3.cbegin(); // 发生vector<int>::const_iterator到const vector<int>::const_iterator的强制类型转换；
```

cbegin()和cend()是C++新标准引入的，与auto配合使用可以简化代码；

### 9.2.4 容器定义和初始化

`array`容器初始化至少需要指定元素类型与长度（或者说长度是`array`类型的一部分），其他容器则至少指定元素类型即可。

1. 默认初始化

  ```c++
  array<int, 5> a; // 默认初始化也需要指定长度，此时会执行元素类型（即int）的值初始化；
  vector<int> v;   // 默认初始化不需指定长度，v为空;
  ```

2. 列表初始化

  ```c++
  list<string> authors = {"Milton", "Shakespeare", "Austen"}; // 列表初始化，会创建元素数目等于列表长度的容器；
  vector<const char *> articles{"a", "an", "the"};            // 是否写等号两种方式等效
  // array的列表初始化提供的初始值个数，必须不多于array定义中的元素数目的个数
  array<int, 5> a = {1, 2, 3, 4, 5}; // 合法
  array<int, 5> b = {1, 2, 3};       // 合法，b的后两个元素执行默认初始化，即b的元素依次为{1，2，3，0，0}
  ```

3. 拷贝初始化

  ```c++
  array<int, 5> b(a);  // array的操作类似内置数组，但是比内置数组多出来的操作是拷贝初始化；
  array<int, 5> b = a; // 两种方式等效，拷贝初始化要求两个array长度一致；
  // 其他类型的拷贝初始化只需要类型一致；
  list<string> list2(authors);                            // 合法，容器类型与元素类型均匹配
  deque<string> dq(authors);                              // 非法，容器类型不匹配
  vector<string> vec(articles);                           // 非法，元素类型不匹配
  vector<string> words(articles.begin(), articles.end()); // 合法，使用begin和end的写法不要求容器类型相同，也不要求元素类型相同，只要元素类型之间能发生合法的类型转换即可，这种方式不属于拷贝初始化
  ```

4. 除`array`外的顺序容器,初始化时也可指定长度，对于元素类型没有默认初始化的，则必须指定初始值，其他按需指定;

  ```c++
  vector<int> nums(10, 1); // 10个元素均初始化为1
  vector<int> nums(10);    // 10个元素均执行默认初始化，即初始化为0
  ```

### 9.2.5 赋值和swap

1. 拷贝赋值

  - 除`array`之外的顺序容器经过拷贝赋值，长度可能发生变化，变为等号右边元素的长度, 等号两边类型要求一致；

  ```c++
  vector<int> a(5, 0);
  vector<int> b(10, 1);
  b = a;         // b的元素数目变成5，值都为1；
  b = {0, 1, 2}; // b的元素数目变成3；
  ```

  - `array`拷贝赋值时，

    - 使用对象拷贝赋值，等号两边元素长度必须相等；
    - 使用初始化列表赋值，等号右边的长度不能超过等号左边的长度,等号右边元素数目不足时，等号左边的元素数目也不会减少，末尾若干个元素保持原始值；

  ```c++
  array<int, 5> c = {0, 1, 2, 3, 4};
  array<int, 5> d = {0}; // 列表初始化了第一个元素，其他元素执行默认初始化
  array<int, 10> e = {0};
  d = c;                 // 合法，要求元素数目相同，因为元素数目也是array的类型的一部分
  d = e;                 // 非法，要求元素数目相同
  d = {1, 2, 3};         // 合法，剩余元素保持原来的值
  ```

2. `assign`赋值（仅适用`array`之外的其他顺序容器）

  赋值会导致指向左边容器（被赋值容器）的迭代器，引用，指针全部失效；

  ```c++
  list <string> names;
  vector<const char*> oldstyle;
  names = oldstyle;                               // 非法，类型不同无法赋值；
  names.assign(++oldstyle.begin(), --oldstyle.end()); // 合法，只要元素类型能发生合法的类型转换
  names.assign(10, "a");                          // 另外一种用法，类似构造函数传入数目和初始值的用法，10个元素，每个都是"a"；
  names.assign({"a", "b", "c"});                  // 也可以调用assign
  ```

3. `swap`交换

  用于交换两个相同类型的容器的内容：

  ```c++
  vector<int> vec1(10);
  vector<int> vec2(5);
  swap(vec1, vec2); // 交换两个vector，此时vec1.size()为5，vec2.size()为10，内容也相应交换
  vec1.swap(vec2);  // 两种方式等效，更推荐使用上一种全局swap
  ```

  需要注意的是：

  - `array`容器的交换会真正交换两个容器的元素，时间复杂度O(n);
  - 其他容器交换只是交换指针，时间复杂度O(1)，不对任何元素进行拷贝删除插入等操作;
  - 除`string`外，交换之后，指向之前的元素的迭代器、引用、指针依然有效，但他们属于交换后的新容器，不在原容器了；
  - `string`交换之后，迭代器、引用、指针失效；
  - 标准库中既提供成员版本的`swap`，也提供非成员版本的`swap`，后者更常用；

### 9.2.6 容器大小操作

每个容器z都有三个与大小相关的操作：`size`, `empty`和 `max_size`； 只有一个例外，forward_list不支持size操作，原因下一节解释；

### 9.2.7 关系运算符

- `==`和`!=`: 所有容器都适用；
- `>`, `>=`, `<`, `<=`: 除无序关联容器外，其他容器都适用；
- 关系符左右两边的容器类型和元素类型必须一致；
- 比较方式与`string`的比较方法类似，都是逐个元素比较：

  - 两个容器大小相等，所有元素两两相等，则两个容器相等；
  - 两个容器大小不等，较小容器的每个元素都等于较大容器的对应元素，则较小容器小于较大容器；
  - 如果每个容器都不是另外一个容器的前缀子序列，则取决于第一个不相等的元素的比较结果；

- 需要注意的是，由于容器的比较实际上依赖于容器中元素的比较，所以元素必须对相应得比较运算符有定义；

## 9.3 顺序容器操作

上一节介绍了所有元素（大部分）都适用的操作，本节介绍顺序容器特有的操作；

### 9.3.1 向顺序容器添加元素

```c++
c.push_back(t);  
c.emplace_back(args);  // 尾插，返回void;
c.push_front(c);
c.emplace_front(args); // 头插，返回void;
c.insert(p, t);
c.emplace(p, args);    // 在迭代器p所指的位置插入元素t或者args创建的元素，返回指向新添加的元素的迭代器；
c.insert(p, n, t);     // 在迭代器p所指的位置之前插入n个元素t；返回指向新添加的第一个元素的迭代器；如果n=0,返回p;
c.insert(p, b, e);     // 在迭代器p所指的位置之前插入b和e所指区间内的所有元素；返回指向新添加的第一个元素的迭代器；如果范围为空,返回p;
c.insert(p, il);       // 在迭代器p所指的位置之前插入列表il的所有元素；返回指向新添加的第一个元素的迭代器；如果列表为空,返回p;
```

注意：

- `array`不适用上述所有操作；
- `forward_list`有自己专有版本的`insert`和`emplace`操作; 不支持`push_back`和`emplace_back`;
- `vector`和`string`不支持`push_front`和`emplace_front`;
- 向`vector`, `string`, `deque`插入元素会使所有指向容器的迭代器、引用和指针失效；
- 将一个对象插入到容器中时，实际插入的是对象的拷贝，此后针对这个对象的修改，不会再引起容器中元素的变化；

#### `push_back`和`push_front`

```c++
string word = "book";
word.push_back('s'); // 等价于word += 's';
list<int> ilist;
for(size_t ix = 0; ix != 4; ++ix) {ilist.push_front(ix);}  // ilist = {3, 2, 1, 0}
```

#### `insert`

`insert`的第一个参数为迭代器（单个或范围），可指向容器中任何位置（包括尾后位置），将元素插入到此迭代器所指位置之前，插入后，迭代器恰好指向新元素（范围开头）

```c++
// 单个迭代器
vector<string> svec;
svec.insert(svec.begin(), "Hello");            // 虽然vector没有push_front, 但可以用insert实现，但效率慢
// 返回指向第一个新元素的迭代器， 如果插入范围为空，返回传入insert的第一个参数
list<string> lst;
auto iter = lst.begin();
while (cin >> word) { iter = insert(iter, word)};  //  等价于push_front

// 范围
svec.insert(svec.end(), 10, "Anna");           // 向末尾插入若干重复元素
svec.insert(svec.end(), {"Hello", "World"});   // 向末尾插入整个列表
v = {"Hello", "World", "Hello", "World"};
svec.insert(svec.end(), v.end() - 2, v.end()); // 向末尾插入另外一个容器的一部分， 但不能插入与目的容器相同的容器内的元素
```

#### `emplace`, `emplace_back`和 `emplace_front`

调用`insert`, `push_back`, `push_front`时，是将所指对象的拷贝放入容器中；而`emplace`等函数则直接在容器管理的内存空间上，使用传入的参数调用构造函数生成对象，省去了拷贝的过程

```c++
vector<Sales_data> c;
c.emplace_back("9780067", 25, 15.99);  // 调用Sales_data的构造函数，在容器尾部的内存空间上创建一个对象
c.push_back(Sales_data("9780067", 25, 15.99)); // 与上面等价，但是是首先创建了一个Sales_data对象，再拷贝进容器尾部的内存空间
c.push_back("9780067", 25, 15.99); // 非法
// 可以使用Sales_data类的不同构造函数，emplace需指定插入位置，规则与insert一样
c.emplace(c.begin()); 
c.emplace(iter, "9780067")
```

### 9.3.2 访问元素

```c++
c.back()  // 返回尾元素的引用，如果c空，则函数行为未定义
c.front() // 返回首元素的引用，如果c空，则函数行为未定义
c.at(n)   // 返回索引为n的元素的引用，如果索引越界，则抛出out_of_range异常
c[n]      // 返回索引为n的元素的引用，n是无符号整数，如果n >= c.size()， 则函数行为未定义
```

注意：

- `at`和下标操作只适用于`string`, `vector`, `deque`和`array`
- `back`不适用与`forward_list`

```c++
if (!c.empty()) // 首先确认容器非空，否则会引发未定义行为
{
    // 获取首尾元素
    auto val1 = c.front();
    auto val2 = *(c.begin());  // 等价
    auto val3 = c.back();
    auto val4 = *(c.end() - 1); // 等价

    // 返回引用，如果想通过引用改变容器元素，需要用引用类型的变量接住
    auto v = c.front(); // v是首元素的一个拷贝， 改变v不会改变容器的首元素
    auto &v = c.front(); // v是首元素的引用，改变v会改变容器的首元素
}

// 如果想确保下标不越界，at是更安全的访问方法
vector<int> vec = {0, 1, 2, 3, 4};
int a = vec[5];  // 返回0，但实际上越界，导致程序不易排查错误
int b = vec.at(5); // 抛出out_of_range异常
vector<int> vec;  // 空容器
int c = vec[0];  // 段错误，无其他提示
int d = vec.at(0); // 抛出out_of_range异常
int e = vec.front(); // 抛出out_of_range异常
int f = *(vec.begin()); // 抛出out_of_range异常
```

### 9.3.3 删除元素

```c++
c.pop_back()    // 删除尾元素，返回void；如果c空，则函数行为未定义；
c.pop_front()   // 删除首元素，返回void；如果c空，则函数行为未定义；
c.erase(p)      // 删除迭代器所指p位置元素，返回下一个元素位置的迭代器；如果p是尾后迭代器，则函数行为未定义；
c.erase(b, e)   // 删除迭代器b和e所指范围内的元素，返回下一个元素位置的迭代器；如果e是尾后迭代器，则函数返回尾后迭代器；
```

注意：

- 因为删除操作会改变容器的大小，所以`array`不适用；
- `forward_list`有特殊版本的`erase`；
- `forward_list`不支持`pop_back`; `vector`和`string`不支持`pop_front`；
- 删除`deque`除了首尾之外的任何元素都会使所有迭代器、指针和引用失效；
- 删除`vector`和`string`的某元素，此元素之后的所有迭代器、指针和引用失效；
- 删除元素的成员函数不会检查其参数，程序员需自行确保被删除的元素是存在的；

```c++
vector<string> svec;
svec.pop_back();
svec.pop_front();  // 返回void，并不能返回弹出的值；
// 单个迭代器
svec.erase(svec.begin());  // 返回原始容器中第二个元素，现在是新的首元素
svec.erase(svec.end());           //  未定义
svec.erase(svec.end() - 1);  // 正确的删除尾元素的方法，返回尾后元素
// 返回删除后下一个元素的迭代器
auto it = svec.begin();
while (it != svec.end()) {
    // 一些处理；
    it = svec.earse(it);
}  // 可逐个删除元素

// 范围
elem1 = svec.erase(elem1, elem2); // 删除[elem1, elem2）内的元素，运行后，elem1==elem2
svec.erase(svec.end(), v.end()); // 删除所有元素
svec.clear();  // 等价
```

### 9.3.4 特殊的`forward_list`操作

在单向链表中插入或删除元素，需要改变前驱节点的链接，以便指向后继节点； 但是单向链表中访问前驱节点是一件麻烦的事情，所以定义了`insert_after`, `emplace_after`和`erase_after`，对当前迭代器指向的下一个节点进行相应操作； 为了操作首节点，还定义了指向首前节点的迭代器`before_begin`；

```c++
lst.before_begin()             // 返回指向首前节点的迭代器，不能解引用；
lst.cbefore_begin()            // 返回const_iterator版本
lst.insert_after(p, t)         // 在节点p之后插入元素t，返回指向最后一个插入元素的迭代器；如果p为尾后元素，则函数行为未定义
lst.insert_after(p, n, t)      // 插入n个元素t
lst.insert_after(p, b, e)      // 插入区间[b, e), 如果范围空，返回p
lst.insert_after(p, ilist)     // 插入ilist
lst.emplace_after(p, args)     // 在节点p之后插入args构建的元素，返回指向新元素的迭代器
lst.erase_after(p)             // 删除节点p之后的元素，返回指向最后一个被删除元素之后位置的迭代器；如果p为尾后元素，则函数行为未定义
lst.erase_after(b, e)          // 删除区间[b, e)，如果e是尾后迭代器，则返回尾后迭代器；
```

### 9.3.5 改变容器大小

```c++
c.resize(n)                    // 改变容器大小为n，如果n小于当前大小，则从末尾删除元素；否则，在尾后添加n-size()个元素，新元素进行值初始化
c.resize(n, t)                 // 改变容器大小为n，如果n小于当前大小，则从末尾删除元素；否则，在尾后添加n-size()个元素，新元素初始化为值t
```

注意：

- 如果`resize`缩小了容器，则指向被删除元素的迭代器、指针、引用都会失效；
- 如果容器内保存类类型，且`resize`需要向容器内添加新元素，则类必须提供默认构造函数，否则我们必须提供初始值；

### 9.3.6 容器操作可能使迭代器失效

添加和删除元素可能导致迭代器、引用和指针失效。使用失效的指针是一种严重的程序设计错误； 对于不同的容器类型，失效的情况有所不同：

- 在p处添加元素：

  - `vector`和`string`: 如果存储空间未被重新分配过，则p之前的迭代器、指针和引用仍有效，p以及之后的迭代器、指针和引用无效；如果存储空间被重新分配过，则全部失效；
  - `deque`: 插入到首尾之外的位置会导致所有迭代器、指针和引用失效；如果在首尾插入，迭代器失效，指针和引用不会失效；
  - `list`和`forward_list`: 迭代器（包括首前和尾后）、指针和引用都不会失效；

- 在p处删除元素, 指向被删除元素的迭代器、指针、引用都直接失效（毕竟已经销毁了）

  - `vector`和`string`: 被删元素之前的迭代器、指针和引用仍然有效；
  - `deque`: 删除首尾之外的元素会导致所有迭代器、引用和指针失效；删除尾元素，尾后迭代器失效，其他不受影响；删除首元素，不受影响；
  - `list`和`forward_list`: 指向容器其他位置的迭代器（包括首前和尾后）、指针和引用都不会失效；

由于添加和删除元素可能导致迭代器失效，所以将迭代器保持在有效范围内很有必要，或者保证**每次操作后正确地重新定位迭代器**； 例如，循环条件常用

```c++
while(p != v.end())
```

而不是

```c++
end = v.end()
while(p != end)  // 一般会导致无限循环
```

的原因就是`end`可能会被改变，所以应该每次重新计算`v.end()`；

## 9.4 `vector`对象是如何增长的

为了支持快速随机访问，`vector`将元素连续存储在一段内存中，每个元素紧挨着前一个元素存储；当向`vector`添加元素时，如果空间不足，需要将所有元素复制到更大的内存中，然后将新元素插入到新的内存中； 这样的话，每次插入都会导致`vector`重新分配内存，重新复制元素，效率很低；标准库为了提高效率，通常在为`vector`开辟新的内存空间时，分配比其需求更大的内存空间，作为备用；这就是`vector`的**内存空间分配策略**；

`vector`提供了一些成员函数，允许程序员与内存分配部分互动；

```c++
c.shrink_to_fit();  // 请求释放多余的内存空间，即把capacity()减少到与size()相同大小； 适用vector, string和deque;
c.capacity();       // 不重新分配内存的话，c可以保存的元素数目；  适用vector, string;
c.reserve(n);       // 请求分配至少能容纳n个元素的内存空间；   适用vector, string;
```

依赖于不同的标准库的实现， `shrink_to_fit`不一定保证能退回内存空间； `reserve`请求的数目小于等于当前`capacity`时，不会退回空间；反之，`reserve`之后至少分配与需求一样大的内存空间（有可能更大）； `capacity`与`size`的关系：

```c++
vector<int> vec;  // 空容器
cout << vec.size() << " " << vec.capacity() << endl; // 0, 0
for (vector<int>::size_type i = 0; i < 24; ++i)
{
    vec.push_back(i);
}
cout << vec.size() << " " << vec.capacity() << endl; // 24, 32, 容器第一次开辟了32的内存空间
vec.reserve(30);
cout << vec.size() << " " << vec.capacity() << endl; // 24, 32，reserve的数目小于capacity,内存空间未减少
vec.reserve(50);
cout << vec.size() << " " << vec.capacity() << endl; // 24, 50，reserve的数目大于capacity,内存空间增加
while (vec.size() != vec.capacity())   // 填满内存空间
{
    vec.push_back(0);
}
cout << vec.size() << " " << vec.capacity() << endl; // 50, 50
vec.push_back(0);
cout << vec.size() << " " << vec.capacity() << endl; // 51, 100，再push一个就必须开辟新的内存空间了，第二次开辟了100的内存空间（50double,但不是所有编译器都使用double规则）
vec.shrink_to_fit();
cout << vec.size() << " " << vec.capacity() << endl; // 51, 51，回退内存空间，标准库不保证退还内存
```

## 9.5 额外的`string`操作

包括与C风格字符串的转换，以及使用下标代替迭代器的版本；

### 9.5.1 构造`string`的其他方法

除了前面顺序容器介绍过的构造方法，`string`还有额外的构造方法与子字符串操作:

```c++
string s(cp);    // cp指向的数组的拷贝，必须以空字符结尾
string s(cp, n);   // cp指向的数组(至少有n个字符，指的是C风格字符串）中，前n个字符的拷贝，构造成s
string s(s2, pos2); // string s2 从下标pos2开始的字符的拷贝，构造成s; 如果pos2 > s2.size(), 则抛出out_of_range
string s(s2, pos2, len2)   // string s2 从下标pos2开始的len2个字符的拷贝，构造成s; 无论len2的值为多少，构造函数最多拷贝（s2.size()-pos2）个字符，不会抛出out_of_range
string s = s2.substr(pos2); // 返回从pos2开始到结尾的所有字符的拷贝，如果pos2 > s2.size(), 则抛出out_of_range
string s = s2.substr(pos2, len2); // 返回s2从pos2开始的len2个字符的拷贝，无论len2的值为多少，构造函数最多拷贝（s2.size()-pos2）个字符，不会抛出out_of_range
```

### 9.5.2 改变`string`的其他方法

本节的方法有多种重载版本，上面只介绍少部分，其他用法可以在使用时查阅C++ Primer 5th或网上博客。

- 定义了额外的`insert`和`erase`版本，接受下标，指示开始删除的位置，或者插入到此位置之前；

```c++
s.insert(s.size(), 5, '!');  // 末尾插入5个字符
s.erase(s.size() - 5, 5);    // 删除最后5个字符
```

- 定义了接收C风格字符串的`insert`和`assign`版本

  ```c++
  const char* cp = "Stately, plump Buck";
  s.assign(cp, 7);    // 使用cp的前7个字符替换s的内容
  s.insert(s.size(), cp + 7); // 从cp的第8个字符开始，插入到s的末尾
  ```

- 定义了`append`和`replace`方法，`append`是在`string`末尾插入的简单形式，`replace`是调用`erase`和`insert`的组合;

```c++
string s("Cpp Primer"), s2 = s;
s.insert(s.size(), " 4th Ed.");
s2.append(" 4th Ed."); // 等价方法
s.erase(11, 3); // 在位置11处开始删除3个字符；
s.insert(11, "5th"); // 插入新字符串
s2.replace(11, 3, "5th");  // 等价方法，可以替换任意长度字符；
```

### 9.5.3 `string`搜索操作

提供了6个不同的搜索函数，每个函数还有4个重载版本；所有函数都返回一个`string::size_type`值，表示匹配位置的下标;搜索失败返回`string::npos`;

本节的方法有多种重载版本，上面只介绍少部分，其他用法可以在使用时查阅C++ Primer 5th或网上博客。

```c++
// find函数和rfind函数
string river("Mississippi");
auto pos = river.find("is"); // pos == 1, 注意这里不要用int类型接住；find函数大小写敏感；
auto pos = river.rfind("is"); // pos == 4

// find_first_of函数和find_first_not_of函数
string numbers("0123456789"), name("r2d2");
auto pos = name.find_first_of(numbers); // pos == 1，即name中第一个数字的下标；
auto pos = name.find_first_not_of(numbers); // pos == 0，即name中第一个非数字的下标；

// find_last_of函数和find_last_not_of函数
string str("ci9582mcndv935fjwsv");
auto pos = str.find_last_of(numbers); // pos == 13，即str中最后一个数字的下标；
auto pos = str.find_last_not_of(numbers); // pos == 18，即str中最后一个非数字的下标；
```

也可以传递给上述函数一个可选的开始位置，指出从哪个位置开始往后搜索，默认为0；示例：

```c++
string name{"ab2c3d7R4E6"};
string numbers{"0123456789"};
string::size_type pos = 0;
while ((pos = name.find_first_of(numbers, pos)) != string::npos)
{
    cout << "index: " << pos << " number: " << name[pos] << endl;
    ++pos;
}
```

### 9.5.4 `compare`函数

类似C标准库的`strcmp`，根据`s`是等于、大于、小于给定字符串，`s.compare`返回0、正数、负数；

`compare`函数提供了6个重载版本，包括与字符串的比较，与字符数组的比较，以及比较整个或一部分数组；

```c++
s.compare(s2);   // 比较s和string类型的s2
s.compare(pos1, n1, s2); // 比较s[pos1, pos1+n1)与s2
s.compare(pos1, n1, s2, pos2, n2); // 比较s[pos1, pos1+n1)与s2[pos2, pos2+n2)
s.compare(cp);   // 比较s与const char* 类型的cp（必须以空字符结尾）
s.compare(pos1, n1, cp);  // 比较s[pos1, pos1+n1)与cp
s.compare(pos1, n1, cp, n2);  // 比较s[pos1, pos1+n1)与cp指向的地址开始的n2个字符
```

### 9.5.5 数值转换

一般从文件中读入的数字是`string`形式，而运算需要的是数值形式（例如`int`）, 新标准提供了互相转换的方式；

```c++
int i = 42;
string s = to_string(i);  // 整数转字符串
double d = stod(s);  // 字符串转浮点数
string s2 = "pi = 3.14";
d = stod(s2.substr(s2.find_first_of("+_.0123456789")));
```   

## 9.6 容器适配器

`stack`，`queue`, `priority_queue`是顺序容器适配器； 适配器是一种概念，容器、迭代器、函数都有适配器，可以使一种事物的行为看起来像另一种事物； 例如`stack`适配器接受一个`vector`容器,可以使得操作看起来像栈一样；

所有容器适配器都支持的操作有：

```c++
size_type    // 足以保存当前类型的最大对象的大小
value_type   // 元素类型
container_type // 实现适配器的底层容器类型
A a;          // 创建空适配器
A a(c);       // 创建适配器a，带有容器c的一个拷贝
==, !=, <, <=, >, >=  // 返回底层容器的比较结果
a.empty()          // 返回a是否为空
a.size()           // 返回a中元素的个数
swap(a, b)            
a.swap(b)          // 交换a和b的内容，a和b的类型以及底层容器类型必须相同
```

默认情况下，`stack`和`queue`是基于`deque`实现的，`priority_queue`是基于`vector`实现的；在创建适配器时，可以自行重载容器类型；

```c++
stack<string, vector<string>> str_stk; // 定义了一个在vector上实现的空栈
stack<string, vector<string>> str_stk2(svec);  // 定义了一个在vector上实现的栈，初始化保存了svec的拷贝
str_stk.push_back("Hello"); // 非法，不能使用底层容器的操作，只能使用适配器的操作
```

不同种类的适配器，可以重载的底层容器类型也有所不同：

- 所有适配器都要求容器具有添加和删除元素的能力，所以不能使用`array`和`forward_list`构造适配器；
- `stack`只要求`push_back`, `pop_back`和`back`操作，所以可以基于`list`, `deque`和`vector`构造；
- `queue`要求`push_back`, `back`, `push_front`和`front`操作，所以只能基于`list`, `deque`构造；
- `priority_queue`要求`push_back`, `pop_back`和`front`操作，还要求随机访问能力，所以只能基于`vector`和`list`构造；

除了上述所有适配器共同支持的操作外，每个适配器都定义了自己的特殊操作。

```c++
// stack, 定义在stack头文件中
stk.pop()  // 删除栈顶元素, 不返回该元素
stk.top()  // 返回栈顶元素, 不删除该元素
stk.push(item) // 添加元素到栈顶, 该元素可以通过拷贝或者移动item而来，
stk.emplace(args)  // 添加元素到栈顶, 元素由args构造

// queue和priority_queue, 定义在queue头文件中
q.pop()  // 删除队列首元素(或优先级最高的元素), 不返回该元素
q.front()  q.back()  // 返回队列首尾元素, 不删除该元素,只适用queue
q.top()  // 返回优先级最高的元素, 不删除该元素,只适用priority_queue
q.push(item) // 添加元素到队列尾(优先级恰当的位置), 该元素可以通过拷贝或者移动item而来;
q.emplace(args)  // 添加元素到队列尾(优先级恰当的位置), 元素由args构造
```

- stack: FILO，先进后出
- queue: FIFO，先进先出
- priority_queue: 优先级最高的元素先出队,优先级默认使用`<`确定，可以自行重载；



