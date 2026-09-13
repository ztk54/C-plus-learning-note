## 命名空间
### 定义
在c语言里面比方说两个库都定义“LIst”会冲突，在C++里面我们使用namespace给名字外注上一层域，有了这一层域则不会产生冲突。
```cpp
namespace hg               // 定义一个叫 hg 的命名空间
{
    int rand = 10;         // 这个 rand 只在 hg 域里
    int Add(int a, int b) { return a + b; }
    namespace inner        // 可以嵌套
    {
        int x = 1;
    }
}
```
我们在main函数中调用hg中的rand，如果没有命名空间，那么编译器会默认去找自带的rand函数。但是有了命名空间我们就可以使用`using hg::rand;`将这个命名空间中的rand展开，或者说直接`using namespace hg;`C++标准库也是在一个命名为std的命名空间中，平常的小型练习中我们可以使用`using namespace std;`将标准库中的函数展开。
### 三种使用方法
```cpp

// 方式① 指定命名空间访问 —— 项目里推荐

std::cout << "hello" << std::endl;
hg::Add(1, 2);  

// 方式② 展开某个成员 —— 这个成员经常用且不冲突时推荐
using std::cout;
using std::endl;
cout << "hello" << endl;

// 方式③ 展开全部 —— 项目里不推荐（等于把隔离墙拆了）
using namespace std;
cout << "hello" << endl;

```
## 输入输出IO流
```cpp
#include <iostream>
#include <list>
#include <algorithm>

int main()
{
    // endl 既是换行 '\n'，又会刷新缓冲区
    std::cout << "hello world\n" << std::endl;
    std::cout << "hello world" << std::endl;
    int a[10] = { 1, 2, 3, 4 };      // 后面自动补 0
    std::list<int> lt;
    std::sort(a, a + 10);
    for (size_t i = 0; i < sizeof(a) / sizeof(a[0]); ++i)
        std::cout << a[i] << " ";
    std::cout << std::endl;
    int i = 0;
    double d = 0;
    std::cin >> i >> d;               // 可以连着读，自动按空白分隔
    std::cout << i << " " << d << std::endl;
    return 0;
}
```
这里的cin和cout是C++标准库中的函数，标准输入输出流，可以直接输入或者输出C++默认的参数类型，无需像C语言中指定占位符。
同时在后续我们学习完运算符重载后，即可用cin和cout来输入输出各种各样的类，更加直观易懂。
```cpp
int a = 1;  double b = 1.1;  char c = 'x';
printf("%d %f %c\n", a, b, c);   // C：要手写格式，错了不报错、只出乱码
cout << a << " " << b << " " << c << endl;   // C++：类型自己认
```
## 缺省参数
### 基本规则
```cpp
// 全缺省
void Func(int a = 10, int b = 20, int c = 30) { ... }
// 半缺省
void Func(int a, int b = 20, int c = 30) { ... }
Func();           // a=10 b=20 c=30   （只有全缺省才能这样调）
Func(1);          // a=1  b=20 c=30
Func(1, 2);       // a=1  b=2  c=30
Func(1, 2, 3);    // 全给
```
###  三条硬规则
1. **半缺省参数必须从右往左连续给，不能跳跃**。
   ```cpp
   void f(int a, int b = 1, int c);   // ❌ 错：c 没给，中间断了
   void f(int a, int b = 1, int c = 2); // ✅ 对
   ```
   我们可以这样来理解，假如说半缺省是这样的`void f(int a=10;int b;int c)`那我们传参f（5,10）这时候编译器就蒙了，这个5我到底是传给a还是b呢，因此会产生歧义。

2. **调用时实参从左往右依次传**，不能跳过。
   ```cpp
   Func(1, , 3);   // ❌ 没有这种语法
   ```

3. **声明与定义分离时，缺省值只能写在声明里**。
   ```cpp
   // Stack.h
   void STInit(ST* ps, int n = 4);      // ✅ 缺省值写这里
   // Stack.cpp
   void STInit(ST* ps, int n) { ... }   // ✅ 定义里绝对不能重复写
   ```
