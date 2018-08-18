[TOC]

*最新版本: 01/19/2011*

翻译自[Windows ::DATA alternate data stream](https://www.owasp.org/index.php/Windows_::DATA_alternate_data_stream "Windows ::DATA alternate data stream")

# 漏洞介绍
- NTFS文件系统包含对交换数据流的支持。这不是一个众所周知的功能，主要是为了提供与Macintosh文件系统中文件的兼容性。交换数据流允许文件包含多个数据流。每个文件至少包含一个数据流。在Windows下，默认的数据流被称为`:$DATA`。
- Windows资源管理器不提供查看文件中的交换数据流的方法（或者在不删除文件的情况下删除它们的方法），但可以轻松创建和访问它们。由于它们很难找到，它们经常被黑客用来隐藏他们已经入侵的机器上的文件（可能是rootkit的文件）。交换数据流中的可执行文件可以从命令行执行，但它们不会显示在Windows资源管理器（或控制台）中。关于创建和访问交换数据流的信息请参考示例1。
- 由于每个文件都存在`:$DATA`交换数据流，因此它可以作为访问任何文件的备用方式。参考示例2获悉关于访问文本文件里`:$DATA`交换数据流的信息。任何创建文件或查看或取决于文件名（或扩展名）末尾的应用程序都应该意识到这些交换数据流的可能性。如果使用未处理的用户输入来创建或引用文件名，攻击者可以利用`:$DATA`来改变软件的行为。旧版本的IIS中存在一个众所周知的这种性质的漏洞。当IIS看到对具有ASP扩展名文件的请求时，它将ASP文件发送到与该扩展名关联的应用程序。此应用程序将运行ASP文件中的服务器端代码并为请求生成HTML响应。由于这些版本的IIS的解析扩展存在缺陷，`filename.asp::$DATA`与扩展名不匹配，并且由于没有为`asp::$DATA`扩展名注册的应用程序，所以将asp源代码返回给攻击者。
- 正确的用户输入过滤是抵御此类攻击的最佳防御措施。

# 攻击示例
## 示例1 - 创建交换数据流

```shell
C:\> type C:\windows\system32\notepad.exe > c:\windows\system32\calc.exe:notepad.exe
C:\> start c:\windows\system32\calc.exe:notepad.exe
```

## 示例2 - 访问`:$DATA`交换数据流

```shell
C:\> start c:\textfile.txt::$DATA
```

## 示例3 - 执行ASP交换数据流泄露代码漏洞

普通访问:
`http://www.alternate-data-streams.com/default.asp`

访问`:$DATA`交换数据流显示代码:
` http://www.alternate-data-streams.com/default.asp::$DATA`

在包含漏洞的版本中，IIS将此文件的扩展名解析为`asp::$DATA`，而不是ASP。因此，与ASP扩展关联的应用程序未被调用，并且攻击者可以查看ASP源代码。

# 相关威胁
# 相关攻击
# 相关的漏洞

- [无限制的文件上传](https://www.owasp.org/index.php/Unrestricted_File_Upload "无限制的文件上传")

# 相关防御措施

- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")
