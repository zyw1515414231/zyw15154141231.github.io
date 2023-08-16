

# Chapter8 IO库

cin, cout, cerr都是istream类或者ostream类的对象，为什么没有实例化就能直接用了呢？

有进行实例化，在头文件中就实例化了，是一个静态对象，主要是方便使用；

## 8.1 IO类

除了从**控制台**进行IO操作，常见的还有对**文件**和**内存**中的string进行IO操作：

- 头文件iostream 控制台IO，包含istream类，ostream类，iostream类，以及对应的宽字符版本wistream类，wotream类和wiostream类；
- 头文件fstream 文件IO, 包含ifstream类，ofstream类，iofstream类，对应宽字符版本......
- 头文件sstream 字符串IO, 包含istreamstring类，ostreamstring类，iostringstream类，对应宽字符版本......
- 基本所有类都继承自iostream类；

### 8.1.1 IO对象无拷贝无赋值

- 不能对IO对象（cin，cout， cerr）等进行拷贝与赋值；
- 所以不能将形参或返回类型设置成流类型，使用**引用方式**传参或传返回值；
- 读写IO对象实际上是改变这个IO对象，所以传递的引用不能是const的；

### 8.1.2 条件状态

IO库定义了一些**函数**和**标志**，用于访问IO流的状态， strm代表某个IO类，s代表一个IO对象（一个流）：

- 标志：strm::iostate类型，取值包括strm::badbit, strm::failbit, strm::eofbit，strm::goodbit，都是constexpr；

  - badbit表示不可恢复的系统级错误，流被置为badbit之后一般无法再使用；
  - failbit可恢复错误, 例如期望得到整数但是得到一个字符，或者文件到达末尾；
  - eofbit表示文件到达末尾；
  - goodbit为0表示流未发生错误；

- 函数：

  - s.eof(), s.fail(), s.bad(), s.good(), s.clear()：返回bool类型；其他三个均未置位，s.good()返回true；badbit置位时，failbit返回true;
  - s.clear(),将s设置为有效；s.clear(flags)， s.setstate(flags)：将流设置成flags条件状态位（iostate类型）：返回void;
  - s.rdstate(), 返回流的当前条件状态位：返回iostate类型；

通常在使用流之前检查流的状态，一般作为一个条件来使用:

```c++
while (cin >> word)
{
  // 后续操作
}
```

### 8.1.3 管理输出缓冲

当执行

```c++
cout << "Hello world";
```

时，字符串被保存到缓冲区，可能不会被立即被打印；因为写操作一般比较耗时，所以操作系统可能会将多次输出到缓冲区的数据合并成一次输出，这个操作称为**缓冲区刷新**。

导致缓冲刷新的原因：

- 程序结束，return
- 缓冲区满
- 操纵符endl, flush, ends

  ```c++
  cout << "Hello world" << endl; // 输出缓冲区内容+换行，并刷新缓冲区
  cout << "Hello world" << flush; // 输出缓冲区内容，并刷新缓冲区
  cout << "Hello world" << ends; // 输出缓冲区内容+空字符，并刷新缓冲区
  ```

- 操纵符unitbuf

  ```c++
  cout << unitbuf; // 输出的任何内容都会导致缓冲区刷新
  // 程序异常终止时，可能有信息储存在缓存区并未被刷新，导致无法识别程序在何处崩溃，所以unitbuf经常用于调试
  cout << nonunitbuf;  // 回到正常的缓冲方式
  ```

- 关联的输入流和输出流 交互式系统中，所有输出在提示用户输入之前打印出来，是很友好且必须的；当输入流被关联到输出流时，任何试图从输入流读取的操作都会导致输出流的刷新。关联的方法是使用tie函数：

  ```c++
  // 第一种重载
  cin.tie() // 返回cin关联的输出流的指针，如果没有关联到输出流，返回空指针
  // 第二种重载
  cin.tie(&ostream); // 设置cin关联到ostream对象，返回之前关联的ostream对象
  cin.tie(nullptr); // 取消cin关联
  ```

  每个流只能关联到一个流，但一个流可以被多个流关联；可以将一个istream或者ostream关联到ostream。

## 8.2 文件输入输出

fstream头文件用于文件IO操作。其中fstream, ifstream, ofstream三个类型继承自iostream,可以使用IO运算符<<, >>, getline等，此外还有一些特有操作:

