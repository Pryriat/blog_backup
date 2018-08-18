[TOC]

# RTTI(运行阶段类型识别)
- RTTI可在程序运行过程中动态地识别基类指针/引用所指向的类对象（派生类or基类）
- 全局示例代码：

    ```cpp
    class a{...};  a* biu
    class b : public a{......} b* test
    class c : public b{......} c
    ```

## RTTI运算符：
### dynamic_cast:
- 使用示例：
  `type * pm = dynamic_cast<type*> (item)  `
  如

```cpp
 b* test;
c*pm = dynamic_cast<c*>(test)
```

- 用途：如果指针item的类型可以安全地转换为type*类型（派生类到基类），则运算符返回对象的地址，否则返回一个空指针。
- dynamic_cast可以与if语句连用，实现多态的同时保证安全

```cpp
if(test = dynamic_cast<b*>(point))
	test->say();
```

b及其派生类可以调用say，而基类a无法安全转换返回NULL，无法调用
### typeid:
- typeid运算符用来确定两个对象是否为同种类型，可以接收两种参数：类名和结果为对象的表达式
- ypeid运算符返回一个对type_info对象的引用，其中type_info是在头文件typeinfo定义的一个类。
- typeinfo重载了==和！=运算符以对类型进行比较
- 用法示例：

```cpp
typeid(test)==typeid(biu)
```

该式返回bool值，如果指针为NULL则抛出bad_typeid异常
- 如果发现在扩展的if-else语句中使用了typeid作判断，则应考虑是否应使用虚函数和dynamic_cast

## 类型转换运算符：
### const_cast:
- const_cast运算符用于执行改变值为const或volatile的转换
- 使用示例：

```cpp
const_cast<type_name>(expression)
```

expression为指针，返回指针

```cpp
const a* tt = biu;
a* qq = const_cast<a*>(tt)
//该转换使qq成为可以修改biu对象的指针，去除const标签
a* qq = const_cast<b*>(tt) //非法，该转换不能修改变量类型
```

-  注意：仅当指向的对象不是const类型时，const_cast返回的指针才能对该对象进行修改
-  由于可能无意间改变变量类型和常量特征，因此使用const_cast更安全
###  static_cast:
- 用法：

```cpp
static_cast<typename>(expresion)
```
expression为指针，返回指针
- 当且仅当type_name可被隐式转换为expression所属的类型或expression所属的类型可被隐式转换为typename的类型时，转换才是合法的：

```cpp
a* aa = static_cast<a*>(test)  //合法，隐式向上转换
b* bb = static_cast<b*>(biu)  //合法，隐式向下转换
b* tt = static_cast<another_class*>(test)  //非法，无关类之间不存在隐式转换 
```

- 可用static_cast进行枚举值与整型、整型与浮点型的转换
### reinterpret_cast:
- reinterpret_cast用于多种类型的转换，即使是不安全的
- 用法：

```cpp
reinterpret_cast<typename>(expression) 
```

- 使用示例

```cpp
struct date{int a,int c};
long b = 0x231233124;
date* pd = reinterpret_cast<date*>(&b);
cout<<pd->a<<pd->c;
```

- reinterpret_cast不支持函数指针与数据指针的相互转换，也不支持将指针转换为更小的整型或浮点型（int* -> char)

# 智能指针
- 智能指针是行为类似于指针的类对象，当这类对象析构时会自动释放所指向的内存空间
- 智能指针的模版有三个：auto_ptr、uniqe_ptr、shared_ptr
- 创建智能指针对象须包含头文件memory。智能指针类接收一个类型名作模版参数，接收地址作构造函数参数。使用通常的模版语法实例化智能指针对象

```cpp
auto_ptr<double> pd(new double)
```

- 将普通指向动态内存的指针赋予智能指针时需调用构造函数

  ```cpp
  double* pt = new double;
  auto_ptr<double> pp ; //auto_ptr<double>(pp)
  pp = auto_ptr<double>(pt);
  ```

- 智能指针不应指向静态内存空间，否则在对象过期析构时会将delete应用于栈上，引发错误
  - 相较于auto_ptr，uniqe_ptr和shared_ptr在内存控制方面更加严格。
    - 示例：

    ```cpp
    auto_ptr<int> g= new int(3);
    auto_ptr<int> p = g;
    ```

    - 在g和p过期时，会分别调用delete释放同一块内存空间。智能指针对象基于两种方法避免此种情况
    - uniqe_ptr和auto_ptr基于建立所有权的概念，只能有一个对象能拥有所指向的内存空间。当g赋值给p时，g将指向地址的所有权转让给p，而后g将调用析构销毁自身。遇到赋值语句时，若uniqe_ptr是一个临时右值（如函数返回值），则编译器允许这样做，若uniqe_ptr对象为存在一段时间的自动变量或常量，uniqe_ptr会让赋值语句失效（编译阶段非法）。
      - 若想对uniqe_ptr强制赋值，可使用move()函数:p=move(g)
      - uniqe_ptr有new[]版本：

      ```cpp
      uniqe_ptr<int[]>(new int[3])
      ```

      - 当uniqe_ptr为右值时，可赋值给shared_ptr
    - shared_ptr使用引用计数跟踪智能指针对象。如赋值时，指针对象加一，智能指针过期时，计数减一。当且仅当最后一个指针过期时才调用delete

