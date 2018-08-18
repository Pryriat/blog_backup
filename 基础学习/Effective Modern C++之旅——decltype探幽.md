# 理解decltype

在C++中，对于给定的变量或表达式，`decltype`能够告诉你变量或表达式的型别。大部分情况下，它告诉你的结果和你预测的是一致的，不过偶尔也会有一些“非正常”情况，让你面对推导结果时抓耳挠腮。

先从一般的情况讲起——那些不会引发意外的案例

```cpp
const int i = 0;//decltype(i)是const int

bool f(const Widget& i);//decltype(w)是const Widget&，decltype(f)是bool(*)(const Widget&)

Widget W;

if(f(W))//decltype(f(W))是bool
{
    ...
};

std::vector<int> v;
.....
if(v[0] == 0)//decltype(v[0])为int&
```

是不是与你自己推导的型别一样，没有任何意外吧。

在C++11中，`decltype`的主要用途大概在于声明那些返回值型别依赖于形参型别的函数模版。例如，假设我们想实现一个函数，其形参包括一个容器，重载了方括号下标运算符`[]`，并会在返回下标操作结果前进行用户验证。函数的返回值型别须与下标操作结果的返回值型别相同。一般来说，含有模版`T`的对象容器，其`operator[]`的返回类型为`T&`，如`std::deque`和`std::vector`，但`std::vector<bool>`对应的`operator[]`返回的不是`bool&`，而是一个全新的对象。重要的是，容器的`operator[]`的返回类型取决于容器本身。举个例子：

```cpp
template<typename T>
T& getIndex(std::vector<T> a)
{
    return a[0];
}


std::vector<int> a0{1,2,3};

std::vector<bool> b0{true,false};

std::cout<<getIndex(a0);//可以编译运行，因为a0[0]的类型为int&
std::cout<<getIndex(b0);//无法编译，因为b0[0]的类型为bool，无法转换为bool&
```



`decltype`让决定容器方法的返回值简单易行，下面是上述想法的简单实现，其中使用了`decltype`来计算返回值的型别。

```cpp
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i)->decltype(c[i])
{
    authenticateUser();
    return c[i];
}
```

函数名称前的`auto`仅是一个占位符，说明函数的返回值型别将出现在形参列表之后。使用后置返回类型声明的好处在于可以指定函数返回值为形参的类型。比如，在上述函数中，我们可以使用`c`和`i`来指定型别，而传统的声明方式是无法做到的——返回值型别的声明在形参前，怎能用形参来决定返回值类型呢？采用了返回类型后置的声明后，`operator[]`返回的是什么型别，`authAndAccess`返回的即是什么型别，二者类型一致。

C++11允许对单表达式的lambda式的返回值型别进行推导，而在C++14中这个许可扩展到了一切lambda表达式和函数。对于上述函数来说，意味着可以去掉返回类型后置的语法，只保留前置的`auto`。此时，编译器会根据函数的实现来实施返回类型推导

```cpp
/*C++14及后续版本*/
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i)
{
    authenticateUser();
    return c[i];//返回值类型会根据c[i]来推导
}
```

虽然使用`auto`作为返回值类型看起来一劳永逸，但也会产生隐患。一如前面所讨论的，大多数含有模版`T`的对象容器的`operator[]`会返回`T&`，但是在模版型别推导的过程中，初始化表达式的引用性会被忽略，由此会引发问题，参看以下代码：

```cpp
std::deque<int> d;
...
authAndAccess(d,5) = 10;//验证用户，返回d[5]，并赋值为10。无法通过编译
```

看起来没有任何问题是吧，为什么无法通过编译呢？这里的`d[5]`返回类型本应该是`int&`，但由于`auto`作为返回型别引发的模版型别推导，剥夺了其引用饰词，返回值变为了`int`，且作为函数的返回值，该`int`是一个右值，无法进行赋值操作，因此无法通过编译。

由此看来，要让`authAndAccess`正常运作，对返回类型进行`decltype`型别推导是必不可少的。在C++14中，可以通过`decltype(atuo)`来实现自动返回类型后置的功能

```cpp
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index i)
{
    authenticateUser();
    return c[i];//返回值类型会根据c[i]来推导
}
```

