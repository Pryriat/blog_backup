[TOC]
# 流和缓冲区概念
- C++把输入和谁出看作字节流。输入时，程序从输入流中抽取字节；输出时程序将字节插入输出流中。字节为构成数值或字符的二进制表示。
- C++处理字节流的步骤为
    - 将输入流、输出流与标准输入/输出关联
    - 在内存中创建缓冲区，从输入流中读取字节放入内存中
    - 缓冲区满或检测到特定输入（如回车）刷新缓冲区，同时将内存中的字节流通过输出流传入文件
- 处理字节流的常用类
  - ios_base：表示流的一般特征，如是否可读取、是二进制流还是文本等
      - ios：继承自ios_base，包括一个指向streambuf对象的指针
          - ostream：继承自ios，提供输出方法
          - istream：继承自ios，提供输入方法
              - iostream：继承自ostram和istream，继承输入输出方法
  - streambuf：为缓冲区提供内存，并提供用于填充缓冲区、访问缓冲区内容、刷新缓冲区和管理缓冲区内存的类方法
- 在程序中包含iostream文件，将自动创建8个流对象（4个用于窄字符流，4个用于宽字符流）
  - cin/wcin：对应标准输入流。在默认情况下，这个流被关联到标准输入设备（通常为键盘）。wcin处理wchar_t（之后同理）
  - cout/wcout：对应标准输出流。在默认情况下，这个流被关联到标准输出设备（通常为显示器），cout处理char
  - cerr/wcerr：对应标准错误流，可用于显示错误信息。在默认情况下，这个流被关联到标准输出设备。这个流没有被缓冲，意味着信息将直接发送给屏幕而不会等到缓冲区填满或换行符输入。
  - clog/wclog：对应标准错误流。在默认情况下，这个流被关联到标准输出设备，这和个流被缓冲。
- 对象代表流。当iostream文件为程序声明一个cout对象时，该对象将包含存储了与输出有关的信息数据成员，如显示字段宽度、小数位数等

# 输出流方法
- `ostream& put(char)`，为ostream对象的成员函数
- `basic_ostream<charT, traits>& write(const chartype* s, streamsize m)`，第一个参数为要显示的字符串地址，第二个参数为读取字节长度。write()方法不会再遇到空字符时停止输出（允许越界），常用于比特输出（不会修改数据，如在字符串后加'\0'）
- `flush/endl`：刷新缓冲区。`ostream<<flush\endl、flush(ostream)`
- `ostream.width(int)`：调整下一次字段宽度，宽度小于数据长度则增宽，大于则使用填充字符
- `ostream.fill(char)`：指定填充字符，修改将一直有效，直到更改为止
- `ostream.precision(int)`：设置输出浮点数小数位，精度将一直有效
- `ostream.setf()`
  - `ostream.setf(ios_base::showpoint)`：将浮点数末尾的0显示
  - `ostream.setf(ios_base::boolalpha)`：输入和输出bool值，可以为true或false
  - `ostream.setf(ios_base::showbase)`：对输出是同C++基数前缀（0、0x）
  - `ostream.setf(ios_base::showpos)`：在正数前加'+'
  - `ostream.setf(ios_base::uppercase)`：对十六进制输出使用大写字母表示法
  - `ostream.setf(ios_base::adjustfield)`：恢复默认设置
- ostream控制符，通过ostream<<调用，如`cout<<hex`
  - boolalpha/noboolalpha
  - showbase
  - showpoint
  - shoupos
  - uppercase
  - internal
  - left：左对齐
  - right：右对齐
  - dec
  - hex
  - oct
  - fixed
  - scientific
- 头文件iomainip包含许多格式设置的方法
  - setprecision(int)：设置精度
  - setfill(char)：设置填充字符
  - setw(int)：设置宽度

# 输入流方法
- cin遇到非有效字符时将其留在输入缓冲区
- 流状态（位于strm名称空间中）
  - eofbit：到达文件尾置1
  - badbit：流被破坏置1
  - failbit：输出输出操作未能处理预期字符
  - goodbit：0
- istream对象获取流状态方法
  - good()
  - eof()
  - bad()
  - fail()
  - rdstate()：返回流状态
  - exceptions()：返回位掩码，指出异常
  - exceptions(iostate ex)：设置那些状态导致clear()引发异常
  - clear(iostate s)：将流状态置为s
  - setstate(iostate s)：调用clear(rdstate()|s)，设置S与中设置位为对应位的状态位
- get(char&)、get(void)（非格式化输入函数）
  - istream&get(char&)将字符赋给参数，包括空格、换行符，返回调用对象引用，到达文件尾返回false
  - int get(void)将输入字符转换为整形并返回，到达文件尾返回EOF
