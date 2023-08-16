# Chapter10 泛型算法

顺序容器定义的操作比较少，有很多常用的操作并未包含。例如查找、删除、替换特定元素等等。

标准库没有为每个容器都定义相应的成员函数，而是定义了一组**泛型(generic, 通用的)算法**，实现了一些经典算法的公共接口，可以用于多种容器类型（甚至可能包括内置数组类型等等）

## 10.1 概述

大多数算法定义在头文件`algorithm`中，标准库还在头文件`numeric`中定义了一组数值泛型算法；一般来说这些算法并不直接操作容器，而是通过两个迭代器指定的元素范围进行操作；

例如：

```c++
auto result = find(v.cbegin(), v.cend(), 42);   // 返回迭代器
cout << (result == v.cend() ? "42 not found" : "42 found") << endl;
// 可以用于vector，string，甚至内置数组;
// 泛型算法直接操作迭代器，不会执行容器的操作，所以和容器类型无关，但也无法改变容器大小，插入删除元素；后者需要特殊的迭代器来完成（而非使用算法完成）
// 但算法依然对具体元素的类型有依赖，例如上例中需要容器内元素对==有定义，如果没有，需要自定义;
```

## 10.2 初识泛型算法

大多数算法的前两个参数接受两个迭代器，作为算法操作的元素的范围；

### 10.2.1 只读算法

一些算法只会读取范围内的元素，不会改变元素；这种算法中参数最好写作`cbegin`和`cend`的形式；

包括`find`，`conut`，`acculumate`，`equal`等；

```c++
// acculumate
int sum = acculumate(v.cbegin(), v.cend(), 0); // 求和, 基数是0；
string sum = acculumate(v.cbegin(), v.cend(), string("")); // 可以将所有string连接起来，注意最后一个参数不能是""(const *char类型)，必须是string;
// equal,用于对比两个序列是否有相同的值；传入三个迭代器，只有一个迭代器用来表示第二个序列，这种算法一般都假定第二个序列长度大于等于第一个序列
vector<string> v;
list<const char*> w;
bool is_equal = equal(v.cbegin(), v.cend(), w.cbegin());  // 容器类型可以不同；v和w元素类型也可以不一样，只要能用来比较即可；如果v是w的前缀序列，那么也返回true;
```

### 10.2.2 写容器元素的算法

一些算法会修改容器中的元素，使用这类算法时必须保证序列大小至少不小于要求写入的元素数目；因为算法不能改变容器的大小；

```c++
// 算法fill接受一对迭代器表示的范围，所以写入操作是比较安全的
fill(v.begin(), v.end(), 42);  // 所有元素都写入42
// 算法fill_n只接受一个起始迭代器dest和一个计数器n，所以需要程序员自己确保dest之后至少有n个元素
vector<int> vec;
fill_n(vec.begin(), 10, 42);  // 写入10个42,非法，因为vec中没有元素；
// 算法copy接受三个迭代器，前两个迭代器表示源序列的范围，第三个迭代器表示目标序列起始位置，需要程序员自己确保目标序列有足够空间
int a1[] = {0,1,2,3,4,5,6,7,8,9};
int a2[sizeof(a1) / sizeof(*a1)];  // 保证a1和a2一样大
auto ret = copy(begin(a1), end(a1), a2);  // 返回一个迭代器，指向a2被拷贝的最一个元素之后的位置，这里是尾后元素
```

**插入迭代器**可以将新元素添加到容器中，而不是修改容器中已有元素的值；10.4节中会详细介绍，这里先介绍`back_iterator`(定义在头文件`iterator`中)

```c++
vector<int> vec;
auto it = back_iterator(vec);  // 返回一个迭代器，通过它赋值会将元素添加到vec的末尾
*it = 42;  // 添加42
fill_n(back_iterator(vec), 10, 42);  // 添加10个42
```

### 10.2.3 重排容器元素的算法

最明显的例子是`sort`,可以将容器中元素按运算符`<`排序；接下来使用一个例子介绍`unique`去重操作：

```c++
void elimDups(vector<string>& words)
{
    sort(words.begin(), words.end());  // 排序，方便unique调用，因为unique只能处理那些相同元素相邻的情况
    auto end_unique = unique(words.begin(), words.end());  // unique不操作容器，所以实际没有删除元素，只是将不重复的元素放到容器前面，并返回最后一个不重复元素之后的位置；
    words.erase(end_unique, words.end());  // erase才能真正删除，即使原始容器中没有重复元素也是合法的
}
```

## 10.3 定制操作

很多算法都会比较容器中的元素，一般来说是使用`<`运算符，但允许我们提供自己定义的操作来代替默认的运算符；

### 10.3.1 向算法传递函数

一个例子，现在希望调用`elimDups`之后打印`vector`的内容，希望单词按长度排序，大小相同的再按字典序排序；

那么需要对`sort`设置第三个参数**谓词**，来代替默认的`<`。标准库使用的谓词有一元谓词（接受一个参数）和二元谓词（接受两个参数）两种; 谓词是一个函数对象。

```c++
bool isShorter(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}
elimDups(words);  // 先按字典序排序并消除重复
stable_sort(words.begin(), words.end(), isShorter);  // 按长度排序, stable_sort可以保证长度相同的按照字典序排序
```

### 10.3.2 lambda表达式

谓词只能使用一个或两个参数，有时候需要使用其他参数，超出了算法对谓词的限制；例如上一节中的练习题，打印容器中长度大于5的单词,就不得不将5写死；

```c++
bool predicate(const string &s)
{
    return s.size() >= 5;
}

vector<string> v = {"a", "as", "aasss", "aaaaassaa", "aaaaaabba", "aaa"};
auto pivot = partition(v.begin(), v.end(), predicate);
for (auto it = v.begin(); it != pivot; ++it)
{
    cout << *it << " ";
}
return 0;
```

`sort`算法接受的谓词实际上是一个**可调用对象**，可调用对象包括函数、函数指针、重载了函数调用运算符的类、lambda表达式；

lambda表达式是C++的一个语法糖，可以理解为**一个没有函数名称的内联函数**，也称**匿名函数**；lambdad表达式具有如下形式：

```c++
[capture list] (parameter list) mutable exception specification -> return type { function body }
```

- `capture list``: 捕获列表，用于捕获外部作用域中的变量，并可以在函数体中使用;
- `parameter list`参数列表，`return type`返回类型，`function body`函数体和普通函数中相同；
- 参数列表和返回类型可以忽略，但必须包含捕获列表和函数体；

### 10.3.3 lambda捕获和返回

### 10.3.4 参数绑定

## 10.4 再探迭代器

## 10.5 泛型算法结构

## 10.6 特定容器算法