# 异常
- 程序在运行期间遇到非预期的错误会引发异常，如不进行处理可能会造成程序崩溃或数据丢失。异常处理能够让引发的异常可控
  - 示例代码:

  ```cpp
    //异常规范
    //void test() throw(const char* a);  
  void test()  
  {  
      ...... 
      if(...)  
          throw "error!";  
      else
          throw 2;
  }
  try
    {
        test();
    }
  catch (const char* a)
      cout<<a;
  catch(int b)
      cout<<b+1;
  ```

- C++中的异常是对程序运行中发生的异常情况进行响应。异常提供了将控制权从一个部分传递到另一个部分的途径。

- 对异常的处理包括三个部分：
  - 引发异常(throw)
      - 程序在出现问题时将引发异常，throw关键字表示引发异常，紧随其后的值（const char*或int）表示异常的特征。throw语句实质上是跳转，即命令程序跳转到另一条语句。
        - 执行throw语句类似于执行返回语句，它将终止函数的执行，沿函数调用序列后退，同时释放try块和throw之间的整个函数调用序列放在栈中的对象（栈解退），直到找到包含try块的函数.
      - throw;————抛出任何异常
  - 使用处理程序捕获异常(catch)
    - 程序中使用异常处理程序捕获异常，异常处理程序位于要处理问题的程序中。
    - catch关键字表示捕获异常，处理程序以关键字catch开头，随后是位于括号中的类型声明，它指出了异常处理程序所响应的异常类型。然后是一个花括号括起的代码块，指出要采取的措施。catch关键字和异常类型用作标签，指出当异常被引发时，程序应跳到这个位置执行。异常处理程序也称作catch块。
    - 执行完try块中的语句后，如果没有引发任何异常，则程序跳过try块后面的catch块，直接执行处理程序后面的第一条语句。
      - catch(...)————接收任何异常
    - 使用try块
    - try块包含可能抛出特定异常的代码块，它后面跟着一个或多个catch块。try块由关键字try开头，关键字try后面是由花括号括起的代码块，表明需要注意其中的代码所引发的异常。

- 使用类对象作为异常类型：
  - 类对象可以作为异常类型，与内置类型相比，可以使用不同的异常类型来区分不同的函数在不同的情况下引发的异常。另外，对象可以携带信息，可以根据信息确定引发异常的原因，同时catch块可以根据信息处理相应的异常。
  - 引发异常时编译器总是创建一个临时拷贝做参数，即使catch块中指定的是引用，因为引发异常时的栈解退将使原对象不复存在，因此使用副本。
  - 如果有一个使用异常类继承层次结构，应注意catch块的排序顺序：将捕获位于派生链底端一场类的catch置于最顶端，将捕获基类的catchh语句置于底端。因为基类的引用作为参数的catch块可处理派生链内所有异常类的异常（隐式向上转换）

- 异常引发的问题：
  - 异常被引发后，在两种情况下会导致问题。首先，如果它是在带异常规范的函数中引发的，则必须与规范列表中的某种异常匹配（在继承层次中，类类型与这个类及其派生类对象匹配），否则称为意外异常。如果异常不是在函数中引发的，则必须捕获它，否则会产生未捕获异常。默认情况下，这两种异常都将导致程序终止。
  - 可以修改程序对此两种异常的反应：
    - 未捕获的异常产生时，程序会先调用函数terminate()，默认情况下程序将调用abort()函数。可以通过头文件exception内的函数set_ terminate()函数来指定terminate()应调用的函数:

      ```cpp
      #include<exception>
      void myQuit(){......};
      int main()
      {
              set_terminate(myQuit);
          ...
      }
      ```

      - 发生意外异常时，程序将调用unexpected()，这个函数将调用terminate()，后者在默认情况下调用abrot()。可以通过exception函数中的set_unexcepted()函数修改其默认调用

          ```cpp
            #include<exception>
            void muQuit(){...}
            int main()
            {
          set_unexpected(myQuit);
          ...
            }
          ```
      ```
      
      ```

- 异常和动态内存分配
  - 引发异常时，作用域对应栈内的自动变量、类对象都将被自动销毁。但对于动态内存分配的对象

    ```cpp
        void test
    {
          double *bad = new double;
          if(*bad == 0)
            throw;
          delete bad;
    }
    ```

      解退栈时，bad作为自动变量被销毁，但throw语句于delete语句前执行，bad所指向的内存空间并没有被释放，
    且变得不可访问。
  - 可以通过一些清理代码，在函数内包含catch块来解决这个问题

      ```cpp
      void test()
      {
          double *bad = new double;
          try
          {
              if(*bad == 0)
                  throw exception();
          }
          catch (exception)
          {
              delete bad;
              throw another_exception;
          }
          delete bad;
      }
      ```

