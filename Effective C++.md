# 导读

声明式：告诉编译器某个东西的名称和类型，但是省略去实现细节

```C++
int x   //内置类型也是声明式
```

定义式：提供编译器一些声明式所遗漏的细节

**读懂变量声明的黄金法则：**

第一步; 找到变量名，如果没有变量名，找到最里面的结构

第二步，向右看，读出你看到的东西，但是不要跳过括号

第三步，再向左看， 读出你看到的东西，但是不要跳过括号

第四步， 如果有括号的话，跳出一层括号

第五步，重复上述步骤，知道读出最终的类型

```C++
int *a[5] 
// 变量 a
// 数组5
// 是个int型指针
a是一个指向int的5个指针数组

int (*a)[5]
// 有括号不能跳过
// a是个指针
// a指向了数组5
// 数组类型是int

int (*(*v)[])()
// v 是个指针
//指向了一个数组
// 数组的类型是指针 (*(*v)[])
// 向右看，指针指向一个函数，
// 向左看， 该函数是个int型

v指向一个数组，数组元素是一个函数指针


int const a 等价于 const int  a

int const *r // 找到变量r， r是个指针，向左，是指向const int指针，  *r不能改变，但r本身可以改变

int *const r //找到变量r， 向左看到*， r是个const 指针，指向int， r不能改变， *r可以改变
   
```

- 如果关键字 const 出现在星号左边，表示被指物是常量
- 如果出现在星号右边，表示指针自身是常量

### explicit 

explicit 阻止用来执行隐式类型转换

```C++
class B
{
public:
	expilict B(int x, bool flag = true){  //禁止隐士类型转换
	
	}
};
void dosomething(B object)
B obj1;
dosomething(obj1); // OK
dosomething(25);   // 错误， 25 和B之间不存在隐式类型转换
dosomething(B(28)) //可以
```

**尽量将构造函数声明成为explicit的**

# 条款02：尽量以const，enum, inline 替换#define

```c++
#define pi 3.14
const float pi = 3.14; //替换为常量
```

好处：避免宏定义没有进入符号表 。 减少目标码量

注意：如果是常量指针，要写两次const

```C++
const char* const name = "meye";
```

以#define 实现宏看起来像函数，并且不会导致函数调用带来的开销，但是可能引发错误：

```C++
#define CALL_WITH_MAX(a,b) f((a) > (b) ? (a) : (b))

int a = 5,b = 0;

CALL_WITH_MAX(++a,b); *//a* *被累加* *2* *次*

CALL_WITH_MAX(++a,b + 10); *//a* *被累加* *1* *次*
```

使用 inline 函数可以减轻为**参数加上括号以及参数被核算多次**等问题。同时，inline 可以实现一个“类内

的 private inline 函数”，但一般而言宏无法完成此事



# 条款 03：尽可能使用 const

**const 修饰指针：**

1 const 出现在* 左边，表示被指示物是常量，

```C++
const char *p = greeting; // 被指向物是常量，指针是非常量
```

2 const 出现在 * 右边，表示指针自身是常量，

```c++
char* const p = greeting; // 常量指针， 非常量被指物品
```

3 const 出现在* 两边， 表示指针和被指向物都是常量

```C++
const char * const p = greeting; //常量指针，常量被指物
```

 **注意：**

对于迭代器：

```
const std::vector<int>::iterator iter = vec.begin();
*iter = 10;
++ iter ; // 错误， iter 作用是一个T* const ，指针本身不能变

std::vector<int>::const_iterator iter = vec.begin();
*iter = 10; // 错误， iter 是一个T const *
++ iter ; // 可以
```

**const** **修饰函数**

**函数前const**，

​	普通函数或成员函数（非静态成员函数）前均可加const修饰，表示函数的返回值为const，不可修改。格式为：

```C++
const returnType functionName(param list)
```

**函数后加const**

 ==》 只能类中成员函数加，为常量成员函数。常量成员函数意味着在函数体内不能修改类的成员变量（除非它们被声明为 `mutable`）或调用非常量成员函数。

​	***只有类的非静态成员函数后可以加const修饰***，表示该类的this指针为const类型，不能改变类的成员变量的值，即成员变量为read only，任何改变成员变量的行为均为非法。此类型的函数可称为只读成员函数，格式为：

```C++
returnType functionName(param list) const
```

类中const（函数后面加）与static不能同时修饰成员函数，原因有以下两点①C++编译器在实现const的成员函数时，为了确保该函数不能修改类的实例状态，会在函数中添加一个隐式的参数const this*。但当一个成员为static的时候，该函数是没有this指针的，也就是说此时const的用法和static是冲突的；
②两者的语意是矛盾的。static的作用是表示该函数只作用在类型的静态变量上，与类的实例没有关系；而const的作用是确保函数不能修改类的实例的状态，与类型的静态变量没有关系，因此不能同时用它们。

注意：

- ​	const 类对象只能调用后const成员函数
- ​	非const对象既可以调用const成员函数，又可以调用非const成员函数。

好处：

1. **可读性**：使得接口容易被理解，知道哪个函数可以改动对象哪个函数不行
2. **const** **修饰的成员函数可以作用于** **const** **对象**

使用 const 修饰成员函数时需要注意，C++对常量性的定义是 bitwise constness，即 **const** **成员函**

**数不应该修改对象的任何成员变量**。因此，如果成员变量是一个指针，那么**不修改指针而修改指针所指**

**之物**，也符合 bitwise constness，因此如果不是从 bitwise constness 的角度，这样也是修改了对象：

