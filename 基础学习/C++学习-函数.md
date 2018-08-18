- [TOC]

  # 递归
  -递归函数调用自己，则被调用的函数也将调用自己，这将无限循环下去，除非代码包含终止调用链的内容

  ```cpp
  {
  	statement 1;
  	if(a>0)
      	recurs(a-1);
  	statement 2;
  }
  ```

  - 解析：只要if语句为true，每个recurs调用将执行statement 1，然后再调用recurs，而不会执行statement 2。当if语句为false时，当前调用将执行statement 2，然后结束并返回之前的函数，执行statement 2，以此类推

  # 函数头
  - 函数头由3部分组成，如：

  ```cpp
  	int main()
  ```

  - int：函数返回类型（没有指定返回类型默认返回int类型）
  - main：函数名，每个程序都必须有一个main函数
  - ()：括号中可插入其他函数的信息，空括号代表该函数不接受任何参数  
      P.S. int main()亦可用main () 、int main(void) 、viod main()来表示 void在功能上形同空括号
   - 一个函数可接收多个参数，但只能返回一个值

  # 内联函数
  - 内联函数等价于在main()函数内部声明并调用函数。相比于普通函数，内联函数在运行速度上有优势，但占内存
  - 声明内联函数的方法为：
      -  在函数声明前加上关键字inline

      ```cpp
      		inline typename function();
      ```

      -  在函数定义前加上关键字inline

      ```cpp
      inline typename function()
      {
      	...
      }
      ```

  - 通常的做法是省略原型，将整个定义放在本该提供原型的地方

  # 默认参数
  - 在函数原型（定义或实现）的声明中，在对形参赋值，该形参即为函数的默认参数

      ```cpp
      int test(int a,int b=5)
      ```

  - 调用带有默认参数的函数时，传入的实参可以省略默认参数所对应的参数，此时默认参数使用默认值。若传入了默认参数的实参，默认参数将被置为传入值

    ```cpp
    test(3)————》int test(3,5)
    test(3,6)——————》int test(3,6)
    ```

  - 对于带默认参数的函数，必须从右至左添加默认值，形如int a(int b=3, int c)是不允许的

  # 函数的重载
  - 函数名相同，参数个数相同，区别仅在于形参的类型或返回值类型函数成为函数重载

    ```cpp
    int test(int a);
    int test(int a, char b)
    ...
    ```

  - 传入不同类型的参数会调用不同版本的函数，若无匹配的函数版本，则自动将形参提升为符合该版本形参的类型。注意：仅可提升而不可降低传入参数的精度，否则会有多个版本的函数与之匹配
  - 引用传递与按值传递在匹配参数时等价，下列代码的重载使编译器无法匹配相应的函数版本

    ```cpp
    int test(int& a);
    int test(int a);
    ```

  # 函数模版
  - 函数的模版声明需要包含template和class/typename关键字

    ```cpp
    template<typename T>
    void swap(T t, t q)
    {
    	T temp = t;
    	t = q;
    	q = temp;
    }
    template<classname Q>
    void add(Q& a, const Q& b)
    {
    	a+=b;
    }
    ```

      在示例中，T、Q是一个类型的名称。不同类型的形参调用函数时，会将T、Q替换为传入形参的类型名
  - 使用函数模版不能缩短程序，编程过程最终仍会出现两个函数定义而不会出现模版。使 用模版可以使编程更加的通用化，让多个函数的定义生成简单、可靠。
  - 函数模版的命名应与库函数中的模版函数命名不同，否则会出现函数重载的问题
    模版可以与重载一起使用
  - 函数模版的作用域为接下来定义的一个函数
  - 函数模版可以与重载同时使用

    ```cpp
    template<typename T>
    void swap(T a, T b);
    template<typename T>
    void swap(T a, T b, bool g);
    ```

  - 关键字decltype(C++11)
      - 使用格式：decltype(type) name，如：

      ```cpp
      int a，char c;
      decltype(a) b//b为int类型
      ```

      decltype括号内可以为表达式

      ```cpp
      decltype(a+c) b//b为char+int即为int 型
      decltype(a+6)b//b为int+6即为int类型
      ```

      decltype括号内可以为函数调用,得到变量的类型为函数的返回值类型 

      decltype括号内可以为左值

      ```cpp
      decltype((a)) b//b为 int&
      ```
      - 匹配优先级：
          - 没有用括号括起的变量(decltype(a) b)，则b的类型与a相同
          - 括号内为函数调用(decltype(swap(c,d) b)，得到的变量类型与函数的返回值类型相同
          - 括号内为左值(decltype((c)) b)，得到的变量类型为指向该类型的引用
          - 前面的类型都不满足(decltype(a+c) b)，得到的变量类型与括号内的类型相同
  - 后置返回类型(c++11)
      - 函数定义：auto name(type a, type b)->type
          如：

          ```cpp
          auto arr(int a, float b) ->double
          {
          	......
          }
          ```

      auto为占位符，表示后置返回类型的类型
  - 模版与后置返回类型结合使用，可指定返回类型

    ```cpp
    template<typename T, typename Q>
    auto swap(T t, Q q) -> double
    {
    	...
    }
    ```

  - 显式具体化与隐式具体化
      - 显式具体化
          - 显式具体化的声明格式为template<>函数名<typename>(形参) 

          ```cpp
          template<> swap<double> (double& a, double& b)
          ```

          - 显式具体化意义为：对于指定类型，不使用模版来生成函数定义，而使用专门的指定类型的函数定义。因此，在使用显式具体化时，要有单独的函数定义
      - 显式实例化
          - 显式实例化的声明格式为template 函数名<typename>(形参)

          ```cpp
          template swap<int, int> (int& a, int& b)
          ```

          - 意义为使用swap模版生成int类型的函数定义
      - 使用中，函数调用优先级顺序为：非模版>显式具体化>模版
      - 注意，在同一程序中使用同一类型的显式实例化和显式具体化将导致错误：定义冲突