# auto大法好

使用`auto`不仅可以少打些字，还能阻止那些由于手动指定型别带来的潜在错误和性能问题。另外，某些`auto`型别推导的结果在编程者的视角看起来是错误的，因此，有必要知道如何去引导`auto`推导出正确的结果。

我们从一段“天真无邪”的代码引入

```cpp
int x;
```

`x`忘记初始化了，所以它的值是不确定的，不过也不一定，它有可能被初始化为0。一切全看具体语境。

再来看一段不那么“天真”的代码

```cpp
template<typename It>
void dwim(It b, It e)
{
    while(b != e)
    {
        typename std::iterator_traits<It>::value_type currrValue = *b;
        ....
    }
}
```

绕晕了吧，用`std::iterator_traits<It>::value_type`来表达迭代器所指涉到的值的型别！除非有特殊癖好，这种声明型别的方式没有丝毫乐趣可言。

当然你也可以用闭包的型别来声明局部变量，不过闭包的型别只有编译器知道，程序员是写不出来的，GG。

在`auto`尚未出现的旧标准下，编程就是这么水深火热，但是在`C++11`我们迎来了`auto`，某种意义上写`C++`变得跟写弱类型语言（如`python`）一样流畅了——不过是在变量前加个`auto`而已。

## 强制初始化与简化代码

`auto`的好处之一是强制初始化——用`auto`声明的变量，其型别都推导自初始化物，因此在用`C++`造轮子时，可以完美避免由未初始化变量带来的问题了。

```cpp
int x1;//x1未初始化
auto x1;//错误，必须要有初始化物
auto x3 = 0;
```

对于提领迭代器，`auto`也能游刃有余

```cpp
template<typename It>
void dwim(It b, It e)
{
    while(b != e)
    {
        auto currrValue = *b;
        ....
    }
}

```

而且，由于`auto`使用了型别推导， 就可以用它来表示只有编译器才能掌握的复杂型别

```cpp
auto derefUPLess = 
    [](const std::unique_ptr<Widget>& p1, const std::unique_ptr<Widget>& p2)
	{return *p1 < *p2}
```

我们虽然不知道`Widget`对象比对返回的型别，但交给`auto`就行了——是不是很酷。在C++14中，连lambda表达式的形参中都可以使用`auto`

```cpp
auto derefUPLess = 
    [](const auto& p1, const auto& p2)
	{return *p1 < *p2}
```

酷毙了！

## auto VS std::function

有些人认为不需要声明变量来持有闭包，因为可以用`std::function`来完成这件事。等等，`std::function`是什么？

`std::function`是`C++11`标准库中的一个模板，将函数指针的思想加以推广，它可以指向任何可以被调用的对象。创建一个`std::function`对象，需要指定欲涉的函数的型别，举个例子，声明一个指向任何以`bool(const std::unique_ptr<Widget>&, const std::unique_ptr<Widget>&)`签名调用的对象

```cpp
std::function<bool(const std::unique_ptr<Widget>&, const std::unique_ptr<Widget>&)> func;
```

因为lambda表达式产生可调物对象，`std::function`对象中就可以存储闭包

```cpp
std::function<bool(const std::unique_ptr<Widget>&, const std::unique_ptr<Widget>&)>;
derefUPLess = 
    [](const std::unique_ptr<Widget>& p1, const std::unique_ptr<Widget>& p2)
	{return *p1 < *p2}
```

抛开啰嗦的语法和重复的形参不谈，`std::function`和`auto`在实现闭包时所占用的内存空间也不相同。使用`auto`声明的，存储着一个闭包的变量和该闭包是同一个型别，从而它要求的内存量也和该闭包一样。而使用`std::function`声明的，存储着一个闭包的变量是一个`std::function`实例，所以不管给定的签名是什么，它都占有固定的尺寸，而这个尺寸对于其存储的闭包而言不一定够用。在这种情况下，`std::function`的构造函数会在堆上分配内存来存储闭包。从结果上看，`std::function`对象一般会比使用`auto`声明的变量使用更多的内存。再者，编译器的实现细节一般会限制内联，并产生间接的函数调用，造成额外的开销。将这些因素考虑在内，毫无疑问，`auto`获胜。

## 为变量推导安全且合适的类型

使用`auto`可以避免某些类型安全的问题。比如下面的代码，你肯定见过，甚至还曾写过

```cpp
std::vector<int> v;
unsigned sz = v.size();
```