- getline、get()、ignore()
  - istream& get(char*, int ,char)\get(char*, int)：输入位置、读取长度、结束符（可选），将换行符留在流中，输入空行导致failbit
  - istyream& getline(char*, int, char)\getline(char* ,int)：抽取并丢弃换行符，读取最大数目字符后还有字符设置failbit
  - istream& ignore(int a,int b)：最大字符数、输入分界符。读取并丢弃接下来a个字符或遇到(char)b停止
- istream& read(char* ,int)：读取字符到指定地址中，不将输入转换为字符串（无'\0'）
- char peek()：返回输入流中的下一个字符（原理为先调用get()抽取，后调用putback()）
- int gcount()：返回最后一个非格式化方法抽取的字符数
- istream& putback()：将下一条输入语句的第一个字符插入到输入流中

# 文件IO
- ifstream/ofstream(filename,mode)或创建fstream派生对象调用open()方法
- 与good()方法相比，is_open()方法不仅打开失败返回false，以不合适的文件模式打开亦返回false
- int main(int argc, char* argv[])，main函数内的参数为命令行处理，argc为传入命令的参数个数，argv为字符串指针数组。当程序在命令行中被调用时，程序名后的字符串将被传入。
- 文件模式（于ios_base名称空间中）
  - in -> "r" 读取文件
  - out -> “w" 写入文件
  - ate -> "cmode" 打开文件并移到文件尾
  - app -> "a" 追加到文件尾
  - trunc -> "w" 如文件存在则截短文件
  - binary -> "b" 二进制读写
  - 不同的模式可以通过或运算复合，如ios_base::out|ios_base::app|ios_base::binary 以二进制方式写入文件尾
  - 通过<<运算符输出/输入的数据为文本模式（即使以二进制方式打开），以二进制方式读写需要调用write()\read()方法
- 对string对象进行二进制读取\写入时因使用c_str()方法转化成c指针再调用，因为string对象本身实际上并不包含字符串，而是包含一个指向其中存储字符串的内存的指针。将string对象写入文件时写入的仅是字符串地址，再次读取时该地址无意义。
- 如果类有虚方法，则二进制读写将包含类内指向虚函数的函数表（内存地址），在下次读写时无意义。对含有虚方法的类应使用文本I/O，可用换行符或空白符将数据隔开以便读取。如使用二进制读写时不能将该类对象作为一个整体写入，而应对对象内不同的成员应用read()或write()。在类数据项前加上特殊的标识符（如int、char），在读取时使用switch以区分基类和派生类类型
- fstream为读写流，在构造时需要显式指定文件模式。
  - istream
    - seekg(streamoff , ios_base::seekdir)：第一个参数为偏移字节数（可为负），ios_base::seekdir为ios_base类中定义的整型，有三种可能的值：ios_base::beg为从文件起始处开始偏移，ios_base::cur指从当前位置偏移，ios_base::end从文件尾开始偏移。如seekg(-5,ios_base::end)
    - seekg(streampos)：从文件开始位置偏移，seekg()返回文件头位置
    - tellg()：返回文件当前位置
  - ostream
    - seekp(streamoff, ios_base::seekdir)
    - seekp(streampos)
    - tellp()
- 对文件进行读写操作时，可用clear()方法重置到达文件尾的流，再使用seekg()或seekp()返回到指定位置
- char* tmpnam(char* name)：接受一个字符串指针作为参数作为参数，返回一个随机子串。可重复调用以获得不同子串，但最大调用次数依赖于传入字符串的长度。在cstdio中的L_tmpnam常量限制了文件名最长字符数，TMP_MAX限制了tmpnam()最多可调用次数
- 内核格式化：读取特定类型的对象中的格式化信息或将格式化后的信息写入特定对象的行为称为内核格式化

# 缓存流
- 头文件sstream定义了从ostream/istream派生出的ostringstream/istringstream对象，该类对象可以创建缓冲区暂存输入\输出的信息，例如：

    ```cpp
    string a = "sdada";
    ostringstream of;
    of<<a<<"   ";//此时a被of缓存，并没有输出到屏幕
    string resault = a.str();
    cout<<resault
    ```

- 这两类对象包含接受一个string对象为参数的构造函数  
  istringstream(string) if  
  该读取在EOF或文件错误才会停止，遇空格则跳过\赋值给下个参数。如：使用if读取包含大量字符串格式的整数：

    ```cpp
  int num;
  int temp;
  while(if>>temp)
  	num += temp;
    ```

- 使用方法str("")清空流