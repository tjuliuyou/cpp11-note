一、理解类型推导
===

考虑函数定义（伪代码）如下：

```cpp
template<typenmae T>
void f(ParamType param);
```
调用方式如下：

```cpp
f(expr);  // call f with some expression
```
在编译期间，编译器根据 `expr` 推导两个类型值： **T**（typename）和 **ParamType**。
通常来说两者并不一致，因为 *ParamType* 可能包含其他修饰，例如 **const** **\*** **&** 等。

例如：

```cpp
template<typenmae T>
void f(const T& param);  //ParamType is const T&
```
调用方式如下：

```cpp
int x = 0;
f(x);  // call f with an int
```
则，**T** 被推导为 **int**；但是 *ParamType* 则是 **const T&**。

类型推导大概会遇到以下三种情况：

* *ParamType* 为指针或引用，但是非全局引用（[item24]()）;
* *ParamType* 为全局引用;
* *ParamType* 既不是指针也不是引用。

### 情况一、*ParamType* 为指针或引用，但是非全局引用
在此情况下，类型推导大体原则：

* 如果 `expr` 类型是引用，则忽略引用部分；
* 匹配`expr`对应的*ParamType* 决定 **T**；

举例：

```cpp
template<typename T>void f(T& param); // param is a reference
```
有如下变量：

```cpp
int x = 27;  			// x is an intconst int cx = x; 		// cx is a const intconst int& rx = x; 		// rx is a reference to x as a const int
```
则有如下推导：

```cpp
f(x);       	// T is int, param's type is int&f(cx); 			// T is const int,				// param's type is const int&f(rx); 			// T is const int,				// param's type is const int&
```
第二和第三个指定了为`const`，在这里 **T** 被推导为 `const int`。这一点非常重要。因为传递一个**const**引用对象时，即期望这个对象保持不会被修改。这就是为什么要讲**const**修饰变成**T**类型的一部分。
第三个例子中，即使`rx`本身是引用类型，推导是也将引用忽略。
>**注**：上述所有例子都是左值引用，但其规则也适用于右值引用。

如果将 `f` 的参数由 **T&** 更改为 **const T&**规则回稍稍有些变化。`cx`和`rx`的**const**修饰依然有效，但是 **param** 已经声明为 **ref-to-const**, 所以不再需要为T 类型的的推导添加 **const** 修饰：

```cpp
template<typename T>void f(const T& param); 	// param is now a ref-to-const
int x = 27;				// as beforeconst int cx = x;			// as beforeconst int& rx = x;		// as before

f(x);				// T is int, param's type is const int&
			f(cx);				// T is int, param's type is const int&

f(rx);				// T is int, param's type is const int&
```
如果param 是一个pointer（或者 pointer-to-const）,推导规则也大致入引用：

```cpp
template<typename T> 
void f(T* param);  			// param is now a pointer
int x = 27;					// as beforeconst int *px = &x;			// px is a ptr to x as a const intf(&x); 						// T is int, param's type is int* 
f(px); 						// T is const int,							// param's type is const int*
```

### 情况二、*ParamType* 为全局引用

在此情况下，类型推导大体原则：

* 如果 `expr` 类型是左值引用，则 **T** 和 *ParamType*都被推导为 左值引用；
* 匹配`expr`类型是右值引用， 那么推导原则按照（情况一处理）；

举例：

```cpp
template<typename T> 
void f(T&& param);		// param is now a universal reference
int x = 27;				// as beforeconst int cx = x;			// as beforeconst int& rx = x;		// as before
f(x);				// x is lvalue, so T is int&, 
					// param's type is also int&
f(cx);				// cx is lvalue, so T is const int&, 
					// param's type is also const int&
f(rx);				// rx is lvalue, so T is const int&,
					// param's type is also const int&
f(27);				// 27 is rvalue, so T is int,					// param's type is therefore int&&
```

### 情况三、*ParamType* 既不是指针也不是引用

当 *ParamType* 为 **pass-by-value**时：

```cpp
template<typename T>void f(T param); 			// param is now passed by value
```
即，param是传递对象的一份拷贝：一个全新的对象。这就是说 **T** 的推导结果取决于**expr**：

* 如果expr类型是引用，则忽略引用部分；
* 如果忽略了引用部分，包含**const**修饰，继续忽略；如果包含 **volatile**，继续忽略：

因此：

```cpp
int x = 27;				// as beforeconst int cx = x;			// as beforeconst int& rx = x;		// as before
f(x);					// T's and param's types are both intf(cx);					// T's and param's types are again both intf(rx);					// T's and param's types are still both int
```

### Array 参数

```cpp
```
### function 参数


```cpp
void someFunc(int, double);			// someFunc is a function;
									// type is void(int, double)template<typename T>      void f1(T param);					// in f1, param passed by value				template<typename T>void f2(T& param);					// in f2, param passed by ref
f1(someFunc);						// param deduced as ptr-to-func;									// type is void (*)(int, double)
f2(someFunc);						// param deduced as ref-to-func;									// type is void (&)(int, double)
```