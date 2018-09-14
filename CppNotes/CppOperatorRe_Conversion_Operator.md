这章节是第十四章。这章介绍运算符重载，这种机制允许内置运算符作用于类类型的运算对象。这样我们创建的类型直观上就可以像内置类型一样使用，运算符重载是C++借以实现这一目的的方法之一。<br>
类可以重载的运算符中有一种特殊运算符---函数调用运算符。对于重载了这种运算符的类，我们可以“调用”其对象，就好像它们是函数一样。新标准库中提供了一些设施，使得不同类型的可调用对象可以以一种一致的方式来使用，我们也将介绍这部分内容。<br>
最后将介绍另一种特殊类型的类成员函数---转换运算符。这些运算符定义了类类型对象的隐式转换机制。编译器应用这种转换机制的场合与原因都与内置类型转换是一样的。<br>
那么，在第四章中我们看到C++语言定义了大量的运算符以及内置类型的自动转换规则。这些特性使得程序员能编写出形式丰富、含有多种混合类型的表达式。<br>
当运算符被用于类类型的对象时，C++语言允许我们为其制定新的含义；同时，我们也能自定义类类型之间的转换规则。和内置类型的转换一样，类类型转换隐式地将一种类型的对象转换成另一种我们所需类型的对象<br>
@ 当运算符作用于类类型的运算对象时，可以通过运算符重载重新定义该运算符的含义。<br>
## 基本概念
**重载的运算符时具有特殊名字的函数**：它们的名字由关键字operator和其后要定义的运算符号共同组成。和其他函数一样，重载的运算符也包含返回类型、参数列表以及函数体。<br>
**重载运算符函数的参数数量与该运算符作用的运算对象数量一样多。一元运算符又一个参数，二元运算符有两个。**<br>
@ 对于二元运算符来说，左侧运算对象传递给第一个参数，而右侧运算对象传递给第二个参数。除了重载的函数调用运算符operator()之外，其他重载运算符不能含有默认实参。<br>
@ 如果一个运算符函数是成员函数，则它的第一个（左侧）运算对象绑定到隐式的this指针上，因此，成员运算符函数的（显式）参数数量比运算符的运算对象总数少一个。<br>
📒 当一个重载的运算符是成员函数时，this绑定到左侧运算对象。成员运算符函数的（显式）参数数量比运算对象的数量少一个。<br>
**约定：对于一个运算符函数来说，它或者是类的成员，或者至少含有一个类类型的参数:**<br>
```cpp
// ❌ 不能为int重定义内置的运算符
int operator+(int,int);
```
这一约定意味着当运算符作用于内置类型的运算对象时，我们无法改变该运算符的含义。<br>
**我们可以重载大多数（但不是全部）运算符。**<br>
我们只能重载已有的运算符，而无权发明新的运算符号。<br>
有四个符号（+，-，*，&）即是一元运算符也是二元运算符，所有这些运算符都能被重载，从参数的数量我们可以推断到底定义的是哪种运算符。<br>
@ 对于一个重载的运算符来说，其优先级和结合律与对应的内置运算符保持一致。<br>
可以重载的运算符太多，这里列举不能重载的运算符<br>
```cpp
::    .*    .    ?:
```
#### 直接调用一个重载的运算符函数
通常情况下，我们将运算符作用于类型正确的实参，从而以这种间接方式“调用”重载的运算符函数。然而，我们也能像调用普通函数一样直接调用运算符函数，先指定函数名字，然后传入数量正确、类型适当的实参：
```cpp
// 一个非成员运算符函数的等价调用
data1+data2;
// 基于“调用”的表达式
operator+(data2,data2);
// 对成员运算符函数的等价调用
```
两次调用都是等价的，它们都调用了非成员函数operator+。传入的data1是第一个实参，传入data2作为第二个实参<br>
显式地调用成员运算符函数。
```cpp
// 基于“调用”的表达式
data1 += data2;
data1.operator+=(data2);
// 对成员运算符函数的等价调用 将this绑定到data1的地址、将data2作为实参传入了函数。
```
#### 某些运算符不应该被重载
@ 使用重载的运算符本质上是一次函数调用，所以关于运算对象求值顺序的规则无法应用到重载的运算符上。<br>
特别是，逻辑与运算符、逻辑或运算符和逗号运算符的运算对象求值顺序无法保留下来.除此之外，&&和||运算符的重载版本也无法保留内置运算符的短路求值属性，两个运算对象总是会被求值。<br>
我们一般不重载逗号运算符和取地址运算符。<br>
✨ 通常情况下，不应该重载逗号、取地址、逻辑与和逻辑或运算符<br>

