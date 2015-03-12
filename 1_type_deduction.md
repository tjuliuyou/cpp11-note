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