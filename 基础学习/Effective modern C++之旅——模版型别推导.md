# 模版型别推导

函数模版大致形如

```cpp
template<typename T>
void f(ParamType param);
```

模版函数的调用形如

```cpp
f(expr);
```


在编译期，编译器会通过`expr`推导两个类别。一个是T的类别，一个是`ParamType`的类别。由于`ParamType`常包含修饰词（如`const T&`)，`ParamType`的类别可能会与T类别不同。

T的类别推导结果和传递给函数的实参类型不一定相同。T的类别推导结果，不仅仅与`expr`类型有关，还与`ParamType`的形式有关。分为以下三种情况

- `ParamType`具有指针或引用型别，但不是万能引用
- `ParamType`是万能引用
- `ParamType`即非指针又非引用

## 情况1：`ParamType`是指针或引用，但不是万能引用

在此情况下，型别推导按以下步骤进行

- 若`expr`具有引用类别，先将引用部分忽略
- 尔后，对`expr`的型别和`ParamType`的型别执行模式匹配，来决定T的型别

```cpp
template<typename T>
void f(T& param);//param现为引用类型
int x = 0;
const int cx = x;
const int& rx = x;

f(x);//T的类型是int，param类型为int&
f(cx);//T的类型是const int，param类型为const int&
f(rx);//匹配时先将引用忽略，得到T为const int，param为const int&
```

在第二个及第三个函数调用中，`cx`和`rx`都被指明为`const`，所以无论是`T`还是`param`都被推导为`const`类型。向`T&`模版传入`const`类型的变量是安全的，该对象的常量性会成为推导结果的组成部分。右值引用的形参的推导方式与上述相同，但注意：传给右值引用形参的只能为右值引用实参。

当形参类别从`T&`变为`const T&`时：

```cpp
template<typename T>
void f(const T& param);//param现为引用类型
int x = 0;
const int cx = x;
const int& rx = x;

f(x);//T的类型是int，param类型为const int&
f(cx);//T的类型是int，param类型为const int&
f(rx);//匹配时先将引用忽略，得到T为int，param为const int&
```

T的推导类型可视为`param` -`const`，如`const int` —> `int`

如果`T`和`param`是个指针：

```cpp
template<typename T>
void f(T* param);//param现为引用类型
int x = 0;
const int* px = &x; 

f(x);//T的类型是int，param类型为int*
f(px);//T的类型是const int，param类型为const int*
```

## 情况2：`ParamType`是个万能引用

此类形参的声明方式类似右值引用（如函数模版中有类别`T`时，形参声明为`T&&`），但是当传入的实参是左值时，其表现会有所不同。

- 如果`expr`是个左值，`T`和`ParamType`都会被推导为左值引用。这是在模版型别推导中T被推导为引用型别的唯一类型，其次尽管声明时为右值引用，但型别推导的结果却是左值引用
- 如果`expr`是个右值，则应用常规方法

```cpp
template<typename T>
void f(T&& param);

int x  = 10;
const int cx = x;
const int& rx = x;

f(x);//x是个左值，所以T的类别为int&，param类型为int&
f(cx);//cx是个左值，所以T的类别为const int&，param类型为const int&
f(rx);//rx是个左值，T的类型为const int&，param类型为const int&
f(27);//27是个右值，所以T的类别为int，param类型变成了int&&
```

由此可见，当遇到万能引用时，型别推导规则会区分实参是左值还是右值，而非万能引用不会进行这种区分。

## 情况3：ParamType即非指针又非引用

当`ParamType`即非指针又非引用时，就到了按值传递的情况

```cpp
template<typename T>
void f(T param);
```

这意味`param`是实参的一份副本，即一个全新的对象。推导规则为：

- 若`expr`为引用型别，则忽略其引用部分
- 忽略`expr`的引用性之后，若`expr`是一个`const`对象或`volatile`对象，也忽略之。所以

```cpp
int x = 27;
const int cx = x;
const int& rx = x;
f(x);//T和param类型为int
f(cx);//同上
f(rx);//同上
```

如果expr是一个指向`const`对象的`const`指针呢？

```cpp
template<typename T>
void f(T param);
const char* const ptr = "Hello World!";
f(ptr);//ptr的类型是const cahr*，还是char* const，还是char*？
```

虽然`ptr`的指向不可改变，但`ptr`在传递给`f`时，`ptr`这个指针自己会被按值传递，依照按值传递形参的型别推导规则，`ptr`的常量性会被忽略，`param`的类型会被推导为`const char *`——指针指向可变的，指向一个不可变字符串的指针。

### 数组实参

数组型别有别于指针型别，尽管有时候它们看起来可以互换。形成这种假象的原因是数组可以退化成指向其首元素的指针。下面这段代码之所以能通过编译，就是因为这种退化机制在发挥作用

```cpp
const char name[] = "Ada Wang";
const char* ptrToName = name;
```

在此情况下，型别为`const char*`类型的指针是通过`name`来初始化的，而后者的型别是`const char[8]`，二者型别并不统一，代码能通过编译依赖于数组的退化规则。

由于数组退化导致的数组与指针的等价性，如果直接将数组传入按值传递的模版中，会被推导为指针类别

```cpp
template<typename T>
void f(T param);
f(name);//T的类型被推导为const char*
```

但是，尽管函数无法声明真正的数组型别的形参，但却能够将形参声明为数组的引用。所以，如果修改模版，指定按引用的传递实参，然后向其传递一个数组

```cpp
template<typename T>
void f(T& param);
f(name);//T推导为const char[13]，f为const char(&)[13]
```

此时T的类型会被推导为实际的数组型别，甚至包含数组的长度。可以利用数组声明引用这一能力创造一个模版，用来推导出数组含有元素的个数

```cpp
template<typename T, std::size_t N>
constexpr std::size_t arraysize(T (&)[N]) noexcept
{
    return N;
}
```

然后可以初始化一个长度与未知数组相同的数组或其他STL类型

```cpp
int keys = {1,2,3,4,5,6,7,8,9,10};
int cpy_keys[arraysize(keys)];
std::array<int,arraysize(keys)> exp;
```

### 函数实参

数组并非是C++中唯一可以退化为指针之物，函数型别亦可，并且对数组的推导规则对函数对象同样适用

```cpp
void someFunc(int, double);
template<typename T>
void f1(T param);

template<typename T>
void f2(T& param);

f1(someFunc);//param被推导为函数指针，具体类型为void(*)(int, double)
f2(someFunc);//param被推导为函数引用，具体类型为void(&)(int, double)
```

## 总结

- 在模版型别的推导中，其引用性会被忽略
- 对万能引用形参进行推导时，左值实参会进行特殊处理
- 对按值传递的形参进行推导时，会消除实参中`const`和`volatile`属性
- 在模版型别的推导中，数组或函数型别的实参会退化成对应的指针，除非模版形参为引用型