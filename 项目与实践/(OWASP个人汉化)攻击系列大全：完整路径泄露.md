[TOC]

*最新版本: 11/8/2012*

翻译自[Full Path Disclosure](https://www.owasp.org/index.php/Full_Path_Disclosure "Full Path Disclosure")

# 漏洞描述
完整路径泄漏（FPD）漏洞使攻击者能够看到web根目录/文件的路径。 例如：`/home/omg/htdocs/file/` 某些漏洞（如使用SQL注入中的load_file（）查询来查看页面源）需要攻击者拥有要查看文件的完整路径。

# 风险因素
有关FDP的风险可能产生各种结果。例如，如果web根目录被泄露，攻击者可能会对其进行恶意利用，如将其与文件包含漏洞（请参阅[PHP文件包含](https://www.owasp.org/index.php/PHP_File_Inclusion "PHP文件包含")）结合使用以窃取有关Web应用程序或操作系统的其他配置文件。

```php
Warning: session_start() [function.session-start]: The session id contains illegal characters, 
valid characters are a-z, A-Z, 0-9 and '-,' in /home/example/public_html/includes/functions.php on line 2
```
结合使用PHP函数file_get_contents的未保护功能，攻击者有机会窃取配置文件。

## index.php的源代码

```php
<?php
   echo file_get_contents(getcwd().$_GET['page']);
?>
```

攻击者制作一个如下的URL：
`http://site.com/index.php?page=../../../../../../../home/example/public_html/includes/config.php`
结合相对路径遍历执行FPD。

## config.php泄露的源代码

```php
<?php
   //Hidden configuration file containing database credentials.
   $hostname = 'localhost';
   $username = 'root';
   $password = 'owasp_fpd';
   $database = 'example_site';
   $connector = mysql_connect($hostname, $username, $password);
   mysql_select_db($database, $connector);
?>
```

不考虑上述示例，FPD也可以通过观察文件路径来显示底层操作系统。例如，Windows始终以驱动器号开头，例如：`C:\ `，而基于Unix的操作系统往往以单斜杠开始。

- \*Unix

```shell
Warning: session_start() [function.session-start]: The session id contains illegal characters, 
valid characters are a-z, A-Z, 0-9 and '-,' in /home/alice/public_html/includes/functions.php on line 2
```

- Windows

```shell
Warning: session_start() [function.session-start]: The session id contains illegal characters, 
valid characters are a-z, A-Z, 0-9 and '-,' in C:\Users\bob\public_html\includes\functions.php on line 2
```

FPD可获取的信息比人们通常怀疑的要多得多。上面的两个示例还显示了操作系统上的用户名; “爱丽丝”和“鲍勃”。 用户名当然是重要的凭证。攻击者可以通过多种不同的方式使用攻击者，包括各种协议（SSH，Telnet，RDP，FTP等）的强制攻击，以及启动需要工作用户名的攻击。

# 攻击示例
## 空列表

如果我们有一个网站，使用这种请求页面的方法
`http://site.com/index.php?page=about`
我们可以使用一组大括号，使页面输出一个错误。这个方法看起来像这样：
`http://site.com/index.php?page[]=about`
这使得页面不存在，因此产生一个错误：

```shell
Warning: opendir(Array): failed to open dir: No such file or directory in /home/omg/htdocs/index.php on line 84
Warning: pg_num_rows(): supplied argument ... in /usr/home/example/html/pie/index.php on line 131
```

## 空的会话cookie
另一种生成包含FPD错误的流行且非常可靠的方法是使用JavaScript注入为页面提供一个无效会话。使用这种方法的简单注入看起来就像这样
`javascript:void(document.cookie="PHPSESSID=");`
通过简单地将PHP会话ID的cookie设为空，我们得到一个错误

```shell
Warning: session_start() [function.session-start]: The session id contains illegal characters, 
valid characters are a-z, A-Z, 0-9 and '-,' in /home/example/public_html/includes/functions.php on line 2
```

简单地通过关闭错误报告来防止此漏洞，由此您的代码不会吐出错误
`error_reporting(0);`
错误可能包含对站点所有者有用的信息，所以相比禁用错误报告，通过[display_errors](http://www.php.net/errorfunc.configuration#ini.display-errors "display_errors")来隐藏输出中的错误是更好的选择。


## 非法的会话cookie

作为空的会话cookie的补充，很长的会话也可能产生一个包含FPD的错误。 这也可以使用JavaScript注入来完成，如下所示

`javascript:void(document.cookie='PHPSESSID=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');`

通过简单地将PHP会话IDcookie长度设置为129或更长，PHP可能返回一个warning.
另一种方法是将PHP会话IDcookie保留字节的其中一个
`javascript:void(document.cookie='PHPSESSID=.');`

两种攻击都会产生以下结果

```shell
Warning: session_start(): The session id is too long or contains illegal characters,
valid characters are a-z, A-Z, 0-9 and '-,' in /home/example/public_html/includes/functions.php on line 2
```
与空会话Cookie相同的补救措施适用于此处。[display_errors](http://www.php.net/errorfunc.configuration#ini.display-errors "display_errors")会隐藏输出中的错误。

## 直接访问需要预加载库文件的文件
Web应用程序开发人员有时无法在需要预加载的库/函数文件的文件中添加安全检查。当这些应用程序的URL直接被请求时，很可能显示敏感信息。有时候，这是本地文件包含漏洞的线索。
在使用Mambo CMS的情况下，当我们访问一个直接的URL:
`http://site.com/mambo/mambots/editors/mostlyce/jscripts/tiny_mce/plugins/spellchecker/classes/PSpellShell.php`
然后我们的将得到

```html
<br />
<b>Fatal error</b>:  Class 'SpellChecker' not found in <b>/home/victim/public_html/mambo/mambots/editors/mostlyce/jscripts/tiny_mce/plugins/spellchecker/classes/PSpellShell.php</b> on line <b>9</b><br />
```

# 工具
- 以上三项检查可以借助[inspathx](https://code.google.com/p/inspathx/ "inspathx")工具完成。

# 相关攻击代理
- 内部软件开发商

# 相关攻击方式
- [SQL注入](https://www.andseclab.cn/2018/04/22/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e4%ba%8c%ef%bc%9asql%e6%b3%a8%e5%85%a5/ "SQL注入")
- [相对路径遍历](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%af%e5%be%84%e9%81%8d%e5%8e%86/ "相对路径遍历")

# 相关漏洞
无

# 相关防御措施

- [错误处理](https://www.owasp.org/index.php/Error_Handling "错误处理")
- [边界检查](https://www.owasp.org/index.php/Bounds_Checking "边界检查")
- [安全库](https://www.owasp.org/index.php/Safe_Libraries "安全库")
- [静态代码分析](https://www.owasp.org/index.php/Static_Code_Analysis "静态代码分析")
- [可执行的空间保护](https://www.owasp.org/index.php/Executable_space_protection "可执行的空间保护")
- [地址空间布局随机化（ASLR）](https://www.owasp.org/index.php?title=Address_space_layout_randomization_(ASLR)&action=edit&redlink=1 "地址空间布局随机化（ASLR）")
- [堆栈粉碎保护（SSP）](https://www.owasp.org/index.php/Stack-smashing_Protection_(SSP) "堆栈粉碎保护（SSP）")

# 参考

- [http://www.acunetix.com/vulnerabilities/Full-path-disclosure.htm](http://www.acunetix.com/vulnerabilities/Full-path-disclosure.htm "http://www.acunetix.com/vulnerabilities/Full-path-disclosure.htm")
- [由EnigmaGroup.org撰写的完整路径泄漏文章摘录。](http://www.enigmagroup.org/ "由EnigmaGroup.org撰写的完整路径泄漏文章摘录。")
- [路径泄露漏洞 - 是否严重？](http://yehg.net/lab/pr0js/view.php/path_disclosure_vulnerability.txt "路径泄露漏洞 - 是否严重？")
- inspathx - 内部路径泄露查找器