# 头文件
- 头文件常包含：
  - 函数原型
  - 使用#define或const定义的符号常量
  - 结构声明
  - 类声明
  - 模版声明
  - 内联函数
- 不能将函数定义或变量声明放在头文件中
- 在使用自己的头文件时要用尖括号包含
- 在头文件行首添加#ifndef name #define name，在结束包含后添加 #endif

    ```cpp
    #include.....
    #ifndef name
    #define name
    ......
    #endif
    ```

    作为保护措施，让其忽略第一次包含以外的内容，防止重定义
- 包含同一个自定义头文件的cpp文件可单独编译并合并输出
- include指令有两种写法：
    - #include<文件名>，使用<>写法时，编译器会在C++安装目录的include子目录寻找<>中标明的文件，此方法叫做按标准方式搜索
    - #include"文件名"，使用""写法时，编译器会在当前工程目录寻找""中标明的文件，若没有，再按标准方式搜索

# 名称空间
- 在C++中，名称可以是变量、函数、结构、枚举、类及类和结构成员。不同的名称空间中可以包含多个相同的名称，彼此互不影响
- 创建名称空间的方法为：namespace 空间名称{名称}，如`namespace Ch{char a; int b;}`，名称空间可以是全局的，也可以包含于里另一个名称空间中，但不能包含在代码块中。

    ```cpp
    namespace Ch
    {
        using namespace std;
        int a;
        namespace ch
        {
            int b;
        }
    }
    ...
    cout<<Ch::a<<Ch::ch::b;
    ```

- 默认情况下名称空间具有外部链接性（除非引用常量）
- 名称空间是开放的，可把名称加入已有的名称空间中，如`namspace Ch{double d;}`将d加入到Ch中
- 未被修饰的名称（如a）称为未限定的名称，包含名称空间的名称（如：Ch::a）称为限定名称
- 在名称空间中可包含using声明和编译指令

    ```cpp
    namesapce te
    {
        using Ch::a;
    }
    ```

- 可以给名称空间创建别名，如：

```cpp
namespace BB = Ch;
```
    此时BB与Ch等价

## 匿名名称空间

```cpp
namespace
{
	int a = 5;
}
```

等价于static int a = 5，匿名名称空间里的变量均为静态内部链接性

## using声明和using编译指令

### using声明
- using声明将特定的名称添加到它所属的声明区域中，使特定的标识符可用。使用方法：using + 限定的名称。如上述te示例中使用a替代了Ch::a
    - 在函数外部使用using声明时，将名称添加至全局名称空间中

### using 编译指令
- using编译指令使名称空间中所有名称可用，使用方法：using namespace 名称空间
- 在全局声明区域中使用using指令，使该名称空间全局可用
- 在函数中使用using指令，使名称在函数中可用
- 将using指令放在函数定义之前，则该源代码中所有函数都可以使用std中元素
- 将using指令放在函数中，则只有该函数能使用std中元素

# new运算符
- new运算符可为某种类型的数据分配内存，并返回该内存块的地址。需要给该地址赋予一个指针

    ```cpp
    int * A = new int;
    ```

    A指向的内存地址没有名称，A是一个对象，赋值只能用*A进行
- 为一个对象获得并指定分配内存的通用格式为：类型*指针名= new 类型
- 可在分配空间同时赋值

    ```cpp
    int* a = new int(5);
    double* b = new double[5]{1,2,3,4,5}
    ```

- 分配空间失败时返回NULL

## 定位new运算符
- 需包含头文件new，功能为在特定的地址创建内存空间
- 用法：new (地址) 类型，如：

  ```cpp
  int *a =new (0x0000) int`（直接指定地址）
  char abc[50]; 
  int *a = new (abc) int (70);
  ```
  （将变量的地址赋给新变量）  
- 当两个变量类型相同时，同时改变两个变量的值，类似于指针相等

    ```cpp
    int t = 74;
    int* h  = new(&t) int(7);
    //t = *h = 7
    ```

- 当两个变量类型不同时，新的变量数据会覆盖旧的变量
- 指向特定地址的定位new运算符不能用delete置空，但指向动态地址
  的定位new运算符可以使用delete

    ```cpp
    int t = 74;
    int *q = new int (6);
    int* p = new(&t) int;
    //delete p;//错误
    p = new(q) int; 
    delete p;//正确
    ```

# delete运算符
- delete运算符可释放new请求的内存，该操作将释放指针所指的内存而不会删除指针本身
- 一定要配对地使用new和double，否则会发生严重后果

    ```cpp
    int* a = new int;
    delete a;
    int* b = new int[3];
    delete[] b;
    ```

- delete只能释放new请求的内存，不能释放声明变量占用的内存
- delete释放的是地址而不是值，因此只要指针指的是new创建的内存地址便可用delete释放，delete将该地址声明为未利用空间。
- delete可以释放指针所指向的内存空间，但指针本身没有撤销，即指针所占有的内存空间没有被释放。使用完delete后应该将指针置空，即p = null;使用null空置指针后，指针的指向为地址00000000