### 示例
我们在c语言中曾经学习过栈的初始化，我们要为栈开辟空间，假如说我们要把函数卖给一个非专业人士使用。那他也不懂要开辟多大的空间，这时候怎么办呢，我们的缺省参数就派上了用处。
```cpp
typedef int STDataType;
typedef struct Stack
{
	STDataType *a;
	int top;
	int capacity;
}ST;

// 用缺省参数让"不传容量"也有合理默认值
void STInit(ST* ps;int n=4)
{
	assert(ps);
	ps->a=(STDataType*)malloc(n*sizeof(STDataType));
	if (ps->a == NULL)
    {
        perror("malloc fail");
        return;
    }
    ps->top = 0;
    ps->capacity = n;
}
```
这样子的话，安全性和自主性都有了。
## 函数重载
### 基础概念
**同一作用域**里，函数名相同、**参数列表不同**（个数/类型/顺序），构成重载。
```cpp
int  Add(int a, int b)          { return a + b; }
double Add(double a, double b)  { return a + b; }
int  Add(int a, int b, int c)   { return a + b + c; }
```

- 只看**参数**：返回值不同**不构成**重载。
- C++ 靠**名字修饰（name mangling）**实现，C 语言没有这个概念。

### 与缺省参数撞车 → 二义性

经典例子：

```cpp
void f1()          { cout << "f1()" << endl; }
void f1(int a = 10) { cout << "f1(int)" << endl; }
  
int main()
{
    f1();      // 二义性！编译器不知道调哪个
    f1(20);    // OK，走 f1(int)
    return 0;
}
```

>[!bug]
> **踩坑**：**C2668：对重载函数的调用不明确**。原因：`f1()` 是精确匹配，`f1(int=10)` 也能靠缺省参数匹配，两者都不比对方差 → 编译期无法裁决。

**记住这条，后面构造函数的"默认构造二义性"是同一个原因。**
## 引用
### 定义
引用就相当于给一个变量起别名，比如说一个人叫张伟，他的外号叫张三，今天张伟去上班了，那张三没上班吗？肯定也上了，反过来说张三上班那么也就是张伟上班了。引用**不占**独立内存（底层由指针实现的）
```cpp
int a = 10;
int& ra = a;      // ra 是 a 的别名
ra = 20;          // 等价于 a = 20
cout << &a << " " << &ra << endl;   // 两个地址一模一样
```
三条常见性质：
1. **定义时必须初始化**（不能 `int& r;`）。
2. **一个变量可以有多个引用**（`int& r1 = a; int& r2 = a;`）。
3. **引用一旦引用一个实体，就不能再引用别的**（`r1 = b;` 是赋值，不是改指向）。
### 用途
**用途一：做参数**（最常用）
```cpp
// 指针版
void swap(int* p1, int* p2)
{
    int tmp = *p1;
    *p1 = *p2;
    *p2 = tmp;
}

// 引用版：调用处不用取地址，函数体不用解引用
void swap(int& p1, int& p2)
{
    int tmp = p1;
    p1 = p2;
    p2 = tmp;
}
int main()
{
    int x = 1, y = 2;
    swap(&x, &y);    // 调指针版
    swap(x, y);      // 调引用版
    return 0;
}
```
**用途二：做返回值——修改返回对象**

```cpp
struct SeqList
{
    int a[100];
    int size;
};
  
// ❌ 传值返回：返回时拷贝出一个临时变量，临时变量具有常性

int SLAt(SeqList& s, int i)
{
    assert(i >= 0 && i < s.size);
    return s.a[i];
}
  

// ✅ 传引用返回：返回的就是 s.a[i] 本身
int& SLAt(SeqList& s, int i)
{
    assert(i >= 0 && i < s.size);
    return s.a[i];
}

  
int main()
{
    SeqList s;
    s.size = 5;
    s.a[3] = 0;
  
    SLAt(s, 3) += 1;   // 用引用版才合法
    // 用传值版会报：⚠️ C2106 左操作数必须为左值
    return 0;
}

```

**为什么传值返回不行**：`return s.a[i]` 会把值**拷贝到一个临时变量**，C++ 规定**临时变量具有常性**（不能被修改），所以 `+= 1` 就违法了。

**用途三：做返回值——减少拷贝，提高效率**
```cpp
// 返回大对象的引用，省掉一次拷贝构造
const string& GetBigString() 
{ 
	static string s(10000, 'x'); 
	return s; 
}
```
### 注意事项
1. 不能返回局部变量的引用
```cpp
int& f()
{
	int a=20;
	return a;
}
```
a在出了函数之外就被消耗了，相当于这里的是个野引用，结果不确定，可能会打印随机值
2. ```cpp
	int& r = a;
	int& rr = r;    // ✅ 合法：rr 也是 a 的别名（不是"引用的引用"）
	int& arr[10];   // ❌ 没有"引用数组"
	int&* p;        // ❌ 没有"指向引用的指针"（但可以有"指针的引用"：int*& p）
```