```
class CTextBlock {

public:

 char& operator[](std::size_t position) const // bitwise constness声明

 { return pText[position]; } *//* 但其实不恰当

private:

 *char* pText;

};

const CTextBlock cctb("Hello"); //声明一个常量对象

char *pc = &cctb[0]; *//调用* const operator[]取得一个指针，

 //指向* *cctb* *的数据

*pc = 'J'; *//cctb*  现在有了“Jello”这样的内容
```

还有一种 logical constness：一个 const 成员函数可以修改它所处理的对象内的某些 bits，但只有在客户

端侦测不出的情况下才行：

```C++
class CTextBlock {

public:

 std::size_t length() const;

private:

 char *pText;

 std::size_t textLength; *//* *最近一次计算的文本区块长度*

 bool lengthIsValid; *//* *目前的长度是否有效*

}; 

std::size_t CTextBlock::length() const{

 if (!lengthIsValid) { 

 textLength = std::strlen(pText); //**错误！在* *const* *成员函数内不能复制给*

 lengthIsValid = true; //textLength* *和* *lengthIsValid*

 }

 return textLength;

}
```

但是，C++对常量性的定义是 bitwise constness 的，所以这样的操作非法。

**解决办法是使用** **mutable**:

```C++
class CTextBlock {

public:

 std::size_t length() const;

private:

 char *pText;

 mutable std::size_t textLength; *//* *这些成员变量可能总是会被更改*

 mutable bool lengthIsValid; *//* *即使在* *const* *成员函数内*

}; 

std::size_t CTextBlock::length() const{

 if (!lengthIsValid) { 

 textLength = std::strlen(pText); *//**现在可以这样*

 lengthIsValid = true; //也可以这样

 }

 **return** textLength;

}
```

总的来说，上面提到了 2 种“修改”const 成员函数中修改对象（修改 const 对象）的方法:

```
1 使用指针，通过指针修改指向的内容绕过const成员函数不能修改成员函数的限制

2 使用mutable 修饰成员变量
```

最后，const 和 non-const 版本的函数可能含有重复的代码，如果抽离出来单独成为一个成员函数还是有

重复。如果希望去重，可以使用“**运用** **const** **成员函数实现出其** **non-const** **孪生兄弟**”的技术， 即在no-const 函数中调用const 函数：

```C++
class CTextBlock {

public:

 const char& operator[](size_t pos) const{

 ...

 }

 char& operator[](size_t pos){

	return const_cast<char&>(

	static_cast<const TextBlock&>(*this) //注意此处的强制转换，将this 强转成为const 是为了调用const 成员函数，避免了自身循环调用

 [pos] 

 );

 }

};
```

**为什么不能在一个常量对象中调用非常成员函数？**

因为在默认情况下，this的类型是指向类的非常量版本的常量指针（意思是this的值不能改变，永远指向那个对象，即“常量指针”，但是被this指向的对象本身是可以改变的，因为是非常量版本，这里this相当于是**顶层const**），而this尽管是隐式的，它仍然需要遵循初始化规则，**普通成员函数的隐式参数之一是一个底层非const指针**，在默认情况下我们无法把一个底层const的this指针转化为非const的this指针，因此我们不能在常量对象上调用普通的成员函数。因此在上例中，形参列表后的const就意味着默认this指针应该是一个底层const, 类型是 const ClassName&。而**非常对象却可以调用常成员函数，因为*底层非const可以默认转化为底层const**。*

# 条款 04：确定对象被使用前已先被初始化

读取没有明确初始化值的对象会导致不明确的行为，

- 对于内置类型的对，永远在使用初始化

- **类类型的对象**：初始化责任落在**构造函数身上**

  ​      **构造效率：**

  ​        分清楚赋值和初始化，在构造函数体中的是赋值，初始化发生在函数构造体之前；

  ​	比起先调用 default 构造函数然后再调用 copy assignment 操作符，单只调用一次 copy 构造函数比较高效。因此，

  善用初始化列表有助于提升效率

  ​       内置类型成员的初始化不一定发生在赋值动作的的时间点之前 。对于内置类型成员，

  一般为了保持一致也在初始化列表中给出初始值

  ​	**初始化顺序：**

  ​       base class 更早与derived class被初始化， 成员的初始化顺序与类内声明顺序相同

**static 对象**

 static 对象寿命从被构造出来直到程序结束为止， 程序结束时候static对象会被自动销毁。

-  	函数内的static对象成为local对象，其他的static对象称为non-local static对象。不同编译单元中的non-local  startic对象初始化次序并不明确
- ​	 函数内的 local static 对象会在“该函数被调用期间、首次遇上该对象的定义式”时被初始化

因此，如果一个 non-local static 对象的初始化依赖于另外一个 non-local static 的初始化，那么可能造成

错误。解决方法是使用 local static 对象替换 non-local static 对象：

```
FileSystem& tfs()
{
	static FileSystem fs;
	return fs
}
```



# **条款 05：了解 C++默默编写并调用哪些函数**

**一般情况下，编译器会为类合成下列函数：**

- **default 构造函数**
- **copy 构造函数**：编译器生成的版本只是单纯地将来源对象的每一个 **non-static** **成员变量拷贝**到目标对象
- **copy assignment** **操作符**：编译器生成的版本只是单纯地将来源对象的每一个 **non-static** **成员变量拷贝**到目标对象
- **析构函数**：编译器生成的版本是 **non-virtual** 的

**以下情况编译器不会合成 copy assignment 操作符：**

•	 **含有引用成员**：原因在于这种情况下，赋值的目的不明确。是修改引用还是修改引用的对象？如果

是修改引用，这是被禁止的。因此编译器干脆拒绝这样的赋值行为

• 	**含有** **const** **成员**：const 对象不应该修改

•	 **父类的** **copy assignment** **操作符被声明为** **private**：无法处理基类子对象，因此也就无法合成