```c++
// fstream代指上述三个类型
fstream fstrm;  // 创建一个未绑定的文件流
fstream fstrm(s);  // 创建一个绑定到文件s的文件流，自动调用open打开
fstream fstrm(s, mode);  // 创建一个绑定到文件s的文件流zi自动调用open打开, 打开模式mode
fstrm.open(s); // 打开名为s的文件，并绑定到fstrm
fstrm.close(); // 关闭与fstrm绑定的文件s
fstrm.is_open();  // 返回bool值，指出与fstrm绑定的文件是否打开且尚未关闭
```

### 8.2.1 使用文件流对象

因为fstream继承自iostream，所以在调用形参类型为istream或者ostream的函数，可以传入fstream实参；

常用的打开文件的用法是:

```c++
ifstream in(ifile); // 创建对象in, 绑定到文件ifile并自动调用open函数打开；
ofstream out; // 创建未绑定的对象out
out.open(ifile + ".copy");  // out绑定到文件并打开
// 如果open失败，out会被置位failbit,在操作文件之前最好进行一次判断

if (out)
    // 文件读取成功，后续操作···
```

文件流open成功之后，就绑定到对应文件上，此时再使用open函数打开其他文件的话会导致failbit被置位；如果想把文件流关联到其他文件，需要先调用close关闭已经关联的文件；

```c++
in.close();
in.open(ifile + "2");
```

离开in对象的作用域时，in对象被销毁，close函数会被自动调用，但是写上close依然是个好习惯。

### 8.2.2 文件模式

- in: 读，只可对ifstream和fstream对象使用；
- out: 写，只可对ofstream和fstream对象使用；
- app: 追加，每次写操作前定位到文件末尾；即使没有显示指定out模式，文件实际上也是以out模式被打开；
- trunc: 截断，文件会被清空；out也被设定时，才能设定trunc;只要trunc没被设定，就可以设定app; 也就是说，组合方式就两种：

  - out + app == app: 只指定app也会默认指定out; 用于将数据追加到文件结尾；
  - out + trunc == out: 默认情况下，只指定out实际上也是截断模式；
  - 如果想不截断，但又不想在文件末尾追加，需要同时指定in和out模式；

- ate: 打开文件立即定位到文件末尾；

- binary: 二进制IO；ate和binary可以用于任何类型文件流对象，也可与其他模式任意组合；

```c++
ofstream out;
out.open("test_trunc.txt"); // 隐含指定out和trunc
out.close();  // 文件会被清空
out.open("test_app.txt", ofstream::app); // 隐含指定out和app
// out.open("test_app.txt", ofstream::out | ofstream::app); // 显式指定out和app, 和上一条等效
out.close();  // 文件内容被追加到文件末尾
```

## 8.3 string流

sstream头文件用于字符串string的IO操作。其中istringstream, ostring, stringstream继承自iostream,可以使用IO运算符<<, >>, getline等，此外还有一些特有操作:

```c++
// sstream 代指上述三个类型
sstream strm;  // 创建一个未绑定的字符串流
sstream strm(s);  // 创建sstream对象，保存字符串s的一个拷贝，这个构造函数是explicit的
strm.str()  // 返回strm保存的字符串的拷贝；
strm.str(s)  // 将strm的保存的字符串设置为s，返回void；
```

### 8.3.1 使用istringstream

适合用于处理整行字符串中的单个单词；

`iss >> buf` 与 `ifs >> buf`的区别在于，后者无法判断哪些是一行文本中的单词，而前者可以接收一行文本进行处理；

当文件中有若干行，每一行的单词数量不定，可能某些行有缺失时，外层循环使用`getline(ifs, buf)`读取一行，内层循环使用`iss >> word`读取单词，是较好的处理方式;

ss.clear()只能重置流的状态，不会清空流的缓冲区的数据；要想清空缓冲区数据，需使用ss.str("")，后者还会把流的状态重置为未绑定状态。 [一个经典例子](https://www.cnblogs.com/lfri/p/9364275.html)

### 8.3.2 使用ostringstream

适合用于逐步构造输出，最后希望一起打印的情况；可以先把即将打印的数据存储在oss这块内存中，最后一起打印；