#### 使用内置类型一直的含义
当你开始设计一个类时，首先应该考虑的是这个类将提供哪些操作。在确定类需要哪些操作之后，才能思考到底应该把每个类操作设成普通函数还是重载的运算符。如果某些操作在逻辑上与运算符相关，则它们适合于定义成重载的运算符：<br>
@ 如果类执行IO操作，则定义移位运算符使其与内置类型的IO操作保持一致。<br>
@ 如果类的某个操作是检查相等性，则定义operator==；如果类有了operator==，意味着它通常也应该有operator!=。<br>
@ 如果类包含一个内在的单序比较操作，则定义operator<；如果类有了operator<,则它也应该 含有其他关系操作。<br>
@ 重载运算符的返回类型通常情况下应该与其内置类型版本的返回类型兼容：逻辑运算符和关系运算符应该返回bool，算术运算符应该返回一个类类型的值，赋值运算符和复合赋值运算符则应该返回左侧运算对象的一个引用。<br>
💡 **尽量明智滴使用运算符重载。**<br>
每个运算符在用于内置类型时都有比较明确的含义。以二元+运算符为例，它明显执行的是加法操作。因此，把二元+运算符映射到类类型的一个类似操作上可以极大地简化记忆。<br>
#### 赋值和复合赋值运算符
赋值运算符的行为与复合版本的类似：赋值之后，左侧运算对象和右侧运算对象的值相等，并且运算符应该返回它左侧运算对象的一个引用。**重载的赋值运算应该继承而非违背其内置版本的含义**<br>
#### 选择作为成员或者非成员
当我们定义重载的运算符时，必须首先决定是将其声明为类的成员函数还是声明为一个普通的非成员函数。<br>
将运算符定义为成员函数还是普通的非成员函数的准则以及抉择:<br>
@ 赋值（=），下标（[]),调用(（）)和成员访问箭头（->）运算符必须是成员。<br>
@ 复合赋值运算符一般来说应该是成员，但并非必须，这一点与赋值运算符略有不同。<br>
@ 改变对象状态的运算符或者与给定类型密切相关的运算符，如递增、递减、和解引用运算符，通常应该是成员<br>
@ 具有对称性的运算符可能转换任意一端的运算对象，例如算术、相等性、关系和位运算符等，因此它们通常应该是普通的非成员函数。<br>
程序员希望能在含有混合类型的表达式中使用对称性运算符。🌰 我们能求一个int 和 double的和，因为它们中的任意一个都可以是左侧运算对象或右侧运算对象，所以加法是对称的。**如果我们想提供含有类对象的混合类型表达式，则运算符必须定义非成员函数。**<br>
当我们把运算符定义成成员函数时，它的左侧运算对象必须时运算符所属类的一个对象。<br>
🌰 ：
```cpp
string s = "world";
string t = s+')'; // 可以把一个const char*加到一个string对象中 s.operator+(')')
string u = "hi"+s; // 如果+是string的成员，则产生错误。
```
因为string将+定义成了普通的非成员函数，所以"hi"+s等价于operator+("hi",s)。和任何其他函数调用一样，每个实参都能被转换成形参类型。唯一的要求是至少又一个运算对象是类类型，并且两个运算对象都能准确无误地转换成string.<br>
❓ 在什么情况下重载的运算符与内置运算符有所区别？在什么情况下重载的运算符又与内置运算符一样？<br>
我们可以直接调用重载运算符函数。重置运算符与内置运算符有一样的优先级与结合性。<br>
```cpp
#include <string>
#include <iostream>
class Sales_data{
    friend std::istream& operator>>(istream&, Sales_data&); // input
    friend std::ostream& operator<<(ostream&, const Sales_data&); //output
    friend Sales_data operator+(const Sales&, const Sales_Data&);
    ...
    Sales_data& operator+=(const Sales_data&);
}
```
### 输入和输出运算符
IO标准库分别使用>>和<<执行输入和输出操作。对于两个运算符来说，IO库定义了用其读写内置类型的版本，而类则需要自定义适合其对象的新版本以支持IO操作<br>
#### 重载输出运算符<<
通常情况下，输出运算符的**第一个形参是一个非常量ostream对象的引用**。之所以ostream是非常量是因为向流写入内容会改变其状态；而该形参是引用是因为我们无法直接复制一个ostream对象。<br>
**第二个形参一般来说是一个常量的引用，该常量是我们想要打印的类类型。**第二个形参是引用的原因是我们希望避免复制实参；而之所以该形参可以是常量是因为（通常情况下）打印对象不会改变对象的内容。<br>
为了与其他输出运算符保持一致，operator<<一般要返回它的ostream形参。<br>
##### Sales_data的输出运算符
🌰 
```cpp
ostream &operator<<(ostream &os, const Sales_data &item){
    os<<item.isbn()<<""<<item.units_sold<<""<<item.revenue<<""<<item.avg_price();
    return os;
}
```
##### 输出运算符尽量减少格式化操作
用于内置类型的输出运算符不太考虑格式化操作，尤其不会打印换行符，用户希望类的输出运算符也像如此行事。
##### 输入输出运算符必须是非成员函数
假设输入输出运算符是某个类的成员，则它们也必须是istream或ostream的成员。然而，这两个类属于标准库，并且我们无法给标准库中的类添加任何成员。<br>
因此，如果我们希望为类自定义IO运算符，则必须将其定义成非成员函数。**当然，IO运算符通常需要读写类的非公有数据成员，所以IO运算符一般被声明为友元。**<br>
```cpp
class Sales_data{
    friend ostream &operator<<(ostream &os, const Sales_data &item);
    ...
}
```

