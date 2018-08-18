# 理解auto型别推导

如果你已经了解了有关模版型别推导的规则，那么你已经基本了解有关auto型别推导了，因为auto型别推导除了一种特殊情况外，其他与模板型别推导并无二致，它们之间确实也存在双向的算法变换。

在模版型别推导一章中，编译器会利用传入参数来推导模版参数的型别以实例化模版函数

```cpp
template<typename T>
void f(T param);
f(expr);
```

当某变量使用auto来声明时，`auto`就扮演了模版中的`T`这个角色，而变量类型扮演的是`expr`。下面的代码展现了这种有趣的映射关系

```cpp
auto x = 27;//auto为int类型，类比f(T)->f(27)
const auto cx = x;//auto为int类型，类比f(const T)->f(cx)
const auto& rx = x;//auto类型还是为int，类比f(const T&)->f(rx)
```

可以简单理解为，`auto`就是模版中的可变参，对`auto`的修饰即是对可变参的修饰，根据模版类型的推导规则将可变参的类型确定下来，`auto`自然确定了类型。

与模版型别推导类似，`auto`型别推导根据型别饰词的不同分为三种情况：

- 情况1：型别饰词是指针或引用
- 情况2：型别饰词是万能引用
- 情况3：型别饰词即非指针也非引用

情况1和情况3可以参考上面的代码示例，对于情况2，亦可以套用模版的型别推导规则

```cpp
auto&& ux1 = x;//x是左值，所以ux1的类型是int&
auto&& ux2 = cx;//cx是左值，ux2类型还是int&
auto&& ux3 = 27;//27是右值，ux3的类型推导为int&&
```

模版中关于函数与数组的在非引用型别饰词下发生的指针退化，在`auto`的型别推导中也是成立的

```cpp
const char* name = "Ada Wang";//const char[8]
void someFunc(int, double);

auto arr1 = name;//arr1类型为const char*
auto& arr2 = name;//arr2的类型为const char(&)[13]
auto func1 = someFunc;//func1的类型为void(*)(int, double)
auto& func2 = someFunc;//func2的类型为void(&)(int, double)
```

如此看来，`auto`的推导与模版型别推导的规则一模一样，的确如此，除了一种额外情况：

假设我们要声明一个`int`类型的变量，并将其初始化为27，在支持统一初始化的C++11中，我们有四种方式~~就像回字有四种写法一样~~

```cpp
int x1 = 27;
int x2(27);
int x3 = { 27 };
int x4{ 27 };
```

虽然看起来大相径庭，结果却殊途同归——我们得到了一个值为27的`int`类型变量。

为了方便，相比采用固定型别的声明方法，使用`auto`声明来减轻代码量岂不美哉？于是

```cpp
auto x1 = 27;
auto x2(27);
auto x3 = { 27 };
auto x4{27};
```

轻松+愉快对吧？这些声明都能够通过编译，但结果却与我们设想的不同。`x1`与`x2`都被“正确”地初始化，但`x3`与`x4`被初始化成了一个类型为`std::initializer_list<int>`，且含有单个值为27的元素，本想初始化一个整型，得到的却是一个列表，没想到吧！

这与`auto`的一条特殊推导规则有关。当`auto`声明的变量初始化表达式使用大括号括起时，推导所得的型别就属于`std::initializer_list`。因此，下述代码无法通过编译，因为大括号内型别不一，`std::initializer_list<int>`没法装，`std::initializer_list<int>`也不能装：

```cpp
auto x5{1,2,3.4};//叫你皮
```

要注意，这短短一条语句发生了两种型别推导。第一种源于`auto`，将`x5`推导为`std::initializer_list`，但`std::initializer_list`是一个模版，原型为`std::initializer_list<T>`，则意味着`T`的类型也要被推导出来。而后一次推导就落入模版型别推导的范畴了。

`auto`型别推导时会假定用大括号括起的初始化表达式为`std::initializer_list`，在模版型别推导中也成立吗？不存在的

```cpp
auto x = {1,2,3};//x的类型是std::initializer_list<int>

template<typename T>
void f(T param);

f({1,2,3});//错误，无法推导T的类型
```

所以，`auto`和模版型别推导的唯一区别在于，`auto`型别推导时会假定用大括号括起的初始化表达式为`std::initializer_list`，但模版型别推导不会。

不过，如果指定模版中实参的类型为`std::initializer_list`，编译就可以通过

```cpp
template<typename T>
void f2(std::initializer_list<T> list);

f2({1,2,34});
```

至于二者的推导规则为什么会有这个差别，我也不知道。~~不爽不要玩~~

## C++14

相比起C++11，`auto`在C++14还用在了自动推导函数返回类型和lambda表达式中，而且，这些`auto`类型推导使用的是模版类型推导的规则，而非`auto`。~~不爽不要玩~~

首先，自动推导函数返回类型中，`auto`不会将大括号初始化变量推导为`std::initializer_list`，以下的代码是无法通过编译的

```cpp
auto createlist()
{
    return {1,2,3};
}
```

修改为后置返回类型的语法才能正确推导

```cpp
auto createlist()->std::initializer_list<int>
{
	return {1,2,3};
}
```

同样，用`auto`来指定lambda式的形参类型时，也不能使用大括号括起的初始化表达式

```cpp
std::vector<int> v;
...
auto resetV = [&v](const auto& newValue) { v=newValue; };
resetV({1, 2, 3});//错误，无法推导类型
```

# 总结

- 在一般情况下，`auto`型别推导规则与模版推导规则一样，但是对大括号初始化的型别推导却截然不同
- 在函数返回值或者lambda表达式的形参使用`auto`，推导规则为模版型别推导，而不是`auto`的型别推导