标准规定，`v.size()`的返回型别是`std::vector<int>::size_type`，但是很少人会注意到，`std::vector<int>::size_type`仅仅是一个无符号整型，使用`unsigned`来指定型别可能会导致一些意想不到的后果：在32位Windows上，`unsigned`和`std::vector<int>::size_type`的长度一致，但在64位下，`unsigned`是32位，`std::vector<int>::size_type`却是64位——在应用移植的时候，你可能为一些由于整型溢出导致的奇怪数值浪费时间。但`atuo`绝不会让你在这个方面浪费时间

```cpp
auto sz = v.size();//sz的型别推导为std::vector<int>::size_type
```

对`auto`的大智慧还是一知半解？接着看下面的代码，找出其隐患

```cpp
std::unordered_map<std::string, int> m;
....
for(const std::pair<std::string, int>& p : m)
{
    ....//在p上实施某些操作
}
```

这段代码看起来合情合理，实际上会造成极大的额外性能开销。别忘了，`std::unordered_map`的健值部分是`const`，所以其元素的型别并不是`std::pair<std::string, int>`，而是`std::pair<cosnt std::string int>`。可是在循环中，`p`的类型却不是这个。结果编译器使用了一种低效的方法将`std::pair<cosnt std::string int>`转换为`std::pair<std::string, int>`——对m中的每个对象执行一次复制操作，形成一个p想要绑定型别的临时对象。在循环每次迭代结束时，该临时对象都会被析构一次——虽然你想要的效果只是想把引用p依次绑定到`m`中的每个元素而已。这种无心之错可以利用`auto`轻松化解

```cpp
for(const auto& p: m)
{
    ....
}
```

这样做不仅能提升运行效率，更可以简化代码。更有甚者，在这段代码中，如果对`p`取址，肯定能得到指向m中某个元素的指针，而没有`auto`，取到的只能是一个临时对象的指针。

这两个例子说明了，显式指定型别可能导致你既不想要，也没想到的隐式型别转换。如果你使用`auto`，就完全没必要担心这个问题。

# 不止于auto

虽然有若干种理由让`auto`相比于显式指定型别更加优秀，但是`auto`本身并不完美——每个`auto`变量的型别都是推导出来的，结果可能既不符合期望也不符合要求。这种情况何时会出现，又该如何应对，需要结合大量的知识储备和实践才能掌握。同时，`auto`可能会导致源代码可读性问题——需要自己推导型别而非一目了然。那么，`auto`到底应该如何使用，在何时使用呢？

首先明确一点：`auto`是一个可选项而不是必选项。如果你的代码在显式型别声明的前提下更清晰、可维护性更高、或者其他的好处，当然选择显式声明。`C++`引入`auto`只是借鉴了其他语言中称为性别推导的思想，是一种新的编程方式，而且不会与大型的、工业强度的基础代码产生冲突。有些人可能觉得使用`auto`之后无法一眼看出变量类型，就失去了对代码的掌控。但是IDE的对象型别的推导及显示能缓解这个问题，并且在很多情况下，对型别的抽象理解比了解完整类型更加有用。比如知道对象是个计数器，还是容器，亦或是指针，已经足以应付大多数情况，没必要知道它具体是何种型别。简单来说，`string`比`basic_string<char>`更容易理解——虽然后者是前者的精确表示。事实上，使用`auto`还是显式型别声明，对代码的影响是十分微妙不可捉摸的，有些关乎正确性，有些关系到执行效率。

`auto`的另一个目的是简化重构。由于`auto`的型别随着其初始化表达式的型别动态改变，则意味着通过`auto`可以省略一些重构的代码。假如一个函数原本的声明返回值是`int`，后来想修改成`double`，如果使用函数的调用结果存储在`auto`中，直接修改返回类型就行了。相反，使用显式类型声明，你还得一个个改函数的调用点，修改接受返回值变量的型别

```cpp
int f()
{
    return 1;
}

int x = f();
auto y = f();

//此时对f进行修改，返回值类型变更
double f()
{
    reutrn 1.1
}

auto y = f();//代码依然可以编译通过，不需进行任何修改
double x = f();//使用显式型别声明则需要修改变量类型
```

# 总结

- `auto`变量必须初始化，基本免疫型别不匹配导致的效率和兼容性问题，还可以简化代码
- `auto`变量有其局限性，不是万能，按需使用