```cpp
class String{
    friend ostream &operator<<(ostream &os, const String&);
}

ostream &operator<<(ostream &os, const String &s){
    char *c =  const_cast<char*>(s.c_str());
    while(*c)
        os<<*c++;
    return os;
}
```
### 重载输入运算符>>
@ 输入运算符的第一个形参是运算符将要读取的流的引用。<br>
@ 第二个形参时将要读入到的（非常量）对象的引用。<br>
@ 该运算符通常会返回某个给定流的引用。<br>
第二个形参之所以必须是个非常量是因为输入运算符本身的目的就是将数据读入到这个对象中<br>
##### Sales_data的输入运算符
吃个🌰 ：
```cpp
istream &operator>>(istream &is, Sales_data &item){
    double price; // 不需要初始化，因为我们将先读入数据到price，之后才使用它
    is>>item.bookNo >> item.units_sold >> price;
    if(is)
        item.revenue = item.units_sold * price;
    else
        item = Sales_data();// 输入失败就赋予默认值，确保对象处于正确的状态
    return is;
}
```
📒 输入运算符必须处理输入可能失败的情况，而输出运算符不需要<br>
##### 输入时的错误
@ 当流含有错误类型的数据时读取操作可能失败。<br>
@ 当读取操作到达文件末尾或者遇到输入流的其他错误时也会失败<br>
🏊‍♀️ 当读取操作发生错误时，输入运算符应该负责从错误中恢复。<br>
#### 标示错误
一些输入运算符需要做更多数据验证的工作。<br>
通常情况下，输入运算符只设置failbit。除此之外，设置eofbit表示文件耗尽，而设置badbit表示流被破坏。最好的方式是由IO标准库来标示这些错误。<br>
```cpp
class Sales_data{
    friend istream &operator>>(istream &is, Sales_data &item);
    ...
}
istream &operatr>>(istream &is,Sales_data &item){
    ...
}
```
### 算术和关系运算符
@ 算术和关系运算符定义成非成员函数以允许对左侧或右侧的运算对象进行转换。<br>
@ 因为这些运算符一般不需要改变运算对象的状态，所以形参都是常量的引用。<br>
算术运算符通常会计算它的两个运算对象并得到一个新值，这个值有别于任意一个运算对象，常常位于一个局部变量之内，操作完成后返回该局部变量的副本作为其结果。<br>
如果定义了算术运算符，则它一般也会定义一个对应的复合赋值运算符。此时，最有效的方式是使用复合赋值来定义算术运算符：
```cpp
Sales_data operator+(const Sales_Data &lhs, const Sales_data &rhs){
    Sales_Data sum = lhs; 
    sum += rhs; // 将rhs加到sum中
    return sum;
}
```
与原来的add函数完全等价。<br>
💡 如果类同时定义了算术运算符和相关的复合赋值运算符，则通常情况下应该使用复合赋值来实现算术运算符。<br>