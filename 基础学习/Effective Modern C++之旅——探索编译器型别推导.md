# 掌握查看型别类型推导结果的方法

## IDE编辑器

IDE中的代码编辑器通常会在你将鼠标指针悬停至某个程序实体，如变量、形参、函数等，会提示该实体的型别。例如下面的代码：

```cpp
const int exp = 42;
auto x = exp;
auto y = &exp;
```

IDE会告诉你，x的型别是`int`，y是`const int *`。IDE推导类型的工作原理是让C++编译器（至少也是其前端）在IDE内执行一轮。如果该编译器不能在分析你的代码是得到足够的信息，自然也就无法得出型别推导的结果。对于像`int`这种内置的型别，从IDE那能够得到相对准确的信息。但是一旦面对复杂的型别，IDE显示的信息就不太有用了。

## 编译器的诊断信息

想要让编译器显示其推导出的型别，一条有效的途径是使用该型别来引发某些编译错误，编译器在报告错误时会提及导致该错误实体的型别。比如，我们想要让编译器推导上述`x`和`y`的型别，我们可以声明一个模版，而不去定义它，然后用`x`、`y`来实例化模版

```cpp
template<typename T>
class TD;

TD<decltype(x)> xType;//诱发错误
TD<decltype(y)> yType;
```

只要试图实例化该模版，就会引发编译错误——找不到实例化模版的定义。在编译阶段会得到类似以下的错误信息：

```cpp
error: 'xType' uses undefined class TD<int>
error: 'yType' uses undefined class TD<const int*>
```

于是你便可以确定`x`与`y`的型别了——虽然方法有些奇怪。

## 运行时输出

使用`cout`来输出型别信息的方法只有到了程序运行时期才能奏效，难点在于如何将型别信息用文字显示出来——你可能会认为这个问题平淡无奇，简单地使用`typeid`和`std::type_info::name`就可以解决。

```cpp
std::cout<<typeid(x).name()<<std::endl;
std::cout<<typeid(y).name()<<std::endl;
```

这种方法依赖于以下事实：针对某个对象，对其使用`typeid`方法，返回一个`std::type_info`对象，而后者拥有一个成员函数`name`，该函数返回一个包含型别信息的`const char*`。但是，`std::type_info::name`的返回没有任何规范，各个编译器会在其返回内容上做写文章。例如，`GNU`和`Clang`的编译器会报告`x`的型别是`int`，`y`的型别是`PKI`，`PK`指代"pointer to konst const"，指向不能修改的指针。而微软的编译器输出清晰明了：`x`是`int`，`y`是` int const*`。

以为搞清楚不同编译器的“方言”就万事大吉了？且慢，这只是对简单型别的推导，如果是复杂的型别，你可能需要怀疑编译器是否搞错了。

```cpp
template<typename T>
void f(const T& param)
{
    using std::cout;
    using std::endl;
    cout<<"T = "<<typeid(T).name()<<endl;//显示T的类别
    cout<<"Param = "<<typeid(param).name()<<endl;//显示param的类别
    ....
}

class Widget
{
    ...
}


std::vector<Widget> creater()
{
    
    ...
}

auto vw = creater();
if(!vw.empty())
{
    f(&vw[0]);
}
```

GNU和Clang编译的程序输出中包含以下信息：

```cpp
T = PK6Widget
param = PK6Widget
```

"6"的含义是紧随其后的类名(Widget)的字符数。很明显，编译器告诉我们，`T`和`Param`的类型都是`const Widget*`

微软的编译器也给出了同样的推导结果：

```cpp
T = class Widget const *
param = class Widget const *
```

异口同声，看起来`T`和`param`型别是板上钉钉的事实。但真是如此吗？仔细观察一下，在模版`f`中，`param`的类别是`const T&`，与`T`的型别居然一模一样！正确的结果：`T`为`Widget const* `，`param`为`const Widget const* &`。在这个示例中，编译器只得到了一半的正确结果。

为什么会这样？原因在于，`std::type_info::name`中处理型别的方式仿佛就是向函数模版按值传参一样——又是模版。这么一来，如果该型别是个引用型别，其引用性将被忽略，同时`const`等属性也一并销毁，这就是为什么`param`的型别本该是`const Widget const * &`，却被报告成`Widget const *`的原因。

### 使用Boost库

编译器没做好的事情，就交给`Boost`中的`TypeIndex`来收拾烂摊子吧。虽然说该库不是标准C++的一部分，但`Boost`库开源、跨平台，并且有着公共许可，还能麻利地解决问题——何乐而不为。

```cpp
#include <boost/type_index.hpp>

template<typename T>
void f(const T& param)
{
    using std::cout;
    using std::endl;
    using boost::typeindex::type_id_with_cvr;
    cout<<"T = "<<type_id_with_cvr<T>().pretty_name()<<endl;//显示T的类别
    cout<<"Param = "<<type_id_with_cvr<decltype(param)>().pretty_name()<<endl;//显示param的类别
    ....
}

class Widget
{
    ...
}


std::vector<Widget> creater()
{
    
    ...
}

auto vw = creater();
if(!vw.empty())
{
    f(&vw[0]);
}

```

这种方法的工作原理是：模版函数`boost::typeindex::type_id_with_cvr`接受一个型别实参，而且不会移除其`const`、`volatile`和引用饰词（这就是函数名称中`with_cvr`的含义）。该模版函数返回一个`boost::typeindex::type_index`对象，利用其成员函数`pretty_name`来返回一个可读的`std::string`字符串。

现在，无论在什么编译器，我们都能得到一个准确且统一的结果：

```cpp
T = Widget const*
param = const Widget const* &
```

这种自动推导固然高效准确，但是牢记一点：无论是IDE，还是像`Boost.TypeIndex`这样的库，仅仅是你弄明白你的编译器推导类型的辅助工具。它们十分有用，但说到底，理解模版、`auto`和`decltype`的推导规则是驾驭这些工具的必要前提。

# 总结

- 利用IDE编辑器、错误信息和运行时输出来查看编译器的型别推导结果
- 有些工具的结果可能是不准确甚至是无用的，因此，理解C++的型别推导规则是必要的