乍看上去自相矛盾——怎么又指定类型又自动推导呢？换个角度思考，`auto`指定了要推导的型别，而推导过程套用的是`decltype`的规则，是不是清晰了许多？现在，`authAndAccess`的返回值将与`c[i]`一模一样。

`decltype(auto)`的使用场景并不局限于函数返回值，在变量声明时也可以方便地使用

```cpp
Widget w;

const Widget& cw = w;

auto myWidget = cw;//auto型别推导,myWidget的类型为Widget

decltype(auto) DeWidget = cw;//decltype型别推导，DeWidget的类型为const Widget&
```

万事大吉了吗？非也。在`authAndAccess`的声明中，容器的传递方式为对非常量的左值引用，因此允许客户对容器进行修改，同时也意味着无法向该函数传递右值容器，因为右值无法与左值引用绑定（除非为常量的右值引用）。虽然向`authAndAccess`传入右值容器是一个很罕见的情况，因为一个右值容器，作为一个临时对象，一般会在包含了调用`authAndAccess`的语句结束处被析构，函数返回的引用将被置于悬空状态。但客户可能会想获得临时容器的某元素副本，如

```cpp
std::deque<std::string> maker();//构造函数

auto s = authAndAccess(maker(),5);
```



如果要支持这种用法，要么修改函数的声明，添加一种接收右值的模版；要么使用万能引用，以同时接收两种传参类型，还记得模版类型推导里的万能引用吧，现在到了它大显身手的时候了

```cpp
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i)
```

在本模板中，我们对于容器的操作并不知情，同时对下标对象的型别也一概不知，对未知型别的对象采用直接按值传递的方法有诸多风险，非必要的复制操作将带来性能隐患。但在容器下标这一特定的领域，按值传递是合理的——标准库的实现。不过，为了优化，对于万能引用传参，我们要使用`std::forward`

```cpp
/*C++14版本*/
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}

/*C++11版本*/
template<typename Container, typename Index>
auto authAndAccess(Container&& c, Index i)->decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

当然，除非是那些重度的库实现者，一般不会遭遇除以上所述的额外情况。当然——如果你遇到了，恭喜你，继续玩挖`decltype`这个深坑吧。

将`decltype`应用于一个变量名上，会得到该变量的型别。对于单个的左值表达式变量，`decltype`的行为保持不变，然而，如果是比仅有变量名更加复杂的左值表达式的话，`decltype`保证得出的型别总是左值引用。简单来说，如果表达式不仅是型别为`T`的变量，如返回`T`的函数，那么`decltype`得到的是一个`T&`。这种行为一般而言没有什么影响，因为绝大多数左值表达式都自带一个左值引用的饰词。但这种行为有一个严重的后果，如下

```cpp
int x  = 0;//decltype(x)为int
int (x) = 0;//decltype(x)为int&
```

`x`是一个变量名，但把`x`放入括号中，得到了一个比仅用名称更复杂的表达式`(x)`。在C++的定义中，`x`是一个没有引用修饰的左值，`(x)`也一样，但`decltype((x))`的推导结果却加上了引用修饰。仅仅是把变量名放入括号，就修改了`decltype`的结果！在C++14中，使用`decltype(auto)`来确定返回值型别，不注意上述规则的话可能会造成严重的后果

```cpp
decltype(auto) f1()
{
    int x = 0;
    ....
    return x; //x推导为int
};

decltype(auto) f2()
{
    int y = 0;
    ....
    return (y);//y推导为int&
}    
```

注意，问题不仅仅在于`f2`和`f1`的返回类型不同，更重要的是`f2`返回了一个局部变量的引用，将程序猿送上未定义行为的快车，螺旋升天。由此可见，使用`decltype(auto)`时要小心翼翼，否则看似用不影响型别的不同的写法会引发灾难性的后果。当然，这些仅仅是特殊使用场景，不必为此放弃`decltype`这一简化代码的神器——大多数情况下它还是勤勤恳恳且符合期望的。

# 总结

- 绝大多数情况下，`decltype`会忠实地返回变量或表达式的型别
- 对于型别为`T`的左值表达式，除非该表达式仅有一个名字，否则`decltype`推导结果为`T&`
- C++14支持`decltype(auto)`，`auto`指定要推导的型别，推导过程套用的是`decltype`的规则